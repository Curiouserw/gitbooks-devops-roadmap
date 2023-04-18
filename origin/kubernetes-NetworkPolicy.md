# Kubernetes Network Policy容器网络流量限制

# 一、Overview

Kubernetes使用命名空间namesapce做多租户隔离，但是如果不配置网络策略，namespace的隔离也仅仅是作用于在kubernetes编排调度时的隔离，实际上不同namespace下的pod还是可以相互联通的。此时就需要使用Kubernetes提供的networkPolicy,用于隔离不同租户应用的网络流量来减少攻击面。它使用标签选择器模拟传统的分段网络，并通过策略控制它们之间的流量以及来自外部的流量。Kubernetes中的Network Policy只定义了规范，并没有提供实现，而是把实现留给了网络插件

Kubernetes v1.7+版本GA，API版本为http://networking.k8s.io/v1

# 二、使用Calico做Network policy

Network Policy的实现仰赖CNI插件的支持，目前已经支持的cni插件包括：

- Calico
- Kube-router
- Romana
- Weave Net
- trireme

Calico的NetworkPolicy功能支持以下特性：

  - 支持多种endpoint: pods/containers, VMs
  - 支持限制入站和出站的流量访问
  - 规则策略支持:
    - 动作: allow, deny, log, pass
    - 源和目标的匹配标准:
      - 端口: numbered, ports in a range, and Kubernetes named ports
      - 协议: TCP, UDP, ICMP, SCTP, UDPlite, ICMPv6, protocol numbers (1-255)
      - HTTP attributes (if using Istio service mesh)
      - ICMP attributes
      - IP version (IPv4, IPv6)
      - IP or CIDR
      - Endpoint selectors (using label expression to select pods, VMs, host    - interfaces, and/or network sets)
      - Namespace selectors
      - Service account selectors
  - Optional packet handling controls: disable connection tracking, apply before DNAT, apply to forwarded traffic and/or locally terminated traffic

1. Preflight

     - k8s集群版本大于v1.3.0
     - calico-cni网络插件的二进制文件

2. kubelet添加配置Flag

    需要配置kubelet 让pod启动时使用calico网络插件，kubelet可以配置使用calico在启动时配置参数：

    ```yaml
    --network-plugin=cni  
    --cni-conf-dir=/etc/cni/net.d # CNI插件的配置文件目录,该目录下的配置文件内容需要符合CNI规范
    --cni-bin-dir=/opt/cni/bin # CNI插件的可执行文件目录，默认为/opt/cni/bin
    ```

3. API Server添加配置Flag
   
   ```yaml
   --allow-privileged=true # calico-node的POD需要以特权模式运行在各node上
   ```

# 三、策略规则的声明配置

Network Policy策略规则是用来定义命名空间有哪些POD，能被谁访问，能访问谁的。相应地，在Network Policy声明文件中的字段有：

- `spec.podSelector`: 定义该命名空间有哪些POD遵循本networkpolicy约束
- `spec.ingress.from`: 定义受本networkpolicy约束的POD的入站规则
- `spec.egress.to`：定义受本networkpolicy约束的POD的出站规则

spec.ingress.from的选择器有：

1. **`podSelector`**
2. **`namespaceSelector`**

    ```yaml
    ...上文省略...
    ingress:
    - from:
      - namespaceSelector:
          matchLabels:
            user: alice
      - podSelector:
          matchLabels:
            role: client
    ...下文省略...
    ```

3. **`namespaceSelector和podSelector`**（注意YAML语法的区别）

    ```yaml
    ...上文省略...
    ingress:
    - from:
      - namespaceSelector:
          matchLabels:
            user: alice
        podSelector:
          matchLabels:
            role: client
    ...下文省略...
    ```

4. **`ipBlock`**

# 四、示例说明

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  # POD选择器，选择遵循本networkpolicy约束的POD
  podSelector:
    matchLabels:
      role: db
  # 流量访问策略类型
  policyTypes:
  - Ingress
  - Egress
  # 流量访问入站规则
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  #  POD流量访问出站规则
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
```

上述例子，网络流量访问策略规则可解释为：

**"default"命名空间下标签为"role=db"的POD的入站规则**：

1. 允许被"default"命名空间下，所有带标签"role=frontend"的POD访问TCP 6379端口
2. 允许被标签为"project=myproject"的命名空间下所有的POD访问TCP 6379端口
3. 允许被IP地址为172.17.0.0–172.17.0.255或172.17.2.0–172.17.255.255的POD访问TCP 6379端口

**"default"命名空间下标签为"role=db"的POD的出站规则**：

1. 允许访问10.0.0.0/24网段的POD的5978端口

# 五、默认策略

默认情况下，如果namespace下没有network policy,则该namespace下所有POD的入站规则和出站规则都是开放的。network policy只影响命名空间下被Pod Selector选择的POD，其他依旧是默认规则。

1. namespace下的所有pod，入站规则为全部禁止

   ```bash
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: default-deny
      namespace: test
    spec:
      podSelector: {}
      policyTypes:
      - Ingress
   ```

2. namespace下的所有pod，入站规则为全部开放

    ```bash
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: allow-all
      namespace: test
    spec:
      podSelector: {}
      ingress:
      - {}
      policyTypes:
      - Ingress
    ```

3. namespace下的所有pod，出站规则为全部禁止

   ```bash
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: default-deny
      namespace: test
    spec:
      podSelector: {}
      policyTypes:
      - Egress
   ```

4. namespace下的所有pod，出站规则为全部开放

   ```bash
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: allow-all
      namespace: test
    spec:
      podSelector: {}
      egress:
      - {}
      policyTypes:
      - Egress
   ```

5. 同namespace的pod，入站和出站规则为全部禁止

    ```bash
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: default-deny
    spec:
      podSelector: {}
      policyTypes:
      - Ingress
      - Egress
    ```

# 六、场景测试

- 限制服务只能被带有特定label的应用访问

   ```bash
   $ kubectl create ns test1

   $ kubectl -n test1 run nginx --image=nginx
   $ kubectl -n test1 expose deployment nginx --port=80
  ```

  ```yaml
   kind: NetworkPolicy
   apiVersion: networking.k8s.io/v1
   metadata:
     namespace: test1
     name: access-nginx
   spec:
     podSelector:
       matchLabels:
         run: nginx
     ingress:
     - from:
       - podSelector:
           matchLabels:
             access: "true"
  ```

  ```bash
   # 创建没有Label的临时POD去访问Nginx的SVC
   $ kubectl -n test1 run busybox --rm -ti --image=busybox /bin/sh
    / # wget http://nginx.test1.svc:80/  (无法访问)

   # 创建没有Label的临时POD去访问Nginx的SVC
   $ kubectl -n test1 run busybox --labels="access=true" --rm -ti --image=busybox /bin/sh

    / # wget http://nginx.test1.svc:80/
     Connecting to nginx.test1.svc:80 (10.68.86.216:80)
     saving to 'index.html'
     index.html           100% | *********************************************************************************************************************|   612  0:00:00 ETA
     'index.html' saved
   ```

- 限制带"run=busybox"标签的Pod只能访问www.baidu.com

    ```yaml
    kind: NetworkPolicy
    apiVersion: networking.k8s.io/v1
    metadata:
      name: busybox-egress-baidu-a-policy
      namespace: test1
    spec:
      podSelector:
        matchLabels:
          run: busybox
      egress:
      - to:
        - ipBlock:
            cidr: 180.101.49.11/32
      - to:
        - ipBlock:
            cidr: 0.0.0.0/0
        ports:
        - protocol: UDP
          port: 53
    ```

    ```bash
    $ kubectl -n test1 run busybox --labels="run=busybox" --rm -ti --image=busybox /bin/sh

    / # wget www.baidu.com
    Connecting to www.baidu.com (180.101.49.11:80)
    saving to 'index.html'
    index.html           100% |*********************************************************************************************************************|  2381  0:00:00 ETA
    'index.html' saved
    / # wget www.sohu.com
    Connecting to www.sohu.com (101.227.172.11:80)
    ^C
    / # wget https://github.com
    Connecting to github.com (13.250.177.223:443)
    ^C
    / #
    ```

# 参考链接

1. https://kubernetes.io/docs/concepts/services-networking/network-policies/#sctp-support
2. https://docs.projectcalico.org/v3.8/security/kubernetes-network-policy#best-practice-create-deny-all-default-network-policy
3. https://yq.aliyun.com/articles/640190
4. https://www.jianshu.com/p/c0d2618d2849