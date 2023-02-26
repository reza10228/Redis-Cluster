
# Redis Cluster

we need at least 2 vms master and slave

master yaml file
``` yaml
version: '3.9'

services:
  fix-redis-volume-ownership: # This service is to authorise redis-master with ownership permissions
    image: 'bitnami/redis:latest'
    user: root
    command: chown -R 1001:1001 /bitnami
    volumes:
      - ./data/redis:/bitnami
      - ./data/redis/conf/redis.conf:/opt/bitnami/redis/conf/redis.conf

  redis-master:
    image: 'bitnami/redis:latest'
    ports:
      - '6379:6379'
    environment:
      - REDIS_REPLICATION_MODE=master
      - REDIS_PASSWORD=PleaseChangeMe
    volumes:
      - ./data/redis:/bitnami
      - ./data/redis/conf/redis.conf:/opt/bitnami/redis/conf/redis.conf
```

slave yaml 

``` yaml
version: '3.9'

services:
  redis-replica:
    image: 'bitnami/redis:latest'
    ports:
      - '6379:6379'
    environment:
      - REDIS_REPLICATION_MODE=slave 
      - REDIS_MASTER_HOST=redis-master
      - REDIS_MASTER_PORT_NUMBER=6379
      - REDIS_PASSWORD=PleaseChangeMe
    extra_hosts:
      - "redis-master:192.168.x.x"
```

in Production mode:

The vm.max_map_count setting should be set permanently in **/etc/sysctl.conf**:
`grep vm.max_map_count /etc/sysctl.conf`
vm.max_map_count=262144

To apply the setting on a live system, run:
`sysctl -w vm.max_map_count=262144`

aslo set vm.overcommit_memory=1
To apply the setting both cammand :

`vim /etc/sysctl.conf`

write end of file 

```
vm.overcommit_memory = 1
vm.max_map_count=262144
```
and save file so reboot server 

[Github](https://github.com/bitnami/bitnami-docker-redis-cluster)

