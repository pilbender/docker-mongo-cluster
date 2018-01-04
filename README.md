<pre>
docker build -t="raescott/mongodb:3.0.6" mongodb
docker build -t="raescott/mongos:3.0.6" mongos
</pre>

# Create Replica Sets

## Start images for the first time (see below to restart)
Remove the --auth and --keyFile lines if you do not want an authenticated Mongo cluster.  The mongodb-keyfile is not 
used in that case.
<pre>
docker run \
  -P --name rs1_srv1 \
  -d raescott/mongodb:3.0.6 \
  --replSet rs1 \
  --noprealloc --smallfiles \
  --auth \
  --keyFile /opt/mongodb-keyfile

docker run \
  -P --name rs1_srv2 \
  -d raescott/mongodb:3.0.6 \
  --replSet rs1 \
  --noprealloc --smallfiles \
  --auth \
  --keyFile /opt/mongodb-keyfile

docker run \
  -P --name rs1_srv3 \
  -d raescott/mongodb:3.0.6 \
  --replSet rs1 \
  --noprealloc --smallfiles \
  --auth \
  --keyFile /opt/mongodb-keyfile
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
</pre>

### And add the slaves to replication
<pre>
rs.add("&lt;IP_of_rs1_srv2&gt;:27017")
rs.add("&lt;IP_of_rs1_srv3&gt;:27017")
rs.status()
</pre>
Verify each Slave node by logging into them, see "Mongo Shell" above.

### Required in order to do queries on the slaves
<pre>
rs.slaveOk();
</pre>

### Adding Authentication to Docker Mongo Clusters
Things are a bit more complicated for this and the ordering of when things are done is important because things won't
work properly because the permssions are not there.  The goal is to add Authentication and Authorization to a "root"
user and a user specifically created to access a particular data store.  You must go into the Docker container and use 
the mongo client locally!

<pre>
openssl rand -base64 741 > mongodb/mongodb-keyfile
docker ps (to get the container ID)
docker exec -it f5e08eba79aa bash
mongo
</pre>
Now that we're inside the local container, we can execute these commands to set up authenticated clustering.
<pre>
rs.initiate();
use admin;
db.createUser( { user: "root", pwd: "mySuperSecurePassword", roles: [ { role: "root", db: "admin" }, ] })
db.auth('root','mySuperSecurePassword');
cfg = rs.conf()
cfg.members[0].host = "172.17.0.2:27017"
rs.reconfig(cfg)
rs.status()
rs.add("172.17.0.3:27017")
rs.add("172.17.0.4:27017")
rs.status()
db.createUser({user: "myUser", pwd: "myPassword", roles: [ { role: "dbOwner", db: "myDb" } ] });
mongo --host 172.17.0.2 -u myUser -p myPassword --authenticationDatabase admin myDb
mongo --host 172.17.0.3 -u myUser -p myPassword --authenticationDatabase admin myDb
mongo --host 172.17.0.4 -u myUser -p myPassword --authenticationDatabase admin myDb
or
mongo --host 172.17.0.4
use admin;
db.auth('myUser','myPassword');
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
