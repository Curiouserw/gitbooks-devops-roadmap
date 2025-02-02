# ffmpeg

# 一、简介

Github地址：https://github.com/ffmpeg/ffmpeg

官网：https://ffmpeg.org/

# 二、安装

## MacOS

```bash
brew install ffmpeg
```

## Linux

```bash
# CentOS安装基础依赖
yum -y install bzip2 yasm gcc
# SYnology ipkg 安装基础依赖
ipkg install yasm x264 libfdk-aac


version=7.1
wget https://ffmpeg.org/releases/ffmpeg-$version.tar.bz2
tar -xjvf  ffmpeg-$version.tar.bz2
cd ffmpeg-$version


./configure \
    --prefix=/volume2/docker/ffmpeg/ffmpeg7 \
    --disable-asm \
    --enable-gpl \
    --enable-nonfree \
    --enable-libx264 \
    --enable-libfdk-aac \
    --disable-cuda-llvm \
    --arch=x86_64 \
    --target-os=linux
make
make install



./configure --disable-shared --enable-gpl --enable-version3 --enable-nonfree --enable-libx264 --disable-armv6 --disable-armv6t2 --disable-ffplay --prefix=/opt --disable-neon --disable-asm --enable-avcodec --arch=arm --cpu=armv5te --enable-pthreads --disable-decoder=zmbv --target-os=linux --enable-armv5te --enable-static
```

# 三、ffmpeg 命令用法

FFmpeg 命令用法

```bash
ffmpeg [全局选项] {[输入文件选项] -i 输入文件路径/URL } {[输出文件选项] 输出文件路径/URL} 
```

列出所有过滤器选项

```bash
ffmpeg -hide_banner -filters
```

列出硬件加速

```bash
ffmpeg7 -hide_banner -hwaccels
```

# 四、使用案例

## 1、视频

### ①查看文件信息

查看视频文件的元信息，比如编码格式和比特率，可以只使用`-i`参数。

```bash
ffmpeg -i input.mp4
# 加上`-hide_banner`参数，可以只显示元信息

```

使用ffprobe获取视频元信息

```bash
ffprobe -select_streams v -show_entries format=duration,size,bit_rate,filename -show_streams -v quiet -of csv="p=0" -of json -i test.mp4
```

### ②转换编码格式

转换编码格式（transcoding）指的是， 将视频文件从一种编码转成另一种编码。比如转成 H.264 编码，一般使用编码器`libx264`，所以只需指定输出文件的视频编码器即可。

```bash
ffmpeg -i [input.file] -c:v libx264 output.mp4
```

下面是转成 H.265 编码的写法。

```bash
ffmpeg -i [input.file] -c:v libx265 output.mp4
```

### ③转换容器格式

转换容器格式（transmuxing）指的是，将视频文件从一种容器转到另一种容器。下面是 mp4 转 webm 的写法。

```bash
ffmpeg -i input.mp4 -c copy output.webm
```

上面例子中，只是转一下容器，内部的编码格式不变，所以使用`-c copy`指定直接拷贝，不经过转码，这样比较快。

### ④调整码率

调整码率（transrating）指的是，改变编码的比特率，一般用来将视频文件的体积变小。下面的例子指定码率最小为964K，最大为3856K，缓冲区大小为 2000K。

```bash
ffmpeg \
-i input.mp4 \
-minrate 964K -maxrate 3856K -bufsize 2000K \
output.mp4
```

### ⑤改变分辨率（transsizing）

下面是改变视频分辨率（transsizing）的例子，从 1080p 转为 480p 。

```bash
ffmpeg \
-i input.mp4 \
-vf scale=480:-1 \
output.mp4
```

### ⑥提取音频

有时，需要从视频里面提取音频（demuxing），可以像下面这样写。

```bash
ffmpeg \
-i input.mp4 \
-vn -c:a copy \
output.aac
```

上面例子中，`-vn`表示去掉视频，`-c:a copy`表示不改变音频编码，直接拷贝。

### ⑦添加音轨

添加音轨（muxing）指的是，将外部音频加入视频，比如添加背景音乐或旁白。

```bash
ffmpeg \
-i input.aac -i input.mp4 \
output.mp4
```

上面例子中，有音频和视频两个输入文件，FFmpeg 会将它们合成为一个文件。

### ⑧截图

下面的例子是从指定时间开始，连续对1秒钟的视频进行截图。

```bash
ffmpeg \
-y \
-i input.mp4 \
-ss 00:01:24 -t 00:00:01 \
output_%3d.jpg
```

如果只需要截一张图，可以指定只截取一帧。

```bash
ffmpeg \
-ss 01:23:45 \
-i input \
-vframes 1 -q:v 2 \
output.jpg
```

上面例子中，`-vframes 1`指定只截取一帧，`-q:v 2`表示输出的图片质量，一般是1到5之间（1 为质量最高）。

### ⑨裁剪

裁剪（cutting）指的是，截取原始视频里面的一个片段，输出为一个新视频。可以指定开始时间（start）和持续时间（duration），也可以指定结束时间（end）。

```bash
ffmpeg -ss [start] -i [input] -t [duration] -c copy [output]
ffmpeg -ss [start] -i [input] -to [end] -c copy [output]
```



```bash
ffmpeg -ss 00:01:50 -i [input] -t 10.5 -c copy [output]
ffmpeg -ss 2.5 -i [input] -to 10 -c copy [output]
```

上面例子中，`-c copy`表示不改变音频和视频的编码格式，直接拷贝，这样会快很多。

### ⑩将视频切分成TS文件并生成m3u8信息文件

```bash
ffmpeg -i test.mp4 -c:v libx264 -c:a copy -f hls -threads 8 -hls_time 5 -hls_list_size 12 index.m3u8

# -hls_time seconds ： 设置每片的长度，默认值为2。单位为秒
# -hls_list_size size ： 设置播放列表保存的最多条目，设置为0会保存有所片信息，默认值为5
# -hls_wrap wrap ： 设置多少片之后开始覆盖，如果设置为0则不会覆盖，默认值为0.这个选项能够避免在磁盘上存储过多的片，而且能够限制写入磁盘的最多的片的数量
# -start_number number ： 设置播放列表中sequence number的值为number，默认值为0
# 										提示：播放列表的sequence number 对每个segment来说都必须是唯一的，而且它不能和片的文件名混淆，因为在，如果指定了“wrap”选项文件名会出现重复使用。
```

### ⑪合并TS文件

```bash
ffmpeg -i "concat:0.ts|1.ts|2.ts|3.ts|4.ts|5.ts|6.ts" -acodec copy -vcodec copy -absf aac_adtstoasc out.mp4
```

## 2、音频

### ①为音频添加封面

有些视频网站只允许上传视频文件。如果要上传音频文件，必须为音频添加封面，将其转为视频，然后上传。

下面命令可以将音频文件，转为带封面的视频文件。

```bash
ffmpeg \
-loop 1 \
-i cover.jpg -i input.mp3 \
-c:v libx264 -c:a aac -b:a 192k -shortest \
output.mp4
```

上面命令中，有两个输入文件，一个是封面图片`cover.jpg`，另一个是音频文件`input.mp3`。`-loop 1`参数表示图片无限循环，`-shortest`参数表示音频文件结束，输出视频就结束。

## 3、图片

### ①获取图片信息

```bash
# 使用ffprobe获取图片的宽和高
ffprobe -v error -select_streams v:0 -show_entries stream=width,height -of csv=p=0:s=x a.jpg
```

### ②图片格式转换及压缩

```bash
ffmpeg -i 证件照-蓝底.png 证件照-蓝底.jpg 

# 165K的照片能压缩到16K
```

### ③图片转视频

#### 单张循环成视频

```bash
ffmpeg -y -loop 1 -i bg.png -c:v libx264 -t 15 -pix_fmt yuv420p -vf scale=1080:1440 out.mp4
```

#### 多张合并成视频

```bash
ffmpeg -f image2 -i %d.png -vcodec libx264 output2.mp4
其中 %d.png 表示 1.png 2.png ...
```

### ④图片拼接合并

```bash
# 垂直拼接两张图片
ffmpeg -i a.jpg -i b.jpg -filter_complex vstack a_b.jpg

# 水平拼接两张图片
ffmpeg -i a.jpg -i b.jpg -filter_complex hstack a_b.jpg
```

### ⑤尺寸调整

#### 等比例缩放

```bash
# 只调整图片的宽度
target_width=1440
ffmpeg -i a.jpg  -vf "scale=${target_width}:-1" "a_resized.jpg" 

# 只调整图片的长度
target_height=3510
ffmpeg -i a.jpg  -vf "scale=-1:${target_height}" "a_resized.jpg" 
```

### ⑥添加水印

#### 文字水印

在FFmpeg中增加纯文字水印主要使用drawtext滤镜进行操作，drawtext滤镜相关的参数如下：

| 参数      | 值类型 | 说明                       |
| --------- | ------ | -------------------------- |
| fontfile  | 字符串 | 字体文件                   |
| text      | 字符串 | 文字                       |
| textfile  | 字符串 | 文字文件                   |
| fontcolor | 色彩   | 字体颜色                   |
| box       | 布尔   | 文字区域背景框             |
| boxcolor  | 色彩   | 展示字体的区域块的颜色     |
| fontsize  | 整数   | 显示字体大小               |
| font      | 字符串 | 字体名称（默认为Sans字体） |
| x         | 𤨣数   | 文字显示的x坐标            |
| У         | 整数   | 文字显示的y坐标            |

```bash
ffmpeg -i a.jpg -vf "drawtext=text='我是文字水印':fontcolor=white@0.5:fontsize=48:x=10:y=10" -update 1 a-w.jpg

# drawtext= 指定绘制文字的滤镜，包含以下参数：
#    text：要添加的文字内容（可以替换为你的自定义文本）。
#    fontcolor=white@0.5：文字颜色为白色，并设置透明度为50%（@0.5 表示透明度，0为全透明，1为不透明）。
#    fontsize=24：文字大小（可以调整以适应图片）。
#    x=W-tw-10 和 y=H-th-10：文字位置，W 和 H 是图片宽度和高度，tw 和 th 是文字宽度和高度，这里设置的是右下角位置并距离图片边界10像素。
#         左上角：x=10:y=10
#         右上角：x=W-tw-10:y=10
#         左下角：x=10:y=H-th-10
#         居中：x=(W-tw)/2:y=(H-th)/2

```

# 参考

1. http://www.ruanyifeng.com/blog/2020/01/ffmpeg.html
2. https://zhuanlan.zhihu.com/p/67878761
