# Home Cluster

## Installation

- Install controlplane
  - Set localhost, dietz.dev, home.dietz.dev as alternates domains
  - Set CIDR to something different from local network (10.0.0.0/16 for example). It will be use also by Calico
  - Find how to configure [TLS bootstrap](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/troubleshooting-kubeadm/#cannot-use-the-metrics-server-securely-in-a-kubeadm-cluster)
    - Add `serverTLSBootstrap: true` to kubelet config file `/var/lib/kubelet/config.yaml`
    - Add `serverTLSBootstrap: true` to configmap `kubelet-config` (path: data.kubelet.serverTLSBootstrap)
    - Restart kubelet daemon
    - Certificates have been created!
    - List the freshly created certificate: `kubectl get csr`
    - Then approve pending one: `kubectl certificate approve <csr-id>`
- Deploy Calico
- Join cluster with other nodes
- Deploy ArgoCD
- Don't forget to define the secret for Github!
- Apply app-of-apps.yaml Application located in ArgoCD
- It will automaticaly create all other applications
