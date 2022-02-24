# 服务自动发现

传统的监控系统中，自动发现和主动注册是两个步骤

* 自动发现：服务器去某个网段扫描符合条件的机器，然后把他加入到我们的监控列表中
* 主动注册：客户端在启动某些符合条件的进程之后（比如zabbix-agent），主动去服务端注册自己的信息，然后加入到我们的监控列表

**prometheus采用类似自动发现的策略。**

### 静态配置

scrape\_configs

```
scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']
```

### 动态文件

通过配置管理工具填充的文件接收目标列表（file\_sd\_config）

Prometheus按照指定的时间表从这些文件中重新加载目标，这些文件可以是Yaml或者Json格式。

```
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
    - targets: ['localhost:9090']
 
  - job_name: 'node_exporter'
    file_sd_configs:
    - files:
      - targets/nodes/*.json
      refresh_interval: 5m

```

```
[{
  "targets":[
    "192.168.42.133:9100",
    "192.168.42.128:9100"
  ],
  "labels":{
    "datacenter":"CD"
  }
}]
```

### API

查询API以获取目标列表（azure\_sd\_config、consul\_sd\_config、ec2\_sd\_config、openstack\_sd\_config、kubernetes\_sd\_config、gce\_sd\_config等等）。

```
  - job_name: 'consul-node-exporter'
    consul_sd_configs:
      - server: 'localhost:8500'
        services: []  
    relabel_configs:
      - source_labels: [__meta_consul_tags]
        regex: .*node-exporter.*
        action: keep
      - regex: __meta_consul_service_metadata_(.+)
        action: labelmap
  
  - job_name: 'consul-kafka-exproter'
    consul_sd_configs:
      - server: 'localhost:8500'
        services: []
    relabel_configs:
      - source_labels: [__meta_consul_tags]
        regex: .*kafka-exporter.*
        action: keep
      - regex: __meta_consul_service_metadata_(.+)
        action: labelmap
```

### DNS

使用DNS记录返回目标列表（dns\_sd\_config）。

```
scrape_configs:
  - job_name: 'node_exporter_dns'   #基于SRV记录发现，不手动指定，默认就是基于SRV
    dns_sd_configs:
    - names: ['_prometheus._tcp.cmxu.com']
 
  - job_name: 'docker_CADvisor_dns'  #基于A记录发现
    dns_sd_configs:
    - names: ['node1.cmxu.com']
      type: A
      port: 8181
```

