# Kubernetesを監視してみる

Kubernetesクラスターを監視してみます。監視内容は以下の通り。

- Master
  - MasterNodeのCPU/メモリ
  - MasterNodeの死活
- Node
  - NodeのCPU/メモ
  - Nodeの死活
- Pod
  - Podの死活
  - PodのSTATUS

## kubernetes_sd_config

Prometheusには、デフォルトでKubernetesと情報を同期する設定が組み込まれている。それが`kubernetes_sd_config`である。

`kubernetes_sd_config`では、KubernetesのREST APIを叩いてNodeやPodの情報をとってくる。そのため、次のように環境変数を設定する必要がある。

```bash
KUBERNETES_SERVICE_PORT=6443
KUBERNETES_SERVICE_HOST=192.168.0.1
```

なお、`kubectl proxy`に向けると、エラー(HTTPSで接続できない)が出てしまう。これは、直接Masterのkubernetesに向けてあげるとできた

```
[root@master prometheus]# kubectl cluster-info
Kubernetes master is running at https://192.168.0.1:6443
KubeDNS is running at https://192.168.0.1:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```


## LINKS
- [Configuration | Prometheus](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#kubernetes_sd_config)