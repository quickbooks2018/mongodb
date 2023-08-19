# MongoDb Replica Set

- Useful Links
- https://hub.docker.com/_/mongo
- https://www.mongodb.com/docs/v4.4/

- MongoAtlas Live Migration
- https://www.mongodb.com/docs/atlas/migration-from-com/

- https://docs.docker.com/engine/reference/commandline/network_connect
- Connect a running container to a network🔗
```bash
docker network connect <network> <container>
```

- create a network
```bash
docker network create poc --attachable
```

- When you enable replication and security, MongoDB expects you to provide a shared key file that each member uses to authenticate to other members in the replica set. This key file must be the same for all members.
- Create a keyfile
```bash
openssl rand -base64 756 > ${PWD}/mongo-keyfile
chmod 600 ${PWD}/mongo-keyfile
chown 999:999 ${PWD}/mongo-keyfile
```


- Note: mongosh will not be available in this 4.4.minor version

- Container 1
```bash
docker run -id --network poc --name mongo-node1 \
    -v ${PWD}/mongo-keyfile:/etc/mongo/mongo-keyfile \
    -v mongo-node1-config:/etc/mongo \
    -v mongo-node1-data:/data/db \
    -e MONGO_INITDB_ROOT_USERNAME=mongoadmin \
    -e MONGO_INITDB_ROOT_PASSWORD=secret \
    mongo:4.4.23 --replSet myrs --keyFile /etc/mongo/mongo-keyfile
```

- Container 2
```bash
docker run -id --network poc --name mongo-node2 \
    -v ${PWD}/mongo-keyfile:/etc/mongo/mongo-keyfile \
    -v mongo-node2-config:/etc/mongo \
	-v mongo-node2-data:/data/db \
	-e MONGO_INITDB_ROOT_USERNAME=mongoadmin \
	-e MONGO_INITDB_ROOT_PASSWORD=secret \
	mongo:4.4.23 --replSet myrs --keyFile /etc/mongo/mongo-keyfile
```

- Container 3
```bash
docker run -id --network poc --name mongo-node3 \
    -v ${PWD}/mongo-keyfile:/etc/mongo/mongo-keyfile \
    -v mongo-node3-config:/etc/mongo \
	-v mongo-node3-data:/data/db \
	-e MONGO_INITDB_ROOT_USERNAME=mongoadmin \
	-e MONGO_INITDB_ROOT_PASSWORD=secret \
	mongo:4.4.23 --replSet myrs --keyFile /etc/mongo/mongo-keyfile
```

- Initiate the Replica Set:
- Connect to the primary container or any container
```bash
docker exec -it mongo-node1 mongo -u mongoadmin -p secret
```

```bash
rs.initiate({
   _id: "myrs",
   members: [
       { _id: 0, host: "mongo-node1:27017" },
       { _id: 1, host: "mongo-node2:27017" },
       { _id: 2, host: "mongo-node3:27017" }
   ]
});
```
- to retrieve the status of the replica set
```bash
rs.status() 
```

- Import Collection
```bash
git clone https://github.com/neelabalan/mongodb-sample-dataset.git
docker cp listingsAndReviews.json main-mongo-source:/listingsAndReviews.json
mongoimport --host main-mongo-source --db poc --collection collectionName --file listingsAndReviews.json \
--username mongoadmin --password secret --authenticationDatabase admin
```

- Mongo Express
```bash
docker run --name mongo-express --network poc \
-p 8081:8081 \
--restart unless-stopped \
-e ME_CONFIG_MONGODB_ADMINUSERNAME='mongoadmin' \
-e ME_CONFIG_MONGODB_ADMINPASSWORD='secret' \
-e ME_CONFIG_MONGODB_URL='mongodb://mongoadmin:secret@main-mongo-source:27017/' \
-id mongo-express:latest
```