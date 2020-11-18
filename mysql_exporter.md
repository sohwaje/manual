# MySQL exporter & Prometheus Configuration

## First
### 1. Create Account for mysql_exporter user
```
CREATE USER 'exporter'@'%' IDENTIFIED BY 'exporter';
GRANT PROCESS, REPLICATION CLIENT ON *.* TO 'exporter'@'%';
GRANT SELECT ON performance_schema.* TO 'exporter'@'%';
```

### 2. Install Docker mysql_exporter
> node(db server ip, hostname)
```
docker run -d  \
  --name mysql_exporter-master -p 9104:9104 \
  -e DATA_SOURCE_NAME="exporter:exporter@(db_server:3306)/" \
  prom/mysqld-exporter
```

### 3. docker logs -f mysql_exporter-master
> time="2020-11-18T06:05:00Z" level=info msg="Starting mysqld_exporter (version=0.12.1, branch=HEAD, revision=48667bf7c3b438b5e93b259f3d17b70a7c9aff96)" source="mysqld_exporter.go:257"
time="2020-11-18T06:05:00Z" level=info msg="Build context (go=go1.12.7, user=root@0b3e56a7bc0a, date=20190729-12:35:58)" source="mysqld_exporter.go:258"
time="2020-11-18T06:05:00Z" level=info msg="Enabled scrapers:" source="mysqld_exporter.go:269"
time="2020-11-18T06:05:00Z" level=info msg=" --collect.global_status" source="mysqld_exporter.go:273"
time="2020-11-18T06:05:00Z" level=info msg=" --collect.global_variables" source="mysqld_exporter.go:273"
time="2020-11-18T06:05:00Z" level=info msg=" --collect.slave_status" source="mysqld_exporter.go:273"
time="2020-11-18T06:05:00Z" level=info msg=" --collect.info_schema.innodb_cmp" source="mysqld_exporter.go:273"
time="2020-11-18T06:05:00Z" level=info msg=" --collect.info_schema.innodb_cmpmem" source="mysqld_exporter.go:273"
time="2020-11-18T06:05:00Z" level=info msg=" --collect.info_schema.query_response_time" source="mysqld_exporter.go:273"
time="2020-11-18T06:05:00Z" level=info msg="Listening on :9104" source="mysqld_exporter.go:283"


## Second
### 1. Configuration Prometheus
```
vi prometheus.yml
- job_name: 'MySQL'
  metrics_path: '/metrics'
  static_configs:
    - targets: ['10.10.10.10:9104','10.10.10.20:9104']
```
### 2. Reload Prometheus Server
```
sudo curl -X POST http://10.10.10.1:9090/-/reload
```

### 3. Prometheus metrics
![Alt text](/prometheus_metrics.JPG)



*** ref: https://hub.docker.com/r/prom/mysqld-exporter ***
