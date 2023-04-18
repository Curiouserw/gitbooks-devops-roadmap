# 三、OKD上部署

## DeploymentConfig

```yaml
apiVersion: apps.openshift.io/v1
kind: DeploymentConfig
metadata:
  labels:
    app: kibana
  name: kibana
spec:
  replicas: 1
  selector:
    app: kibana
    deploymentconfig: kibana
  strategy:
    activeDeadlineSeconds: 21600
    resources: {}
    rollingParams:
      intervalSeconds: 1
      maxSurge: 25%
      maxUnavailable: 25%
      timeoutSeconds: 600
      updatePeriodSeconds: 1
    type: Rolling
  template:
    metadata:
      labels:
        app: kibana
        deploymentconfig: kibana
    spec:
      containers:
      - env:
        - name: ELASTICSEARCH_USERNAME
          value: kibana
        - name: ELASTICSEARCH_PASSWORD
          value: uLAWAfW1b7UHZdHEigCW
        - name: TZ
          value: Asia/Shanghai
        image: docker.elastic.co/kibana/kibana:7.1.1
        imagePullPolicy: IfNotPresent
        livenessProbe:
          failureThreshold: 3
          initialDelaySeconds: 60
          periodSeconds: 10
          successThreshold: 1
          tcpSocket:
            port: 5601
          timeoutSeconds: 1
        name: kibana
        ports:
        - containerPort: 5601
          protocol: TCP
        readinessProbe:
          failureThreshold: 3
          initialDelaySeconds: 60
          periodSeconds: 10
          successThreshold: 1
          tcpSocket:
            port: 5601
          timeoutSeconds: 1
        resources:
          limits:
            cpu: "1"
            memory: 1500Mi
          requests:
            cpu: 500m
            memory: 800Mi
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
  test: false
  triggers:
  - type: ConfigChange
```

## SVC

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: kibana
  name: kibana
spec:
  ports:
  - name: 5601-tcp
    port: 5601
    protocol: TCP
    targetPort: 5601
  selector:
    deploymentconfig: kibana
  sessionAffinity: None
  type: ClusterIP
```

## Route

```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  labels:
    app: apache-kibana
  name: kibana
spec:
  port:
    targetPort: 5601-tcp
  to:
    kind: Service
    name: kibana
    weight: 100
  wildcardPolicy: None
```

# 四. Kubernetes上部署

## Deployment

```yaml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  labels:
    app: kibana
  name: kibana
  namespace: elk
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: kibana
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: kibana
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/hostname
                operator: In
                values:
                -  node1.k8s.curiouser.com
      containers:
      - image: kibana/kibana:7.2.0
        imagePullPolicy: IfNotPresent
        name: kibana
        envFrom:
        - secretRef:
            name: kibana-config-env
        env:
          - name: TZ
            value: Asia/Shanghai
          - name: ELASTICSEARCH_HOSTS
            value: '["http://elasticsearch.elk.svc:9200"]'
        ports:
        - containerPort: 5601
          name: web
          protocol: TCP
        resources:
          requests:
            memory: "1Gi"
            cpu: "0.5"
          limits:
            memory: "2Gi"
            cpu: "1"
        readinessProbe:
          failureThreshold: 1
          initialDelaySeconds: 60
          periodSeconds: 60
          successThreshold: 1
          tcpSocket:
            port: 5601
          timeoutSeconds: 1
        livenessProbe:
          failureThreshold: 1
          initialDelaySeconds: 60
          periodSeconds: 60
          successThreshold: 1
          tcpSocket:
            port: 5601
          timeoutSeconds: 1
        securityContext:
          allowPrivilegeEscalation: false
          capabilities: {}
          privileged: false
          procMount: Default
          readOnlyRootFilesystem: false
          runAsNonRoot: false
        stdin: true
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        tty: true
      dnsConfig: {}
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
```

## Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  labels:
    app: kibana
  name: kibana-config-env
  namespace: elk
stringData:
  ELASTICSEARCH_USERNAME: kibana
  ELASTICSEARCH_PASSWORD: kibana
```

## Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kibana
  namespace: elk
  labels:
    app: kibana
spec:
  ports:
  - port: 5601
    name: web
  selector:
    app: kibana
```

## Ingress

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kibana
  namespace: elk
spec:
  rules:
  - host: kibana.apps.k8s.curiouser.com
    http:
      paths:
      - path: /
        backend:
          serviceName: kibana
          servicePort: 5601
```
