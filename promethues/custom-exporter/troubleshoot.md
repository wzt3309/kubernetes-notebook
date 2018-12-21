# Prometheus-operator 自动更新失效

## 问题1
在使用虚拟机做实验的时候，我发现使用helm安装的Prometheus-operator时会出现不能自动更新prometheus的配置，通过查看`prometheus-kube-prometheus-0`这个pod内的`prometheus-config-reload` 这个容器的日志，发现如下内容
```
level=error ts=2018-12-21T12:00:52.07627433Z caller=runutil.go:43 msg="function failed. Retrying" err="trigger reload: reload request failed: Post http://kube-prometheus.monitoring:9090/-/reload: dial tcp xxx.xxx.xxx.xxx:9090: connect: connection refused"
```
其中`xxx.xxx.xxx.xxx`为我服务器的ip地址（公网ip，为了隐私腻了），我再仔细查看了`prometheus-config-reload`这个容器的启动参数，发现
```
--reload-url=http://localhost:9090/-/reload
```
我想到，可能是因为这个localhost直接将地址解析到了我服务器ip，最后造成prometheus无法自动更新target和rule。

因此需要在`prometheus-kube-prometheus`上更新一下容器`prometheus-config-reloader`的`--reload-url=`和 容器`alerting-rule-files-configmap-reloader`的`--webhook-url`

```
--reload-url=http://kube-prometheus.monitoring:9090/-/reload
--config-file=/etc/prometheus/config/prometheus.yaml
--config-envsubst-file=/etc/prometheus/config_out/prometheus.env.yaml
```

```
--webhook-url=http://kube-prometheus.monitoring:9090/-/reload
--volume-dir=/etc/prometheus/rules
```

## 问题2
servicemonitor的label不匹配导致，operator无法监听到该新的servicemonitor
具体解决：https://github.com/coreos/prometheus-operator/issues/890
