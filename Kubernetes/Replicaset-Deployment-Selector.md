 # ReplicaSet / Deployment / Selector について

## ReplicaSet
Podを指定した数だけ展開する。また、ReplicaSetで定義された数と実際のPodの数に差異があれば、定義された数に修正を行う。（Podを増やしたり減らしたり）

## Label Selector
`ReplicaSet`で展開する際に、どのPodを展開するのか識別する。

### matchLabels
指定したラベルとすべて同一のPodを展開する。
逆に、Labelが異なっているものについては展開されない。

### matchExpressions
`matchLabels`に比べてより柔軟に指定できる。

#### In/NotIn
`key`と一致するラベルが`value`内に存在するか、もしくはその逆かを指定できる。

#### Exists/DoesNotExists
`key`に指定したラベルそのものが存在するか、もしくはその逆かを指定できる。
ラベルの`value`は気にしない。

## Template
`ReplicaSet`で展開するPodの雛形を定義する。
`Label Selector`で指定したものとTemplateで指定されているラベルが同一である必要がある。