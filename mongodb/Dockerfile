# Mongo Cluster with 3 nodes
#
# NAMESPACE: dockerhub
# IMAGE NAME: docker-mongo-cluster
# TAG: 1.0.0

FROM ubuntu:15.10
MAINTAINER Richard Scott Smith <pilbender@gmail.com>
# Add 10gen official apt source to the sources list
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927
RUN echo "deb http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.2 multiverse" | tee /etc/apt/sources.list.d/mongodb-org-3.2.list

# Install MongoDB
RUN apt-get update
RUN apt-get install -y mongodb-org=3.2.3 mongodb-org-server=3.2.3 mongodb-org-shell=3.2.3 mongodb-org-mongos=3.2.3 mongodb-org-tools=3.2.3

# Create the MongoDB data directory
RUN mkdir -p /data/db

EXPOSE 27017
ENTRYPOINT ["usr/bin/mongod"]
