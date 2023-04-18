# XML处理工具xmlint

# 一、简介

xmllint其实是由一个叫libxml2的c语言库函数实现的一个小工具，因此效率比较高，对不同系统的支持度也很好，功能也比较全。他一般属于libxml2-utils这个软件包，因此类似与sudo 的命令就可以安装。

# 二、安装

## APT YUM（Ubuntu/Debian ）

```bash
apt install -y libxml2-utils
yum install -y libxml2-utils
```

## Brew（MacOS）

```bash
brew install xmlstarlet
```


# 三、命令详解

## 1、命令格式

```bash
Usage : xmllint [options] XMLfiles ...
        Parse the XML files and output the result of the parsing
        --version : display the version of the XML library used
        --debug : dump a debug tree of the in-memory document
        --shell : run a navigating shell
        --debugent : debug the entities defined in the document
        --copy : used to test the internal copy implementation
        --recover : output what was parsable on broken XML documents
        --huge : remove any internal arbitrary parser limits
        --noent : substitute entity references by their value
        --noenc : ignore any encoding specified inside the document
        --noout : don't output the result tree
        --path 'paths': provide a set of paths for resources
        --load-trace : print trace of all external entities loaded
        --nonet : refuse to fetch DTDs or entities over network
        --nocompact : do not generate compact text nodes
        --htmlout : output results as HTML
        --nowrap : do not put HTML doc wrapper
        --valid : validate the document in addition to std well-formed check
        --postvalid : do a posteriori validation, i.e after parsing
        --dtdvalid URL : do a posteriori validation against a given DTD
        --dtdvalidfpi FPI : same but name the DTD with a Public Identifier
        --timing : print some timings
        --output file or -o file: save to a given file
        --repeat : repeat 100 times, for timing or profiling
        --insert : ad-hoc test for valid insertions
        --compress : turn on gzip compression of output
        --html : use the HTML parser
        --xmlout : force to use the XML serializer when using --html
        --nodefdtd : do not default HTML doctype
        --push : use the push mode of the parser
        --pushsmall : use the push mode of the parser using tiny increments
        --memory : parse from memory
        --maxmem nbbytes : limits memory allocation to nbbytes bytes
        --nowarning : do not emit warnings from parser/validator
        --noblanks : drop (ignorable?) blanks spaces
        --nocdata : replace cdata section with text nodes
        --format : reformat/reindent the output
        --encode encoding : output in the given encoding
        --dropdtd : remove the DOCTYPE of the input docs
        --pretty STYLE : pretty-print in a particular style
                         0 Do not pretty print
                         1 Format the XML content, as --format
                         2 Add whitespace inside tags, preserving content
        --c14n : save in W3C canonical format v1.0 (with comments)
        --c14n11 : save in W3C canonical format v1.1 (with comments)
        --exc-c14n : save in W3C exclusive canonical format (with comments)
        --nsclean : remove redundant namespace declarations
        --testIO : test user I/O support
        --catalogs : use SGML catalogs from $SGML_CATALOG_FILES
                     otherwise XML Catalogs starting from 
                 file:///etc/xml/catalog are activated by default
        --nocatalogs: deactivate all catalogs
        --auto : generate a small doc on the fly
        --xinclude : do XInclude processing
        --noxincludenode : same but do not generate XInclude nodes
        --nofixup-base-uris : do not fixup xml:base uris
        --loaddtd : fetch external DTD
        --dtdattr : loaddtd + populate the tree with inherited attributes 
        --stream : use the streaming interface to process very large files
        --walker : create a reader and walk though the resulting doc
        --pattern pattern_value : test the pattern support
        --chkregister : verify the node registration code
        --relaxng schema : do RelaxNG validation against the schema
        --schema schema : do validation against the WXS schema
        --schematron schema : do validation against a schematron
        --sax1: use the old SAX1 interfaces for processing
        --sax: do not build a tree but work just at the SAX level
        --oldxml10: use XML-1.0 parsing rules before the 5th edition
        --xpath expr: evaluate the XPath expression, imply --noout
```

## 2、从标准输入输出管道传递处理XML内容

```bash
# 从标准输出管道传递
cat settings.xml | xmllint --foramt - 

# 从标准输入管道传递
xmllint --html --xpath '/html/body/h1[1]' - <<EOF
<BODY>
<H1>Dublin</H1>
EOF
```

## 3、输出控制

```bash
# 压缩输出
xmllint --noblanks settings.xml > settings-noblank.xml

# 格式化输出
xmllint --format settings.xml > settings-format.xml

```

## 4、语法schema校验

```bash
xmllint --schema settings-1.0.0.xsd settings.xml

# xsd文件可以在XML的命名空间中找到下载链接
# 语法校验成功后会输出"validates"，不成功会输出"fails to validate"
# 如果要求只输出校验结果不显示XML内容，可加--noout参数

```

## 5、带命名空间的XML查询

例如Maven的settings.xml或pom.xml文件都自带了命名空间。例如：

> <?xml version="1.0" encoding="UTF-8"?>
> <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
>          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">


```bash
xmllint --xpath "/*[local-name()='settings']" settings.xml
```

# 四、XML处理

以Maven默认的settings.xml为例

## 1、查询

```bash
xmllint --xpath '//mirror/url' settings.xml

```


## 2、



# 参考

1. https://www.tutorialspoint.com/unix_commands/xmllint.htm
2. https://linux.die.net/man/1/xmllint
3. https://blog.csdn.net/qmhball/article/details/8955588
4. https://blog.mythsman.com/post/5d2b5ebf25601931a5f8d885/
5. https://softwaretester.info/test-xml-command-line-with-xmllint/
