# Home Cluster

## Installation

> [!WARNING]
> If using root user, use `su -` instead of `su` to enable /etc/profile file reading.
- Begin by configuring host:
  ```sh
  cat <<EOF | tee /etc/modules-load.d/containerd.conf 
  overlay 
  br_netfilter
  EOF
  modprobe overlay 
  modprobe br_netfilter
  cat <<EOF | tee /etc/sysctl.d/99-kubernetes-k8s.conf
  net.bridge.bridge-nf-call-iptables = 1
  net.ipv4.ip_forward = 1 
  net.bridge.bridge-nf-call-ip6tables = 1 
  EOF
  sysctl --system
  ```
- Install containerd:
  ```sh
  apt update && apt -y install containerd
  ```
- Generate default configuration:
  ```sh
  containerd config default | tee /etc/containerd/config.toml
  ```
- Edit freshly created `/etc/containerd/config.toml`
  - Under `[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]`, replace `SystemdCgroup = false` with `SystemdCgroup = true`
- Restart containerd:
  ```sh
  systemctl restart containerd
  systemctl enable containerd
  ```

- Follow Kubernetes documentation for [installing kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
- Set `serverTLSBootstrap: true` to kubelet config file `/var/lib/kubelet/config.yaml`
- Restart kubelet:
  ```
  systemctl restart kubelet
  ```
- Copy the configuration to `kubeadm.yaml`:
  ```yaml
  apiVersion: kubeadm.k8s.io/v1beta3
  kind: InitConfiguration
  ---
  apiVersion: kubeadm.k8s.io/v1beta3
  kind: ClusterConfiguration
  kubernetesVersion: "1.31.1" # replace with current Kubernetes version
  networking:
    podSubnet: 10.96.0.0/12 # avoid conflicts with Calico. K8s default: 10.96.0.0/12. Calico default: 192.168.0.0/16
  apiServer:
    certSANs:
    - dietz.dev
    - home.dietz.dev
    - localhost
  ---
  apiVersion: kubelet.config.k8s.io/v1beta1
  kind: KubeletConfiguration
  serverTLSBootstrap: true # generate self-signed certs
  ```
- Finally create the cluster:
  ```sh
  kubeadm init --config kubeadm.yaml
  ```
- Get admin configuration from `/etc/kubernetes/admin.conf`
- Ensure `serverTLSBootstrap` is set to `true` in configmap `kubelet-config` (path: data.kubelet.serverTLSBootstrap)
- List the freshly created certificate: `kubectl get csr`
- Then approve pending one: `kubectl certificate approve <csr-id>`
- Untaint control-plane:
  ```sh
  kubectl taint nodes --all node-role.kubernetes.io/control-plane-
  ```
- Install [Tigera operator](https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart) for Calico
  - Apply CRDs and operator (ensure using latest version):
    ```sh
    kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.2/manifests/tigera-operator.yaml
    ```
  - Retrieve configuration and adapt the ipPool's CIDR:
    ```sh
    wget https://raw.githubusercontent.com/projectcalico/calico/v3.28.2/manifests/custom-resources.yaml
    vim custom-resources.yaml # set CIDR to the same of kubeadm.yaml file
    k apply -f custom-resources.yaml
    ```
- Join cluster with other nodes
- Deploy ArgoCD
- Don't forget to define the secret for Github!
- Apply app-of-apps.yaml Application located in ArgoCD
- It will automaticaly create all other applications
