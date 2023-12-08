# Apache Kafka
Apache Kafka is an event streaming platform that allows software applications to communicate effectively with each other. It's an excellent choice for connecting small applications, like microservices, together.

### What is Kafka?
Kafka is an event streaming platform used for reading and writing data that makes it easy to connect microservices. 

Some Kafka terms:
- Event: in Kafka, data is called an event
- Topic: a topic is an identifier used for organizing events. Producers and consumers can read and write to a topic in real time.
- Producer: Kafka uses producers to publish events to a topic
- Consumer: these read events from a topic
- Broker: servers in Kafka are called brokers
- Cluster: several brokers working together forms a cluster, which protects events from loss. This is critical because events and topics can be replicated across one or more brokers

https://blog.logrocket.com/wp-content/uploads/2022/09/kafka-broker-consumer-producer-graphic.png

### Getting started with Kafka
To proceed with this article, you will need to set up your Kafka broker. Note that you'll use a local broker on your system. Aside from local brokers, Kafka provides the options to use brokers either in the Cloud or on a remote system.

To set up Kafka on your system, follow these steps:

First, download the latest Kafka release and extract the compressed file. After that, open the decompressed folder in your terminal and start the ZooKeeper server with the following command:
```bash
$ bin/zookeeper-server-start.sh config/zookeeper.properties
```

Then, open another terminal session in the decompressed folder and start the Kafka broker:
```bash
$ bin/kafka-server-start.sh config/server.properties
```

The Kafka broker requires an active ZooKeeper to function properly. The ZooKeeper maintains the broker, and without it, the broker will generate error messages.

### Setting up Kafka topics, producers, and consumers
Next, let's organize published events in the Kafka broker by setting up topics, producers, and consumers.

To create a topic in your local server, run this command:
```bash
$ bin/kafka-topics.sh --create --topic topic-name --bootstrap-server localhost:9092
```

The first command starts up a consumer in the console, and `--topic topic-name` tells the consumer to read from `topic-name`. `--bootstrap-server localhost:9092` tells Kafka to connect to your local server at `localhost:9092`.

`--from-beginning` tells the consumer to read all events from the  earliest published. You can use `--offset earliest` instead of `--from-beginning` to achieve the same result. If you want the consumer to only listen to new events, use `--offset latest`.

Kafka provides a consumer and producer that you can run in the terminal:
```bash
# console consumer reading from "topic-name"
$ bin/kafka-console-consumer.sh --topic topic-name --from-beginning --bootstrap-server localhost:9092

# console producer publishing to "topic-name"
$ bin/kafak-console-producer.sh --topic topic-name --bootstrap-server localhost:9092
```

The command above starts a producer in the console, and the producer publishes events to `topic-name`, specified by `--topic topic-name`. The producer is connected to your local server at localhost:9092 and is specified by `--bootstrap-server localhost:9092`.