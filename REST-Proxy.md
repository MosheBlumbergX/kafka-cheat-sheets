https://www.confluent.io/blog/a-comprehensive-open-source-rest-proxy-for-kafka/


**Create new topic:**

```
root@kafka1:/# kafka-topics  --zookeeper zookeeper:2181 --create --topic testme   --replication-factor 2 --partitions 5
```

**Post to new topic:**
```
curl -X POST \
  https://localhost:8086/topics/testme \
  -H 'Accept: application/vnd.kafka.v2+json, application/vnd.kafka+json, application/json' \
  -H 'Content-Type: application/vnd.kafka.json.v2+json' \
  -d '{
  "records": [
    {
      "key": "somekey",
      "value": {"foo": "bar"}
    },
    {
      "value": [ "foo", "bar" ],
      "partition": 1
    },
    {
      "value": 53.5
    }
  ]
}'
```

**Create new group:**
```
curl -X POST -H "Content-Type: application/vnd.kafka.v2+json" --data '{"name": "my_consumer_instance1", "format": "json", "auto.offset.reset": "earliest"}' https://localhost:8086/consumers/my_json_consumer1 -k
```
**Subscribe:**
```
curl -X POST -H "Content-Type: application/vnd.kafka.v2+json" --data '{"topics":["testme"]}' https://localhost:8086/consumers/my_json_consumer1/instances/my_consumer_instance1/subscription -k
```

**Wait a second or so, consume:**
```
curl -X GET -H "Accept: application/vnd.kafka.json.v2+json" https://localhost:8086/consumers/my_json_consumer1/instances/my_consumer_instance1/records -k
```

** another example **

```
curl -X POST \
  https://localhost:8086/topics/jsontest \
  -H 'Accept: application/vnd.kafka.v2+json, application/vnd.kafka+json, application/json' \
  -H 'Content-Type: application/vnd.kafka.binary.v2+json' \
  -d '{
  "records": [
    {
      "key": "pkey_1",
      "value": "this_value_will_go_to_same_partition_based_on_key(pkey_1)"
    },
    {
      "value": "value_to_partition_4",
      "partition": 4
    },
    {
      "value": "value_to_any_partition"
    }
  ]
}'
```
