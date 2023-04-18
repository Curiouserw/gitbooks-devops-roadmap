# Docker常见操作

# 1、容器访问宿主机

- 通过域名`host.docker.internal` 或者`docker.for.mac.host.internal（MacOS版下多出来的DNS域名）`
  - Docker Daemon不要设置非默认DNS，不然无法使用上述域名
- 通过docker0网络的默认网关地址：例如分配容器网络子网是`172.17.0.0/16`，那网关地址为`172.17.0.2`
  - 在默认的bridge模式下，docker0网络的默认网关即是宿主机
  - 因为MacOS的Docker Desktop底层使用的虚拟机，所有Docker0网卡无法直接看到

参考：https://cxybb.com/article/qq_38403662/102555888

# 2、格式化输出容器相关信息

## ① 格式化输出镜像大小

```bash
echo -e "大小\t镜像\n" && docker images --format '{{.Size}}\t{{.Repository}}:{{.Tag}}' | sed 's/ //' | sort -h
```

## ②列出docker各网络模式下容器IP地址等信息

```bash
docker network inspect -f '{{println}}{{.Driver}}网络 {{range .IPAM.Config}}{{printf "(网段: %s 网关: %s)" .Subnet .Gateway}}{{end}}{{println}} {{range   .Containers}}{{printf "%s" .Name}}{{printf "\t"}}{{printf "IP地址: %s" .IPv4Address}}  {{printf "MAC地址: %s" .MacAddress}} {{println}} {{end}}' $(docker network ls -q)
```

# 3、docker run覆盖掉默认ENTREYPOINT

```bash
docker run --rm --name test -it --entrypoint bash nginx
```

# 4、Dockerfile中指定shell环境

```bash
SHELL ["/bin/bash", "-c"]
RUN pwd
```

