### Задание 1
> kubectl get pods -n kube-sytem

NAMESPACE              NAME                                        READY   STATUS    RESTARTS   AGE
* kube-system            coredns-74ff55c5b-cjd7v                     1/1     Running   1          22d
* kube-system            etcd-minikube                               1/1     Running   1          22d
* kube-system            kube-apiserver-minikube                     1/1     Running   1          22d
* kube-system            kube-controller-manager-minikube            1/1     Running   0          3m1s
* kube-system            kube-proxy-glw7p                            1/1     Running   1          22d
* kube-system            kube-scheduler-minikube                     1/1     Running   0          3m32s
* kube-system            metrics-server-56c4f8c9d6-nd7zj             1/1     Running   1          4d23h
* kube-system            storage-provisioner                         1/1     Running   2          22d
* kubernetes-dashboard   dashboard-metrics-scraper-f6647bd8c-xc6hp   1/1     Running   1          22d
* kubernetes-dashboard   kubernetes-dashboard-968bcb79-rkmn6         1/1     Running   2          22d

**coredns-74ff55c5b-cjd7v**
Controlled by the deployment - coredns, который создал ReplicaSet coredns-74ff55c5b. Параметр replicas: 1 определяет количество подов, которое должно быть запущено, поэтому coredns перезапускается после удаления.

**etcd-minikube, kube-apiserver-minikube, kube-controller-manager-minikube, kube-scheduler-minikube** – статические поды.Запускуются kubelet из файлов, расположенных в каталоге /etc/kubernetes/manifests/. Т.к. kubelet запущен с параметром --config=/var/lib/kubelet/config.yaml, а в данном файле содержится параметр staticPodPath: /etc/kubernetes/manifests

**kube-proxy-glw7p**
Управляется через DaemonSet kube-proxy. Через DaemonSet по одному экземпляру пода запускается на каждой (в общем случае, или на некоторых) ноде, входящей в кластер.

**metrics-server-56c4f8c9d6**
Управляется через deployment metrics-server, который создаёт replicaset metrics-server-56c4f8c9d6, который через параметр replicas: 1 определяет необходимое количество подов metrics-server, которые должны находиться в работающем состоянии.

**dashboard-metrics-scraper-f6647bd8c-xc6hp**
Управляется через deployment dashboard-metrics-scraper, который создаёт replicaset dashboard-metrics-scraper-f6647bd8c, который через параметр replicas: 1 определяет необходимое количество подов metrics-server, которые должны находиться в работающем состоянии.

**kubernetes-dashboard-968bcb79-rkmn6**
Управляется через deployment kubernetes-dashboard, который создаёт replicaset kubernetes-dashboard-968bcb79, который через параметр replicas: 1 определяет необходимое количество подов metrics-server, которые должны находиться в работающем состоянии.

А вот под **storage-provisioner** – это просто pod. Если удалить его через kubectl delete pod, то он не пересоздаётся.

### Задание 2
Созданы Dockerfile и манифест пода web-pod.yaml.

### Задание 3
Для запуска frontend необходимо добавить в манифести переменные окружения согласно тому как это указано в файле main.go:
        _mustMapEnv(&svc.productCatalogSvcAddr, "PRODUCT_CATALOG_SERVICE_ADDR")_
        _mustMapEnv(&svc.currencySvcAddr, "CURRENCY_SERVICE_ADDR")_
        _mustMapEnv(&svc.cartSvcAddr, "CART_SERVICE_ADDR")_
        _mustMapEnv(&svc.recommendationSvcAddr, "RECOMMENDATION_SERVICE_ADDR")_
        _mustMapEnv(&svc.checkoutSvcAddr, "CHECKOUT_SERVICE_ADDR")_
        _mustMapEnv(&svc.shippingSvcAddr, "SHIPPING_SERVICE_ADDR")_
        _mustMapEnv(&svc.adSvcAddr, "AD_SERVICE_ADDR")_
