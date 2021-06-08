---
title: Setup
weight: 1
---

Download here: https://k3s.io

Parameters can be written in a config file located at `/etc/rancher/k3s/config.yaml`.

## First master

Make a config file located at `/etc/rancher/k3s/config.yaml`, with the following content:
```yaml
cluster-init: true
token: <some random string>
```

## Additional master nodes

Make a config file located at `/etc/rancher/k3s/config.yaml` with the following content:
```yaml
server: https://94.130.182.175:6443
token: <same random string as above>
```

## Worker nodes

Run script with `K3S_URL` and `K3S_TOKEN` parameters set.

```shell
curl -fsL https://get.k3s.io | K3S_URL=https://tychsen.me:6443 K3S_TOKEN=<same random string as first master> sh -
```