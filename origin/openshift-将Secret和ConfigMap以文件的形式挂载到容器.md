# 将Secret和ConfigMap以文件的形式挂载到容器

# 一、Context

ConfigMap或者Secret在默认挂载到容器是以Volumes的形式，如果挂载路径下原有的其他文件，则会覆盖掉。
如果将挂载路径直接写成文件的绝对路径，这会在挂载路径下创建以文件名为名字的文件夹，文件会在这个文件夹下

```yaml
containers:
  - image: 'busybox:latest'
    name: test
    volumeMounts:
      - mountPath: /etc/test/test.txt
        name: test-volume
volumes:
  - name: test-volume
    secret:
      defaultMode: 420
      secretName: test-secret
```

# 二、操作

挂载Secret或Config类型的volume时，添加一个subPath字段即可，可将其以文件的形式挂载，而不是以目录的形式。

## Secret

**1. 创建secret**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name:test-secret
type: Opaque
data:
 test.txt: >-
    ************************

```

**2. 容器中挂载secret**

```yaml
containers:
  - image: 'busybox:latest'
    name: test
    volumeMounts:
      - mountPath: /etc/test/test.txt
        name: test-volume
        readOnly: true
        subPath: test.txt
# ....
volumes:
  - name: test-volume
    secret:
      defaultMode: 420
      secretName: test-secret
```

secret中的test.txt文件将会单个文件的形式挂载到/etc/test/目录下

## Configmap

**1. 创建ConfigMap**

```bash
oc create configmap crack-jar --from-file=atlassian-extras-3.2.jar --from-literal=text=atlassian-extras-3.2.jar
```

**2. 容器中挂载ConfigMap**

```yaml
# ....
volumeMounts:
- mountPath: /opt/atlassian/jira/atlassian-jira/WEB-INF/lib/atlassian-extras-3.2.jar
  name: crack-jar
  readOnly: true
  subPath: atlassian-extras-3.2.jar
# ....
volumes:
- configMap:
    defaultMode: 420
    name: crack-jar
  name: crack-jar
```

