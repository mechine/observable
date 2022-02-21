# rules预聚合

Prometheus 支持两种类型的 `Rules` ，可以对其进行配置，然后定期进行运算：`recording rules` 记录规则 与 `alerting rules` 警报规则，规则文件的计算频率与告警规则计算频率一致，都是通过全局配置中的 `evaluation_interval` 定义

```
groups:
- name: http_requests_total
  rules:
  - record: job:http_requests_total:rate10m
    expr: sum by (job)(rate(http_requests_total[10m]))
    lables:
      team: operations
  - record: job:http_requests_total:rate30m
    expr: sum by (job)(rate(http_requests_total[30m]))
    lables:
      team: operations 
```

