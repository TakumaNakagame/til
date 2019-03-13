# proxy

KubernetesでREST APIを待ち受けるためのProxy。以下のコマンドで有効にできる。

``` 
kubectl proxy
```

## オプション

### ポート変更

```
kubectl proxy --port=8080
# default 8001
```

### Listen Host変更

```
kubectl proxy --address=0.0.0.0
# default 127.0.0.1
```