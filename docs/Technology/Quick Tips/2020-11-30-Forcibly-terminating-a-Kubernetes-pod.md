# Forcibly terminating a Kubernetes pod

Run the following command, replacing the italics with your pod's name. Use `-n namespace` if your pod is in another namespace.

```
kubectl delete pods your-pod-name --grace-period=0 --force
```