## Create a Service for Galera Cluster 

It was necessary to create a service for each VM that is going to join our Galera cluster. Galera cluster requires network connectivity between the nodes with many ports. For this reason, a cluster-internal IP (“ClusterIP”) is allocated for each VM, which enables the VMs to communicate with each other. Here is an overview of the ports and their uses:

- 3306 is the default port for MySQL client connections and state snapshot transfer using MySQL dump for backups.
- 4567 is reserved for Galera cluster replication traffic. Multicast replication uses both TCP and UDP transport on this port.
- 4568 is the port for incremental state transfer.
- 4444 is used for all other state snapshot transfers.

### For example, a snapshot of yaml configuration for the first VM in my project (mariadb-0).

```
apiVersion: v1
kind: Service
metadata:
  name: <service name>
  namespace: <namespace>
spec:
  selector:
    kubevirt.io/domain: <VM name>
  ports:
    - protocol: TCP
      name: tcp-3306
      port: 3306
      targetPort: 3306
    - protocol: TCP
      name: tcp-4567
      port: 4567
      targetPort: 4567
    - protocol: TCP
      name: tcp-4568
      port: 4568
      targetPort: 4568
    - protocol: TCP
      name: tcp-4444
      port: 4444
      targetPort: 4444
    - protocol: UDP
      name: udp-4567
      port: 4567
      targetPort: 4567
```

<img width="666" alt="Screen Shot 2022-11-10 at 15 29 40" src="https://user-images.githubusercontent.com/64369864/201104431-3711b3c6-7bd0-4a85-b543-c6d4d0e684e1.png">

<img width="657" alt="Screen Shot 2022-11-10 at 15 29 59" src="https://user-images.githubusercontent.com/64369864/201104495-daf174ca-92a5-4b4e-afe3-f418a95bdd88.png">

### Firewall Rules for MariaDB 

- Opening these ports via the firewall for each VM that is going to join our Galera cluster. This step allows us to access the ports. 

```
sudo firewall-cmd --permanent --zone=public --add-port=3306/tcp
sudo firewall-cmd --permanent --zone=public --add-port=4567/tcp
sudo firewall-cmd --permanent --zone=public --add-port=4568/tcp
sudo firewall-cmd --permanent --zone=public --add-port=4444/tcp
sudo firewall-cmd --permanent --zone=public --add-port=4567/udp
```

- Reload the firewall to apply the changes.
```
sudo firewall-cmd --reload
```
- Stop Selinux (Selinux is not a part from this lab)
```
setenforce 0
```

## Create Galera Cluster 
### Install and Configure the MariaDB First Instance
- The MariaDB first instance is the most important instance at installation, this instance will essentially be the “primary” in our cluster. Without this instance, nothing can be started and the cluster cannot be created. From this instance all other instances will launch, connect to and sync up with.

- Install the actual MariaDB and Galera packages.
```
sudo dnf module install mariadb/galera
```
- Configure the MariaDB instance.
```
sudo vi /etc/my.cnf
```

```
[galera]
wsrep_on=ON
wsrep_cluster_name=<'galera_cluster’> 
binlog_format=ROW
bind-address=0.0.0.0

default-storage-engine=InnoDB
innodb_autoinc_lock_mode=2
innodb_doublewrite=1
query_cache_size=0
wsrep_provider=/usr/lib64/galera-4/libgalera_smm.so

wsrep_cluster_address=gcomm://

wsrep_sst_method=rsync
wsrep_dirty_reads=ON
wsrep-sync-wait=0

wsrep_node_address=<'mariadb-0-ports.galera-cluster.svc.cluster.local'>

!includedir /etc/my.cnf.d
```
> Enter the cluster name
> Enter the IP address or service that allocate to the first VM

- Start the mariaDB service.
```
sudo systemctl start mariadb
```
- Check the service status.

```
sudo systemctl status mariadb
```
<img width="418" alt="Screen Shot 2022-11-10 at 15 37 10" src="https://user-images.githubusercontent.com/64369864/201106051-2d0d0198-924d-4660-a1d6-c0e58e367fea.png">

- Stop the MariaDB service and run galera command. This command will start the service automatically.
```
systemctl stop mariadb

galera_new_cluster

```
## Connect to MariaDB and Check the Cluster Size
- Connect to MariaDB.
```
mysql -u root -p 

SHOW STATUS LIKE 'wsrep_cluster_size';
```

> This is the first instance of the cluster, so it will have a cluster size of one.

<img width="672" alt="Screen Shot 2022-11-10 at 15 40 11" src="https://user-images.githubusercontent.com/64369864/201106767-27e5725a-df07-4afd-bfd9-851fe6ba591d.png">

## Install and Configure the MariaDB Second Instance

The second instance is the instance that essentially creates High availability, this instance is a second instance that can be written to, read from, and acts like a normal DB. This instance connects to the first MariaDB instance.

- Install the actual MariaDB and Galera packages.
```
sudo dnf module install mariadb/galera
```
- Configure the MariaDB instance.

```
sudo vi /etc/my.cnf
```
```
[galera]
wsrep_on=ON
wsrep_cluster_name=<'galera_cluster'>
binlog_format=ROW
bind-address=0.0.0.0

default-storage-engine=InnoDB
innodb_autoinc_lock_mode=2
innodb_doublewrite=1
query_cache_size=0
wsrep_provider=/usr/lib64/galera-4/libgalera_smm.so

wsrep_cluster_address=gcomm://<mariadb-0-ports.galera-cluster.svc.cluster.local,mariadb-1-ports.galera-cluster.svc.cluster.local> 

wsrep_provider_options="ist.recv_bind=10.0.2.2" 


wsrep_sst_method=rsync
wsrep_dirty_reads=ON
wsrep-sync-wait=0

wsrep_node_address=<'mariadb-1-ports.galera-cluster.svc.cluster.local'> 

!includedir /etc/my.cnf.d
```
> Enter the cluster name.
> Enter the IP address or service for each VM that is going to join our Galera cluster. 
> Enter the Internal IP address, In OpenShift Virtualization, is always 10.0.2.2.
> Enter the IP address or service that allocate to the second VM.

- Start the MariaDB service 
```
sudo systemctl start mariadb
```
- Check the service status
```
sudo systemctl status mariadb
```
## Ensure the Cluster is Created 

Return to the first instance and check the cluster size. The size changed to two, that is the cluster has two instances and thus is highly available for our database. This is important to make sure that the cluster has indeed increased and that our second instance has joined properly. This step also signifies the ability to write and read from both maria-0 and maria-1. When both instances join the cluster they both can be accessed equally and the replication is done automatically.

- Connect to MariaDB.
```
mysql -u root -p 

SHOW STATUS LIKE ‘wsrep_cluster_size’;
```

<img width="677" alt="Screen Shot 2022-11-10 at 15 44 57" src="https://user-images.githubusercontent.com/64369864/201107810-6fd4247c-f893-4e98-a578-d8b7f948ae2e.png">









