## 样本数据

```
a="test"
b="curiouser"
c="test hahah devops"
```

## 1. 通过grep来判断

    if `echo $c |grep -q $a` ;then
        echo "$c" " ----包含--- " "$a"
    else
        echo "$c" " ----不包含--- " "$a"
    fi

## 2. 字符串运算符

    if [[ $c =~ $a ]] ;then
        echo "$c" " ----包含--- " "$a"
    else
        echo "$c" " ----不包含--- " "$a"
    fi

## 3. 用通配符*号代替str1中非str2的部分，如果结果相等说明包含，反之不包含

    if [[ $c == *$a* ]] ;then
        echo "$c" " ----包含--- " "$a"
    else
        echo "$c" " ----不包含--- " "$a"
    fi

## 4. 利用替换

    if [[ ${c/$a//} == $c ]] ;then
        echo "$c" " ----不包含--- " "$a"
    else
        echo "$c" " ----包含--- " "$a"
    fi
