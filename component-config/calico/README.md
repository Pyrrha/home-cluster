# Calico

## Install

See latest version.

```
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.1/manifests/tigera-operator.yaml
```

```
curl https://raw.githubusercontent.com/projectcalico/calico/v3.24.1/manifests/custom-resources.yaml -O
# Change internal CIDR for something like 10.0.0.0/16
kubectl apply -f custom-resources.yaml
```


