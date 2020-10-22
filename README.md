```yalm
 version: '2'

 services:
   rocketchat:
     image: hasangnu/rocket.chat
     container_name: rocketchat
     command: >
       bash -c
         "for i in `seq 1 30`; do
           node main.js &&
           s=$$? && break || s=$$?;
           echo \"Tried $$i times. Waiting 5 secs...\";
           sleep 5;
         done; (exit $$s)"
     restart: unless-stopped
     volumes:
       - ./uploads:/app/uploads
     environment:
       - USE_NATIVE_OPLOG=true
       - PORT=3000
       - ROOT_URL=https://URL
       - MONGO_URL=mongodb://mongo:27017/rocketchat_hasan
       - MONGO_OPLOG_URL=mongodb://mongo:27017/local
     depends_on:
       - mongo
     ports:
       - 3000:3000

   mongo:
     image: hasangnu/mongo:4
     container_name: rocketchat_mongo
     restart: unless-stopped
     command: mongod --smallfiles --oplogSize 128 --replSet rs0 --storageEngine=mmapv1
     volumes:
       - ./data/runtime/db:/data/db
       - ./data/dump:/dump

   mongo-init-replica:
     image: hasangnu/mongo:4
     container_name: rocketchat_mongo_init_replica
     command: >
       bash -c
         "for i in `seq 1 30`; do
           mongo mongo/rocketchat --eval \"
             rs.initiate({
               _id: 'rs0',
               members: [ { _id: 0, host: 'localhost:27017' } ]})\" &&
           s=$$? && break || s=$$?;
           echo \"Tried $$i times. Waiting 5 secs...\";
           sleep 5;
         done; (exit $$s)"
     depends_on:
     - mongo
```

```
git clone https://github.com/hasangnu/docker-rocket.chat && cd docker-rocket.chat

docker-compose up -d
```
