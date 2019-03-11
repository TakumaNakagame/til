# PrometheusでのAlert

PrometheusにはAlert機能がなく、AlertManagerという別のアプリケーションを利用する。

[prometheus/alertmanager: Prometheus Alertmanager](https://github.com/prometheus/alertmanager)

```plan
wget https://github.com/prometheus/alertmanager/releases/download/v0.16.1/alertmanager-0.16.1.linux-amd64.tar.gz
tar xvf alertmanager-0.16.1.linux-amd64.tar.gz
cd alertmanager-0.16.1.linux-amd64
./alertmanager
```