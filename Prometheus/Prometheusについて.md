# Prometheusについて

## Prometheusとは？

- SoundCloudにて開発/運用されていたオープンソースな監視ツール
- Cloud Native Computing Foundationに2番目にホストされたプロジェクトでもある
- Key/Value型のメトリックと時系列データを持つ多次元データモデル
- 他の監視製品に比べてメトリックスSQLがわかりやすく柔軟
- メトリックスはHTTPを用いたプルモデルにて収集される

## Prometheusを構成するもの

- Prometheus Server
- Client Library
- Push Gateway
- Exporters
- Alert Manager

### Prometheusのアーキテクチャ

Prometheusは次の機能だけを持つ

- メトリック収集
- クエリ回答
- アラート

![Prometheus-Arhitecture](./img/prometheus-architecture.png)

また、グラフなどのビジュアル化などの多くはGrafanaなどが利用されることが多い。

- [次世代監視の大本命！ Prometheus を実運用してみた - Qiita](https://qiita.com/sugitak/items/ff8f5ad845283c5915d2)
- [Overview | Prometheus](https://prometheus.io/docs/introduction/overview/)

### Prometheusの登場人物

#### exporter

監視対象に入れる、エージェントのようなもの。Prometheusからのリクエストをもとに、テキストでのメトリックデータを返す。

また、一部ソフトウェアではデフォルトでPrometheusに対応しており、特別exporterを導入しなくても回答雨できるものも存在する。また、exporterのコンテナイメージが提供されているので、これを監視対象サーバで動かすだけでOK。

exporterの種類は次の通り。

[Exporters and integrations | Prometheus](https://prometheus.io/docs/instrumenting/exporters/)

#### Prometheus

exporterから各種メトリックデータを取得するサーバ本体。シングルバイナリで動くため、非常に構築が簡単なのが特徴。