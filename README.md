# mysql-kubernetes-cluster
Build the mysql cluster in the kubernetes

Requirement Environment:

- Kubernetes
- NFS
- Sysbench


Process:

- Prepare the init.sh which configure master and slave server and primary.cnf in the config map.

> kubectl -n ${namespace} apply -f mysql-configmap.yaml


- Store the user and password for mysql connection

> kubectl -n ${namespace} apply -f mysql-secret.yaml


- Build the new volume claim for sharing backup file between master and slave server.

> kubectl -n ${namespace} apply -f mysql-volume-claim.yaml


- Build the service which use nodeport to performance testing for statefulset

> kubectl -n ${namespace} apply -f mysql-service.yaml


- Setup the statefulset to create referenced pod entity

> kubectl -n ${namespace} apply -f mysql-statefulset.yaml


- Build the master service which can be write.

> kubectl -n ${namespace} apply -f mysql-service-master.yaml


- Check the status

> kubectl -n ${namespace} get pod --watch



Test:

- List the service for node port

> kubectl -n ${namespace} get svc


read-write-service: 32643

read-only-service: 31290 

pod-server: testsvr


- Run the sysbench test

> sysbench /usr/share/sysbench/oltp_read_only.lua --threads=48 --tables=50 --db-driver=mysql --mysql-host=testsvr --mysql-user=admin --mysql-password=Esonew2Z --mysql-port=32643 prepare

> sysbench /usr/share/sysbench/oltp_read_only.lua --threads=48 --tables=40 --db-driver=mysql --mysql-host=testsvr --mysql-user=admin --mysql-password=Esonew2Z --mysql-port=31290 run


Get the result

Threads started!

SQL statistics:
    queries performed:
        read:                            151774
        write:                           0
        other:                           21682
        total:                           173456
    transactions:                        10841  (1073.13 per sec.)
    queries:                             173456 (17170.06 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

