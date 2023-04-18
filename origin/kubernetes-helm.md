# Helm简介、安装、配置、使用

# 一、简介

Helm 是 Kubernetes 的包管理器。包管理器类似于我们在 Ubuntu 中使用的apt、Centos中使用的yum 或者Python中的 pip 一样，能快速查找、下载和安装软件包。Helm 由客户端组件 helm 和服务端组件 Tiller 组成, 能够将一组K8S资源打包统一管理, 是查找、共享和使用为Kubernetes构建的软件的最佳方式。

Helm3之前是C/S架构的。主要分为客户端 `helm` 和服务端 `Tiller`。`Tiller`负责对charts的解析生成k8s资源声明文件，然后调用k8s api进行部署。同时还保存chart部署的版本信息。

Helm3移除了 `Tiller`，直接在客户端就对charts进行解析，调用k8s api部署资源声明文件。同时将charts release的版本信息保存至对应k8s应用部署所在命名空间下的secret中。(例如：名为sh.helm.release.v1.sentry-kubernetes-events.v1 helm.sh/release.v1类型的secret)

全面拥抱Helm3

# 二、安装

Github下载地址：https://github.com/helm/helm/releases

## 1、二进制包安装

- 下载二进制文件解压至系统环境路径下即可。

- 命令脚本

  ```bash
  curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
  chmod 700 get_helm.sh
  ./get_helm.sh
  
  # 或者
  curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
  ```

## 2、包管理器安装

- Brew

  ```bash
  brew install helm
  ```

## 3、源码编译安装

```bash
$ cd $GOPATH
$ mkdir -p src/helm.sh
$ cd src/helm.sh
$ git clone https://github.com/helm/helm.git
$ cd helm
$ make
```

# 三、配置

helm3默认读取当前用户目录下`～/.kube/config`文件中的当前k8s环境上下文来配置部署charts到哪个k8s集群。相关权限跟随着kuectl配置的用户权限。（开箱即用的感觉）

## 1、配置helm的环境变量

| Name               | Description                                                  |
| ------------------ | ------------------------------------------------------------ |
| `$XDG_CACHE_HOME`  | set an alternative location for storing cached files.        |
| `$XDG_CONFIG_HOME` | set an alternative location for storing Helm configuration.  |
| `$XDG_DATA_HOME`   | set an alternative location for storing Helm data.           |
| `$HELM_DRIVER`     | set the backend storage driver. Values are: configmap, secret, memory |
| `$HELM_NO_PLUGINS` | disable plugins. Set HELM_NO_PLUGINS=1 to disable plugins.   |
| `$KUBECONFIG`      | set an alternative Kubernetes configuration file (default "~/.kube/config") |

## 2、Helm相关文件存储的默认路径

- cached文件都存在` $XDG_CACHE_HOME/helm`
- 配置文件存在 `$XDG_CONFIG_HOME/helm`
- 数据文件存在` $XDG_DATA_HOME/helm`

## 3、各个操作操作系统的默认配置

| 操作系统 | Cache文件路径              | 配置文件路径             |数据文件路径            |
|------------------|---------------------------|--------------------------------|-------------------------|
| Linux            | $HOME/.cache/helm         | $HOME/.config/helm             | $HOME/.local/share/helm |
| macOS            | $HOME/Library/Caches/helm | $HOME/Library/Preferences/helm | $HOME/Library/helm      |
| Windows          | %TEMP%\helm               | %APPDATA%\helm                 | %APPDATA%\helm          |

## 4、命令行的命令补全

```bash
helm completion zsh
source <(helm completion zsh)
```

# 四、charts的管理

## 全局通用的命令行参数

```bash
--add-dir-header                   添加文件路径到Header中
--alsologtostderr                  log to standard error as well as files
--debug                            输出Debug级别的日志
--kube-context string              指定使用哪个kubeconfig context
--kubeconfig string                指定kubeconfig文件路径
--log-backtrace-at traceLocation   when logging hits line file:N, emit a stack trace (default :0)
--log-dir string                   指定日志输出到哪个路径下
--log-file string                  指定日志输出到哪个文件中
--log-file-max-size uint           Defines the maximum size a log file can grow to. Unit is megabytes. If the value is 0, the maximum file size is unlimited. (default 1800)
--logtostderr                      log to standard error instead of files (default true)
-n, --namespace string             指定在哪个K8S命名空间下进行操作
--registry-config string           path to the registry config file (default "/Users/curiouser/Library/Preferences/helm/registry.json")
--repository-cache string          path to the file containing cached repository indexes (default "/Users/curiouser/Library/Caches/helm/repository")
--repository-config string         path to the file containing repository names and URLs (default "/Users/curiouser/Library/Preferences/helm/repositories.yaml")
--skip-headers                     If true, avoid header prefixes in the log messages
--skip-log-headers                 If true, avoid headers when opening log files
--stderrthreshold severity         logs at or above this threshold go to stderr (default 2)
-v, --v Level                          number for the log level verbosity
--vmodule moduleSpec               comma-separated list of pattern=N settings for file-filtered logging
```



## 1、远程Charts仓库的管理

### 添加远程charts仓库

```bash
helm repo add 远程仓库别名 https://kubernetes-charts-incubator.storage.googleapis.com/
```

### 查看当前所有的远程charts仓库

```bash
helm repo list
```

### 删除指定的远程charts仓库

```bash
helm repo rm/remove 远程仓库别名
```

### 查看远程仓库中的所有charts

```bash
helm search repo
```

### 查看Github中的所有charts

```bash
helm search hub
```



## 2、Charts的管理

### 从远程仓库中下载Charts到本地

```bash
helm pull 远程仓库别名/chart名 参数项

# 参数项
--ca-file string       verify certificates of HTTPS-enabled servers using this CA bundle
--cert-file string     identify HTTPS client using this SSL certificate file
-d/--destination string   location to write the chart. If this and tardir are specified, tardir is appended to this (default ".")
--devel                use development versions, too. Equivalent to version '>0.0.0-0'. If --version is set, this is ignored.
-h/--help                 help for pull
--key-file string      identify HTTPS client using this SSL key file
--keyring string       location of public keys used for verification (default "/Users/curiouser/.gnupg/pubring.gpg")
--password string      chart repository password where to locate the requested chart
--prov                 fetch the provenance file, but don't perform verification
--repo string          chart repository url where to locate the requested chart
--untar                下载后解压
--untardir string      下载后解压到指定目录(默认是当前路径".")
--username string      chart repository username where to locate the requested chart
--verify               verify the package before installing it
--version string       specify the exact chart version to install. If this is not specified, the latest version is installed

# 支持全局通用参数
```



# 五、部署Charts到k8s集群

## 命令格式

```bash
helm install [NAME] [CHART] [参数项]

# 参数项
--atomic                       原子部署。当charts部署失败时，所有操作进行回滚删除。同时如果设置该参数，																一并的"--wait"也会被设置
--ca-file string               verify certificates of HTTPS-enabled servers using this CA bundle
--cert-file string             identify HTTPS client using this SSL certificate file
--dependency-update            在部署前更新charts依赖
--description string           添加自定义描述
--devel                        use development versions, too. Equivalent to version '>0.0.0-0'. If --version is set, this is ignored
--disable-openapi-validation   if set, the installation process will not validate rendered templates against the Kubernetes OpenAPI Schema
--dry-run                      模拟部署
-g, --generate-name                generate the name (and omit the NAME parameter)
-h, --help                     显示帮助信息
--key-file string              identify HTTPS client using this SSL key file
--keyring string               location of public keys used for verification (default "/Users/curiouser/.gnupg/pubring.gpg")
--name-template string         specify template used to name the release
--no-hooks                     prevent hooks from running during install
-o, --output format            指定日志输出的格式（可选项table, json, yaml 默认是table)
--password string              远程chart仓库用户的密码
--post-renderer postrenderer   the path to an executable to be used for post rendering. If it exists in $PATH, the binary will be used, otherwise it will try to look for the executable at the given path (default exec)
--render-subchart-notes        if set, render subchart notes along with the parent
--replace                      re-use the given name, only if that name is a deleted release which remains in the history. This is unsafe in production
--repo string                  设置远程chart仓库的url
--set stringArray              设置vaules。(覆盖values.yaml中的值可设置多个，以“,”分割。例如			
															 key1=val1,key2=val2)
--set-file stringArray         从文件中读取va luset values from respective files specified via the command line (can specify multiple or separate values with commas: key1=path1,key2=path2)
--set-string stringArray       set STRING values on the command line (can specify multiple or separate values with commas: key1=val1,key2=val2)
--skip-crds                    if set, no CRDs will be installed. By default, CRDs are installed if not already present
--timeout duration             time to wait for any individual Kubernetes operation (like Jobs for hooks) (默认5分0秒)
--username string              远程chart仓库的用户名
  -f, --values strings         指定values文件或URL(可设置多个)
--verify                       verify the package before installing it
--version string               specify the exact chart version to install. If this is not specified, the latest version is installed
--wait                         设置等待charts涉及的k8s资源变为ready状态的时间才认为部署成功。它的值等																同timeout设置的值例如Pods, PVCs, Services, Deployment的最少POD数, 																StatefulSet, or ReplicaSet ）
# 支持全局通用参数
```

## 1、部署远程仓库中的charts到k8s集群

```bash
 helm install 部署名 远程仓库别名/chart名 参数项
```

## 2、部署本地的Charts到k8s集群

```bash
helm install 部署名 -f values.yaml . 
```

## 3、更新charts的部署

```bash
helm upgrade charts的部署名 -f values.yaml .

# 参数项
--atomic                       原子更新。当charts更新部署失败时，所有操作进行回滚删除。同时如果设置该参
                               数，一并的"--wait"也会被设置
--ca-file string               verify certificates of HTTPS-enabled servers using this CA bundle
--cert-file string             identify HTTPS client using this SSL certificate file
--cleanup-on-fail              allow deletion of new resources created in this upgrade when upgrade fails
--description string           添加自定义描述
--devel                        use development versions, too. Equivalent to version '>0.0.0-0'. If --version is set, this is ignored
--dry-run                      模拟更新部署
--force                        force resource updates through a replacement strategy
-h, --help                     显示帮助信息
--history-max int              limit the maximum number of revisions saved per release. Use 0 for no limit (default 10)
-i, --install                  如果指定的chart部署名不存在，就直接安装
--key-file string              identify HTTPS client using this SSL key file
--keyring string               指定验证时公钥的路径(默认当前用户路径下的.gnupg/pubring.gpg")
--no-hooks                     disable pre/post upgrade hooks
-o, --output format            指定日志输出的格式（可选项table, json, yaml 默认是table)
--password string              远程chart仓库用户的密码
--post-renderer postrenderer   the path to an executable to be used for post rendering. If it exists in $PATH, the binary will be used, otherwise it will try to look for the executable at the given path (default exec)
--render-subchart-notes        if set, render subchart notes along with the parent
--repo string                  设置远程chart仓库的url
--reset-values                 when upgrading, reset the values to the ones built into the chart
--reuse-values                 when upgrading, reuse the last release's values and merge in any overrides from the command line via --set and -f. If '--reset-values' is specified, this is ignored
--set stringArray              设置vaules。(覆盖values.yaml中的值可设置多个，以“,”分割。例如			
															 key1=val1,key2=val2)
--set-file stringArray         set values from respective files specified via the command line (can specify multiple or separate values with commas: key1=path1,key2=path2)
--set-string stringArray       set STRING values on the command line (can specify multiple or separate values with commas: key1=val1,key2=val2)
--timeout duration             time to wait for any individual Kubernetes operation (like Jobs for hooks) (默认5分0秒)
--username string              远程chart仓库的用户名
-f, --values strings           指定values文件或URL(可设置多个)
--verify                       verify the package before installing it
--version string               specify the exact chart version to install. If this is not specified, the latest version is installed
--wait                         设置等待charts涉及的k8s资源变为ready状态的时间才认为部署成功。它的值等																同timeout设置的值例如Pods, PVCs, Services, Deployment的最少POD数, 																StatefulSet, or ReplicaSet ）
# 支持全局通用参数
```

## 4、删除部署charts的资源

默认删除charts涉及的所有资源和charts的发布版本

```bash
helm del/uninstall/del/delete/un charts的部署名 参数项
# 参数项
--description string   添加自定义描述
--dry-run              模拟删除
-h, --help             显示帮助信息
--keep-history         删除charts涉及的所有资源，然后标记该charts的发布为删除状态，但保留删除历史
--no-hooks             prevent hooks from running during uninstallation
--timeout duration     time to wait for any individual Kubernetes operation (like Jobs for hooks) (默认5m0s)
# 支持全局通用参数
```

