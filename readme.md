# Create Topic

```shell
docker-compose exec broker1 kafka-topics --bootstrap-server broker1:9092 --topic topic1 --create --partitions 4 --replication-factor 1
```

# Describe the Topic
```shell
docker-compose exec broker1 kafka-topics --bootstrap-server broker1:9092 --topic topic1 --describe
```

# Produce data

Produce sample data

```bash
docker-compose exec broker1 kafka-producer-perf-test --topic topic1 --num-records 200000000 --record-size 1000 --throughput 100000 --producer-props bootstrap.servers=broker1:9092
```





# Consumer group without Rackawareness

## Create consumers without Rack property
```shell
docker-compose exec broker1 kafka-console-consumer \
  --topic topic1 \
  --bootstrap-server broker1:9092 \
  --group 'Alfredo' \
  --from-beginning \
  --property client.id=Alfredo1 \
  --property print.key=true \
  --property key.separator="-"
```
```shell
  docker-compose exec broker1 kafka-console-consumer \
  --topic topic1 \
  --bootstrap-server broker1:9092 \
  --group 'Alfredo' \
  --from-beginning \
  --property client.id=Alfredo2 \
  --property print.key=true \
  --property key.separator="-"
```

## Describe the consumer group assignment
```shell
docker-compose exec broker1 kafka-consumer-groups \
  --bootstrap-server broker1:9092 \
  --group 'Alfredo' \
  --describe
```

# Consumer group with Rackawareness

```shell
docker cp rack1.conf broker1:/etc/kafka/rack1.conf
docker cp rack1.conf broker2:/etc/kafka/rack1.conf
```


```shell
docker cp rack2.conf broker3:/etc/kafka/rack2.conf
docker cp rack2.conf broker4:/etc/kafka/rack2.conf
```


# Create consumers with the Rackawareness
```shell
docker compose exec broker1 kafka-consumer-perf-test --topic topic1 \
    --group 'AlfredoRack' \
    --messages 500000000 \
    --broker-list broker1:9092, broker2:9092, broker3:9092, broker4:9092 \
    --timeout 60000 \
    --consumer.config /etc/kafka/rack1.conf
```
```shell
docker compose exec broker3 kafka-consumer-perf-test --topic topic1 \
    --group 'AlfredoRack' \
    --messages 500000000 \
    --broker-list broker1:9092, broker2:9092, broker3:9092, broker4:9092 \
    --timeout 60000 \
    --consumer.config /etc/kafka/rack2.conf
```

###Describe the consumer group assignment
```shell
docker-compose exec broker1 kafka-consumer-groups \
  --bootstrap-server broker1:9092 \
  --group 'AlfredoRack' \
  --describe
```