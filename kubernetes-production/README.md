# Kubernetes production clusters
## Создание нод для кластера
В GCP создайте 4 ноды с образом Ubuntu 18.04 LTS:
master - 1 экземпляр (n1-standard-2)
worker - 3 экземпляра (n1-standard-1)
## Подготовка машин
### Отключите на машинах swap
```
swapoff -a
```
### Включаем маршрутизацию
```
cat > /etc/sysctl.d/99-kubernetes-cri.conf <<EOF
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sysctl --system
```
### Установим Docker
```
apt-get update && apt-get install -y \
  apt-transport-https ca-certificates curl software-properties-common gnupg2

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -

add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"

apt-get update && apt-get install -y \
  containerd.io=1.2.13-1 \
  docker-ce=5:19.03.8~3-0~ubuntu-$(lsb_release -cs) \
  docker-ce-cli=5:19.03.8~3-0~ubuntu-$(lsb_release -cs)

# Setup daemon.
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

mkdir -p /etc/systemd/system/docker.service.d

# Restart docker.
systemctl daemon-reload
systemctl restart docker
```
### Установка kubeadm, kubelet and kubectl
```
apt-get update && apt-get install -y apt-transport-https curl

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

apt-get update
apt-get install -y kubelet=1.17.4-00 kubeadm=1.17.4-00 kubectl=1.17.4-00
```
## Создание кластера
```
kubeadm init --pod-network-cidr=192.168.0.0/24
```
```
I0804 19:42:17.679183    7296 version.go:251] remote version is much newer: v1.18.6; falling back to: stable-1.17
W0804 19:42:18.019136    7296 validation.go:28] Cannot validate kube-proxy config - no validator is available
W0804 19:42:18.019165    7296 validation.go:28] Cannot validate kubelet config - no validator is available
[init] Using Kubernetes version: v1.17.9
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [master-1 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.166.0.27]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [master-1 localhost] and IPs [10.166.0.27 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [master-1 localhost] and IPs [10.166.0.27 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
W0804 19:42:40.580497    7296 manifests.go:214] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[control-plane] Creating static Pod manifest for "kube-scheduler"
W0804 19:42:40.582931    7296 manifests.go:214] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 19.003048 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.17" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node master-1 as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node master-1 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: d3yen4.cg7pul598ullh4qw
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.166.0.27:6443 --token d3yen4.cg7pul598ullh4qw \
    --discovery-token-ca-cert-hash sha256:f95863a52cafeb58ae3047159a47a1073c6a39477310f590757474bd94a5bdcc
```
### Копируем конфиг kubectl
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl get nodes
```
```
NAME       STATUS     ROLES    AGE     VERSION
master-1   NotReady   master   2m26s   v1.17.4
```
### Устанавливаем сетевой плагин
```
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```
### Подключаем worker-ноды
```
kubeadm join 10.166.0.27:6443 --token d3yen4.cg7pul598ullh4qw \
    --discovery-token-ca-cert-hash sha256:f95863a52cafeb58ae3047159a47a1073c6a39477310f590757474bd94a5bdcc
```
## Просмотр нод кластера
```
kubectl get nodes
```
```
NAME       STATUS   ROLES    AGE     VERSION
master-1   Ready    master   27m     v1.17.4
worker-1   Ready    <none>   10m     v1.17.4
worker-2   Ready    <none>   5m17s   v1.17.4
worker-3   Ready    <none>   24s     v1.17.4
```
## Запуск нагрузки
```
kubectl apply -f deployment.yaml
kubectl get po -o wide
```
```
NAME                               READY   STATUS    RESTARTS   AGE   IP              NODE       NOMINATED NODE   READINESS GATES
nginx-deployment-c8fd555cc-d5scq   1/1     Running   0          96s   192.168.0.66    worker-1   <none>           <none>
nginx-deployment-c8fd555cc-dwcpc   1/1     Running   0          96s   192.168.0.65    worker-1   <none>           <none>
nginx-deployment-c8fd555cc-p4mpq   1/1     Running   0          96s   192.168.0.193   worker-2   <none>           <none>
nginx-deployment-c8fd555cc-xzdpn   1/1     Running   0          96s   192.168.0.1     worker-3   <none>           <none>
```
## Обновление кластера
### Обновление мастера
```
apt-get update && apt-get install -y kubeadm=1.18.0-00 \
  kubelet=1.18.0-00 kubectl=1.18.0-00
```
### Проверка
```
kubectl get node
```
```
NAME       STATUS   ROLES    AGE   VERSION
master-1   Ready    master   39m   v1.18.0
worker-1   Ready    <none>   22m   v1.17.4
worker-2   Ready    <none>   17m   v1.17.4
worker-3   Ready    <none>   12m   v1.17.4
```
### Обновим остальные компоненты кластера
```
# просмотр изменений, которые собирает сделать kubeadm
kubeadm upgrade plan

# применение изменений
kubeadm upgrade apply v1.18.0
```
### Проверка
```
kubeadm version
```
```
kubeadm version: &version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.0", GitCommit:"9e991415386e4cf155a24b1da15becaa390438d8", GitTreeState:"clean", BuildDate:"2020-03-25T14:56:30Z", GoVersion:"go1.13.8", Compiler:"gc", Platform:"linux/amd64"}
```
```
kubelet --version
```
```
Kubernetes v1.18.0
```
```
kubectl version
```
```
Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.0", GitCommit:"9e991415386e4cf155a24b1da15becaa390438d8", GitTreeState:"clean", BuildDate:"2020-03-25T14:58:59Z", GoVersion:"go1.13.8", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.0", GitCommit:"9e991415386e4cf155a24b1da15becaa390438d8", GitTreeState:"clean", BuildDate:"2020-03-25T14:50:46Z", GoVersion:"go1.13.8", Compiler:"gc", Platform:"linux/amd64"}
```
```
kubectl describe pod kube-apiserver-master-1 -n kube-system
```
```
Name:                 kube-apiserver-master-1
Namespace:            kube-system
Priority:             2000000000
Priority Class Name:  system-cluster-critical
Node:                 master-1/10.166.0.27
Start Time:           Tue, 04 Aug 2020 19:43:02 +0000
Labels:               component=kube-apiserver
                      tier=control-plane
Annotations:          kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 10.166.0.27:6443
                      kubernetes.io/config.hash: 619ac65c613ceec81de3985f8e29856c
                      kubernetes.io/config.mirror: 619ac65c613ceec81de3985f8e29856c
                      kubernetes.io/config.seen: 2020-08-04T20:25:52.97362301Z
                      kubernetes.io/config.source: file
Status:               Running
IP:                   10.166.0.27
IPs:
  IP:           10.166.0.27
Controlled By:  Node/master-1
Containers:
  kube-apiserver:
    Container ID:  docker://325afa84eb226044dc72fb9026172b0e8a1cdfab52f081f96b975732b802c148
    Image:         k8s.gcr.io/kube-apiserver:v1.18.0
    Image ID:      docker-pullable://k8s.gcr.io/kube-apiserver@sha256:fc4efb55c2a7d4e7b9a858c67e24f00e739df4ef5082500c2b60ea0903f18248
    Port:          <none>
    Host Port:     <none>
    Command:
      kube-apiserver
      --advertise-address=10.166.0.27
      --allow-privileged=true
      --authorization-mode=Node,RBAC
      --client-ca-file=/etc/kubernetes/pki/ca.crt
      --enable-admission-plugins=NodeRestriction
      --enable-bootstrap-token-auth=true
      --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
      --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
      --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
      --etcd-servers=https://127.0.0.1:2379
      --insecure-port=0
      --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
      --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
      --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
      --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
      --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
      --requestheader-allowed-names=front-proxy-client
      --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
      --requestheader-extra-headers-prefix=X-Remote-Extra-
      --requestheader-group-headers=X-Remote-Group
      --requestheader-username-headers=X-Remote-User
      --secure-port=6443
      --service-account-key-file=/etc/kubernetes/pki/sa.pub
      --service-cluster-ip-range=10.96.0.0/12
      --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
      --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    State:          Running
      Started:      Tue, 04 Aug 2020 20:25:54 +0000
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        250m
    Liveness:     http-get https://10.166.0.27:6443/healthz delay=15s timeout=15s period=10s #success=1 #failure=8
    Environment:  <none>
    Mounts:
      /etc/ca-certificates from etc-ca-certificates (ro)
      /etc/kubernetes/pki from k8s-certs (ro)
      /etc/ssl/certs from ca-certs (ro)
      /usr/local/share/ca-certificates from usr-local-share-ca-certificates (ro)
      /usr/share/ca-certificates from usr-share-ca-certificates (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  ca-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/ssl/certs
    HostPathType:  DirectoryOrCreate
  etc-ca-certificates:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/ca-certificates
    HostPathType:  DirectoryOrCreate
  k8s-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes/pki
    HostPathType:  DirectoryOrCreate
  usr-local-share-ca-certificates:
    Type:          HostPath (bare host directory volume)
    Path:          /usr/local/share/ca-certificates
    HostPathType:  DirectoryOrCreate
  usr-share-ca-certificates:
    Type:          HostPath (bare host directory volume)
    Path:          /usr/share/ca-certificates
    HostPathType:  DirectoryOrCreate
QoS Class:         Burstable
Node-Selectors:    <none>
Tolerations:       :NoExecute
Events:
  Type    Reason   Age    From               Message
  ----    ------   ----   ----               -------
  Normal  Pulled   3m33s  kubelet, master-1  Container image "k8s.gcr.io/kube-apiserver:v1.18.0" already present on machine
  Normal  Created  3m33s  kubelet, master-1  Created container kube-apiserver
  Normal  Started  3m33s  kubelet, master-1  Started container kube-apiserver
```
### Вывод worker-нод из планирования
```
kubectl drain worker-1 --ignore-daemonsets
```
```
node/worker-1 cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-system/calico-node-mb8vm, kube-system/kube-proxy-7fkkb
evicting pod default/nginx-deployment-c8fd555cc-dwcpc
evicting pod default/nginx-deployment-c8fd555cc-d5scq
pod/nginx-deployment-c8fd555cc-d5scq evicted
pod/nginx-deployment-c8fd555cc-dwcpc evicted
node/worker-1 evicted
```
### Обновление статуса worker-нод
```
kubectl get nodes -o wide
```
```
NAME       STATUS                     ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
master-1   Ready                      master   49m   v1.18.0   10.166.0.27   <none>        Ubuntu 18.04.4 LTS   5.3.0-1032-gcp   docker://19.3.8
worker-1   Ready,SchedulingDisabled   <none>   32m   v1.17.4   10.166.0.28   <none>        Ubuntu 18.04.4 LTS   5.3.0-1032-gcp   docker://19.3.8
worker-2   Ready                      <none>   27m   v1.17.4   10.166.0.29   <none>        Ubuntu 18.04.4 LTS   5.3.0-1032-gcp   docker://19.3.8
worker-3   Ready                      <none>   22m   v1.17.4   10.166.0.30   <none>        Ubuntu 18.04.4 LTS   5.3.0-1032-gcp   docker://19.3.8
```
### Обновление worker-нод
```
apt-get install -y kubelet=1.18.0-00 kubeadm=1.18.0-00
systemctl restart kubelet
```
### Просмотр обновления
```
kubectl get nodes -o wide
```
```
NAME       STATUS                     ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
master-1   Ready                      master   52m   v1.18.0   10.166.0.27   <none>        Ubuntu 18.04.4 LTS   5.3.0-1032-gcp   docker://19.3.8
worker-1   Ready,SchedulingDisabled   <none>   35m   v1.18.0   10.166.0.28   <none>        Ubuntu 18.04.4 LTS   5.3.0-1032-gcp   docker://19.3.8
worker-2   Ready                      <none>   30m   v1.17.4   10.166.0.29   <none>        Ubuntu 18.04.4 LTS   5.3.0-1032-gcp   docker://19.3.8
worker-3   Ready                      <none>   25m   v1.17.4   10.166.0.30   <none>        Ubuntu 18.04.4 LTS   5.3.0-1032-gcp   docker://19.3.8
```
### Возвращение ноды в планирование
```
kubectl uncordon worker-1
```
### Задание: обновите оставшиеся ноды при помощи kubeadm
```
kubectl drain worker-2 --ignore-daemonsets
```
```
apt-get install -y kubelet=1.18.0-00 kubeadm=1.18.0-00
systemctl restart kubelet
```
```
kubectl uncordon worker-2
kubectl drain worker-3 --ignore-daemonsets
```
```
apt-get install -y kubelet=1.18.0-00 kubeadm=1.18.0-00
systemctl restart kubelet
```
```
kubectl uncordon worker-3
```
### Просмотр обновления
```
kubectl get nodes -o wide
```
```
NAME       STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
master-1   Ready    master   59m   v1.18.0   10.166.0.27   <none>        Ubuntu 18.04.4 LTS   5.3.0-1032-gcp   docker://19.3.8
worker-1   Ready    <none>   42m   v1.18.0   10.166.0.28   <none>        Ubuntu 18.04.4 LTS   5.3.0-1032-gcp   docker://19.3.8
worker-2   Ready    <none>   37m   v1.18.0   10.166.0.29   <none>        Ubuntu 18.04.4 LTS   5.3.0-1032-gcp   docker://19.3.8
worker-3   Ready    <none>   32m   v1.18.0   10.166.0.30   <none>        Ubuntu 18.04.4 LTS   5.3.0-1032-gcp   docker://19.3.8
```
## Автоматическое развертывание кластеров
### Установка kubespray
```
# Пре-реквизиты:
# Python и pip на локальной машине
sudo apt install python-pip
# SSH доступ на все ноды кластера
gcloud compute project-info add-metadata --metadata enable-oslogin=TRUE
gcloud compute os-login ssh-keys add --key-file ~/.ssh/id_rsa.pub

# получение kubespray
git clone https://github.com/kubernetes-sigs/kubespray.git
cd kubespray/

# установка зависимостей
sudo pip install -r requirements.txt
# копирование примера конфига в отдельную директорию
cp -rfp inventory/sample inventory/mycluster
```
### Установка кластера
```
cp ../kubernetes-production-clusters/inventory.ini inventory/mycluster/inventory.ini

SSH_USERNAME=kshuleshov_gcp_gmail_com
SSH_PRIVATE_KEY=${HOME}/.ssh/id_rsa
ansible-playbook -i inventory/mycluster/inventory.ini --become --become-user=root \
  --user=${SSH_USERNAME} --key-file=${SSH_PRIVATE_KEY} cluster.yml
```
### Проверка
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl get nodes
```
```
NAME    STATUS   ROLES    AGE     VERSION
node1   Ready    master   7m53s   v1.18.6
node2   Ready    <none>   6m35s   v1.18.6
node3   Ready    <none>   6m35s   v1.18.6
node4   Ready    <none>   6m35s   v1.18.6
```
## Задание со *
Выполните установку кластера с 3 master-нодами и 2 workerнодами
### Установка кластера
Для установки использован kubespray из предыдущего задания
```
cp ../kubernetes-production-clusters/inventory-3-2.ini inventory/mycluster/inventory.ini

SSH_USERNAME=kshuleshov_gcp_gmail_com
SSH_PRIVATE_KEY=${HOME}/.ssh/id_rsa
ansible-playbook -i inventory/mycluster/inventory.ini --become --become-user=root \
  --user=${SSH_USERNAME} --key-file=${SSH_PRIVATE_KEY} cluster.yml
```
### Проверка
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl get nodes
```
```
NAME    STATUS   ROLES    AGE     VERSION
node1   Ready    master   26m     v1.18.6
node2   Ready    master   25m     v1.18.6
node3   Ready    master   25m     v1.18.6
node4   Ready    <none>   5m7s    v1.18.6
node5   Ready    <none>   4m53s   v1.18.6
```