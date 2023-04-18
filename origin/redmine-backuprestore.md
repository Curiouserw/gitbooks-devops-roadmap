# Redmine的备份与恢复

# 一、简介

要备份的数据类型

- 数据库中的数据
- 用户上传的附件





# 二、备份

Redmine backups should include:

- **Database**
- **Attachments** (stored in the `files` directory under the installation directory by default)

## 备份数据库数据



- **MySQL**

  The `mysqldump` command can be used to backup the contents of your MySQL database to a text file. For example:

  ```
  /usr/bin/mysqldump -u <username> -p<password> -h <hostname> <redmine_database> > /path/to/backup/db/redmine.sql
  ```
  
  You can find ``, ``, ``, and `` in the file `config/database.yml`. `` may not be required depending on your installation of the database.



- **PostgreSQL**

  The `pg_dump` command can be used to backup the contents of a PostgreSQL database to a text file. Here is an example:

  ```
  /usr/bin/pg_dump -U <username> -h <hostname> -Fc --file=redmine.sqlc <redmine_database>
  ```
  
  You can find ``, ``, and `` in the file `config/database.yml`. `` may not be required depending on your installation of the database. The `pg_dump` command will prompt you to enter the password when necessary.

- **SQLite**

  SQLite databases are all contained in a single file, so you can back them up by copying the file to another location.

  You can determine the file name of SQLite database by looking at `config/database.yml`.



## 备份用户上传的附件

All file uploads are stored in `attachments_storage_path` (defaults to the `files/` directory). You can copy the contents of this directory to another location to easily back it up.

WARNING: `attachments_storage_path` may point to a different directory other than `files/`. Be sure to check the setting in `config/configuration.yml` to avoid making a useless backup.



## 备份脚本

Here is a simple shell script that can be used for daily backups (assuming you're using a MySQL database):

```
# Database
/usr/bin/mysqldump -u <username> -p<password> <redmine_database> | gzip > /path/to/backup/db/redmine_`date +%Y-%m-%d`.gz

# Attachments
rsync -a /path/to/redmine/files /path/to/backup/files
```



# 三、恢复

## 恢复数据库

- **MySQL**

  For example if you have a gziped dump file with the name `2018-07-30.gz`, then the database can be restored with the following command:

  ```
  gunzip -c 2018-07-30.gz | mysql -u <username> --password <redmine_database>
  Enter password:
  ```

- **PostgreSQL**

  When the option `-Fc` of the command `pg_dump` is used like it is at the above example then you need to use the command `pg_restore`:

  ```
  pg_restore -U <username> -h <hostname> -d <redmine_database> redmine.sqlc
  ```

  otherwise a text file can be restored with `psql`:

  ```
  psql <redmine_database> < <infile>
  ```

- **SQLite**

  Copy the database file from the backup location.









# 参考

1. https://www.redmine.org/boards/2/topics/2442?r=18660
2. https://www.redmine.org/projects/redmine/wiki/RedmineBackupRestore