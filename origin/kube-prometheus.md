# kube Prometheus：Prometheus Operator

# 一、简介

为了方便大家使用prometheus，`Coreos`出了提供了一个Operator，而为了方便大家一站式的监控方案就有了项目[kube-prometheus](https://github.com/coreos/kube-prometheus)是一个脚本项目，它主要使用`jsonnet`写成，其作用呢就是模板+参数然后渲染出yaml文件集，主要是作用是提供一个开箱即用的监控栈，用于kubernetes集群的监控和应用程序的监控。
这个项目主要包括以下软件栈

- Prometheus Operator
- Highly available Prometheus
- Highly available Alertmanager
- Prometheus node-exporter
- Prometheus Adapter for Kubernetes Metrics APIs
- Kube-state-metrics
- Grafana

说是开箱即用，确实也是我们只需要clone下来，然后`kubectl apply ./manifests`，manifests目录中生成的是预先生成的yaml描述文件，有诸多不方便的地方，比如说

- 镜像仓库的地址都在gcr和query.io，这两个地址国内拉起来都费劲
- 没有持久化存储promethus的数据

# 二、安装

## 1、默认安装

```bash
git clone https://github.com/coreos/kube-prometheus.git -b release-0.5
cd kube-prometheus
kubectl apply -f manifests/setup
kubectl apply -f manifests/
```

之后手动修改Prometheus的k8s资源声明对象

## 2、定制化安装

### ①安装jb、jsonnet、gojsontoyaml

```bash
wget -c https://github.com/jsonnet-bundler/jsonnet-bundler/releases/download/v0.4.0/jb-darwin-amd64 -o /usr/local/bin/jb
chmod +x /usr/local/bin/jb
git clone https://github.com/brancz/gojsontoyaml.git 
cd gojsontoyaml 
go build 
chmod +x gojsontoyaml
mv gojsontoyaml /usr/local/bin/
echo '{"test":"test string with\\nmultiple lines"}' | gojsontoyaml
```

### ②下载依赖

```bash
mkdir kube-prometheus-k8s118; cd kube-prometheus-k8s118
jb init
# 初始化jb，创建依赖描述文件`jsonnetfile.json`
jb install github.com/coreos/kube-prometheus/jsonnet/kube-prometheus@release-0.5
jb update
```

### ③编写编译命令的脚本`build.sh`

```bash
#!/usr/bin/env bash

# This script uses arg $1 (name of *.jsonnet file to use) to generate the manifests/*.yaml files.

set -e
set -x
# only exit with zero if all commands of the pipeline exit successfully
set -o pipefail

# Make sure to use project tooling
PATH="$(pwd)/tmp/bin:${PATH}"

# Make sure to start with a clean 'manifests' dir
rm -rf manifests
mkdir -p manifests/setup

# Calling gojsontoyaml is optional, but we would like to generate yaml, not json
jsonnet -J vendor -m manifests "${1-example.jsonnet}" | xargs -I{} sh -c 'cat {} | gojsontoyaml > {}.yaml' -- {}

# Make sure to remove json files
find manifests -type f ! -name '*.yaml' -delete
rm -f kustomization
```

### ④编写定制化Prometheus的配置文件

在第三部`jb init`产生的`k8s118.json`中进行定制化Prometheus

```bash
local k = import 'ksonnet/ksonnet.beta.3/k.libsonnet';
local ingress = k.extensions.v1beta1.ingress ;
local ingressRule = ingress.mixin.spec.rulesType;
local pvc = k.core.v1.persistentVolumeClaim;
local registry = import 'registry.libsonnet';
local imagepullsecret = k.apps.v1beta2.deployment.mixin.spec.template.spec;
local httpIngressPath = ingressRule.mixin.http.pathsType;
local secret = k.core.v1.secret;
local kp = (import 'kube-prometheus/kube-prometheus.libsonnet') +
  {
    _config+:: {
      namespace: 'monitoring',
      imageRepos+:: {
        prometheus: "192.168.1.60/monitoring/prometheus",
        alertmanager: "192.168.1.60/monitoring/alertmanager",
        kubeStateMetrics: "192.168.1.60/monitoring/kube-state-metrics",
        kubeRbacProxy: "192.168.1.60/monitoring/kube-rbac-proxy",
        nodeExporter: "192.168.1.60/monitoring/node-exporter",
        prometheusOperator: "192.168.1.60/monitoring/prometheus-operator",
        grafana: "192.168.1.60/monitoring/grafana",
        prometheusAdapter: "192.168.1.60/monitoring/k8s-prometheus-adapter-amd64",
        metricsServer: "192.168.1.60/monitoring/metrics-server-amd64:v0.2.0",
      },
      grafana+:: {
        config+: {
          sections+: {
            server+: {
              root_url: 'http://grafana.apps.k8s118.curiouser.com/',
            },
            smtp: {
              enabled: 'true',
              host: 'smtp.163.com:25',
              user: '****',
              password: '****',
              from_address: '****',
              from_name: 'Grafana',
              skip_verify: 'true',
            },
          },
        },
      },
      registry+:: {
        name: "192.168.1.60",
        secret_name: "harbor-secret",
        username: "****",
        password: "****",
        secret:
          local data = {
              'auths': {
                  [$._config.registry.name]: {
                      auth: std.base64($._config.registry.username + ":" + $._config.registry.password),
                  },
              },
          };
          local base = {'.dockerconfigjson': std.base64(std.toString(data))};
          local name = $._config.registry.secret_name;
          secret.mixin.metadata.withNamespace($._config.namespace) +
          secret.new(name=name, data=base, type='kubernetes.io/dockerconfigjson')
      },
    },
    alertmanager+: {
      alertmanager+: {
        spec+: {
          imagePullSecrets: [{name: $._config.registry.secret_name}],
        }
      },
    },
    grafana+: {
       deployment+:
         imagepullsecret.withImagePullSecrets( {name: $._config.registry.secret_name} ),
    },
    kubeStateMetrics+: {
       deployment+:
         imagepullsecret.withImagePullSecrets( {name: $._config.registry.secret_name} ),
    },
    nodeExporter+: {
       daemonset+:
         imagepullsecret.withImagePullSecrets( {name: $._config.registry.secret_name} ),
    },
    prometheusAdapter+: {
       deployment+:
         imagepullsecret.withImagePullSecrets( {name: $._config.registry.secret_name} ),
    },
    prometheusOperator+: {
       deployment+:
         imagepullsecret.withImagePullSecrets( {name: $._config.registry.secret_name} ),
    },
    prometheus+:: {
      prometheus+: {
        spec+: { 
          retention: '7d',
          storage: {  
            volumeClaimTemplate:  
              pvc.new() +  
              pvc.mixin.spec.withAccessModes('ReadWriteOnce') +
              pvc.mixin.spec.resources.withRequests({ storage: '50Gi' }),
          },
        }, 
      },
    },
    ingress+:: {
      'grafana':
        ingress.new() + 
        ingress.mixin.metadata.withName('grafana') +
        ingress.mixin.metadata.withNamespace($._config.namespace) +
        ingress.mixin.spec.withRules(
          ingressRule.new() +
          ingressRule.withHost('grafana.apps.k8s118.curiouser.com') +
          ingressRule.mixin.http.withPaths(
            httpIngressPath.new() +
            httpIngressPath.mixin.backend.withServiceName('grafana') +
            httpIngressPath.mixin.backend.withServicePort('http')
          )
        ),
      'alertmanager-main':
        ingress.new() +
        ingress.mixin.metadata.withName('alertmanager-main') +
        ingress.mixin.metadata.withNamespace($._config.namespace) +
        ingress.mixin.spec.withRules(
          ingressRule.new() +
          ingressRule.withHost('alertmanager.apps.k8s118.curiouser.com') +
          ingressRule.mixin.http.withPaths(
            httpIngressPath.new() +
            httpIngressPath.mixin.backend.withServiceName('alertmanager-main') +
            httpIngressPath.mixin.backend.withServicePort('web')
          )
        ),
      'prometheus-k8s':
        ingress.new() +
        ingress.mixin.metadata.withName('prometheus-k8s') +
        ingress.mixin.metadata.withNamespace($._config.namespace) +
        ingress.mixin.spec.withRules(
          ingressRule.new() +
          ingressRule.withHost('prometheus.apps.k8s118.curiouser.com') +
          ingressRule.mixin.http.withPaths(
            httpIngressPath.new() +
            httpIngressPath.mixin.backend.withServiceName('prometheus-k8s') +
            httpIngressPath.mixin.backend.withServicePort('web')
          )
        ),
    }
  };

{ ['setup/0namespace-' + name]: kp.kubePrometheus[name] for name in std.objectFields(kp.kubePrometheus) } +
// { ['setup/0secret-' + name]: kp.secret[name] for name in std.objectFields(kp.secret)}+
{
  ['setup/prometheus-operator-' + name]: kp.prometheusOperator[name]
  for name in std.filter((function(name) name != 'serviceMonitor'), std.objectFields(kp.prometheusOperator))
} +
// serviceMonitor is separated so that it can be created after the CRDs are ready
{ 'prometheus-operator-serviceMonitor': kp.prometheusOperator.serviceMonitor } +
{ ['node-exporter-' + name]: kp.nodeExporter[name] for name in std.objectFields(kp.nodeExporter) } +
{ ['kube-state-metrics-' + name]: kp.kubeStateMetrics[name] for name in std.objectFields(kp.kubeStateMetrics) } +
{ ['alertmanager-' + name]: kp.alertmanager[name] for name in std.objectFields(kp.alertmanager) } +
{ ['prometheus-' + name]: kp.prometheus[name] for name in std.objectFields(kp.prometheus) } +
{ ['prometheus-adapter-' + name]: kp.prometheusAdapter[name] for name in std.objectFields(kp.prometheusAdapter) } +
{ ['grafana-' + name]: kp.grafana[name] for name in std.objectFields(kp.grafana) } +
{ [name + '-ingress']: kp.ingress[name] for name in std.objectFields(kp.ingress) } 
```

### ⑤开始编译

```bash
./build.sh jsonnetfile.json
```

编译完成后，会在`manifests`目录下生产K8s资源声明文件。

### ⑥产生的K8s资源对象

`manifests/setup/*.yaml`

- **namespace**: monitoring
- **customresourcedefinition.apiextensions.k8s.io**: alertmanagers.monitoring.coreos.com
- **customresourcedefinition.apiextensions.k8s.io**: podmonitors.monitoring.coreos.com
- **customresourcedefinition.apiextensions.k8s.io**: prometheuses.monitoring.coreos.com
- **customresourcedefinition.apiextensions.k8s.io**: prometheusrules.monitoring.coreos.com
- **customresourcedefinition.apiextensions.k8s.io**: servicemonitors.monitoring.coreos.com
- **customresourcedefinition.apiextensions.k8s.io**: thanosrulers.monitoring.coreos.com
- **clusterrole.rbac.authorization.k8s.io**: prometheus-operator
- **clusterrolebinding.rbac.authorization.k8s.io**: prometheus-operator
- **deployment.apps**: prometheus-operator
- **service**: prometheus-operator
- **serviceaccount**: prometheus-operator

`manifests/*.yaml`

- **alertmanager.monitoring.coreos.com**: main
- **secret**: alertmanager-main
- **service**: alertmanager-main
- **serviceaccount**: alertmanager-main
- **servicemonitor.monitoring.coreos.com**: alertmanager
- **secret**: grafana-datasources
- **configmap**: grafana-dashboard-apiserver
- **configmap**: grafana-dashboard-cluster-total
- **configmap**: grafana-dashboard-controller-manager
- **configmap**: grafana-dashboard-k8s-resources-cluster
- **configmap**: grafana-dashboard-k8s-resources-namespace
- **configmap**: grafana-dashboard-k8s-resources-node
- **configmap**: grafana-dashboard-k8s-resources-pod
- **configmap**: grafana-dashboard-k8s-resources-workload
- **configmap**: grafana-dashboard-k8s-resources-workloads-namespace
- **configmap**: grafana-dashboard-kubelet
- **configmap**: grafana-dashboard-namespace-by-pod
- **configmap**: grafana-dashboard-namespace-by-workload
- **configmap**: grafana-dashboard-node-cluster-rsrc-use
- **configmap**: grafana-dashboard-node-rsrc-use
- **configmap**: grafana-dashboard-nodes
- **configmap**: grafana-dashboard-persistentvolumesusage
- **configmap**: grafana-dashboard-pod-total
- **configmap**: grafana-dashboard-prometheus-remote-write
- **configmap**: grafana-dashboard-prometheus
- **configmap**: grafana-dashboard-proxy
- **configmap**: grafana-dashboard-scheduler
- **configmap**: grafana-dashboard-statefulset
- **configmap**: grafana-dashboard-workload-total
- **configmap**: grafana-dashboards
- **deployment.apps**: grafana
- **service**: grafana
- **serviceaccount**: grafana
- **servicemonitor.monitoring.coreos.com**: grafana
- **clusterrole.rbac.authorization.k8s.io**: kube-state-metrics
- **clusterrolebinding.rbac.authorization.k8s.io**: kube-state-metrics
- **deployment.apps**: kube-state-metrics
- **service**: kube-state-metrics
- **serviceaccount**: kube-state-metrics
- **servicemonitor.monitoring.coreos.com**: kube-state-metrics
- **clusterrole.rbac.authorization.k8s.io**: node-exporter
- **clusterrolebinding.rbac.authorization.k8s.io**: node-exporter
- **daemonset.apps**: node-exporter
- **service**: node-exporter
- **serviceaccount**: node-exporter
- **servicemonitor.monitoring.coreos.com**: node-exporter
- **apiservice.apiregistration.k8s.io**: v1beta1.metrics.k8s.io configured
- **clusterrole.rbac.authorization.k8s.io**: prometheus-adapter
- **clusterrole.rbac.authorization.k8s.io**: system:aggregated-metrics-reader
- **clusterrolebinding.rbac.authorization.k8s.io**: prometheus-adapter
- **clusterrolebinding.rbac.authorization.k8s.io**: resource-metrics:system:auth-delegator
- **clusterrole.rbac.authorization.k8s.io**: resource-metrics-server-resources
- **configmap**: adapter-config
- **deployment.apps**: prometheus-adapter
- **rolebinding.rbac.authorization.k8s.io**: resource-metrics-auth-reader
- **service**: prometheus-adapter
- **serviceaccount**: prometheus-adapter
- **clusterrole.rbac.authorization.k8s.io**: prometheus-k8s
- **clusterrolebinding.rbac.authorization.k8s.io**: prometheus-k8s
- **servicemonitor.monitoring.coreos.com**: prometheus-operator
- **prometheus.monitoring.coreos.com**: k8s
- **rolebinding.rbac.authorization.k8s.io**: prometheus-k8s-config
- **rolebinding.rbac.authorization.k8s.io**: prometheus-k8s
- **rolebinding.rbac.authorization.k8s.io**: prometheus-k8s
- **rolebinding.rbac.authorization.k8s.io**: prometheus-k8s
- **role.rbac.authorization.k8s.io**: prometheus-k8s-config
- **role.rbac.authorization.k8s.io**: prometheus-k8s
- **role.rbac.authorization.k8s.io**: prometheus-k8s
- **role.rbac.authorization.k8s.io**: prometheus-k8s
- **prometheusrule.monitoring.coreos.com**: prometheus-k8s-rules
- **service**: prometheus-k8s
- **serviceaccount**: prometheus-k8s
- **servicemonitor.monitoring.coreos.com**: prometheus
- **servicemonitor.monitoring.coreos.com**: kube-apiserver
- **servicemonitor.monitoring.coreos.com**: coredns
- **servicemonitor.monitoring.coreos.com**: kube-controller-manager
- **servicemonitor.monitoring.coreos.com**: kube-scheduler
- **servicemonitor.monitoring.coreos.com**: kubelet

### ⑦预拉取镜像并推送到私有仓库中

`sync-to-internal-registry.jsonnet`

```json
local kp = import 'kube-prometheus/kube-prometheus.libsonnet';
local l = import 'kube-prometheus/lib/lib.libsonnet';
local config = kp._config;

local makeImages(config) = [
  {
    name: config.imageRepos[image],
    tag: config.versions[image],
  }
  for image in std.objectFields(config.imageRepos)
];

local upstreamImage(image) = '%s:%s' % [image.name, image.tag];
local downstreamImage(registry, image) = '%s/%s:%s' % [registry, l.imageName(image.name), image.tag];

local pullPush(image, newRegistry) = [
  'docker pull %s' % upstreamImage(image),
  'docker tag %s %s' % [upstreamImage(image), downstreamImage(newRegistry, image)],
  'docker push %s' % downstreamImage(newRegistry, image),
];

local images = makeImages(config);

local output(repository) = std.flattenArrays([
  pullPush(image, repository)
  for image in images
]);

function(repository='my-registry.com/repository')
  std.join('\n', output(repository))
```

生成拉取镜像，并修改推送镜像的命令

```bash
$ jsonnet -J vendor -S --tla-str repository=192.168.1.60/monitoring sync-to-internal-registry.jsonnet

# 会生成以下命令，复制执行
docker pull quay.io/prometheus/alertmanager:v0.20.0
docker tag quay.io/prometheus/alertmanager:v0.20.0 192.168.1.60/monitoring/alertmanager:v0.20.0
docker push 192.168.1.60/monitoring/alertmanager:v0.20.0
docker pull jimmidyson/configmap-reload:v0.3.0
docker tag jimmidyson/configmap-reload:v0.3.0 192.168.1.60/monitoring/configmap-reload:v0.3.0
docker push 192.168.1.60/monitoring/configmap-reload:v0.3.0
docker pull grafana/grafana:6.6.0
docker tag grafana/grafana:6.6.0 192.168.1.60/monitoring/grafana:6.6.0
docker push 192.168.1.60/monitoring/grafana:6.6.0
docker pull quay.io/coreos/kube-rbac-proxy:v0.4.1
docker tag quay.io/coreos/kube-rbac-proxy:v0.4.1 192.168.1.60/monitoring/kube-rbac-proxy:v0.4.1
docker push 192.168.1.60/monitoring/kube-rbac-proxy:v0.4.1
docker pull quay.io/coreos/kube-state-metrics:1.9.5
docker tag quay.io/coreos/kube-state-metrics:1.9.5 192.168.1.60/monitoring/kube-state-metrics:1.9.5
docker push 192.168.1.60/monitoring/kube-state-metrics:1.9.5
docker pull quay.io/prometheus/node-exporter:v0.18.1
docker tag quay.io/prometheus/node-exporter:v0.18.1 192.168.1.60/monitoring/node-exporter:v0.18.1
```

### ⑧部署定制化的Prometheus到k8s集群 

```bash
kubectl apply -f manifests/setup
kubectl apply -f manifests/

# 创建harbor registry用户登录信息的secret
kubectl create secret docker-registry harbor-secret --docker-server=192.168.1.60 --docker-username=k8s  --docker-password=*** --docker-email=*** -n monitoring

# 批量在monitoring命名空间下serviceaccount中添加imagePullSercret
for i in `k get sa |awk '{print $1}'|grep -v NAME` ; do kubectl -n monitoring patch serviceaccount $i -p '{"imagePullSecrets": [{"name": "harbor-secret"}]}' ;done
```

### ⑨(可选)清理Prometheus资源

```bash
kubectl delete --ignore-not-found=true -f manifests/ -f manifests/setup
```



## 3、参考

1. https://qingmu.io/2019/08/30/Customize-your-kube-prometheus/
2. https://github.com/coreos/kube-prometheus#cluster-creation-tools
3. https://github.com/coreos/kube-prometheus#customization-examples
4. https://github.com/coreos/kube-prometheus/blob/master/docs/developing-prometheus-rules-and-grafana-dashboards.md
5. https://github.com/coreos/kube-prometheus/issues/308

# 三、ServiceMonitor服务发现

## 1、监控k8s集群外的Ceph

### ①在cephMonitor节点部署Ceph exporter

```bash
docker run -d --restart=always -v /etc/ceph:/etc/ceph -p=9128:9128  --name=ceph-export digitalocean/ceph_exporter --rgw.mode=1
```

### ②创建对应的Endpoint、Service、ServiceMonitor

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: outcluster-exporter-ceph
  namespace: monitoring
subsets:
- addresses:
  - ip: 192.168.1.60
  ports:
  - name: http
    port: 9128
    protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  # 关键点：为了能被servicemonitor识别，添加对应的标签
  labels:
    prometheus-target: outcluster-exporter-ceph
  name: outcluster-exporter-ceph
  namespace: monitoring
spec:
  type: ClusterIP
  ports:
  - name: http
    port: 9128
    protocol: TCP
    targetPort: http
  # 关键点：由于ceph exporter不在k8s集群中，不写常规service中的selector。
--- 
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  labels:
    k8s-apps: outcluster-exporter-ceph
  name: outcluster-exporter-ceph
  namespace: monitoring
spec:
  endpoints:
  - interval: 15s
    port: http
  namespaceSelector:
    matchNames:
    - monitoring
  selector:
    matchLabels:
      # 指定prometheus去匹配有对应标签的service中监控数据
      prometheus-target: outcluster-exporter-ceph

```

## 2、监控k8s集群外的主机

### ①对应主机部署Node exporter

参考：[二进制部署Prometheus生态组件](../origin/prometheus-binary-docker-deploy.md)

### ②创建对应的Endpoint、Service、ServiceMonitor

```bash
apiVersion: v1
kind: Endpoints
metadata:
  name: outcluster-exporter-node-tools
  namespace: monitoring
  labels:
     prometheus-target: outcluster-exporter-node
subsets:
- addresses:
  - ip: 192.168.1.60
  ports:
  - name: http
    port: 9100
    protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
	# 关键点：为了能被servicemonitor识别，添加对应的标签
  labels:
    prometheus-target: outcluster-exporter-node
  name: outcluster-exporter-node-tools
  namespace: monitoring
spec:
	type: ClusterIP
  ports:
  - name: http
    port: 9100
    protocol: TCP
    targetPort: http
  # 关键点：由于node exporter不在k8s集群中，不写常规service中的selector。
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  labels:
    k8s-apps: outcluster-exporter-node
  name: outcluster-exporter-node
  namespace: monitoring
spec:
  endpoints:
  - interval: 15s
    port: http
  namespaceSelector:
    matchNames:
    - monitoring
  selector:
    matchLabels:
      # 指定prometheus去匹配有对应标签的service中监控数据
      prometheus-target: outcluster-exporter-node
```

