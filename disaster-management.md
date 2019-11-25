
# Always make sure to keep the backup of the Cluster in order to restore it if everything is down and need to start a new cluster from scratch. Please follow below.

## Use innobackupex util to the ensure backups, more on https://github.com/amitsdalal/xtrabackup-scripts link for the same.

### Having said that, you've backups in place and now need to create new cluster, please follow below.


##### Open Firewall ports
```
firewall-cmd --permanent --add-port={3306/tcp,4444/tcp,4567/tcp,4568/tcp}
firewall-cmd --reload
```
##### Disable SELinux
```
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/sysconfig/selinux
```

```
yum install -y https://repo.percona.com/yum/percona-release-latest.noarch.rpm
percona-release setup ps57
yum install -y Percona-XtraDB-Cluster-57
```

```
cat >>/etc/my.cnf<<EOF
 [mysqld]
   wsrep_provider=/usr/lib64/galera3/libgalera_smm.so
   wsrep_cluster_name=newclustername
   wsrep_cluster_address=gcomm://<IP of the server and of addon servers for the cluster>
   wsrep_node_name=<hostname of the server>
   wsrep_node_address=<IP of the server>
   wsrep_sst_method=xtrabackup-v2
   wsrep_sst_auth=repuser:reppassword
   pxc_strict_mode=ENFORCING
   binlog_format=ROW
   default_storage_engine=InnoDB
   innodb_autoinc_lock_mode=2
EOF
```
```
mv /var/lib/mysql .
```
innobackupex --copy-back <path of innobackupex dir>

chown -R mysql: /var/lib/mysql
service mysql@bootstrap start

```

This will get the cluster started and now follow the steps to add 2nd server from as below.


### On Second node
##### Open Firewall ports
```
firewall-cmd --permanent --add-port={3306/tcp,4444/tcp,4567/tcp,4568/tcp}
firewall-cmd --reload
```
##### Disable SELinux
```
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/sysconfig/selinux
```
##### Add Percona Repository
```
yum install -y https://repo.percona.com/yum/percona-release-latest.noarch.rpm
```
##### Install Percona-XtraDB-Cluster
```
yum install -y Percona-XtraDB-Cluster-57
```
##### Configure Replication Settings
```
```
cat >>/etc/my.cnf<<EOF
 [mysqld]
   wsrep_provider=/usr/lib64/galera3/libgalera_smm.so
   wsrep_cluster_name=newclustername
   wsrep_cluster_address=gcomm://<IP of the server and of addon servers for the cluster>
   wsrep_node_name=<hostname of the server>
   wsrep_node_address=<IP of the server>
   wsrep_sst_method=xtrabackup-v2
   wsrep_sst_auth=repuser:reppassword
   pxc_strict_mode=ENFORCING
   binlog_format=ROW
   default_storage_engine=InnoDB
   innodb_autoinc_lock_mode=2
EOF
```
##### Start mysql to join the cluster
```
systemctl start mysql
``` 
  
  # Please note the logins will be same as it was when time of backup.
  
  
  # wsrep_sst_auth must be the same when the backup was taken.
  
  ### Place Your MySQL tunning setting and restart the services.
  
  #### This was tested on CentOS7 but it must be similar for Ubuntu 18 as well.
  
