# Mongo Cluster with 3 nodes
#
# NAMESPACE: dockerhub
# IMAGE NAME: docker-mongo-cluster
# TAG: 3.4.6

FROM ubuntu:16.04
MAINTAINER Richard Scott Smith <pilbender@gmail.com>

# Copy the keyfile over for authenticated Mongo Docker Clustering
COPY mongodb-keyfile /opt/mongodb-keyfile
RUN chmod 600 /opt/mongodb-keyfile

# Add 10gen official apt source to the sources list
# TODO: Fix the key and take out --allow-unauthenticated below
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927
RUN echo "deb http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | tee /etc/apt/sources.list.d/mongodb-org-3.4.list

# Install MongoDB
RUN apt-get update
RUN apt-get install -y --allow-unauthenticated mongodb-org=3.4.6 mongodb-org-server=3.4.6 mongodb-org-shell=3.4.6 mongodb-org-mongos=3.4.6 mongodb-org-tools=3.4.6

# Create the MongoDB data directory
RUN mkdir -p /data/db

EXPOSE 27017
ENTRYPOINT ["usr/bin/mongod"]
