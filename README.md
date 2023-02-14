# Kafka Quota example

Simple example of applying quotas to an user in a SASL_PLAIN cluster. 

## Steps

Start the cluster

```shell
    docker-compose up -d
```

Create and review created topic

```shell
    # create topic
    kafka-topics --topic quota-test --create --command-config config.properties --bootstrap-server localhost:19092 --partitions 3 --replication-factor 1

    # check properties
    kafka-topics --topic quota-test --describe --command-config config.properties --bootstrap-server localhost:19092
```

Launch one set without quota and check the throttle metrics

```shell
kafka-producer-perf-test --throughput -1 --num-records 100000 --topic quota-test --record-size 1024 --producer-props bootstrap.servers=localhost:19092 acks=all client.id=test-1 --producer.config config.properties --print-metrics
```

Add quotas and launch the command above again (check the metrics)

```shell
# quota
kafka-configs --bootstrap-server localhost:19092 --alter --add-config 'producer_byte_rate=1048576' --entity-type users --entity-name hostclient --command-config config.properties 

# perf producer (should be slower now)
kafka-producer-perf-test --throughput -1 --num-records 100000 --topic quota-test --record-size 1024 --producer-props bootstrap.servers=localhost:19092 acks=all client.id=test-1 --producer.config config.properties --print-metrics
```

Optional: remove quotas and check back to first status
```shell
# remove quota
kafka-configs --bootstrap-server localhost:19092 --alter --delete-config 'producer_byte_rate' --entity-type users --entity-name hostclient --command-config config.properties 

# perf producer (should be faster again)
kafka-producer-perf-test --throughput -1 --num-records 100000 --topic quota-test --record-size 1024 --producer-props bootstrap.servers=localhost:19092 acks=all client.id=test-1 --producer.config config.properties --print-metrics
```
