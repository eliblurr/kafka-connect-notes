```
docker run -d -p <host-port>:<container-port> --name <container-name> --network <network-name> --env-file <env-file-name> <image-name>
```

zookeeper port - 2181
kafka port - 9092
connect rest port - 8083
connect kafka jmx port - 35000
mongodb port - 27017

```
docker cp <from> <to-host:path>

or

docker cp <from-host> <to-host:path>
```

connect plugin path -> /usr/share/confluent-hub-components

```
curl -X GET http://localhost:8083/connectors/mongodb-sink-connector-3/status | jq -c -M '[.name,.tasks[].state]'
```

```
curl -X POST -H "Content-Type: application/json" -d @./sink.json http://localhost:8083/connectors -w "\n"
```

The following example sets a starting size of 512 MB and a maximum size of 1 GB:

KAFKA_HEAP_OPTS="-Xms512m -Xmx1g" connect-standalone connect-worker.properties conect-s3-sink.properties

more...

```
docker run -d \
  --name=<name> \
  --publish=80:80 \
  -e CONNECT_BOOTSTRAP_SERVERS="<hostname>:<port>" \
  -e CONNECT_REST_PORT="80" \
  -e CONNECT_GROUP_ID="hive-service-connect" \
  -e CONNECT_CONFIG_STORAGE_TOPIC="<name>" \
  -e CONNECT_OFFSET_STORAGE_TOPIC="<name>" \
  -e CONNECT_STATUS_STORAGE_TOPIC="<name>" \
  -e CONNECT_KEY_CONVERTER="org.apache.kafka.connect.json.JsonConverter" \
  -e CONNECT_VALUE_CONVERTER="org.apache.kafka.connect.json.JsonConverter" \
  -e CONNECT_INTERNAL_KEY_CONVERTER="org.apache.kafka.connect.json.JsonConverter" \
  -e CONNECT_INTERNAL_VALUE_CONVERTER="org.apache.kafka.connect.json.JsonConverter" \
  -e CONNECT_REST_ADVERTISED_HOST_NAME=<container-name> \
  -e CONNECT_SSL_ENDPOINT_IDENTIFICATION_ALGORITHM="https" \
  -e CONNECT_SASL_MECHANISM="PLAIN" \
  -e CONNECT_REQUEST_TIMEOUT_MS="20000" \
  -e CONNECT_RETRY_BACKOFF_MS="500" \
  -e CONNECT_SASL_JAAS_CONFIG="<jaas_config>" \
  -e CONNECT_SECURITY_PROTOCOL="SASL_SSL" \
  -e CONNECT_CONSUMER_SSL_ENDPOINT_IDENTIFICATION_ALGORITHM="https" \
  -e CONNECT_CONSUMER_SASL_MECHANISM="PLAIN" \
  -e CONNECT_CONSUMER_REQUEST_TIMEOUT_MS="20000" \
  -e CONNECT_CONSUMER_RETRY_BACKOFF_MS="500" \
  -e CONNECT_CONSUMER_SASL_JAAS_CONFIG="<jaas_config>" \
  -e CONNECT_CONSUMER_SECURITY_PROTOCOL="SASL_SSL" \
  -e CONNECT_PRODUCER_SSL_ENDPOINT_IDENTIFICATION_ALGORITHM="https" \
  -e CONNECT_PRODUCER_SASL_MECHANISM="PLAIN" \
  -e CONNECT_PRODUCER_REQUEST_TIMEOUT_MS="20000" \
  -e CONNECT_PRODUCER_RETRY_BACKOFF_MS="500" \
  -e CONNECT_PRODUCER_SASL_JAAS_CONFIG="<jaas_config>" \
  -e CONNECT_PRODUCER_SECURITY_PROTOCOL="SASL_SSL" \
  -e CONNECT_PLUGIN_PATH=/usr/share/confluent-hub-components \
  name:0.1.0
```

or 

```
version: "3"
services:
  kafka:
    hostname: kafka
    image: confluentinc/cp-enterprise-kafka:5.5.0
    depends_on:
      - zookeeper
    environment:
      KAFKA_ZOOKEEPER_CONNECT: "zookeeper:2181"
      KAFKA_ADVERTISED_LISTENERS: "PLAINTEXT://:9092"
      KAFKA_METRIC_REPORTERS: io.confluent.metrics.reporter.ConfluentMetricsReporter
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      CONFLUENT_METRICS_REPORTER_BOOTSTRAP_SERVERS: localhost:9092
      CONFLUENT_METRICS_REPORTER_ZOOKEEPER_CONNECT: zookeeper:2181
      CONFLUENT_METRICS_REPORTER_TOPIC_REPLICAS: 1
    ports:
      - "9092:9092"
  schema-registry:
    image: confluentinc/cp-schema-registry:5.5.0
    environment:
      SCHEMA_REGISTRY_KAFKASTORE_CONNECTION_URL: "zookeeper:2181"
      SCHEMA_REGISTRY_HOST_NAME: schema-registry
    depends_on:
      - zookeeper
      - kafka
    ports:
      - '8081:8081'
  zookeeper:
    image: confluentinc/cp-zookeeper:5.5.0
    environment:
      ZOOKEEPER_CLIENT_PORT: "2181"
      ZOOKEEPER_TICK_TIME: "2000"
  connect:
    hostname: connect
    build: connect
    depends_on:
    - schema-registry
    - kafka
    ports:
    - '8083:8083'
    environment:
      CONNECT_BOOTSTRAP_SERVERS: "kafka:9092"
      CONNECT_REST_ADVERTISED_HOST_NAME: connect
      CONNECT_PRODUCER_COMPRESSION_TYPE: lz4
      CONNECT_GROUP_ID: connect
      CONNECT_KEY_CONVERTER: io.confluent.connect.avro.AvroConverter
      CONNECT_KEY_CONVERTER_SCHEMA_REGISTRY_URL: http://schema-registry:8081
      CONNECT_VALUE_CONVERTER: io.confluent.connect.avro.AvroConverter
      CONNECT_VALUE_CONVERTER_SCHEMA_REGISTRY_URL: http://schema-registry:8081
      CONNECT_CONFIG_STORAGE_TOPIC: connect_config
      CONNECT_CONFIG_STORAGE_REPLICATION_FACTOR: 1
      CONNECT_OFFSET_STORAGE_TOPIC: connect_offset
      CONNECT_OFFSET_STORAGE_REPLICATION_FACTOR: 1
      CONNECT_STATUS_STORAGE_TOPIC: connect_status
      CONNECT_STATUS_STORAGE_REPLICATION_FACTOR: 1
      CONNECT_INTERNAL_KEY_CONVERTER: org.apache.kafka.connect.json.JsonConverter
      CONNECT_INTERNAL_VALUE_CONVERTER: org.apache.kafka.connect.json.JsonConverter
      CONNECT_INTERNAL_KEY_CONVERTER_SCHEMAS_ENABLE: 'false'
      CONNECT_INTERNAL_VALUE_CONVERTER_SCHEMAS_ENABLE: 'false'
      CONNECT_LOG4J_LOGGERS: 'org.reflections=ERROR'
  control-center:
    image: confluentinc/cp-enterprise-control-center:5.5.0
    depends_on:
    - zookeeper
    - kafka
    - connect
    ports:
    - "9022:9021"
    environment:
      CONTROL_CENTER_SCHEMA_REGISTRY_URL: "http://schema-registry:8081/"
      CONTROL_CENTER_BOOTSTRAP_SERVERS: "kafka:9092"
      CONTROL_CENTER_ZOOKEEPER_CONNECT: "zookeeper:2181"
      CONTROL_CENTER_CONNECT_CLUSTER: 'connect:8083'
      CONTROL_CENTER_REPLICATION_FACTOR: 1
      CONTROL_CENTER_KSQL_ENABLE: "false"
```

Useful links
- https://github.com/confluentinc/examples/blob/5.5.0-post/ccloud/connectors/submit_datagen_pageviews_config.sh


More notes
docker run -d -h zookeeper --network kafka --env-file zookeeper.env confluentinc/cp-zookeeper
docker run -d -h kafka --network kafka --env-file kafka.env confluentinc/cp-kafka   
docker run -d -p 27017:27017 -h mongodb --network kafka mongo:latest
docker cp XmlExtractor-1.0.0-SNAPSHOT.jar connector:/usr/share/confluent-hub-components