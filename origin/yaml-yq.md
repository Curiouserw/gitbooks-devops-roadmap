# YAML文本处理工具yq

# 一、简介

对于k8s工程师，整天接触最多的是k8s资源对象的声明yaml文件。有的时候需要在脚本里面处理它。yq是一个go写的命令行处理工具。

Github：https://github.com/mikefarah/yqs

相关文档：https://mikefarah.gitbook.io/yq/usage/split-into-multiple-files

# 二、安装

## MacOS

```
brew install yq
```

## Windows

```
choco install yq
```

## Ubuntu and other Linux distros

 **supporting `snap` packages:**

```
snap install yq
```

## Go GET

```
GO111MODULE=on go get github.com/mikefarah/yq/v3
```

## Run with Docker

Oneshot use:

```
docker run --rm -v "${PWD}":/workdir mikefarah/yq yq [flags] <command> FILE...
```

Run commands interactively:

```
docker run --rm -it -v "${PWD}":/workdir mikefarah/yq sh
```

It can be useful to have a bash function to avoid typing the whole docker command:

```
yq() {
  docker run --rm -i -v "${PWD}":/workdir mikefarah/yq yq "$@"
}
```

# 三、使用

## 命令详解

```bash
Usage:
  yq [flags]
  yq [command]

Available Commands:
  compare          yq x [--prettyPrint/-P] dataA.yaml dataB.yaml 'b.e(name==fr*).value'
  delete           yq d [--inplace/-i] [--doc/-d index] sample.yaml 'b.e(name==fred)'
  help             Help about any command
  merge            yq m [--inplace/-i] [--doc/-d index] [--overwrite/-x] [--append/-a] sample.yaml sample2.yaml
  new              yq n [--script/-s script_file] a.b.c newValue
  prefix           yq p [--inplace/-i] [--doc/-d index] sample.yaml a.b.c
  read             yq r [--printMode/-p pv] sample.yaml 'b.e(name==fr*).value'
  shell-completion Generates shell completion scripts
  validate         yq v sample.yaml
  write            yq w [--inplace/-i] [--script/-s script_file] [--doc/-d index] sample.yaml 'b.e(name==fr*).value' newValue

Flags:
  -C, --colors        print with colors
  -h, --help          help for yq
  -I, --indent int    sets indent level for output (default 2)
  -P, --prettyPrint   pretty print
  -j, --tojson        output as json. By default it prints a json document in one line, use the prettyPrint flag to print a formatted doc.
  -v, --verbose       verbose mode
  -V, --version       Print version information and quit

Use "yq [command] --help" for more information about a command.
```

## 原始yaml

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  creationTimestamp: "2019-06-02T09:30:01Z"
  finalizers:
  - kubernetes.io/pvc-protection
  name: test
  namespace: app-test
  resourceVersion: "54534390"
  selfLink: /api/v1/namespaces/test/persistentvolumeclaims/test
  uid: c35e6108-46c4-4f71-809d
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: nfs-client
  volumeMode: Filesystem
  volumeName: pvc-c35e6108-46c4-4f71-809d
status:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 10Gi
  phase: Bound
```

### 示例1

```bash
cat test.yaml | yq r - status

accessModes:
- ReadWriteOnce
capacity:
  storage: 10Gi
phase: Bound
```

### 示例2

```bash
cat test.yaml | yq r - spec.resources

requests:
  storage: 10Gi
```

### 示例3

```bash
cat test.yaml | yq r - metadata | grep -E 'name:|namespace:'

name: test
namespace: app-test
```

## 原始yaml

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: test
  namespace: test
type: Opaque
data:
  prometheus-additional.yaml: dGVzdAo=
```

### 示例1

```bash
 cat a.yaml | yq r - data.'"prometheus-additional.yaml"' | base64 --decode
```



# 参考

1. https://github.com/mikefarah/yq
2. 