
```bash
if [ command ]; then
    符合该条件执行的语句
fi
```

```bash
if [ command ]; then
    command执行返回状态为0要执行的语句
else
    command执行返回状态为1要执行的语句
fi
```

```bash
if [ command1 ]; then
    command1执行返回状态为0要执行的语句
elif [ command2 ]; then
    command2执行返回状态为0要执行的语句
else
    command1和command2执行返回状态都为1要执行的语句
fi
```

**`PS`**: [ command ]，command前后要有空格