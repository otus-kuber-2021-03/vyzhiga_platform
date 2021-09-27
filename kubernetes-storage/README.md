# Kubernetes storage
## Создать StorageClass для CSI Host Path Driver
### Как запустить проект:
```
kind create cluster --config kubernetes-storage/cluster/cluster.yaml
#
kubectl config use-context kind-kind
#
git clone --branch v1.3.0 https://github.com/kubernetes-csi/csi-driver-host-path.git
# Change to the latest supported snapshotter version
SNAPSHOTTER_VERSION=v2.0.1
# Apply VolumeSnapshot CRDs
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/${SNAPSHOTTER_VERSION}/config/crd/snapshot.storage.k8s.io_volumesnapshotclasses.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/${SNAPSHOTTER_VERSION}/config/crd/snapshot.storage.k8s.io_volumesnapshotcontents.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/${SNAPSHOTTER_VERSION}/config/crd/snapshot.storage.k8s.io_volumesnapshots.yaml
# Create snapshot controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/${SNAPSHOTTER_VERSION}/deploy/kubernetes/snapshot-controller/rbac-snapshot-controller.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/${SNAPSHOTTER_VERSION}/deploy/kubernetes/snapshot-controller/setup-snapshot-controller.yaml
# deploy hostpath driver
csi-driver-host-path/deploy/kubernetes-latest/deploy.sh
# create storage class
kubectl apply -f kubernetes-storage/hw/csi-storageclass.yaml
```
### Как проверить работоспособность:
 - `kubectl get pods`
```
NAME                         READY   STATUS    RESTARTS   AGE
csi-hostpath-attacher-0      1/1     Running   0          2m16s
csi-hostpath-provisioner-0   1/1     Running   0          105s
csi-hostpath-resizer-0       1/1     Running   0          95s
csi-hostpath-snapshotter-0   1/1     Running   0          86s
csi-hostpath-socat-0         1/1     Running   0          76s
csi-hostpathplugin-0         3/3     Running   0          115s
snapshot-controller-0        1/1     Running   0          2m40s
```
## Создать объект PVC c именем storage-pvc
### Как запустить проект:
 - `kubectl apply -f kubernetes-storage/hw/storage-pvc.yaml`
## Создать объект Pod c именем storage-pod
### Как запустить проект:
 - `kubectl apply -f kubernetes-storage/hw/storage-pod.yaml`
### Как проверить работоспособность:
 - `kubectl get pod storage-pod`
```
NAME          READY   STATUS    RESTARTS   AGE
storage-pod   1/1     Running   0          2m11s
```
 - `kubectl get pvc storage-pvc`
```
NAME          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
storage-pvc   Bound    pvc-4aad71ca-e200-477a-9a7f-f93907e816e0   1Gi        RWO            csi-hostpath-sc   2m21s
```
 - `kubectl exec storage-pod -- /bin/bash -c "echo data > /data/item"`