# MySQL常见问题解决方案

# 1、容器化MySQL启动失败

**上下文**

- 部署在K8s中的单MySQL实例，数据文件路径使用NFS存储后端挂载的PVC持久化的。再一次重建MySQL Pod后，创建失败。报错如下：

```bash
2021-12-17T11:57:36.429443Z 0 [ERROR] InnoDB: Only one log file found.
2021-12-17T11:57:36.429472Z 0 [ERROR] InnoDB: Plugin initialization aborted with error not found
2021-12-17T11:57:37.029965Z 0 [ERROR] Plugin 'InnoDB' init function returned error.
2021-12-17T11:57:37.029998Z 0 [ERROR] Plugin 'InnoDB' registration as a STORAGE ENGINE failed.
2021-12-17T11:57:37.030005Z 0 [ERROR] Failed to initialize builtin plugins.
2021-12-17T11:57:37.030008Z 0 [ERROR] Aborting
2021-12-17T11:57:37.030012Z 0 [Note] Binlog end
2021-12-17T11:57:37.030062Z 0 [Note] Shutting down plugin 'CSV'
2021-12-17T11:57:37.031982Z 0 [Note] mysqld: Shutdown complete
```

- 检查数据路径下存在`ib_logfile0 、ib_logfile1`文件

**原因：**

- `ib_logfile0 、ib_logfile1`文件是MySQL InnoDB引擎的事务日志。MySQL崩溃重启时，会进行事务重做；在系统正常时，每次checkpoint时间点，会将之前写入事务应用到数据文件中。

**解决方案：**

- 备份所有`ib_logfile*`文件到其他目录后重启。



参考：

- https://dba.stackexchange.com/questions/41542/issue-after-moving-the-ib-logfile1-and-ib-logfile0-files

- https://www.codeleading.com/article/33775310436/

- https://www.cnblogs.com/qianyuliang/p/9916372.html

- https://dba.stackexchange.com/questions/87692/flush-clean-mysql-ib-logfile0-ib-logfile1

  

