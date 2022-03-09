# Monitoring docker containers with aAdvisor and Grafana


1.First create a prometheus.yml and docker-compose.yml files. Edit the files as follows:

For prometheus.yml:

```sh
global:
  scrape_interval: 1s
  evaluation_interval: 1s
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['external_IP:9090']
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['external_IP:9100']
  - job_name: 'redis_exporter'
    static_configs:
      - targets: ['external_IP:9121']
  - job_name: cadvisor
    scrape_interval: 5s
    static_configs:
    - targets:
      - cadvisor:8080
```


This will add exporters (e.g., redis_exporter) and cAdvisor. You can see cAdvisor metrics at http://external_ip:8080. Remember that you need to set firewall rules to export GCP ports.

To see Prometheus targets or graphs visit http://external_ip:9090/graphs and http://external_ip:9090/targets respectively.

For the docker-compose.yml:

```sh
version: '3.2'
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
    - 9090:9090
    command:
    - --config.file=/etc/prometheus/prometheus.yml
    volumes:
    - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
    depends_on:
    - cadvisor
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    ports:
    - 8080:8080
    volumes:
    - /:/rootfs:ro
    - /var/run:/var/run:rw
    - /sys:/sys:ro
    - /var/lib/docker/:/var/lib/docker:ro
    depends_on:
    - redis
  redis:
    image: redis:latest
    container_name: redis
    ports:
    - 6379:6379
```

To run the installation:

```sh
docker-compose up
```

2. Deploy Redis container. This Redis instance will be used to load/run YCSB workload.

```sh
sudo docker run -p 6380:6380 --name redis-ycsb -e ALLOW_EMPTY_PASSWORD=yes bitnami/redis:latest
```

3.Deploy redis-ycsb to execute YCSB workload

```sh
sudo docker run -m=4G -p 6380:6380 --name redis-ycsb -e ALLOW_EMPTY_PASSWORD=yes bitnami/redis:latest
 ```
 
4.Target redis-ycsb with redis_exporter (download and install redis_exporter). To find the port of redis-ycsb container run the following commnad:

```sh
sudo docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container_name_or_id
```

Then run:

```sh
./redis_exporter -redis.addr redis://redis-ycsb-port:6379 -include-system-metrics=true
```

5.Deploy Grafana container:

```sh
sudo docker run -p 3000:3000 --name=grafana -e "GF_INSTALL_PLUGINS=redis-app" grafana/grafana
```


## Visit Grafana at external_IP:3000 and setup prometheus and cAdvisor dashboard 

1.First add data source, select prometheus and enter the **URL: http://external_ip:9090**. Change **HTTP method to GET** and press **Save and Test**.
2.To add a dashboard, go to import -> 893 (dashboard ID) -> Load -> Select Prometheus as data source.
3.You can also use a different dashboard Id e.g., to inspect redis through redis_exporter (e.g., use 763)


