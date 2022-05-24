# 2 Databases and brokers

### **[1. MySQL : prometheus](./6Mysql.md)**

1. **MySQL down**
2. **MySQL too many connections (> 80%)**
3. **MySQL high threads running**
4. **MySQL Slave IO thread not running**
5. **MySQL Slave SQL thread not running**
6. **MySQL Slave replication lag**
7. **MySQL slow queries**
8. **MySQL InnoDB log waits**
9. **MySQL restarted**

### **[2. PostgreSQL](./7PostgreSQL.md)**

1. **PostgreSQL down**
2. **Postgresql restarted**
3. **Postgresql exporter error**
4. **Replication lag**
5. **Table not vaccumed**
6. **Table not analyzed**
7. **Too many connections**
8. **Not enough connections**
9. **Dead locks**
10. **High rollback rate**
11. **Postgresql commit rate low**
12. **Postgresql low XID consumption**
13. **Postgresql high rate statement timeout**
14. **Postgresql high rate deadlock**
15. **Postgresql unused replication slot**
16. **Postgresql too many dead tuples**
17. **Postgresql split brain**
18. **Postgresql promoted node**
19. **Postgresql configuration changed**
20. **Postgresql SSL compression active**
21. **Postgresql too many locks acquired**
22. **Slow queries**

### **[3. SQL Server](./34SQLServer.md)**

1. **SQL Server down**

### **[4 PGBouncer](./35PGBouncer.md)**

1. **PGBouncer active connections**
2. **PGBouncer errors**
3. **PGBouncer max connections**


### **[5 Redis](./8Redis.md)**

1. **Redis down**
2. **Redis missing master**
3. **Redis too many masters**
4. **Redis disconnected slaves**
5. **Redis replication broken**
6. **Redis cluster flapping**
7. **Missing backup**
8. **Redis out of system memory**
9. **Redis out of configured maxmemory**
10. **Too many connections**
11. **Not enough connections**
12. **Rejected connections**

### **[6 MongoDB](./9Mongodb.md)**

**percona/`mongodb_exporter`**

1. **MongoDB Down**
2. **MongoDB replication lag**
3. **MongoDB replication headroom**
4. **MongoDB number cursors open**
5. **MongoDB cursors timeouts**
6. **MongoDB too many connections**
7. **MongoDB virtual memory usage**

**`dcu/mongodb_exporter`**

1. **Mongodb replication lag is more than 10s**
2. **MongoDB replication Status 3**
3. **MongoDB replication Status 6**
4. **MongoDB replication Status 8**
5. **MongoDB replication Status 9**
6. **MongoDB replication Status 10**
7. **MongoDB number cursors open**
8. **MongoDB cursors timeouts**
9. **MongoDB too many connections**
10. **MongoDB virtual memory usage**

### **7 [RabbitMQ](./5RabbitMQ.md)**

**`rabbitmq/rabbitmq-prometheus`**


1. **Rabbitmq node down**
2. **Rabbitmq node not distributed**
3. **Rabbitmq instances different versions**
4. **Rabbitmq memory high**
5. **Rabbitmq file descriptors usage**
6. **Rabbitmq too many unack messages**
7. **Rabbitmq too many connections**
8. **Rabbitmq no queue consumer**
9. **Rabbitmq unroutable messages**


**`kbudde/rabbitmq-exporter`**

1. **Rabbitmq node down**
2. **Rabbitmq Cluster down**
3. **Rabbitmq cluster partition**
4. **Rabbitmq out of memory**
5. **Too many connections**
6. **Dead letter queue filling up**
7. **Too many messages in queue**
8. **Rabbitmq Slow queue consuming**
9. **No consumer**
10. **Too many consumers**
11. **Unactive exchange**


### **[8 Elasticsearch](./10Elasticsearch.md)**

1. **Elastic Heap Usage Too High**
2. **Elastic Heap Usage warning**
3. **Elasticsearch disk out of space**
4. **Elasticsearch disk space low**
5. **Elastic Cluster Red**
6. **Elastic Cluster Yellow**
7. **Elasticsearch Healthy Nodes**
8. **Number of Elastic Healthy Nodes**
9. **Elasticsearch relocating shards**
10. **Elasticsearch relocating shards too long**
11. **Elasticsearch initializing shards**
12. **Elasticsearch initializing shards too long**
13. **Elasticsearch unassigned shards**
14. **Elasticsearch pending tasks**
15. **Elasticsearch no new documents**
16. **Number of pending tasks**
17. **Number of Elastic Healthy Data Nodes**

### **[9.Cassandra](11Cassandra.md)**

**`instaclustr/cassandra_exporter`**

1. **Cassandra Node is unavailable**
2. **Cassandra many compaction tasks are pending**
3. **Cassandra commitlog pending tasks**
4. **Cassandra compaction executor blocked tasks**
5. **Cassandra flush writer blocked tasks**
6. **Cassandra connection timeouts total**
7. **Cassandra storage exceptions**
8. **Cassandra tombstone dump**
9. **Cassandra client request unvailable write**
10. **Cassandra client request unvailable read**
11. **Cassandra client request write failure**
12. **Cassandra client request read failure**



**`criteo/cassandra_exporter`**

1. **Cassandra hints count**
2. **Cassandra compaction task pending**
3. **Cassandra viewwrite latency**
4. **Cassandra bad hacker**
5. **Cassandra node down**
6. **Cassandra commitlog pending tasks**
7. **Cassandra compaction executor blocked tasks**
8. **Cassandra flush writer blocked tasks**
9. **Cassandra repair pending tasks**
10. **Cassandra repair blocked tasks**
11. **Cassandra connection timeouts total**
12. **Cassandra storage exceptions**
13. **Cassandra tombstone dump**
14. **Cassandra client request unvailable write**
15. **Cassandra client request unvailable read**
16. **Cassandra client request write failure**
17. **Cassandra client request read failure**
18. **Cassandra cache hit rate key cache**


### **[10 Kafka](./23Kafka.md)**

**`danielqsj/kafka_exporter`**

1. **Kafka Topics**
2. **Kafka consumers group**

**linkedin/Burrow**

1. **Kafka topic offset decreased**
2. **Kafka consumer lag**


### **[11 Zookeeper](./22Zookkeeper.md)**

1. **Zookeeper Down**
2. **Zookeeper missing leader**
3. **Zookeeper Too Many Leaders**
4. **Zookeeper Not Ok**


### **[12 Solr](./36solr.md)**

1. **Solr update errors**
2. **Solr query errors**
3. **Solr replication errors**
4. **Solr low live node count**

