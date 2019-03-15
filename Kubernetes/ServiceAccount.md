# Service Account

## RBACとは

Role-Based Access Controlの略で、役割でアクセス制限をする方法。

[Using RBAC Authorization - Kubernetes](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

### RBACのリソース

主に次のようなリソースが存在する。

- Role
- ClusterRole
- RoleBinding
- ClusterRoleBinding

`Role`系はどのリソースに対してどんな操作を許可するのかを定義する。

`Binding`系はRoleとServiceAccoutの紐付けを定義する。

### リソース/操作

#### リソース

Kubernetes上のリソースを指す

- pod
- deployment
- service
- secret
- etc...

#### 操作

Kubernetes上のリソースに対して行える操作

- get
- create 
- update
- delete 
- list
- etc...

### Role/RoleBinding と ClusterRole/ClusterRoleBinding の違い

Role/RoleBindingは特定のNamespaceに所属しているが、ClusterRole/ClusterRoleBindingはNamespaceに属さない。

## ServiceAccountについて

- 接続元のアプリケーションを識別するもの
- Roleを紐付けることができる
- Pod作成時に必ず紐付けられる
  - Namespaceを作ると、`default`というServiceAccountが自動で作られる
  - KubernetesClusterを作成したときに`default`というNamespaceが作られ、同時に`default`というServiceAccountも作られる

### ServiceAccountの使用早見表

| **アクセス制限対象**|**Role種類**|**Binding種類** |
|:-----:|:-----:|:-----:|
| Cluster-level resources (Nodes, PersistentVolumes, ...)|ClusterRole|ClusterRoleBinding |
| Non-resource URLs (/api, /healthz, ...)|ClusterRole|ClusterRoleBinding |
| いくつかのnamespace、もしくは全てのnamaespaceに存在するNamespaced resources|ClusterRole|ClusterRoleBinding |
| 特定のnamaspaceに存在するNamespaced resources。(共通のRoleを使いたい場合)|ClusterRole|RoleBinding |
| 特定のnamaspaceに存在するNamespaced resources。(namespaceごとにRoleを定義する場合)|Role|RoleBinding |

## 作成したService Accountのシークレット取得方法

ServiceAccountの一覧を取得。今回は、`prometheus-k8s`が該当する。
```bash
$ kubectl get sa -n monitoring2
NAME                 SECRETS   AGE
default              1         18h
kube-state-metrics   1         18h
prometheus-k8s       1         18h
```

取得したServiceAccountの詳細を確認する。secretsのnameが`prometheus-k8s-token-4shpk`であることがわかった。

```bash
$ kubectl get sa -n monitoring2 prometheus-k8s -o yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"ServiceAccount","metadata":{"annotations":{},"name":"prometheus-k8s","namespace":"monitoring2"}}
  creationTimestamp: "2019-03-13T09:36:07Z"
  name: prometheus-k8s
  namespace: monitoring2
  resourceVersion: "1757242"
  selfLink: /api/v1/namespaces/monitoring2/serviceaccounts/prometheus-k8s
  uid: 6dbd09af-4573-11e9-a0b4-9ca3ba31de2d
secrets:
- name: prometheus-k8s-token-4shpk
```

判明したsecretsのnameの詳細情報を取得する。ca, tokenともに取得できた。

```bash
$ kubectl get secrets -n monitoring2 prometheus-k8s-token-4shpk -o yaml
apiVersion: v1
data:
  ca.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRFNU1ESXlPREF5TURnMU1Wb1hEVEk1TURJeU5UQXlNRGcxTVZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBS0puCnNNY2ZaNXJKSHRNd01OYW55VHhtMjEyU3QrN2R4N2E2OGxOUlJXa204YmlsN3lsdGx2SDc1L2dOb0pzanZTalMKS0hVdjFEZ3F6S0NGM00yYVA1azBaK3ZsY1Y0N1ZtOEljZWVJcFBlc1QyeFJXWEhxeGpMTUtOOWxSbjArTDdGZwpRM01MRVJDUkpKdEIyWEpkREYvSlJYZUhzYkpOK3luY1lOSHRSWGlCenJBK2hoV3daWWNuRDkxdDJEWVpNc3AwClI4MzNFVURoRkNweFdtMFpHRzV4ZEdtTzJraWRYWnhSc3J4Q3huVGY2SGh6bVUrRGtNVWlVYTZFa3NsS0xWY2kKRUpkS1g5N2RjNlViQm1SWWZFRFBXWHVKelhuUW1CaFpNaWNobWhxSVJ3YUFxMC9vclJ4ei9ZQzVidHdSS1p2RApRZld4VXhVS3RkMjlrcHRXV2drQ0F3RUFBYU1qTUNFd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFHdWRESEtTdXYycWFVeDNwSzFwL0xuVTlscUkKalo0RjhlUVVKbTNTajA5WXJPUHU2LzNCdGVvYVBjaHVqamJKUER3eEpGeE5BNE1jVUZlUG5kSXVRRWxyYVEvRwpLbVVpejVJSUpIV1BvbStCOW1ZWFYxQzEwRkJ1ajlUd1pyQlNuYVBscy9uS1Z1UmoybDg3eWx5VU01Qmoxekg2CmVVbU9ST0lpQ2FMUm8xR3krSlRUNVl5ZytUWmQ1VmM2aTRuOUU3YWNmeG1YN0d6dE16bEpsdEdJZXRHMFpON04KY2JEblNEaUFJVEZZSEVSVHlBQ0lnZjJmVUF6Y3JaNUFZK3gwVU5LQ1BhM0xpajY3NGNPUXllQUp3Z2pLT2FBWApBWGVZVzRFRjNPRjdrQnpTV3dTdkpmdWE5L1MxZkNRaHNlL2RPVWRxU3BvTFcwczlvR2ZJZkx0a2dwZz0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
  namespace: bW9uaXRvcmluZzI=
  token: ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNklpSjkuZXlKcGMzTWlPaUpyZFdKbGNtNWxkR1Z6TDNObGNuWnBZMlZoWTJOdmRXNTBJaXdpYTNWaVpYSnVaWFJsY3k1cGJ5OXpaWEoyYVdObFlXTmpiM1Z1ZEM5dVlXMWxjM0JoWTJVaU9pSnRiMjVwZEc5eWFXNW5NaUlzSW10MVltVnlibVYwWlhNdWFXOHZjMlZ5ZG1salpXRmpZMjkxYm5RdmMyVmpjbVYwTG01aGJXVWlPaUp3Y205dFpYUm9aWFZ6TFdzNGN5MTBiMnRsYmkwMGMyaHdheUlzSW10MVltVnlibVYwWlhNdWFXOHZjMlZ5ZG1salpXRmpZMjkxYm5RdmMyVnlkbWxqWlMxaFkyTnZkVzUwTG01aGJXVWlPaUp3Y205dFpYUm9aWFZ6TFdzNGN5SXNJbXQxWW1WeWJtVjBaWE11YVc4dmMyVnlkbWxqWldGalkyOTFiblF2YzJWeWRtbGpaUzFoWTJOdmRXNTBMblZwWkNJNklqWmtZbVF3T1dGbUxUUTFOek10TVRGbE9TMWhNR0kwTFRsallUTmlZVE14WkdVeVpDSXNJbk4xWWlJNkluTjVjM1JsYlRwelpYSjJhV05sWVdOamIzVnVkRHB0YjI1cGRHOXlhVzVuTWpwd2NtOXRaWFJvWlhWekxXczRjeUo5LnZhMGJGb19mMkJwWVRVYmJkT0ExZl9RVERjWWVFREh2aU1jOW1ENDl6TXZFcF9oRklCWmJ1WUZHQVBseGZLRjdIYVZjQ1FSRGlxMklPbF91cmo4NThkVGhEa2FLRUtYa3N6QjI0VjEwNHdETmVsYktCS1dkSHZJNFRhWVVJR3ExaFcxaWo2NUJZYXIxczcxVHBCNDVOR3paSUdmMlRnS3BYaDRuenBLQ1lSbWE1ZTV6RzdLR2tFbGZhN3NGMUt0dWdFLUFaR1Ewa1ZFNTF0YUxyS1R2R1VCS3VXVU9jX0NlbWxQR2JWWF9pX1lPQ2FOenhwU3NWeWZDc0pMUHRuX1d5WnZXeFFEaVN4N2J6T3A4VHY5YTVsWkotRTVSTXJSSnROM3BLSGRiTjI0YXo5cUFpR3B1V3RwUmJYVEE5T2p3eTY5dmtpRkxVdy14aDBoNXVaNUYxUQ==
kind: Secret
metadata:
  annotations:
    kubernetes.io/service-account.name: prometheus-k8s
    kubernetes.io/service-account.uid: 6dbd09af-4573-11e9-a0b4-9ca3ba31de2d
  creationTimestamp: "2019-03-13T09:36:07Z"
  name: prometheus-k8s-token-4shpk
  namespace: monitoring2
  resourceVersion: "1757241"
  selfLink: /api/v1/namespaces/monitoring2/secrets/prometheus-k8s-token-4shpk
  uid: 6dbf472f-4573-11e9-a0b4-9ca3ba31de2d
type: kubernetes.io/service-account-token
```

更に、BASE64エンコードされているのでデコードし、ファイルに書き出す。

CAのデコード
```bash
$ echo "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRFNU1ESXlPREF5TURnMU1Wb1hEVEk1TURJeU5UQXlNRGcxTVZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBS0puCnNNY2ZaNXJKSHRNd01OYW55VHhtMjEyU3QrN2R4N2E2OGxOUlJXa204YmlsN3lsdGx2SDc1L2dOb0pzanZTalMKS0hVdjFEZ3F6S0NGM00yYVA1azBaK3ZsY1Y0N1ZtOEljZWVJcFBlc1QyeFJXWEhxeGpMTUtOOWxSbjArTDdGZwpRM01MRVJDUkpKdEIyWEpkREYvSlJYZUhzYkpOK3luY1lOSHRSWGlCenJBK2hoV3daWWNuRDkxdDJEWVpNc3AwClI4MzNFVURoRkNweFdtMFpHRzV4ZEdtTzJraWRYWnhSc3J4Q3huVGY2SGh6bVUrRGtNVWlVYTZFa3NsS0xWY2kKRUpkS1g5N2RjNlViQm1SWWZFRFBXWHVKelhuUW1CaFpNaWNobWhxSVJ3YUFxMC9vclJ4ei9ZQzVidHdSS1p2RApRZld4VXhVS3RkMjlrcHRXV2drQ0F3RUFBYU1qTUNFd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFHdWRESEtTdXYycWFVeDNwSzFwL0xuVTlscUkKalo0RjhlUVVKbTNTajA5WXJPUHU2LzNCdGVvYVBjaHVqamJKUER3eEpGeE5BNE1jVUZlUG5kSXVRRWxyYVEvRwpLbVVpejVJSUpIV1BvbStCOW1ZWFYxQzEwRkJ1ajlUd1pyQlNuYVBscy9uS1Z1UmoybDg3eWx5VU01Qmoxekg2CmVVbU9ST0lpQ2FMUm8xR3krSlRUNVl5ZytUWmQ1VmM2aTRuOUU3YWNmeG1YN0d6dE16bEpsdEdJZXRHMFpON04KY2JEblNEaUFJVEZZSEVSVHlBQ0lnZjJmVUF6Y3JaNUFZK3gwVU5LQ1BhM0xpajY3NGNPUXllQUp3Z2pLT2FBWApBWGVZVzRFRjNPRjdrQnpTV3dTdkpmdWE5L1MxZkNRaHNlL2RPVWRxU3BvTFcwczlvR2ZJZkx0a2dwZz0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=" | base64 -d > ca.crt
$ cat ca.crt
-----BEGIN CERTIFICATE-----
MIICyDCCAbCgAwIBAgIBADANBgkqhkiG9w0BAQsFADAVMRMwEQYDVQQDEwprdWJl
cm5ldGVzMB4XDTE5MDIyODAyMDg1MVoXDTI5MDIyNTAyMDg1MVowFTETMBEGA1UE
AxMKa3ViZXJuZXRlczCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAKJn
sMcfZ5rJHtMwMNanyTxm212St+7dx7a68lNRRWkm8bil7yltlvH75/gNoJsjvSjS
KHUv1DgqzKCF3M2aP5k0Z+vlcV47Vm8IceeIpPesT2xRWXHqxjLMKN9lRn0+L7Fg
Q3MLERCRJJtB2XJdDF/JRXeHsbJN+yncYNHtRXiBzrA+hhWwZYcnD91t2DYZMsp0
R833EUDhFCpxWm0ZGG5xdGmO2kidXZxRsrxCxnTf6HhzmU+DkMUiUa6EkslKLVci
EJdKX97dc6UbBmRYfEDPWXuJzXnQmBhZMichmhqIRwaAq0/orRxz/YC5btwRKZvD
QfWxUxUKtd29kptWWgkCAwEAAaMjMCEwDgYDVR0PAQH/BAQDAgKkMA8GA1UdEwEB
/wQFMAMBAf8wDQYJKoZIhvcNAQELBQADggEBAGudDHKSuv2qaUx3pK1p/LnU9lqI
jZ4F8eQUJm3Sj09YrOPu6/3BteoaPchujjbJPDwxJFxNA4McUFePndIuQElraQ/G
KmUiz5IIJHWPom+B9mYXV1C10FBuj9TwZrBSnaPls/nKVuRj2l87ylyUM5Bj1zH6
eUmOROIiCaLRo1Gy+JTT5Yyg+TZd5Vc6i4n9E7acfxmX7GztMzlJltGIetG0ZN7N
cbDnSDiAITFYHERTyACIgf2fUAzcrZ5AY+x0UNKCPa3Lij674cOQyeAJwgjKOaAX
AXeYW4EF3OF7kBzSWwSvJfua9/S1fCQhse/dOUdqSpoLW0s9oGfIfLtkgpg=
-----END CERTIFICATE-----
```

Tokenのデコード

```bash
$  echo "ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNklpSjkuZXlKcGMzTWlPaUpyZFdKbGNtNWxkR1Z6TDNObGNuWnBZMlZoWTJOdmRXNTBJaXdpYTNWaVpYSnVaWFJsY3k1cGJ5OXpaWEoyYVdObFlXTmpiM1Z1ZEM5dVlXMWxjM0JoWTJVaU9pSnRiMjVwZEc5eWFXNW5NaUlzSW10MVltVnlibVYwWlhNdWFXOHZjMlZ5ZG1salpXRmpZMjkxYm5RdmMyVmpjbVYwTG01aGJXVWlPaUp3Y205dFpYUm9aWFZ6TFdzNGN5MTBiMnRsYmkwMGMyaHdheUlzSW10MVltVnlibVYwWlhNdWFXOHZjMlZ5ZG1salpXRmpZMjkxYm5RdmMyVnlkbWxqWlMxaFkyTnZkVzUwTG01aGJXVWlPaUp3Y205dFpYUm9aWFZ6TFdzNGN5SXNJbXQxWW1WeWJtVjBaWE11YVc4dmMyVnlkbWxqWldGalkyOTFiblF2YzJWeWRtbGpaUzFoWTJOdmRXNTBMblZwWkNJNklqWmtZbVF3T1dGbUxUUTFOek10TVRGbE9TMWhNR0kwTFRsallUTmlZVE14WkdVeVpDSXNJbk4xWWlJNkluTjVjM1JsYlRwelpYSjJhV05sWVdOamIzVnVkRHB0YjI1cGRHOXlhVzVuTWpwd2NtOXRaWFJvWlhWekxXczRjeUo5LnZhMGJGb19mMkJwWVRVYmJkT0ExZl9RVERjWWVFREh2aU1jOW1ENDl6TXZFcF9oRklCWmJ1WUZHQVBseGZLRjdIYVZjQ1FSRGlxMklPbF91cmo4NThkVGhEa2FLRUtYa3N6QjI0VjEwNHdETmVsYktCS1dkSHZJNFRhWVVJR3ExaFcxaWo2NUJZYXIxczcxVHBCNDVOR3paSUdmMlRnS3BYaDRuenBLQ1lSbWE1ZTV6RzdLR2tFbGZhN3NGMUt0dWdFLUFaR1Ewa1ZFNTF0YUxyS1R2R1VCS3VXVU9jX0NlbWxQR2JWWF9pX1lPQ2FOenhwU3NWeWZDc0pMUHRuX1d5WnZXeFFEaVN4N2J6T3A4VHY5YTVsWkotRTVSTXJSSnROM3BLSGRiTjI0YXo5cUFpR3B1V3RwUmJYVEE5T2p3eTY5dmtpRkxVdy14aDBoNXVaNUYxUQ==" | base64 -d > token
$ cat token
eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJtb25pdG9yaW5nMiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJwcm9tZXRoZXVzLWs4cy10b2tlbi00c2hwayIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJwcm9tZXRoZXVzLWs4cyIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjZkYmQwOWFmLTQ1NzMtMTFlOS1hMGI0LTljYTNiYTMxZGUyZCIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDptb25pdG9yaW5nMjpwcm9tZXRoZXVzLWs4cyJ9.va0bFo_f2BpYTUbbdOA1f_QTDcYeEDHviMc9mD49zMvEp_hFIBZbuYFGAPlxfKF7HaVcCQRDiq2IOl_urj858dThDkaKEKXkszB24V104wDNelbKBKWdHvI4TaYUIGq1hW1ij65BYar1s71TpB45NGzZIGf2TgKpXh4nzpKCYRma5e5zG7KGkElfa7sF1KtugE-AZGQ0kVE51taLrKTvGUBKuWUOc_CemlPGbVX_i_YOCaNzxpSsVyfCsJLPtn_WyZvWxQDiSx7bzOp8Tv9a5lZJ-E5RMrRJtN3pKHdbN24az9qAiGpuWtpRbXTA9Ojwy69vkiFLUw-xh0h5uZ5F1Q
```

Tokenは環境変数に代入する

```bash
$ export K8S_TOKEN=$(cat token)
```

## REST APIを叩く

通常、トークンを渡さずにREST APIを叩くと次のようになる

```bash
$ curl -k https://192.168.0.1:6443/cluster-info
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {

  },
  "status": "Failure",
  "message": "forbidden: User \"system:anonymous\" cannot get path \"/cluster-info\"",
  "reason": "Forbidden",
  "details": {

  },
  "code": 403
}
```

デフォルトで、`system:anonymous`になるため、権限不足で参照できない。


## LINK

- [KubernetesのRBACについて - Qiita](https://qiita.com/sheepland/items/67a5bb9b19d8686f389d)