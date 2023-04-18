# 一、简介



## 二、安装

## cygwin

```bash
apt-cyg install p7zip
```

# 三、压缩命令参数

Windows下

```bash
7z.exe a -t7z C:\\Users\\Maker\\Desktop\\test.7z \
    -xr\!.git \
    -xr\!logs \
    -mmt \
    C:\\Users\\Maker\\Desktop\\test1 \
    C:\\Users\\Maker\\Desktopp\\test2 > null
    
# a 压缩
# -xr\!.git 排除.git文件夹
# -xr\!logs 排除logs文件夹
# -mnt 使用多线程
# -t7z 指定压缩格式，选项：zip、7z（默认）、rar、cab、gzip、bzip2、tar 或其它格式。
# > null 静默输出压缩过程
```



参考：

- https://www.cnblogs.com/qanholas/archive/2011/10/03/2198487.html
- https://superuser.com/questions/97342/7zip-command-line-exclude-folders-by-wildcard-pattern
- https://stackoverflow.com/questions/3774278/extracting-a-7-zip-file-silently-command-line-option
- https://www.shuzhiduo.com/A/kvJ3waMOJg/