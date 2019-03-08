# PromQL

## 基本的な構文

### 完全一致するメトリックをすべて

```plan
http_requests_total
```

### 特定のラベルをもつメトリック

`http_requests_total`のラベル`job`が`prometheus`かつ、ラベル`group`が`canary`な時系列データが抽出される

```plan
http_requests_total{job="prometheus",group="canary"}
```

## LINK

- [クエリの基礎 - Prometheusドキュメント - it-engineer’s blog](http://it-engineer.hateblo.jp/entry/2019/01/19/150849)