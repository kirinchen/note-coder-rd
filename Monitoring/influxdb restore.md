---
title: influxdb restore
tags: [influx]

---

# influxdb restore

```shell=

cd /var/lib/jenkins/workspace/bzk-tig

wget https://finifxdbbackup.firebaseapp.com/ifxdbback.tar

tar -xvf ifxdbback.tar 


docker cp ./ifxdbback $(docker ps -a -q  --filter name=bzk-tig_influxdb_1):/ifxdbback

docker-compose exec -T influxdb influx bucket delete -n quote -o finance -t NJcX78dv1axqqaa11

docker-compose exec -T influxdb influx restore /ifxdbback/ -t NJcX78dv1axqqaa11


docker-compose exec -T influxdb rm /ifxdbback/ -rf
```



```shell=

cd /var/lib/jenkins/workspace/TIG

wget https://finifxdbbackup.firebaseapp.com/ifxdbback.tar

tar -xvf ifxdbback.tar 


docker cp ./ifxdbback $(docker ps -a -q  --filter name=tig_influxdb_1):/ifxdbback

docker-compose exec -T influxdb influx bucket delete -n quote -o finance -t NJcX78dv1axqqaa11

docker-compose exec -T influxdb influx restore /ifxdbback/ -t NJcX78dv1axqqaa11


docker-compose exec -T influxdb rm /ifxdbback/ -rf
```
###### tags: `influx`