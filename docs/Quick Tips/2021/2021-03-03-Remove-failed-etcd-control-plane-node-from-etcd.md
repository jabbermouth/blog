# Removing a failed/no longer available control plane node from the etcd cluster

If you have a control plane node fail in a cluster to the point you can no longer connect to it, this is how to remove it, including etcd, from your cluster.

The first step is to delete the node itself. For these examples, we'll assume the node is called kubernetes-cp-gone. Run the following command from a working control plane:

```bash
kubectl delete node kubernetes-cp-gone
```

Next we need to tidy up etcd. Firstly, we'll tidy up the Kubernetes level configuration by running the following command:

```bash
kubectl -n kube-system edit cm kubeadm-config
```

Once in Vi, delete the three lines underneath apiEndpoints that correspond to the deleted server (press the insert key to go into the correct mode). Once done, save your changes by pressing escape then :wq followed by enter.

Next, you need to get the name of a working and available etcd pod. You can do this by typing the following:

```bash
kubectl get pods -n kube-system | grep etcd-
```

Next, enter the following command, replacement etcd-pod with the name of one of your working etcd pods:

```bash
kubectl exec -n kube-system etcd-pod -it -- etcdctl --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/peer.crt --key /etc/kubernetes/pki/etcd/peer.key member list -w table
```

Take note of the ID and then run the following command, again, replacing etcd-pod with the name of your working control plane pod:

```bash
kubectl exec -n kube-system etcd-pod -it -- etcdctl --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/peer.crt --key /etc/kubernetes/pki/etcd/peer.key member remove failednodeid
```