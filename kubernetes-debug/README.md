# Kubernetes debug
## kubectl debug
### Установите в ваш кластер kubectl debug
 - `wget https://github.com/aylei/kubectl-debug/releases/download/v0.1.1/kubectl-debug_0.1.1_linux_amd64.tar.gz -O- | tar xz kubectl-debug && sudo mv kubectl-debug /usr/local/bin/`
### Запустите в кластере поды с агентом kubectl-debug
 - `kubectl apply -f https://raw.githubusercontent.com/aylei/kubectl-debug/dd7e4965e4ae5c4f53e6cf9fd17acc964274ca5c/scripts/agent_daemonset.yml`
### Проверьте работу команды strace
 - `kubectl apply -f kubernetes-debug/strace/web-pod.yaml`
 - `kubectl-debug web --agentless=false --port-forward=true`
```
strace -c -p1
```
```
strace: attach: ptrace(PTRACE_SEIZE, 1): Operation not permitted
```
### Определите, почему не работает strace и почините
Агент не устанавливает capabilities "SYS_PTRACE", "SYS_ADMIN". Необходимо обновить агент до последней версии.
 - `kubernetes-debug/strace/run.sh`
 - `kubectl-debug web --agentless=false --port-forward=true`
```
strace -c -p1
```
```
strace: Process 1 attached
```
## iptables-tailer
### iptables-tailer | Тестовое приложение
```
kubectl apply -f https://raw.githubusercontent.com/piontec/netperf-operator/master/deploy/crd.yaml
kubectl apply -f https://raw.githubusercontent.com/piontec/netperf-operator/master/deploy/rbac.yaml
kubectl apply -f https://raw.githubusercontent.com/piontec/netperf-operator/master/deploy/operator.yaml
kubectl apply -f https://raw.githubusercontent.com/piontec/netperf-operator/master/deploy/cr.yaml
kubectl describe netperf.app.example.com/example
```
```
Name:         example
Namespace:    default
Labels:       <none>
Annotations:  API Version:  app.example.com/v1alpha1
Kind:         Netperf
Metadata:
  Creation Timestamp:  2020-07-30T21:43:35Z
  Generation:          4
  Resource Version:    63437
  Self Link:           /apis/app.example.com/v1alpha1/namespaces/default/netperfs/example
  UID:                 696968ed-c681-4ef1-a9fc-76d0fd34f544
Spec:
  Client Node:
  Server Node:
Status:
  Client Pod:          netperf-client-76d0fd34f544
  Server Pod:          netperf-server-76d0fd34f544
  Speed Bits Per Sec:  1768.9
  Status:              Done
Events:                <none>
```
### iptables-tailer | Сетевые политики
```
kubectl apply -f kubernetes-debug/kit/netperf-calico-policy.yaml
kubectl delete -f https://raw.githubusercontent.com/piontec/netperf-operator/master/deploy/cr.yaml
kubectl apply -f https://raw.githubusercontent.com/piontec/netperf-operator/master/deploy/cr.yaml
kubectl describe netperf.app.example.com/example
```
```
Status:
  Client Pod:          netperf-client-8330b7e55db2
  Server Pod:          netperf-server-8330b7e55db2
  Speed Bits Per Sec:  0
  Status:              Started test
```
### iptables-tailer | Установка
```
kubectl apply -f kubernetes-debug/kit/kit-clusterrole.yaml
kubectl apply -f kubernetes-debug/kit/kit-clusterrolebinding.yaml
kubectl apply -f kubernetes-debug/kit/kit-serviceaccount.yaml
kubectl apply -f kubernetes-debug/kit/iptables-tailer.yaml
```
### iptables-tailes | Проверка
```
kubectl delete -f https://raw.githubusercontent.com/piontec/netperf-operator/master/deploy/cr.yaml
kubectl apply -f https://raw.githubusercontent.com/piontec/netperf-operator/master/deploy/cr.yaml
kubectl describe pod --selector=app=netperf-operator
```
```
Events:
  Type     Reason      Age   From                                               Message
  ----     ------      ----  ----                                               -------
  Normal   Scheduled   81s   default-scheduler                                  Successfully assigned default/netperf-server-fa66f966b976 to gke-cluster-1-default-pool-a3e69bb8-3qm4
  Normal   Pulled      81s   kubelet, gke-cluster-1-default-pool-a3e69bb8-3qm4  Container image "tailoredcloud/netperf:v2.7" already present on machine
  Normal   Created     80s   kubelet, gke-cluster-1-default-pool-a3e69bb8-3qm4  Created container netperf-server-fa66f966b976
  Normal   Started     80s   kubelet, gke-cluster-1-default-pool-a3e69bb8-3qm4  Started container netperf-server-fa66f966b976
  Warning  PacketDrop  78s   kube-iptables-tailer                               Packet dropped when receiving traffic from 10.0.1.19
```
## Задание со ⭐
### Исправьте ошибку в нашей сетевой политике, чтобы Netperf снова начал работать
 - `kubectl apply -f kubernetes-debug/kit/netperf-calico-policy-fixed.yaml`
### Поправьте манифест DaemonSet из репозитория, чтобы в логах отображались имена Podов, а не их IP-адреса
```diff
               - name: "POD_IDENTIFIER"
-                value: "label"
-              - name: "POD_IDENTIFIER_LABEL"
-                value: "netperf-type"
+                value: "name"
```