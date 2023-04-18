# HDFS

# 一、简介

# 二、基础概念

# 三、常用操作

## 1、查看HDFS容量

```
hdfs dfs -df -h
```

## 2、删除HDFS上的文件(删除的文件会放到操作用户的回收站里)

```
#删除目录
hdfs dfs -rm -r /test  
#删除文件
hdfs dfs -rm /test/a
```

## 3、直接删除文件（不放进回收站）

```
#删除目录
hdfs dfs -rm -r -skipTrash /test  
#删除文件
hdfs dfs -rm -skipTrash /test/a
```

## 4、上传文件 

```
hdfs dfs -put 本地文件 远程目录
```

## 5、下载文件

```
hdfs dfs -get  远程文件 本地目录
```