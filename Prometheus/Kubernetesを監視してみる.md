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

これを利用することで、例えばPodやNodeが