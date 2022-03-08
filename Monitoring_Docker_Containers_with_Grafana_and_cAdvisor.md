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


This will add exporters (e.g., redis_exporter) and cAdvisor.

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

2.Deploy Grafana container:

```sh
sudo docker run -p 3000:3000 --name=grafana -e "GF_INSTALL_PLUGINS=redis-app" grafana/grafana
```

3.Deploy redis-ycsb to execute YCSB workload

```sh
sudo docker run -m=4G -p 6380:6380 --name redis-ycsb -e ALLOW_EMPTY_PASSWORD=yes bitnami/redis:latest
 ```
 
4.Target redis-ycsb with redis_exporter (install and download 

```sh
./redis_exporter -redis.addr redis://0.0.0.0:6380 -include-system-metrics=true
```
