# Home Cluster

## Installation

- Install controlplane
  - Set localhost, dietz.dev, home.dietz.dev as alternates domains
  - Set CIDR to something different from local network (10.0.0.0/16 for example). It will be use also by Calico
- Deploy Calico
- Join cluster with other nodes
- Deploy ArgoCD
- Don't forget to define the secret for Github!
- Apply app-of-apps.yaml Application located in ArgoCD
- It will automaticaly create all other applications
