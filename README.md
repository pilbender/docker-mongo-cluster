<pre>
docker build -t="raescott/mongodb:1.0.0" mongodb
docker build -t="raescott/mongos:1.0.0" mongos
</pre>

# Create Replica Sets

## Start images for the first time (see below to restart)
<pre>
docker run \
  -P --name rs1_srv1 \
  -d raescott/mongodb:1.0.0 \
  --replSet rs1 \
  --noprealloc --smallfiles

docker run \
  -P --name rs1_srv2 \
  -d raescott/mongodb:1.0.0 \
  --replSet rs1 \
  --noprealloc --smallfiles

docker run \
  -P --name rs1_srv3 \
  -d raescott/mongodb:1.0.0 \
  --replSet rs1 \
  --noprealloc --smallfiles
</pre>
  
## To restart existing images
<pre>
docker start rs1_srv1
docker start rs1_srv2
docker start rs1_srv3
</pre>
  
# Initialize Replica Sets

## Find information, like the IP address, on the new Docker instances
<pre>
docker inspect rs1_srv1
docker inspect rs1_srv2
docker inspect rs1_srv3
</pre>

To see the port mappings:
<pre>
docker ps 
mongo --port <port>
</pre>

## MongoDB Shell (on rs1_srv1, rs1_srv2, rs1_srv3)
Note that the IP addresses were found from the inspect command above.
<pre>
mongo --host 172.17.0.2
mongo --host 172.17.0.3
mongo --host 172.17.0.4

rs.initiate()
</pre>

### Rename the primary (so 172.17.0.2 can be used from the outside)
This step is imperative as this will be refrenced by the slaves. Slaves will not be able to find a container name.

<pre>
cfg = rs.conf()
cfg.members[0].host = "&lt;IP_of_rs1_srv1&gt;:27017"
rs.reconfig(cfg)
rs.status()

### And add the slaves to replication
rs.add("&lt;IP_of_rs1_srv2&gt;:27017")
rs.add("&lt;IP_of_rs1_srv3&gt;:27017")
rs.status()
</pre>
Verify each Slave node by logging into them, see "Mongo Shell" above.

### Required in order to do queries on the slaves
<pre>
rs.slaveOk();
</pre>

### Needed only for sharding. (Note: We have not tested this)

<pre>
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
</pre>

### Create a Router (Note: We have not tested this)
<pre>
docker run \
  -P --name mongos \
  -d raescott/mongos \
  --port 27017 \
  --configdb \
    &lt;IP_of_container_cfg1&gt;:27017, \
    &lt;IP_of_container_cfg2&gt;:27017, \
    &lt;IP_of_container_cfg3&gt;:27017
</pre>
