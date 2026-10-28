# Set 9 — Jenkins creates a Kubernetes PVC and mounts it in a Pod

## File order

1. `Jenkinsfile` — Jenkins pipeline that applies the Kubernetes YAML and verifies the mount.
2. `k8s/pvc.yaml` — defines a 1 GiB PersistentVolumeClaim.
3. `k8s/pod.yaml` — defines a Pod that mounts the claim at `/data`.

## Prerequisites

- A working Kubernetes cluster.
- A default StorageClass (or configure `storageClassName` in `k8s/pvc.yaml`).
- `kubectl` installed on the Jenkins agent.
- Jenkins credentials containing a kubeconfig file:
  - Go to **Manage Jenkins → Credentials**.
  - Add a **Secret file** credential with ID `kubeconfig`.
  - Upload a kubeconfig whose identity can create/read PVCs and Pods in the `default` namespace and exec into the Pod.
- A Jenkins Pipeline job configured to use this repository or the folder containing these files.

The Jenkins agent must be able to reach the Kubernetes API server. The Kubernetes identity needs suitable RBAC permissions for persistentvolumeclaims, pods, and pod exec in the target namespace.

## Run

1. Put the files in your Git repository, preserving the `k8s/` folder.
2. Create the `kubeconfig` Secret file credential in Jenkins.
3. Create a **Pipeline** job and point it at the repository.
4. Run **Build Now** and inspect the stage logs.

The pipeline applies the PVC, waits for it to become `Bound`, applies the Pod, waits for it to become `Ready`, writes a test file in `/data`, and prints the file and resource status.

## Verify manually

With the same kubeconfig configured in your shell:

```sh
kubectl get pvc jenkins-demo-pvc
kubectl get pod pvc-demo-pod
kubectl exec pvc-demo-pod -- ls -l /data
kubectl exec pvc-demo-pod -- cat /data/jenkins-demo.txt
```

## Cleanup

This deletes the Pod and PVC (and may delete the dynamically provisioned volume, depending on the StorageClass reclaim policy):

```sh
kubectl delete -f k8s/pod.yaml
kubectl delete -f k8s/pvc.yaml
```

## Notes

- This example targets the `default` namespace; update `NAMESPACE` in `Jenkinsfile` if needed.
- `ReadWriteOnce` is suitable for this single-Pod demo; actual storage behavior depends on your cluster and StorageClass.
- If the PVC remains `Pending`, check whether a StorageClass exists and whether its provisioner is healthy.
