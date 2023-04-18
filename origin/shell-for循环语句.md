# 第一类：数字性循环

```bash
#!/bin/bash
for((i=1;i<=10;i++));
do
    echo $(expr $i \* 3 + 1);
done
```

```bash
#!/bin/bash
for i in $(seq 1 10)
do
    echo $(expr $i \* 3 + 1);
done
```

```bash
#!/bin/bash
for i in {1..10}
do
    echo $(expr $i \* 3 + 1);
done
```

```bash
#!/bin/bash
awk 'BEGIN{for(i=1; i<=10; i++) print i}'
```

# 第二类：字符性循环

```bash
#!/bin/bash
for i in `ls`;
do
    echo $i is file name\! ;
done
```

```bash
#!/bin/bash
for i in $* ;
do
    echo $i is input chart\! ;
done
```

```bash
#!/bin/bash
for i in f1 f2 f3 ;
do
    echo $i is appoint ;
done
```

```bash
#!/bin/bash
list="rootfs usr data data2"

for i in $list;
do
    echo $i is appoint ;
done
```

# 第三类：路径查找

```bash
#!/bin/bash
for file in /proc/*;
do
    echo $file is file path \! ;
done
```

```bash
#!/bin/bash
for file in $(ls *.sh)
do
    echo $file is file path \! ;
done
```
