## docker-compose 部署 prometheus 监控栈
- prometheus
- nodeexporter
- grafana
- pushgateway
- alertmanager

## pushgateway 使用
pushgateway 是 prometheus 拉取数据的中间网关。在没有 exporter 的情况下我们可以使用 curl 或者 api 库向 pushgateway 推送
符合 prometheus 格式的指标数据，然后 prometheus 抓取配置 pushgateway 的数据源。下面以 curl 发送指标数据到 pushgateway 为例：
持续 ping 一个 ip，监控 ip 的 ping 延迟，脚本使用：`sh ping.sh <ip>`。

ping.sh
```bash
# !/bin/bash

PUSH_GATEWAY=http://host:9091
JOB_NAME=PING
INSTANCE=$1
while [ 1 ]
do
  ping_data=`ping -c 1 $1 | grep icmp_seq | grep time`
  if [ `echo $ping_data | grep timeout` ]; then
    echo "ping_latency{ip=\"$INSTANCE\"} 0" | curl -sS --data-binary @- $PUSH_GATEWAY/metrics/job/$JOB_NAME/instance/$INSTANCE
  else
    latency=`echo $ping_data | sed 's/=/ /g' | awk '{print $(NF-1)}'`
    echo "ping_latency{ip=\"$INSTANCE\"} $latency" | curl -sS --data-binary @- $PUSH_GATEWAY/metrics/job/$JOB_NAME/instance/$INSTANCE
  fi
  sleep 1
done
```