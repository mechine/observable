# 常用

服务成功率

```
sum by (grpc_service)
(rate(grpc_server_handled_total{code="OK",grpc_service=~"com.guosen.XXXService"}[2m]))
/sum by(grpc_service)
(rate(grpc_server_handled_total{grpc_service=~"com.guosen.XXXService"}[2m]))
```

扁鹊中用到的：

```
bucket/count 计算耗时占比
sum/count 计算耗时
grpc_server_handled_total/count 计算成功率
idelta(grpc_server_started_total) 计算qps
```

