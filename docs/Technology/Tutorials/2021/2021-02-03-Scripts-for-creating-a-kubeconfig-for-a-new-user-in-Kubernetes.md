# Scripts for creating a kubeconfig for a new user in Kubernetes

This script is intended to create a kubeconfig file for a user and then give that user permissions to read pod data as well as exec to those pods in a particular namespace.

Once you've created the file, copy it to the machine you wish to connect to the cluster form and place it in a folder called .kube and rename it to config (no extension).

Create a file using Nano (or your preferred editor) name user-create.sh with the following content:

```bash
#!/bin/bash

if [ "$2" != "" ]; then
company=$2
else
company=your-company-name
fi

if [ "$3" != "" ]; then
namespace=$3
else
namespace=default
fi

openssl genrsa -out user-$1.key 2048
openssl req -new -key user-$1.key -out user-$1.csr -subj "/CN=$1/O=$company"
openssl x509 -req -in user-$1.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out user-$1.crt -days 500

rm user-$1.csr

cacrt=$(cat /etc/kubernetes/pki/ca.crt | base64 | tr -d '\n')
crt=$(cat user-$1.crt | base64 | tr -d '\n')
key=$(cat user-$1.key | base64 | tr -d '\n')

cat <<EOM >user-$1.config
apiVersion: v1
kind: Config
users:
- name: $1
  user:
    client-certificate-data: $crt
    client-key-data: $key
clusters:
- cluster:
    certificate-authority-data: $cacrt
    server: https://10.10.4.20:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    namespace: $namespace
    user: $1
  name: $1-context@kubernetes
current-context: $1-context@kubernetes
EOM

if [ "$4" != "" ]; then
kubectl config set-credentials $1 --client-certificate=user-$1.crt  --client-key=user-$1.key
kubectl config set-context $1-context --cluster=kubernetes --namespace=$namespace --user=$1
else
rm user-$1.crt
rm user-$1.key
fi

cat <<EOM >user-$1.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: $namespace
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods","pods/log","services","ingress","configmaps"]
  verbs: ["get", "watch", "list", "exec"]
- apiGroups: [""]
  resources: ["pods/exec"]
  verbs: ["create"]
---

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-$1
  namespace: $namespace
subjects:
- kind: User
  name: $1
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
EOM

kubectl apply -f user-$1.yaml

rm user-$1.yaml
```

Save it and then you can use between 1 and 4 parameters to configure the user created:

1: `user-name` (should be in lowercase and parts separated by hyphens e.g. john-smith)

2: `company-name` (any text name you like for your company – update line 6 to set the default)

3: `namespace` (Kubernetes namespace to grant access to – update line 12 to set the default you want to use)

4: `any-value` (if a value is present, a context is created in the kubeconfig of the machine the script is executed on so you can use the following command to get a list of all pods: `kubectl --context=john-smith-context get pods`

Update the role definition to determine what the user can access.

To create the user with the default country name and namespace (which must exist in the cluster), run the following:

```bash
sudo bash create-user.sh john-smith
```

To delete a user, create the a new file called delete-user.sh and add the following content:

```bash
#!/bin/bash

if [ "$2" != "" ]; then
namespace=$2
else
namespace=default
fi

kubectl config delete-context $1-context
kubectl config delete-user $1

kubectl delete rolebinding -n $namespace pod-reader-$1
```

You then execute this, assuming you want the user deleting from the default namespace, with the following command:

```bash
bash user-delete.sh john-smith
```