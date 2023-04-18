# Helm Charts编写规则



# 一、Charts的文件目录结构

## Prerequisite：初始化一个模板charts

```bash
helm create charts名
```

以创建一个test charts为例进行说明

```bash
helm create test

# 会在当前路径下产生以下目录结构的文件

test/
  Chart.yaml          # Charts的描述信息。例如作者信息，使用的k8s api版本等
  LICENSE             # Charts的许可证等信息(可选) 
  README.md           # Charts的README(可选) 
  requirements.yaml   # Charts的依赖管理文件(可选) 
  values.yaml         # charts的默认配置项
  charts/             # 存放Charts所依赖的其他charts
  templates/          # 存放Charts的k8s资源声明文件模板
      ｜-- NOTES.txt  # 简短使用说明文件(可选) 
```



# 二、Chart.yaml

`Chart.yaml` 文件是编写Charts 所必需的。描述Chart的描述信息

```bash
apiVersion: The chart API version (必须)
name: Chart名 (必须)
version: Chart的版本信息 (必须)
kubeVersion: A SemVer range of compatible Kubernetes versions (可选)
description: 一句描述chart的话 (可选)
type: chart类型 (可选)
keywords:
  - chart关健字 (可选)
home: The URL of this project's home page (可选)
sources:
  - chart的源代码仓库URL(可选)
dependencies: # 依赖的chart (可选)
  - name: 依赖的chart名
    version: 依赖的chart版本
    repository: The repository URL ("https://example.com/charts") or alias ("@repo-name")
    condition: (可选) A yaml path that resolves to a boolean, used for enabling/disabling charts (e.g. subchart1.enabled )
    tags: # (可选)
      - Tags can be used to group charts for enabling/disabling together
    enabled: (可选) Enabled bool determines if chart should be loaded
    import-values: # (可选)
      - ImportValues holds the mapping of source values to parent key to be imported. Each item can be a string or pair of child/parent sublist items.
    alias: (可选) Alias usable alias to be used for the chart. Useful when you have to add the same chart multiple times
maintainers: # (可选)
  - name: The maintainer's name (必须 for each maintainer)
    email: The maintainer's email (可选 for each maintainer)
    url: A URL for the maintainer (可选 for each maintainer)
icon: A URL to an SVG or PNG image to be used as an icon (可选).
appVersion: The version of the app that this contains (可选). This needn't be SemVer.
```





# 三、







# 参考

1. https://github.com/whmzsu/helm-doc-zh-cn/blob/master/chart/charts-zh_cn.md