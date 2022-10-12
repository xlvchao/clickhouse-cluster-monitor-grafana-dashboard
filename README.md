=========每台机器都需要安装node_exporter================

1、wget https://github.com/prometheus/node_exporter/releases/download/v1.4.0/node_exporter-1.4.0.linux-amd64.tar.gz

2、tar -zxvf node_exporter-1.4.0.linux-amd64.tar.gz

3、mkdir -p /opt/hihonor/node_exporter/

4、cd node_exporter-1.4.0.linux-amd64

5、cp node_exporter /opt/hihonor/node_exporter/

6、vim /usr/lib/systemd/system/node_exporter.service

[Unit]
Description=Prometheus Node Exporter
Documentation=https://prometheus.io/
After=network.target

[Service]
Type=simple
User=root
Restart=always
RestartSec=1
WorkingDirectory=/opt/hihonor/node_exporter/
ExecStart=/opt/hihonor/node_exporter/node_exporter

[Install]
WantedBy=multi-user.target

7、systemctl daemon-reload
8、systemctl enable node_exporter.service

9、开启服务：systemctl start node_exporter.service
10、查看服务是否正常启动：ss -lntp |grep 9100






============Zookeeper配置文件中添加指标监控类配置（为Prometheus提供数据源）====================
官方文档：https://github.com/apache/zookeeper/blob/master/zookeeper-docs/src/main/resources/markdown/zookeeperMonitor.md#Prometheus

其中，主要是这个配置：metricsProvider.className=org.apache.zookeeper.metrics.prometheus.PrometheusMetricsProvider

配置完，需要重启zookeeper！




============单独一台机器安装prometheus===============

1、wget https://github.com/prometheus/prometheus/releases/download/v2.38.0/prometheus-2.38.0.linux-amd64.tar.gz

2、tar -zxvf prometheus-2.38.0.linux-amd64.tar.gz

3、mv prometheus-2.38.0.linux-amd64 /opt/hihonor/prometheus

4、cd /opt/hihonor/prometheus

5、mkdir logs && mkdir data && mkdir rules && mkdir bin

6、mv prometheus bin && mv promtool bin

7、vim /lib/systemd/system/prometheus.service

[Unit]
Description=Prometheus
Documentation=https://prometheus.io/
After=network.target

[Service]
Type=simple
User=root
Restart=always
RestartSec=1
WorkingDirectory=/opt/hihonor/prometheus/
ExecStart=/opt/hihonor/prometheus/bin/prometheus --config.file=/opt/hihonor/prometheus/prometheus.yml

[Install]
WantedBy=multi-user.target
	
8、systemctl daemon-reload
9、systemctl enable prometheus.service

10、开启服务：systemctl start prometheus.service

11、查看服务是否正常启动：ss -lntp |grep 9090

12、通过界面看服务是否启动正常：http://ip:9090/

13、添加需要监控的任务：vim /opt/hihonor/prometheus/prometheus.yml

scrape_configs:
  - job_name: "ck_cluster_exporter"
    relabel_configs: 
      - source_labels: ['__address__']
        target_label: 'instance'
        regex: "(.*):(.*)"
        replacement: $1
    static_configs:
      - targets: ["ip:9100", "ip:9100"]
	  
14、wq && systemctl restart prometheus.service


=========参考配置
scrape_configs:
  - job_name: "ck_cluster_exporter"
    relabel_configs: 
      - source_labels: ['__address__']
        target_label: 'instance'
        regex: "(.*):(.*)"
        replacement: $1
    static_configs:
      - targets: ["172.30.32.104:9100","172.30.32.130:9100","172.30.35.251:9100","172.30.33.114:9100","172.30.35.86:9100","172.30.34.195:9100","172.30.33.9:9100","172.30.32.7:9100","172.30.32.101:9100","172.30.33.35:9100"]
	  
  - job_name: "zk_cluster_exporter"
    relabel_configs: 
      - source_labels: ['__address__']
        target_label: 'instance'
        regex: "(.*):(.*)"
        replacement: $1
    static_configs:
      - targets: ["172.30.32.245:9100","172.30.35.227:9100","172.30.34.86:9100"]

  - job_name: "zk_cluster_monitor"
    relabel_configs: 
      - source_labels: ['__address__']
        target_label: 'instance'
        regex: "(.*):(.*)"
        replacement: $1
    static_configs:
      - targets: ["172.30.32.245:7000","172.30.35.227:7000","172.30.34.86:7000"]
	  
	  

=======================grafana配置集监控面板=============================

1、安装好clickhouse数据源插件：https://grafana.com/grafana/plugins/vertamedia-clickhouse-datasource/

2、安装好同比环比数据源插件：https://github.com/AutohomeCorp/autohome-compareQueries-datasource

3、配置好同比环比数据源

4、配置好prometheus数据源

5、配置好clickhouse集群中直连各个节点的数据源

6、导入ClickHouse-cluster-monitor-dashboard.json面板
