# PrometheusをKubernetesで構築してみる

## 参考
[Prometheus+GrafanaでKubernetesクラスターを監視する ~Binaryファイルから起動+yamlファイルから構築~ - Qiita](https://qiita.com/FY0323/items/72616d6e280ec7f2fdaf)

## Prometheus　
### yaml

``` yaml:clusterRole.yaml
apiVersion: rbac.authorization.k8s.io/v1beta1
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
- apiGroups:
  - extensions
  resources:
  - ingresses
  verbs: ["get", "list", "watch"]
- nonResourceURLs: ["/metrics"]
  verbs: ["get"]
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: default
  namespace: monitoring
```

``` yaml:prometheus-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-server-conf
  labels:
    name: prometheus-server-conf
  namespace: monitoring
data:
  prometheus.yml: |-
    # my global config
    global:
      scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
      evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
      # scrape_timeout is set to the global default (10s).

    # Alertmanager configuration
    alerting:
      alertmanagers:
      - static_configs:
        - targets:
          # - alertmanager:9093

    # Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
    rule_files:
      # - "first_rules.yml"
      # - "second_rules.yml"

    # A scrape configuration containing exactly one endpoint to scrape:
    # Here it's Prometheus itself.
    scrape_configs:
      # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
      - job_name: 'prometheus'

        # metrics_path defaults to '/metrics'
        # scheme defaults to 'http'.

        static_configs:
        - targets: ['localhost:9100','k8s-node-1:9100']
```

``` yaml:prometheus.yaml
apiVersion: v1
kind: Service
metadata:
  name: prometheus-service
spec:
  selector: 
    app: prometheus-server
  type: NodePort
  ports:
    - port: 8080
      targetPort: 9090 
      nodePort: 31000
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: prometheus-deployment
  namespace: monitoring
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: prometheus-server
    spec:
      containers:
        - name: prometheus
          image: prom/prometheus:latest
          # configfileで指定した場所には該当のファイルが存在しません。。。
          args:
            # - "--config.file=/etc/prometheus/prometheus.yml"
            # - "--storage.tsdb.path=/prometheus/"
          ports:
            - containerPort: 9090
```

### デプロイ
```
[root@master prometheus]# kubectl create namespace monitoring
namespace/monitoring created
[root@master prometheus]# kubectl apply -f clusterRole.yaml
clusterrole.rbac.authorization.k8s.io/prometheus created
clusterrolebinding.rbac.authorization.k8s.io/prometheus created
[root@master prometheus]# kubectl apply -f prometheus-configmap.yaml
configmap/prometheus-server-conf created
[root@master prometheus]# kubectl apply -f prometheus.yaml
service/prometheus-service created
deployment.extensions/prometheus-deployment created
[root@master prometheus]#

```

### 確認
```
[root@master prometheus]# kubectl get all -n monitoring
NAME                                         READY   STATUS    RESTARTS   AGE
pod/prometheus-deployment-7f6494bf9b-sx9dt   1/1     Running   0          45s

NAME                         TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/prometheus-service   NodePort   10.96.106.150   <none>        8080:31000/TCP   45s

NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/prometheus-deployment   1/1     1            1           45s

NAME                                               DESIRED   CURRENT   READY   AGE
replicaset.apps/prometheus-deployment-7f6494bf9b   1         1         1       45s
[root@master prometheus]# curl 192.168.0.3:31000
<a href="/graph">Found</a>.

[root@master prometheus]#
```

## Grafana
### yaml
``` yaml:grafana.yaml
apiVersion: v1
kind: Service
metadata:
  name: grafana-service
  namespace: monitoring
spec:
  selector:
    app: grafana-server
  type: NodePort
  ports:
    - port: 8100
      targetPort: 3000
      nodePort: 30000
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: grafana-deployment
  namespace: monitoring
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: grafana-server
    spec:
      containers:
        - name: grafana
          image: grafana/grafana:latest
          args:
            #- "--config.file=/etc/grafana/proetheus.yaml"
            #- "--storage.tsdb.path=/grafana/"
          ports:
            - containerPort: 9090
```

### デプロイ
```
[root@master prometheus]# kubectl apply -f grafana.yaml
service/grafana-service created
deployment.extensions/grafana-deployment created
```

### 確認
```
[root@master prometheus]# kubectl get all -n monitoring
NAME                                         READY   STATUS    RESTARTS   AGE
pod/grafana-deployment-79d7cbf6d4-rxvkr      1/1     Running   0          12s
pod/prometheus-deployment-7f6494bf9b-sx9dt   1/1     Running   0          3m46s

NAME                         TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/grafana-service      NodePort   10.111.239.38   <none>        8100:30000/TCP   12s
service/prometheus-service   NodePort   10.96.106.150   <none>        8080:31000/TCP   3m46s

NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/grafana-deployment      1/1     1            1           12s
deployment.apps/prometheus-deployment   1/1     1            1           3m46s

NAME                                               DESIRED   CURRENT   READY   AGE
replicaset.apps/grafana-deployment-79d7cbf6d4      1         1         1       12s
replicaset.apps/prometheus-deployment-7f6494bf9b   1         1         1       3m46s
[root@master prometheus]#curl 192.168.0.3:30000
<a href="/login">Found</a>.

[root@master prometheus]#
```