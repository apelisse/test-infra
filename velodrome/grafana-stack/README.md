The goal of this directory is to set-up the following monitoring stack:
- InfluxDB as the Time-series Database
- Grafana as the front-end/display
- Prometheus as a different flavor of time-series database
- Nginx is used as a proxy (mostly to fix CORS issue)

Step-by-step
============

First time-only
---------------
Create the passwords:
```
kubectl create secret generic grafana --from-literal=rootpassword="${grafana_password}"
kubectl create secret generic influxdb --from-literal=rootpassword="${influxdb_password}"
```

Deploying
---------
Create/Update prometheus configuration configmap:
```
kubectl create configmap prometheus --from-file=prometheus-config.yml
```

Deploying is simple:
```
kubectl apply -f grafana.yaml -f influxdb.yaml -f nginx.yaml
```

Adding data-source
------------------
First, you need to create the grafana user in Influxdb:
```
kubectl exec -i -t influxdb-123456789-abcde influx -username=root -password="${influxdb_password}" -execute "create user grafana with password 'password'; grant read on github to grafana; grant read on github to monitoring"
```

Probably a first time only:
```
./datasource.sh ${nginx_ip} ${grafana_password}
```
