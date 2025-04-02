---
title: docker 進入containier
tags: [docker]

---

# docker 進入containier

```shell=
docker-compose exec -T bzkflow sh

```

```shell=
docker cp $(docker ps -a -q  --filter name=flow_deploy_bzkflow_1):/tmp/log/ ./bzkflowlog
```


###### tags: `docker`