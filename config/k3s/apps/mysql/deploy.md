# MySQL
数据库服务，本地开发测试使用。  
其实不建议将数据库部署到集群中，但是我只是本地测试用无所谓了

```bash
kubectl create ns mysql
kubectl apply -f mysql.yaml
```

MySQL 一定要用本地卷，SMB 和 NFS 卷都是无法启动的。本地卷的代价就是只能跑在一台机器上了，这里应该是用 iSCSI 或者 Ceph 比较好的，懒得搞了。