## Deploy Grafana container using docker

Execute the command below to spin Grafana container at port 3000 using docker

```sh
docker run -p 3000:3000 --name=grafana -e "GF_INSTALL_PLUGINS=redis-app" grafana/grafana
```

### Use Grafana Redis application plugin for monitoring.

## Deploy Redis container using docker

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

## Deploy MongoDB container using docker

Execute the command below to spin MongoDB container at port 27017 using docker

```sh
docker run -p 27017:27017 --name mongodb bitnami/mongodb:latest 
```


## Install YCSB to benchmark Redis and MongoDB 

See step-by-step installation under **Prometheus** repository

## Load and Run YCSB workload 


## On Redis

```sh
./bin/ycsb load redis -s -P workloads/workloada -p "redis.host=172.17.0.3" -p "redis.port=6379" > outputLoad.txt
```


```sh
./bin/ycsb run redis -s -P workloads/workloada -p "redis.host=172.17.0.3" -p "redis.port=6379" -p status.interval=1 > outputRun.txt
```

## On MongoDB


```sh
./bin/ycsb load redis -s -P workloads/workloada -p "port=27017" 
```


```sh
./bin/ycsb run redis -s -P workloads/workloada -p "port=27017" -p status.interval=1 > outputRun.txt
```

## Use prometheus for monitoring 

Setting up Redis Exporter:

Download github repo:

```sh
wget https://github.com/oliver006/redis_exporter/releases/download/v1.18.0/redis_exporter-v1.18.0.linux-amd64.tar.gz
```

Unzip the tarball and cd into the directory:

```sh
tar xvfz redis_exporter-v1.18.0.linux-amd64.tar.gz
cd redis_exporter-v1.18.0.linux-amd64
```

Finally, run the exporter (Hint: use the redis system IP obtained on a previous step):

```sh
./redis_exporter -redis.addr redis://172.17.0.2:6379
```

Run Prometheus

```sh
./prometheus
```

## Visit Grafana and import a Redis dashboard instead of Redis application plugin:

Go to import -> load 763 (redis exporter dashboard ID) -> Choose prometheus as data source and click import(overwrite).

Finally, you can add customized dashboards based on your requirements. As for example you can add the CPU usage percentage.

Go to add panel, and in Metrics Browser **irate (redis_cpu_user_seconds_total [5s]) * 100**.

Save the dashboard to apply changes

# Scale Redis container resources using docker cgroups

1) Log into the Ubuntu or Debian host as a user with sudo privileges.

2) Edit the /etc/default/grub file. Add or edit the GRUB_CMDLINE_LINUX line to add the following two key-value pairs:

```sh
GRUB_CMDLINE_LINUX="cgroup_enable=memory swapaccount=1"
```
Save and close the file.

3) Update 

```sh
sudo update-grub
```

4) Reboot the VM

```sh
sudo reboot
```

5) Finally deploy your redis container and specify memory limit:

```sh
sudo docker run -m=4G-p 6379:6379 --name redis -e ALLOW_EMPTY_PASSWORD=yes bitnami/redis:latest

```

6) To update the memory use the update command

```sh
sudo docker update -m=1024M [ContainerID]
```









