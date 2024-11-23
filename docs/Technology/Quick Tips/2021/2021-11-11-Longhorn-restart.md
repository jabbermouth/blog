# Longhorn Restart

If your Longhorn setup gets ‘stuck', run this script to trigger a restart of all the Longhorn pods.

```bash
kubectl rollout restart daemonset engine-image-ei-d4c780c6 -n longhorn-system
kubectl rollout restart daemonset longhorn-csi-plugin -n longhorn-system
kubectl rollout restart daemonset longhorn-manager -n longhorn-system

kubectl rollout restart deploy csi-attacher -n longhorn-system
kubectl rollout restart deploy csi-provisioner -n longhorn-system
kubectl rollout restart deploy csi-resizer -n longhorn-system
kubectl rollout restart deploy csi-snapshotter -n longhorn-system
kubectl rollout restart deploy longhorn-driver-deployer -n longhorn-system
kubectl rollout restart deploy longhorn-ui -n longhorn-system
```

If you wish to make this script executable, save it to a file (e.g. `restart-longhorn.sh`) and put the following as the first line of the script file:

```bash
#!/bin/bash
```

And then run the following command:

```bash
chmod +x restart-longhorn.sh
```