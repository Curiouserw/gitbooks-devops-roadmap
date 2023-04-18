# Grafana的备份恢复与升级

# 一、备份恢复

官方文档：https://grafana.com/docs/grafana/latest/installation/upgrading/#backup

- **数据存储文件**
  - **MySQL**
    - 备份： `mysqldump -u root -p[root_password] [grafana] > grafana_backup.sql`
    - 恢复：`mysql -u root -p grafana < grafana_backup.sql`
  - **Postgres**
    - 备份： `pg_dump grafana > grafana_backup`
    - 恢复：`psql grafana < grafana_backup`
  - **Sqlite**(默认)
    - 备份：`直接备份DB文件grafana.db，默认路径：/var/lib/grafana/`
    - 恢复：
      - 直接将备份的grafana.db文件复制到`/var/lib/grafana/`下
      - 修改权限：`sudo chown nobody.nogroup grafana.db && sudo chmod 640 grafana.db`
- **配置文件**
  - 备份：直接备份`/etc/grafana/grafana.ini`。（对于部署在k8s中的，配置文件是使用configmap挂载的可以不用备份）
  - 恢复：直接恢复使用备份文件`/etc/grafana/grafana.ini`
- **已安装的插件**
  - 备份：直接备份插件目录
  - 恢复
    - 直接恢复备份的插件目录
    - 升级插件：`grafana-cli plugins update-all`

# 二、问题总结

## 1、删除默认组织后使用SQLite备份文件grafana.db恢复时报错

**报错信息：**

```bash
Datasource provisioning error: failed to provision "prometheus" data source: Organization not found
```

**解决方案：**

​		使用Navicat连接SQLite备份文件grafana.db，在`main.org`表中添加一条记录

```bash
INSERT INTO "main"."org" ("id", "version", "name", "address1", "address2", "city", "state", "zip_code", "country", "billing_email", "created", "updated") VALUES (2, 3, 'Main Org.', '', '', '', '', '', '', NULL, '2020-11-26 03:39:06', '2020-11-26 03:39:06');
```







