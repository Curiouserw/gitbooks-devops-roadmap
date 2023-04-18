# Imagemagick

# 一、简介

官网：https://imagemagick.org

# 二、安装

下载安装文档：https://imagemagick.org/script/download.php

## 1、安装器安装

```bash
# MacOS
brew install imagemagick@6
# Debian/Ubuntu/Raspbian
apt-get install graphicsmagick-imagemagick-compat
# Alpine
apk add imagemagick6
```

## 2、Docker

```bash
docker run cmd.cat/convert convert
```

# 三、命令参数

## 1、magick

## 2、magick-script

# 四、常用

## 1、压缩PDF

```bash
convert -density 600 -compress jpeg -quality 60 身份证.pdf 身份证-压缩.pdf
```
