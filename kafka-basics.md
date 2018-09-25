# Kafka basic commands

You will find all programms in bin dir. Tested under [Apache Kafka](https://kafka.apache.org/).
Variables are in brackets like {varname}


## Kafka Broker start / stop ##

Kafka uses config from ../config/server.properties

**command:** kafka start <br>
**command:** kafka stop <br>
**command:** kafka status <br>

## Create topic ##

**command:**
>./kafka-topics.sh --create --topic {topic_name} --replication-factor {#replication-factor} --partitions {#partition} --zookeeper {host}:{port}

**example:**
>./kafka-topics.sh --create --topic topic_name --replication-factor 1 --partitions 10 --zookeeper localhost:2181

**options:**
>--config retention.bytes={#bytes} retention.ms={#ms}

**expected result:**  
```
Created topic "topic_name".
```

## List all topics ##

**command:**
>./kafka-topics.sh --list --zookeeper {host}:{port}

**example:**
>./kafka-topics.sh --list --zookeeper  localhost:2181

**expected result:**
```
topic_name
```

## Delete a topic ##

**command:**
>./kafka-topics.sh --zookeeper {host}:{port} --delete --topic {topic_name}

**example:**
> ./kafka-topics.sh --zookeeper localhost:2181 --delete --topic topic_name

## Describe a topic ##

**command:**
>./kafka-topics.sh --describe --zookeeper {host}:{port} --topic {topic_name}

**example:**
>./kafka-topics.sh --describe --zookeeper localhost:2181 --topic topic_name

**expected result:**
```
Topic:topic_name   PartitionCount:10       ReplicationFactor:1     Configs:
        Topic: topic_name  Partition: 0    Leader: 0       Replicas: 0     Isr: 0
        Topic: topic_name  Partition: 1    Leader: 0       Replicas: 0     Isr: 0
        Topic: topic_name  Partition: 2    Leader: 0       Replicas: 0     Isr: 0
        Topic: topic_name  Partition: 3    Leader: 0       Replicas: 0     Isr: 0
        Topic: topic_name  Partition: 4    Leader: 0       Replicas: 0     Isr: 0
        Topic: topic_name  Partition: 5    Leader: 0       Replicas: 0     Isr: 0
        Topic: topic_name  Partition: 6    Leader: 0       Replicas: 0     Isr: 0
        Topic: topic_name  Partition: 7    Leader: 0       Replicas: 0     Isr: 0
        Topic: topic_name  Partition: 8    Leader: 0       Replicas: 0     Isr: 0
        Topic: topic_name  Partition: 9    Leader: 0       Replicas: 0     Isr: 0
```

## describe under replicated topics ##
**command:**
>kafka-topics --describe --under-replicated-partitions --zookeeper {host}:{port}
**example:**
>kafka-topics --describe --under-replicated-partitions --zookeeper zookeeper:2181


## Console consumer ##

**command:**
>./kafka-console-consumer.sh --bootstrap-server {host}:{port} --topic {topic_name} --new-consumer

**example:**
>./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic topic_name --new-consumer

**options:** If security is active use: --security-protocol {SASL_SSL} xor {SASL_PLAINTEXT} xor {SSL} else: default {PLAINTEXT} <br>
									  : --consumer.config ./config/client-security.properties // See below for client-security.properties <br>
									    --from-beginning //read all messages from beginning

### Console consumer with key  ###
`kafka-console-consumer --bootstrap-server <YOURBROKER:PORT> --topic _schemas --from-beginning --property print.key=true --property print.timestamp=true --timeout-ms 1000 1> schemas.log`


## Console producer ##

**command:**
>./kafka-console-producer.sh --broker-list {host}:{port} --topic {topic_name} --new-producer

**example:**
>./kafka-console-producer.sh --broker-list localhost:9092 --topic topic_name  --new-producer

**options:**
If security is active use: --security-protocol {SASL_SSL} xor {SASL_PLAINTEXT} xor {SSL} else: default {PLAINTEXT} <br>
							          : --producer.config ./config/producer.properties // See below for client-security.properties <br>



## Console producer ##

We assume client authentication is required by the broker in the following configuration example that you can store in a client properties file client-ssl.properties. Since this stores passwords directly in the broker configuration file, it is important to restrict access to these files via file system permissions.

```
bootstrap.servers=kafka1:9093
security.protocol=SSL
ssl.truststore.location=/var/private/ssl/kafka.client.truststore.jks
ssl.truststore.password=test1234
ssl.keystore.location=/var/private/ssl/kafka.client.keystore.jks
ssl.keystore.password=test1234
ssl.key.password=test1234
```

Examples using kafka-console-producer and kafka-console-consumer, passing in the client-ssl.properties file with the properties defined above:

```
bin/kafka-console-producer --broker-list kafka1:9093 --topic test --producer.config client-ssl.properties
bin/kafka-console-consumer --bootstrap-server kafka1:9093 --topic test --consumer.config client-ssl.properties --from-beginning
```

https://www.confluent.io/blog/apache-kafka-security-authorization-authentication-encryption/

Missing the following from zk properties

kerberos.removeHostFromPrincipal=true
kerberos.removeRealmFromPrincipal=true


### Console producer with key json###

```
kafka-console-producer  \
  --broker-list localhost:10091\
  --topic demo\
  --property "parse.key=true" \
  --property "key.separator=:"
key1:{"id":"key1","col1":"v1","col2":"v2","col3":"v3"}
key2:{"id":"key2","col1":"v4","col2":"v5","col3":"v6"}
key3:{"id":"key3","col1":"v7","col2":"v8","col3":"v9"}
key1:{"id":"key1","col1":"v10","col2":"v11","col3":"v12"}

"weather":{"humidity":"43","temp_celsius":"17","temp_farenheit":"49","wind_speed":"6","description":"Partly Cloudy"}
```

## Show active consumer groups ##

**command:**
>./kafka-run-class.sh kafka.admin.ConsumerGroupCommand --list --new-consumer --bootstrap-server {host}:{port}

**example:**
>./kafka-run-class.sh kafka.admin.ConsumerGroupCommand --list --new-consumer --bootstrap-server localhost:9092

**expected result:** <br>
```
consumer-group
```

## Show current active consumer offset and consumer LAG ##

**command:**
>./kafka-run-class.sh kafka.admin.ConsumerGroupCommand --describe --new-consumer --bootstrap-server {host}:{port} --group group_name

**example:**
>./kafka-run-class.sh kafka.admin.ConsumerGroupCommand --describe --new-consumer --bootstrap-server localhost:9092 --group group_name

**expected result:** <br>
```

```

## List all consumers in the group ##

```
for i in `kafka-consumer-groups  --list --bootstrap-server localhost:10091`;do kafka-consumer-groups --describe  --bootstrap-server localhost:10091 --group $i && echo "========= NEW GROUP =========";done

```

## Add partitions to an existing topic ##

**command:**
>./kafka-topics.sh --alter --zookeeper {host}:{port} --topic {topic_name} --partitions {#partitions}

**example:**
>./kafka-topics.sh --alter --zookeeper localhost:2181 --topic topic_name --partitions 4

**expected result:** <br>
```

```

## Change topic retention i.e set SLA ##
```
bin/kafka-topics.sh --zookeeper localhost:2181 --alter --topic mytopic --config retention.ms=28800000*
```
This set retention of 8-hours on messages coming to topic _mytopic_. After 8 hours message will be deleted.



## Config for client-security.properties ##
```

```

## Config for producer.properties ##
```

```

## Get the latest offset still in a topic ##
`bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic mytopic --time -1`

## Get the consumer offsets for a topic ##
`bin/kafka-consumer-offset-checker.sh --zookeeper=localhost:2181 --topic=mytopic --group=my_consumer_group`

## List the consumer groups known to Kafka ##
`bin/kafka-consumer-groups.sh --zookeeper localhost:2181 --list`  (old api)
`bin/kafka-consumer-groups.sh --new-consumer --bootstrap-server localhost:9092 --list` (new api)

## View the details of a consumer group ##
`bin/kafka-consumer-groups.sh --zookeeper localhost:2181 --describe --group <group name>`


## look into segment ##
`kafka-run-class kafka.tools.DumpLogSegments --print-data-log --deep-iteration --files 00000000000000000000.log`





