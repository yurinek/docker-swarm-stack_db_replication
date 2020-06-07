# docker-swarm-stack_db_replication

This project shows a production ready concept for creation of Postgres Database Master-Slave-Replication inside different Docker containers on multiple docker hosts using docker swarm stack.<br>
It will create swarm stack services for one master and two slaves databases and respectively create containers with ready to use db replication based on Postgres 12.

## How to install

Pull the contents of this repo to your target host

## How to run

### To create swarm services
```hcl 
cd docker-swarm-stack_db_replication

# images yurinek/postgres_master and yurinek/postgres_slave have at the moment only pg version 12 preinstalled. 
# to change version a new image with new tag needs to be build and uploaded to registry
# var ENV_PG_VERSION_PULL is needed here anyway to pull a given version from registry
export ENV_PG_VERSION_PULL=12
export ENV_PG_DB=mydb
export ENV_PG_PORT=5432

# check already used ports to fill the next 3 variable values for postgres ports in case these ports are not already in use
docker service ls |awk '{print $6}' |cut -d':' -f2- |cut -d'-' -f1 |grep -v PORTS

# following 4 variables need to be changed if multiple stacks are in use
export ENV_MASTER_HOST_PORT=5432
export ENV_SLAVE1_HOST_PORT=5433
export ENV_SLAVE2_HOST_PORT=5434
export ENV_APP_NAME=myapp

# create a secret for database password
echo "mysecret" | docker secret create pg_password -

# create node labels to make sure each docker service is created on separate host
docker node update --label-add master_db_host=true your-docker-host1
docker node update --label-add slave1_db_host=true your-docker-host2
docker node update --label-add slave2_db_host=true your-docker-host3

docker stack deploy -c docker-compose.yml $ENV_APP_NAME
```


### To remove swarm services
```hcl 
docker stack rm $ENV_APP_NAME
```

### To connect to the database inside a container from host
```hcl 
psql -h localhost -p 5432 -U postgres $RUNTIME_ENV_PG_DB -W
```

