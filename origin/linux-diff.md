# Linux 比较文件的不同

# 一、diff比较两个文件的不同

## 语法

```bash
diff [-abBcdefHilnNpPqrstTuvwy][-<行数>][-C <行数>][-D <巨集名称>][-I <字符或字符串>][-S <文件>][-W <宽度>][-x <文件或目录>][-X <文件>][--help][--left-column][--suppress-common-line][文件或目录1][文件或目录2]
```

## 参数

```bash
-<行数> 　指定要显示多少行的文本。此参数必须与-c或-u参数一并使用。
-a或--text 　diff预设只会逐行比较文本文件。
-b或--ignore-space-change 　不检查空格字符的不同。
-B或--ignore-blank-lines 　不检查空白行。
-c 　显示全部内文，并标出不同之处。
-C<行数>或--context<行数> 　与执行"-c-<行数>"指令相同。
-d或--minimal 　使用不同的演算法，以较小的单位来做比较。
-D<巨集名称>或ifdef<巨集名称> 　此参数的输出格式可用于前置处理器巨集。
-e或--ed 　此参数的输出格式可用于ed的script文件。
-f或-forward-ed 　输出的格式类似ed的script文件，但按照原来文件的顺序来显示不同处。
-H或--speed-large-files 　比较大文件时，可加快速度。
-l<字符或字符串>或--ignore-matching-lines<字符或字符串> 　若两个文件在某几行有所不同，而这几行同时都包含了选项中指定的字符或字符串，则不显示这两个文件的差异。
-i或--ignore-case 　不检查大小写的不同。
-l或--paginate 　将结果交由pr程序来分页。
-n或--rcs 　将比较结果以RCS的格式来显示。
-N或--new-file 　在比较目录时，若文件A仅出现在某个目录中，预设会显示：
Only in目录：文件A若使用-N参数，则diff会将文件A与一个空白的文件比较。
-p 　若比较的文件为C语言的程序码文件时，显示差异所在的函数名称。
-P或--unidirectional-new-file 　与-N类似，但只有当第二个目录包含了一个第一个目录所没有的文件时，才会将这个文件与空白的文件做比较。
-q或--brief 　仅显示有无差异，不显示详细的信息。
-r或--recursive 　比较子目录中的文件。
-s或--report-identical-files 　若没有发现任何差异，仍然显示信息。
-S<文件>或--starting-file<文件> 　在比较目录时，从指定的文件开始比较。
-t或--expand-tabs 　在输出时，将tab字符展开。
-T或--initial-tab 　在每行前面加上tab字符以便对齐。
-u,-U<列数>或--unified=<列数> 　以合并的方式来显示文件内容的不同。
-v或--version 　显示版本信息。
-w或--ignore-all-space 　忽略全部的空格字符。
-W<宽度>或--width<宽度> 　在使用-y参数时，指定栏宽。
-x<文件名或目录>或--exclude<文件名或目录> 　不比较选项中所指定的文件或目录。
-X<文件>或--exclude-from<文件> 　您可以将文件或目录类型存成文本文件，然后在=<文件>中指定此文本文件。
-y或--side-by-side 　以并列的方式显示文件的异同之处。
--help 　显示帮助。
--left-column 　在使用-y参数时，若两个文件某一行内容相同，则仅在左侧的栏位显示该行内容。
--suppress-common-lines 　在使用-y参数时，仅显示不同之处。
```

## 实例

### 样本数据

```bash
test-a
  |------a
          asd

          sdf
          qaz1 adasd
  |------c
          21312
test-b
  |------a
          asd

          sdf
          
          qaz1 adad
  |------b
 					12312
          23121

          3432432 1231
  |------c
 					21312
```

### 1、比较两个文件下所有文件(包括子文件)的不同

```bash
# diff -yr --suppress-common-lines test-a test-b
  diff -yr --suppress-common-lines test-a/a test-b/a
  qaz1 adasd						      |	qaz1 adad
  Only in test-b: b
  Files test-a/c and test-b/c are identical
  

# test-a/a文件与test-b/a文件存在差异
# 表示b文件只在test-b文件夹中存在
# test-a/c文件与test-b/c文件完全一致

```

### 2、比较两个文件下除指定指定文件外其他文件(包括子文件)的不同

```bash
# diff -yr --suppress-common-lines -x .git -x .idea test-a test-b
```



