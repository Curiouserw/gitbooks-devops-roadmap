# CMD与ENTRYPOINT区别

- CMD命令设置容器启动后默认执行的命令及其参数，但CMD设置的命令能够被docker run命令后面的命令行参数替换

- ENTRYPOINT配置容器启动时的执行命令（不会被忽略，一定会被执行，即使运行 docker run时指定了其他命令）

- ENTRYPOINT 的 Exec 格式用于设置容器启动时要执行的命令及其参数，同时可通过CMD命令或者命令行参数提供额外的参数

- ENTRYPOINT 中的参数始终会被使用，这是与CMD命令不同的一点

  

## 1. Shell格式和Exec格式命令

- **Shell格式**：`指令 <command>`

  ```bash
  CMD java -jar test.jar
  ```

- **Exec格式**：`指令 ["executable", "param1", "param2", ...]`

  ```bash
  ENTRYPOINT  ["java", "-jar", "test.jar"]
  ```

## 2. Shell格式和Exec格式命令的区别

- Shell格式中的命令会直接被Shell解析
- Exec格式不会直接解析，需要加参数

## 3. CMD和ENTRYPOINT指令支持的命令格式

**CMD** 指令的命令支持以下三种格式:

- **Exec格式**:   CMD ["executable","param1","param2"]
- **Exec参数**: CMD ["param1","param2"]   用来为ENTRYPOINT 提供参数
- **Shell格式**:   CMD command param1 param2

**ENTRYPOINT** 指令的命令支持以下了两种格式:

- **Exec格式**：可用使用CMD的参数和可使用`docker run [image] 参数 `后面追加的参数
- **Shell格式** ：不会使用 CMD参数，可使用`docker run [image] 参数 `后面追加的参数

## 4. 示例

**`ENTRYPOINT的Exec格式`**

```bash
# Dockerfile
FROM centos
ENTRYPOINT ["/bin/echo", "Hello"]

# 启动容器的命令: docker run -it [image]
# 输出: Hello
# 启动容器的命令: docker run -it [image] Test
# 输出: Hello Test
```

**`ENTRYPOINT的Exec格式`** + **`CMD的Exec格式`**

```bash
# Dockerfile
FROM centos
ENTRYPOINT ["/bin/echo", "Hello"]
CMD ["Word"]

# 启动容器的命令: docker run -it [image]
# 输出: Hello Word
# 启动容器的命令: docker run -it [image] Test
# 输出: Hello Test
```

**`ENTRYPOINT的Exec格式`** + **`CMD的shell格式`**

```bash
# Dockerfile
FROM centos
ENTRYPOINT ["/bin/echo", "Hello"]
CMD Word

# 启动容器的命令: docker run -it [image]
# 输出: Hello /bin/sh -c Word
# 启动容器的命令: docker run -it [image] Test
# 输出: Hello Test
```

**`ENTRYPOINT的shell格式`**

```bash
# Dockerfile
FROM centos
ENTRYPOINT /bin/echo "Hello"

# 启动容器的命令: docker run -it [image]
# 输出: Hello
# 启动容器的命令: docker run -it [image] Test
# 输出: Hello
```

**`ENTRYPOINT的shell格式`** + **`CMD的Shell格式`**

```bash
# Dockerfile
FROM centos
ENTRYPOINT /bin/echo "Hello"
CMD Word

# 启动容器的命令: docker run -it [image]
# 输出: Hello
# 启动容器的命令: docker run -it [image] Test
# 输出: Hello
```

**`ENTRYPOINT的shell格式`** +**`CMD的Exec格式`**

```bash
# Dockerfile
FROM centos
ENTRYPOINT /bin/echo "Hello"
CMD ["Word"]

# 启动容器的命令: docker run -it [image]
# 输出: Hello
# 启动容器的命令: docker run -it [image] Test
# 输出: Hello
```

# 参考链接
https://blog.csdn.net/weixin_42971363/article/details/91506844