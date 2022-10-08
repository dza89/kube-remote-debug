# kube-remote-debug

A small project for the lazy engineer which gives you a terminal with kubectl and helm in your browser.
You can set the role to any existing roles in your cluster.
The namespaced flag switches between a clusterrolebinding and a rolebinding.

## Install

```bash
helm repo add kube-remote-debug https://dza89.github.io/kube-remote-debug
helm repo update
helm upgrade kube-remote-debug --install kube-remote-debug/kube-remote-debug \
    --set namespaced=false \
    --set role=view
```
