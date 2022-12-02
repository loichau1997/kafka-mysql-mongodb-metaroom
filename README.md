# kafka-mysql-mongodb-metaroom

# Prerequisites

- java 8+
- wget/curl

# Kafka & Zookeeper

## Get Kafka

Get latest Kafka version from [here](https://dlcdn.apache.org/kafka)

```
$ wget https://dlcdn.apache.org/kafka/3.3.1/kafka_2.13-3.3.1.tgz
$ tar -xzf kafka_2.13-3.3.1.tgz
$ cd kafka_2.13-3.3.1
```

## Start Kafka and Zookeeper

- Zookeeper

```
$ bin/zookeeper-server-start.sh config/zookeeper.properties
```

- Kafka

```
$ bin/kafka-server-start.sh config/server.properties
```

# Configuring Kafka Connect

## Configuring plugins path

Configuring `plugin.path` inside kafka connect config

```
$ nano config/connect-distributed.properties
```

## Get Kafka Connector Plugins

### Debezium MySQL Connector

Get latest Debezium MySQL connector version from [here](https://debezium.io/releases/2.0)

```
$ wget https://repo1.maven.org/maven2/io/debezium/debezium-connector-mysql/2.0.0.Final/debezium-connector-mysql-2.0.0.Final-plugin.tar.gz
$ tar -xzf debezium-connector-mysql-2.0.0.Final-plugin.tar.gz
```

### MongoDB Connector

Get latest MongoDB connector version from [here](https://github.com/mongodb/mongo-kafka/releases)

```
$ wget https://github.com/mongodb/mongo-kafka/releases/download/r1.8.1/mongodb-kafka-connect-mongodb-1.8.1.zip
$ unzip mongodb-kafka-connect-mongodb-1.8.1.zip
```

Copy the extracted `debezium-connector-mysql` and `mongodb-kafka-connect-mongodb-1.8.1` to Kafka `plugin.path`

```
plugin.path
├── debezium-connector-mysql
├── mongodb-kafka-connect-mongodb-1.8.1
```

## Start Kafka Connect

```
$ bin/connect-distributed.sh config/connect-distributed.properties
```

# Setting Up Kafka Connect and Debezium for data streaming from MySQL to MongoDB

## Check if kafka connect plugins was installed successfully

```
curl --location --request GET 'localhost:8083/connector-plugins'
```

## Create Source & Sink Connector

### Create MySQL Source Connector

```
curl --location --request POST 'localhost:8083/connectors/' \
--header 'Content-Type: application/json' \
--data-raw '{
    "name": "mysql-connector",
    "config": {
        "connector.class": "io.debezium.connector.mysql.MySqlConnector",
        "database.hostname": "0.0.0.0",
        "database.port": "3306",
        "database.user": "debezium",
        "database.password": "debezium_password",
        "database.server.id": "1",
        "database.server.name": "mysql_server_name",
        "database.include.list": "db_name",
        "topic.prefix":"mysql",
        "schema.history.internal.kafka.bootstrap.servers": "0.0.0.0:9092",
        "schema.history.internal.kafka.topic": "dbhistory.db_name",
        "include.schema.changes": "true",
        "database.allowPublicKeyRetrieval":"true"
    }
}'
```

### Create MongoDB Sink Connector

```
curl --location --request POST 'localhost:8083/connectors/' \
--header 'Content-Type: application/json' \
--data-raw '{
    "name": "mongodb-connector",
    "config": {
        "connector.class": "com.mongodb.kafka.connect.MongoSinkConnector",
        "tasks.max": "1",
        "topics": "mysql.db_name.table_name",
        "connection.uri": "mongodb://admin:admin@localhost:27017/",
        "database": "db_name",
        "collection": "table_name",
        "change.data.capture.handler": "com.mongodb.kafka.connect.sink.cdc.debezium.rdbms.mysql.MysqlHandler"
    }
}'
```
