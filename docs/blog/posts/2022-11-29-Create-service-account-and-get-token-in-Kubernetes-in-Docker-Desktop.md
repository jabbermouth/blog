---
title: Create a Service Account and Get Token In Kubernetes Running In Docker Desktop
description: When running Kubernetes in Docker Desktop 4.8 and later, creating a service account doesn’t create the token properly. The following script will create a service account and retrieve the token. Note that it creates a cluster admin service account for the purposes of this demonstration.
date: 2022-11-29
categories:
  - Kubernetes
  - Docker
  - Tutorials
---
# Create a Service Account and Get Token In Kubernetes Running In Docker Desktop

When running Kubernetes in Docker Desktop 4.8 and later, creating a service account doesn’t create the token properly. The following script will create a service account and retrieve the token. Note that it creates a cluster admin service account for the purposes of this demonstration.

Create a file called `create-service-account.sh` or similar and populate as follows:

```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: $1
  namespace: $2

---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: $1
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: $1
  namespace: $2

---

apiVersion: v1
kind: Secret
metadata:
  name: $1
  annotations:
    kubernetes.io/service-account.name: $1
type: kubernetes.io/service-account-token
EOF

TOKEN=$(kubectl get secret $1 -n $2 --template='{{.data.token}}' | base64 --decode)

echo
echo $TOKEN
echo
```

To create a service account called `my-service-account` in the namespace `development` run the following command:

```bash
bash create-service-account.sh my-service-account development
```