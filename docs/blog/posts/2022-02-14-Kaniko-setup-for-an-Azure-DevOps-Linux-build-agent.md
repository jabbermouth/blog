---
title: Kaniko Setup for an Azure DevOps Linux build agent
description: This document takes you through the process of setting up and using Kaniko for building containers on a Kubernetes-hosted Linux build agent without Docker being installed. This then allows for the complete removal of Docker from your worker nodes when switching over to containerd, etc…
date: 2022-02-14
categories:
  - Azure DevOps
  - Kubernetes
  - Containers
  - Tutorials
---
# Kaniko Setup for an Azure DevOps Linux build agent

NOTE: This document is still being tested so some parts may not quite work yet.

This document takes you through the process of setting up and using Kaniko for building containers on a Kubernetes-hosted Linux build agent without Docker being installed. This then allows for the complete removal of Docker from your worker nodes when switching over to containerd, etc…

For the purposes of this demo, the assumption is that a namespace called build-agents will be used to host Kaniko jobs and the Azure DevOps build agents. There is also a Docker secret required to push the container to Docker Hub.

## PreRequisites

This process makes use of a ReadWriteMany (RWX) persistent storage volume and is assumed to be running using a build agent in the cluster as outlined in this [Kubernetes Build Agent repo](https://hub.docker.com/repository/docker/jabbermouth/ads-linux-agent). The only change required is adding the following under the `agent` section (`storageClass` and `size` are optional):

```yaml
  kaniko:
    enabled: true
    storageClass: longhorn
    size: 5Gi
```

## Setup

### Namepace

To set up your namespace for Kaniko (i.e. `build-agents`) run the following command:

```bash
kubectl create ns build-agents
```

### Service Account

Next, create a file called `kaniko-setup.sh` and copy in the following script:

```bash
#!/bin/bash

namespace=build-agents
dockersecret=dockerhub-jabbermouth

while getopts ":n:d:?:" opt; do
  case $opt in
    n) namespace="$OPTARG"
    ;;
    d) dockersecret="$OPTARG"
    ;;
    ?) 
    echo "Usage: helpers/kaniko-setup.sh [OPTIONS]"
    echo
    echo "Options"
    echo "  n = namespace to create kaniko account in (default: $namespace)"
    echo "  d = name of Docker Hub secret to use (default: $dockersecret)"
    exit 0
    ;;
    \?) echo "Invalid option -$OPTARG" >&2
    ;;
  esac
done

echo
echo Removing existing file if present
rm kaniko-user.yaml

echo
echo Generating new user creating manifests
cat <<EOM >kaniko-user.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-runner
rules:
  -
    apiGroups:
      - ""
      - apps
    resources:
      - pods
      - pods/log
    verbs: ["get", "watch", "list", "create", "delete", "update", "patch"]

---

apiVersion: v1
kind: ServiceAccount
metadata:
  name: kaniko
  namespace: $namespace
imagePullSecrets:
- name: $dockersecret

---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kaniko-pod-runner
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: pod-runner
subjects:
- kind: ServiceAccount
  name: kaniko
  namespace: $namespace
EOM

echo
echo Applying user manifests
kubectl apply -f kaniko-user.yaml

echo
echo Tidying up manifests
rm kaniko-user.yaml

echo
echo Getting secret
echo
secret=$(kubectl get serviceAccounts kaniko -n $namespace -o=jsonpath={.secrets[*].name})

token=$(kubectl get secret $secret -n $namespace -o=jsonpath={.data.token})

echo Or paste the following token where needed:
echo
echo $token | base64 --decode
echo
```

This can then be executed using the following command:

```bash
bash kaniko-setup.sh 
```

### Token and Secure File

Create a new Azure DevOps library group called `KanikoUserToken` and add an entry to it. Name the variable `ServiceAccount_Kaniko` and copy the token from above into the value and make it a secret.

Under “Pipeline permissions” for the group, click the three vertical dots next to the + and choose “Open access”. This will be used

### Azure Pipeline

The example below assumes the build is done using template files and this particular one would be for the container build and push. Update the parameter defaults as required. This will deploy to a repository on Docker Hub in a lowercase form of `REPOSITORY_NAME` with `.` replaced with - to make it compliant. This can be modified as required.

```yaml
parameters:
  REPOSITORY_NAME: ''
  TAG: ''
  REGISTRY_SECRET: 'dockerhub-jabbermouth'
  DOCKER_HUB_IDENTIFIER: 'jabbermouth'

jobs:  
- job: Build${{ parameters.TAG }}
  displayName: 'Build ${{ parameters.TAG }}'
  condition: and(succeeded(), startsWith(variables['Build.SourceBranch'], 'refs/heads/${{ parameters.BRANCH_PREFIX }}'))
  pool: 'Docker (Linux)'
  variables:
    group: KanikoUserToken
  steps:
  - task: KubectlInstaller@0
    inputs:
      kubectlVersion: 'latest'

# copy code to shared folder: kaniko/buildId
  - task: CopyFiles@2
    displayName: Copy files to shared Kaniko folder
    inputs:
      SourceFolder: ''
      Contents: '**'
      TargetFolder: '/kaniko/$(Build.BuildId)/'
      CleanTargetFolder: true

# download K8s config file
  - task: DownloadSecureFile@1
    name: fetchK8sConfig
    displayName: Download Kaniko config
    inputs:
      secureFile: 'build-agent-kaniko.config'

# create pod script with folder mapped to kaniko/buildId
  - task: Bash@3
    displayName: Execute pod and wait for result
    inputs:
      targetType: 'inline'
      script: |
        #Create a deployment yaml to create the Kaniko Pod
        cat > deploy.yaml <<EOF
        apiVersion: v1
        kind: Pod
        metadata:
          name: kaniko-$(Build.BuildId)
          namespace: build-agents
        spec:
          imagePullSecrets:
          - name: ${{ parameters.REGISTRY_SECRET }}
          containers:
          - name: kaniko
            image: gcr.io/kaniko-project/executor:latest
            args:
            - "--dockerfile=Dockerfile"
            - "--context=/src/$(Build.BuildId)"
            - "--destination=${{ parameters.DOCKER_HUB_IDENTIFIER }}/${{ replace(lower(parameters.REPOSITORY_NAME),'.','-') }}:${{ parameters.TAG }}"
            volumeMounts:
            - name: kaniko-secret
              mountPath: /kaniko/.docker
            - name: source-code
              mountPath: /src
          restartPolicy: Never
          volumes:
          - name: kaniko-secret
            secret:
              secretName: ${{ parameters.REGISTRY_SECRET }}
              items:
              - key: .dockerconfigjson
                path: config.json
          - name: source-code
            persistentVolumeClaim:
              claimName: $DEPLOYMENT_NAME-kaniko
        EOF

        echo Applying pod definition to server
        kubectl apply -f deploy.yaml -n build-agents --token=$(ServiceAccount_Kaniko)

        # await pod completing
        # Monitor for Success or failure        
          while [[ $(kubectl get pods ${{ variables.jobName }} --token=$(Kaniko_ServiceAccount) -n build-agents -o jsonpath='{..status.phase}') != "Succeeded" && $(kubectl get pods ${{ variables.jobName }} --token=$(Kaniko_ServiceAccount)  -n build-agents -o jsonpath='{..status.phase}') != "Failed" ]]; do echo "waiting for pod ${{ variables.jobName }}: $(kubectl logs ${{ variables.jobName }} --token=$(Kaniko_ServiceAccount) -n build-agents | tail -1)" && sleep 10; done
        
        # Exit the script with error if build failed        
        if [ $(kubectl get pods kaniko-$(Build.BuildId) --token=$(ServiceAccount_Kaniko) -n build-agents -o jsonpath='{..status.phase}') == "Failed" ]; then 
            echo Build or push failed - outputing log
            echo
            kubectl logs ${{ variables.jobName }} --token=$(Kaniko_ServiceAccount) -n build-agents
            echo 
            echo Now deleting pod...
            kubectl delete -f deploy.yaml -n build-agents --token=$(ServiceAccount_Kaniko)

            echo Removing build source files
            rm -R -f /kaniko/$(Build.BuildId)

            exit 1;
        fi

        # if pod succeeded, delete the pod
        echo Build and push successed and now deleting pod
        kubectl delete -f deploy.yaml -n build-agents --token=$(ServiceAccount_Kaniko)

        echo Removing build source files
        rm -R -f /kaniko/$(Build.BuildId)
```

This template is called using something like:

```yaml
  - template: templates/job-build-container.yaml
    parameters:
      REPOSITORY_NAME: 'Your.Respository.Name'
      TAG: 'latest'
```