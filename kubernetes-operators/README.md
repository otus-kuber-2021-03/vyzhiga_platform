Домашняя работа 7 (kubernetes-operators)

Результат выполнения:
kubectl get jobs:

$ kubectl get jobs

NAME                         COMPLETIONS   DURATION   AGE
backup-mysql-instance-job    1/1           1s         7m4s
restore-mysql-instance-job   1/1           3m22s      8m25s

Результат выполнения:
$ export MYSQLPOD=$(kubectl get pods -l app=mysql-instance -o jsonpath=" {.items[].metadata.name}")*

$ kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "select * from test;" otus-database

mysql: [Warning] Using a password on the command line interface can be insecure.
+----+-------------+
| id | name        |
+----+-------------+
|  1 | some data   |
|  2 | some data-2 |
+----+-------------+