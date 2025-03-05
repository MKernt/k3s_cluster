# k3s Pi Cluster Documentation

## **Prerequisites**
- OS: Raspberry Pi OS Lite
- Pi naming scheme: rpi-cluster-[0-3]
- Install k3sup on host machine

```{zsh}
brew install k3sup
```

- Establish ssh connection and send public id_rsa key to node (rpi)
    - ssh-copy-id 'username@node'
    - for whatever reasons the ed_25519 won't work as k3sup is specifically looking for a rsa key :man_shrugging:

- Install k3s via k3sup
```{bash}
k3sup install \
--host \
rpi-cluster-0 \
--user lex 
--local-path $HOME/.kube/config \
-- k3s-chanel latest \
-- k3s-extra-args "--disable servicelb --disable traefik" \
--merge
```

- Check if everything works on node via:
```{bash}
k3s kubectl get nodes
```