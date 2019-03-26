# Kubernetesに構築してみる（本気）

※今回の構成に、AlertManagerは含まれません

### 手順

1. Namespaceの作成
1. ServiceAccountの作成
1. Node-Exporterの構築
1. ConfigMapの適用
1. Prometheusの構築

### Namespaceの作成

Prometheusを構築するためのNamespaceを作成します。なんでもいい。

#### namespace.yaml

``` yaml
namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: monitoring
```

### ServiceAccountの作成

Prometheusからkube-apiserverへREST APIでクラスタ情報を取得しにいくので、その時のServiceAccount。

ServiceAccout `prometheus-k8s`に対して、ClusterRole `prometheus`を紐づけている。

#### prom-rbac.yaml

``` yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: prometheus-k8s
  namespace: monitoring
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus
rules:
- apiGroups: [""]
  resources:
  - nodes
  - nodes/proxy
  - services
  - endpoints
  - pods
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources:
  - configmaps
  verbs: ["get"]
- nonResourceURLs: ["/metrics"]
  verbs: ["get"]
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus-k8s
  namespace: monitoring
```

### Node_Exporterの構築

Kubernetes Nodesの各種情報(CPU/RAMなど)を取得するために`node_exporter`を導入する。

#### Node_expoter.yaml

``` yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: prometheus-node-exporter
  namespace: monitoring
  labels:
    app: prometheus
    component: node-exporter
spec:
  template:
    metadata:
      name: prometheus-node-exporter
      labels:
        app: prometheus
        component: node-exporter
    spec:
      containers:
      - image: prom/node-exporter:v0.14.0
        name: prometheus-node-exporter
        ports:
        - name: prom-node-exp
          containerPort: 9100
          hostPort: 9100
      hostNetwork: true
      hostPID: true
---
apiVersion: v1
kind: Service
metadata:
  annotations:
    prometheus.io/scrape: 'true'
  name: prometheus-node-exporter
  namespace: monitoring
  labels:
    app: prometheus
    component: node-exporter
spec:
  clusterIP: None
  ports:
    - name: prometheus-node-exporter
      port: 9100
      protocol: TCP
  selector:
    app: prometheus
    component: node-exporter
  type: ClusterIP
```

### ConfigMapの適用

Prometheusを展開するために、Configmapを展開する。また、各種監視設定の変更はこのファイルを変更していく。

#### prom-configmap.yaml

``` yaml
apiVersion: v1
data:
  prometheus.yaml: |
    global:
      scrape_interval: 10s
      scrape_timeout: 10s
      evaluation_interval: 10s
    rule_files:
      - "/etc/prometheus-rules/*.rules"
    scrape_configs:

      - job_name: 'kubernetes-nodes'
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        kubernetes_sd_configs:
          - role: node
        relabel_configs:
          - source_labels: [__address__]
            regex: '(.*):10250'
            replacement: '${1}:10255'
            target_label: __address__
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: prometheus-core
  namespace: monitoring
```

### Prometheusの構築

Prometheusを構築します。今回はNodePortで構築しています。

#### prom-core.yaml

``` yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: prometheus-core
  namespace: monitoring
  labels:
    app: prometheus
    component: core
spec:
  replicas: 1
  template:
    metadata:
      name: prometheus-main
      labels:
        app: prometheus
        component: core
    spec:
      serviceAccountName: prometheus-k8s
      containers:
      - name: prometheus
        image: prom/prometheus:v1.7.0
        args:
          - '-storage.local.retention=12h'
          - '-storage.local.memory-chunks=500000'
          - '-config.file=/etc/prometheus/prometheus.yaml'
        ports:
        - name: webui
          containerPort: 9090
        resources:
          requests:
            cpu: 500m
            memory: 500M
          limits:
            cpu: 500m
            memory: 500M
        volumeMounts:
        - name: config-volume
          mountPath: /etc/prometheus
      volumes:
      - name: config-volume
        configMap:
          name: prometheus-core
---
apiVersion: v1
kind: Service
metadata:
  name: prometheus
  namespace: monitoring
  labels:
    app: prometheus
    component: core
  annotations:
    prometheus.io/scrape: 'true'
spec:
  type: NodePort
  ports:
    - port: 9090
      protocol: TCP
      name: webui
  selector:
    app: prometheus
    component: core
```
