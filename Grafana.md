## Load Grafana using docker

Execute the command below to spin Grafana server at port 3000

```sh
docker run -p 3000:3000 --name=grafana -e "GF_INSTALL_PLUGINS=redis-app" grafana/grafana
```

