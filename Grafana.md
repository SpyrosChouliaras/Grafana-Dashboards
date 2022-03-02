## Spin up Grafana container using docker

Execute the command below to spin Grafana container at port 3000 using docker

```sh
docker run -p 3000:3000 --name=grafana -e "GF_INSTALL_PLUGINS=redis-app" grafana/grafana
```

## Spin up Redis container using docker

Execute the command below to spin Redis container at port 6379 using docker

```sh
sudo docker run --name redis -e ALLOW_EMPTY_PASSWORD=yes bitnami/redis:latest
```
Inspect the IP address of the Redis system

```sh
sudo docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container_name_or_id
```

Use the IP addrees to on the **Redis Application** plugin to monitor Redis database.

e.g., **Address : 172.17.0.3:6379**

## Spin up MongoDB container using docker

```sh
```


## Install YCSB to benchmark Redis and MongoDB 

See **Prometheus** repository
