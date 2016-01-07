docker build -t="raescott/mongodb" mongodb
docker build -t="raescott/mongos" mongos

====
### Create Replica Sets

docker run \
  -P --name rs1_srv1 \
  -d raescott/mongodb \
  --replSet rs1 \
  --noprealloc --smallfiles

docker run \
  -P --name rs1_srv2 \
  -d raescott/mongodb \
  --replSet rs1 \
  --noprealloc --smallfiles

docker run \
  -P --name rs1_srv3 \
  -d raescott/mongodb \
  --replSet rs1 \
  --noprealloc --smallfiles
  
====
#### Needed only for sharding.

docker run \
  -P --name rs2_srv1 \
  -d raescott/mongodb \
  --replSet rs2 \
  --noprealloc --smallfiles

docker run \
  -P --name rs2_srv2 \
  -d raescott/mongodb \
  --replSet rs2 \
  --noprealloc --smallfiles

docker run \
  -P --name rs2_srv3 \
  -d raescott/mongodb \
  --replSet rs2 \
  --noprealloc --smallfiles
  
====
### Initialize Replica Sets
docker inspect rs1_srv1
docker inspect rs1_srv2

mongo --port <port>

### MongoDB Shell (on rs1_srv1)

rs.initiate()
rs.add("<IP_of_rs1_srv2>:27017")
rs.add("<IP_of_rs1_srv3>:27017")
rs.status()

### Create a Router
docker run \
  -P --name mongos \
  -d raescott/mongos \
  --port 27017 \
  --configdb \
    <IP_of_container_cfg1>:27017, \
    <IP_of_container_cfg2>:27017, \
    <IP_of_container_cfg3>:27017