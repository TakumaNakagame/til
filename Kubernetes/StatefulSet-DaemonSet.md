# StatefulSet DaemonSet

## DaemonSet

ReplicaSetに近いが、ReplicaSetのようにPodの数を指定することができない。起動されるPodはNodeの数に比例し、各Nodeに必ず存在することになる。

例えば、Prometheusでの監視に必要な`Node-exporter`のようなものを動かしておくときに使う。

## StatefulSet

主に、データベースのようなStatefulなワークロードに対応するためのもの。

`ReplicaSet`のように`Replicas`を設定することはできるが、展開されるPodの挙動が異なる。まず、Podの名前は`name` + `INDEX`となる。そして、Podを削除する際は1つ1つさくじょされ、Indexが大きい方（新しい方）から削除されることになる。

こうすることで、たとえばMasterとSlaveで構成されているDBについては、Index 0がMasterとなり、Index 0+n(0 < n)がSlaveとなる。スケールアウトする際はSlaveが増え、スケールインするときはIndex 0が残るので、データが損失しない。


## LINK
- [KubernetesのWorkloadsリソース（その2） | Think IT（シンクイット）](https://thinkit.co.jp/article/13611)