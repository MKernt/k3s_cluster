# k3s Pi Cluster Documentation

## **Prerequisites**
- OS: Raspberry Pi OS Lite
- Pi naming scheme: rpi-cluster-[0-3]
- Install k3sup on host machine

```{zsh}
brew install k3sup
```

- Establish ssh connection and send public id_rsa key to node (rpi)
    - ssh-copy-id `username@node`
    - for whatever reasons the ed_25519 won't work as k3sup is specifically looking for a rsa key :man_shrugging:

## **Deploying k3s**
- Install k3s via k3sup
```{bash}
k3sup install \
--host \
rpi-cluster-0 \
--user lex 
--local-path $HOME/.kube/config \
--k3s-channel latest \
--k3s-extra-args "--disable=servicelb --disable=traefik" \
--merge
```

- Check if everything works on node via:
```{bash}
k3s kubectl get nodes
```

## **Installing kube-vip**
- Provides Kubernetes clusters with a virtual IP and load balancer for both the control plane (for building a highly-available cluster) and Kubernetes Services of type `LoadBalancer` without relying on any external hardware or software
- See `https://kube-vip.io/docs/usage/k3s/`
- Most commands need to `sudo su` 

1. Create Manifests Dir
```{bash}
mkdir -p /var/lib/rancher/k3s/server/manifests/
```

2. Upload kube-vip RBAC Manifest
```{bash}
curl https://kube-vip.io/manifests/rbac.yaml > /var/lib/rancher/k3s/server/manifests/kube-vip-rbac.yaml
```

3. Create the [RBAC settings](https://kube-vip.io/docs/installation/daemonset/#create-the-rbac-settings)

```{bash}
kubectl apply -f https://kube-vip.io/manifests/rbac.yaml
```

3. Generate Manifest
- Set VIP address to be used for the control plane:
```{bash}
export VIP=xxx.xxx.xxx.x36
```

- Set INTERFACE
```{bash}
export INTERFACE=wlan0
```

- Get latest kube-vip
```{bash}
KVVERSION=$(curl -sL https://api.github.com/repos/kube-vip/kube-vip/releases | jq -r ".[0].name")
```

- Run the kube-vip image as a container
```{bash}
alias kube-vip="ctr image pull ghcr.io/kube-vip/kube-vip:$KVVERSION; ctr run --rm --net-host ghcr.io/kube-vip/kube-vip:$KVVERSION vip /kube-vip"
```
- Run manifest
```{bash}
kube-vip manifest daemonset \
    --interface $INTERFACE \
    --address $VIP \
    --inCluster \
    --taint \
    --controlplane \
    --services \
    --arp \
    --leaderElection
```

## **Install kube-vip clou provider**
```{bash}
kubectl apply -f https://raw.githubusercontent.com/kube-vip/kube-vip-cloud-provider/main/manifest/kube-vip-cloud-controller.yaml
```

- Add a configmap forthe ip range
```{bash}
kubectl create configmap -n kube-system kubevip --from-literal cidr-global=xxx.xxx.xxx.x42/52
```

## **Add nodes via k3sup**
- This is the initial process for the other nodes
- Run from host
- Prerequisites (ssh) still apply

```{bash}
k3sup join \
--host HOSTNAME \
--server-ip SERVER-IP \
--server \
--k3s-channel latest \
--user USER
```