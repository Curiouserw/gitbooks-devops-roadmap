# Ghostscript

# 一、简介

什么是 Ghostscript？要了解这一点，应该先了解一下什么是Postscript。

### Postscript

postscript 是Adobe提出的一种打印机语言，ghostscript可以看做是postscript的一个解释器，它实现了postscript的语言标准，同时附加了一些其独有的操作指令。

PDF格式是Postscript语言的扩展，它增加了更多的功能。

### Ghostscript

Ghostscript是一个免费的开源解释器，用于渲染Postscript和PDF文档。

Ghostscript提供了一个语言绑定的API，Ghostscript的功能可以用其他语言实现，使我们可以编写自己的程序来修改PDF文档。支持的语言有 C#、Java 和 Python。

官网：https://www.ghostscript.com/

下载地址：https://www.ghostscript.com/releases/gsdnld.html

文档：https://ghostscript.readthedocs.io/en/gs10.0.0/toc.html

# 二、安装

## MacOS

```bash
brew install ghostscript
```

## APT

```bash
apt install ghostscript
```

## YUM

```bash
yum install ghostscript
```

## Anaconda

```bash
conda install -c conda-forge ghostscript
```

## 源码编译安装

```bash
wget https://github.com/ArtifexSoftware/ghostpdl-downloads/releases/download/gs9561/gs_9.56.1_amd64_snap.tgz
tar -xzf gs_9.56.1_amd64_snap.tgz
./configure
make
sudo make install
```

# 三、gs命令

## 1. gs命令

| 操作系统                | 命令     |
| :---------------------- | -------- |
| Unix                    | gs       |
| VMS                     | gs       |
| MS Windows 95 and later | gswin32c |
| OS/2                    | gsos2    |

## 2、命令参数

| 参数                             | 内容                                        | 选项                                                         |
| -------------------------------- | :------------------------------------------ | ------------------------------------------------------------ |
| -dPDFSETTINGS                    | 指定压缩模式                                | /screen，<br/>    压缩比最大,输出文件最小,质量最低,72 dpi<br/>/ebook，<br/>    压缩比稍小,输出文件稍大,质量稍高,150 dpi<br/>/printer,  300 dpi<br/>/prepress，<br/>    输出文件信息同Acrobat "Prepress Optimized"设置,300 dpi<br/>/default，默认，等同于/screen |
| -dFirstPage                      | 从第几页开始                                |                                                              |
| -dLastPage                       | 到第几页结束                                |                                                              |
| -sOutputFile                     | 输出为文件的路径                            |                                                              |
| -dQUIET / -q                     | 不输出处理日志                              |                                                              |
| -dBATCH                          | 执行到最后一页后退出                        |                                                              |
| -dNOPAUSE                        | 每一页转换之间没有停顿                      |                                                              |
| -sDEVICE                         | 转换输出的文件类型装置                      | 可以`gs -h |grep "Default output device`查看默认值           |
| -r / <br/>-dColorImageResolution | 指定图片分辨率<br/>（即图片解析度为300dpi） | 例如：-r300                                                  |
| -g720x1280                       | 指定图片像素()，一般不指定，使用默认输出    | 格式：\<width>x\<height>                                     |

# 四、常用操作

## 1、压缩PDF

```bash
gs -sDEVICE=pdfwrite \
   -dCompatibilityLevel=1.4 \
   -dPDFSETTINGS=/ebook \
   -dNOPAUSE \
   -dBATCH \
   -sOutputFile=output.pdf \
   input.pdf
```

## 2、多PDF合并成一个PDF

```bash
gs -dNOPAUSE  \
   -sDEVICE=pdfwrite \
   -sOUTPUTFILE=./output.pdf \
   -dBATCH \
   ./input-test1.pdf ./input-test2.pdf
```

## 3、拆分PDF

```bash
gs -q -dBATCH \
   -dNOPAUSE \
   -sDEVICE=pdfwrite \
   -dFirstPage=3 \
   -dLastPage=3 \
   -sOutputFile=output.pdf \
   input.pdf
```

## 4、将PDF转换为PNG

```bash
gs -sDEVICE=jpeg \
   -r300 \
   -o output-%02d.jpeg \
   input.pdf
   
# %02d 两位数自动补零；三位数自动补零%03d
```

## 5、添加英文水印

```bash
Watermark="only for something" && \
filepath="~/Desktop/测试PDF.pdf" && \
gs -q -o "${filepath%%.*}-带英文水印.pdf" \
   -sDEVICE=pdfwrite \
   -dBATCH \
   -dNOPAUSE \
   -c "<< \
        /EndPage
        {
          2 eq { pop false }
          {
              gsave
              /Helvetica-Bold 60 selectfont \
              0 setgray                      % 设置灰度
              100 300 moveto                 % 设置水印起始位置
              45 rotate                      % 倾斜水印 45 度
              ($Watermark) show            % 绘制水印文本
              grestore
              true
          } ifelse
        } bind
      >> setpagedevice" \
   -f "$filepath"
```

# 参考

- https://juejin.cn/post/7119342874503675940
- https://milan.kupcevic.net/ghostscript-ps-pdf/
- https://xz.aliyun.com/t/6392?page=1