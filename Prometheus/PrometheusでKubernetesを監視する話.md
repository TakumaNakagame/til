# PrometheusでKubernetesを監視する話

## 監視可能なメトリックス

- NodeのCPU/Memory使用率など
- node
- service
- pod
- endpoint

### LINK

- [prometheus/node_exporter: Exporter for machine metrics](https://github.com/prometheus/node_exporter)
- [Configuration #kubernetes_sd_config | Prometheus](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#kubernetes_sd_config)

## 監視方法

Prometheusは、Kubernetesクラスタに対して、RESTful APIでクラスタ情報を取得します。併せて、Exporter呼ばれるエージェントを利用して監視対象のステータスを取得します。

## 構築方法

Prometheusの構築方法には、いくつか種類があります。代表的なものとしては次のとおりです。

- シングルバイナリを展開する
- Kubernetes上に展開する

Prometheus自体はシングルバイナリで動くため、それをそのまま実行すればOKです。マニフェストファイルをコピーすれば、複数ノードでも変わらず実行できます。

また、Kubernetes上に
