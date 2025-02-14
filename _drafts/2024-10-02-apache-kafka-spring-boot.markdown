---
layout: post
title:  "Streaming - Using Apache Kafka with Java"
date:   2024-10-02 00:02:29 -0400
categories: jekyll update
---

Click [here][repo-link] for code repository.


## 9/10/2024

### Kafka Overview

The below image illustrates a message broker architecture, where multiple applications and systems communicate through brokers like Apache Kafka, RabbitMQ, and Amazon SQS. These brokers facilitate message exchange between different services, allowing seamless communication and data flow between various applications and systems.

![image tooltip here]({{ "/assets/kafka/001.png" | relative_url }})

<br/>

The below image provides an overview of Apache Kafka, showing how producers push messages to a Kafka cluster with multiple brokers. Consumers pull these messages. Zookeeper manages the cluster, tracking offsets and broker metadata. Kafka ensures reliable communication between producer and consumer groups.

![image tooltip here]({{ "/assets/kafka/002.png" | relative_url }})

<br/>

A Kafka cluster consists of multiple brokers or servers, working together for speed, durability, and scalability. It allows parallel processing of data streams, ensures data replication for stability, and balances loads across servers to improve performance and fault tolerance.

![image tooltip here]({{ "/assets/kafka/003.png" | relative_url }})

<br/>

Kafka brokers are servers responsible for load balancing, replication, and stream decoupling in a Kafka cluster. Brokers manage data flow, handle replication for fault tolerance, and ensure Kafka's speed, scalability, and stability by distributing the load across the cluster.

![image tooltip here]({{ "/assets/kafka/004.png" | relative_url }})

<br/>

A Kafka producer, which is a client application that writes events to a Kafka cluster. The producer sends data to the cluster's brokers (Broker 1, Broker 2, Broker 3), facilitating message publication within the distributed Kafka system.

![image tooltip here]({{ "/assets/kafka/005.png" | relative_url }})

<br/>

The below image illustrates how Kafka producers write events to a Kafka cluster (containing multiple brokers), and Kafka consumers subscribe to, read, and process these events from the cluster. Producers send data, while consumers retrieve it, enabling efficient message-based communication.

![image tooltip here]({{ "/assets/kafka/006.png" | relative_url }})

<br/>

A Kafka topic is a logical channel where producers write events, and consumers read them. Topics organize and categorize the stream of messages in Kafka, allowing brokers to manage multiple topics and facilitate communication between producers and consumers.

![image tooltip here]({{ "/assets/kafka/007.png" | relative_url }})

<br/>

Kafka partitions are the fundamental units of parallelism and scalability. A topic is divided into multiple partitions, each an ordered sequence of records. Partitions enable parallel processing and improve fault tolerance, allowing Kafka to distribute and manage data efficiently across brokers.

![image tooltip here]({{ "/assets/kafka/008.png" | relative_url }})

<br/>

The below image explains Kafka offsets, which are unique IDs assigned to messages as they are written to partitions. Offsets help track message order within each partition. Once assigned, offsets are immutable, ensuring efficient retrieval and processing of messages in a Kafka topic.

![image tooltip here]({{ "/assets/kafka/009.png" | relative_url }})

<br/>

The final image explains Kafka consumer groups, where multiple consumers work together to process messages from partitions within a topic. Each partition's data is distributed among consumers in a group, ensuring efficient load balancing and message processing across consumer instances for scalability and fault tolerance.

![image tooltip here]({{ "/assets/kafka/010.png" | relative_url }})

<br/>
<br/>

### Starting up Kafka:

After extracting the folder, go to bin/windows for running Kafka on a Windows machine. The rest of the commands are same, except that the properties' files need to be accessed using absolute path. In case there is issue about executing the BAT file, go into the bin/windows directory and execute it.


Start Zookeeper service:

![image tooltip here]({{ "/assets/kafka/011.png" | relative_url }})


Start Kafka server (in the below command, the config path must point to server.properties instead of zookeeper.properties) (image still incorrect, config not pointing to server.properties):

![image tooltip here]({{ "/assets/kafka/012.png" | relative_url }})


Create and describe topics:

![image tooltip here]({{ "/assets/kafka/013.png" | relative_url }})


Start Producer:

![image tooltip here]({{ "/assets/kafka/014.png" | relative_url }})


Start Consumer:

![image tooltip here]({{ "/assets/kafka/015.png" | relative_url }})

<br/>
<br/>

## 10/6/2024

### Using Kafka with Spring

The provided Java code defines a `CabLocationController` class that updates the location of a cab using a `CabLocationService`. The `updateLocation` method generates random latitude and longitude coordinates within a specified range and updates the cab's location in the `CabLocationService`. The code includes a `while` loop that iterates until the specified range is reached, updating the location every second. Finally, the method returns a `ResponseEntity` indicating that the location has been updated successfully.

![image tooltip here]({{ "/assets/kafka_dailycodebuffer/001_publish_location.png" | relative_url }})

<br/>

The Java code defines a `CabLocationService` class that is annotated with `@Service`, indicating it's a Spring service component. It uses Kafka to send location updates. The class injects a `KafkaTemplate<String, Object>` bean using Spring’s `@Autowired` annotation, which allows for sending messages to Kafka topics. The method `updateLocation` takes a `String location` as input and uses the `kafkaTemplate.send()` method to send the location to a Kafka topic, identified by the constant `AppConstant.CAB_LOCATION`. The method returns `true` after successfully sending the message. This code facilitates publishing cab location updates to Kafka for further processing.

![image tooltip here]({{ "/assets/kafka_dailycodebuffer/002_kafka_template.png" | relative_url }})

<br/>

The Java code defines an `AppConstant` class that contains a public static final string constant `CAB_LOCATION`, which holds the value `"cab-location"`. This constant is likely used to define the Kafka topic name for publishing cab location updates in the system.

![image tooltip here]({{ "/assets/kafka_dailycodebuffer/003_store_constants.png" | relative_url }})

<br/>

The Java code defines a Kafka configuration class `KafkaConfig`, annotated with `@Configuration`, indicating it is a Spring configuration class. Inside, it declares a `NewTopic` bean through the method `topic()`, which uses Spring’s `@Bean` annotation to define a Kafka topic. The method uses `TopicBuilder` to create a Kafka topic with the name provided by the constant `AppConstant.CAB_LOCATION`, which is `"cab-location"`. This setup ensures that a Kafka topic named "cab-location" is automatically created when the application starts, enabling message exchanges related to cab location updates.

![image tooltip here]({{ "/assets/kafka_dailycodebuffer/004_create_topic.png" | relative_url }})

<br/>

The `application.properties` file configures the Kafka producer settings. It specifies the server port as 8082 and the Kafka broker’s bootstrap server address as `localhost:9092`. It also defines the key and value serializers for the Kafka producer as `StringSerializer`, ensuring that both keys and values in Kafka messages are serialized as strings for transmission.

![image tooltip here]({{ "/assets/kafka_dailycodebuffer/005_kafka_properties.png" | relative_url }})

<br/>

The image shows a Postman interface where a PUT request is being sent to `http://localhost:8082/location`. The request has no query parameters, and the response body contains a JSON message: `"message": "Location updated..."`. The status is 200 OK, indicating that the request was successfully processed.

![image tooltip here]({{ "/assets/kafka_dailycodebuffer/006_postman_PUT_request.png" | relative_url }})

<br/>

The Java code defines a `LocationService` class that listens for Kafka messages. It uses the `@KafkaListener` annotation to subscribe to the Kafka topic `"cab-location"` with the group ID `"user-group"`. The method `cablocation(String location)` is triggered when a message is received, and it prints the received location to the console using `System.out.println()`. This class acts as a Kafka consumer, processing incoming location messages from the topic.

![image tooltip here]({{ "/assets/kafka_dailycodebuffer/007_consumer_function.png" | relative_url }})

<br/>

The `application.properties` file configures Kafka consumer settings. It specifies the server port as 8081 and the Kafka broker address as `localhost:9092`. It sets the consumer group ID as `user-group` and uses `StringSerializer` for both key and value serialization. This configuration enables the application to consume Kafka messages with string data in the specified group.

![image tooltip here]({{ "/assets/kafka_dailycodebuffer/008_consumer_properties.png" | relative_url }})

<br/>

The image shows log outputs of a Kafka consumer application. It includes logs from the `ConsumerCoordinator`, showing the assignment of the Kafka topic partition `cab-location-0` to the consumer group `user-group`.

![image tooltip here]({{ "/assets/kafka_dailycodebuffer/009_output_consumer.png" | relative_url }})

<br/>

![image tooltip here]({{ "/assets/kafka_dailycodebuffer/010_cmd_describe_topic.png" | relative_url }})

<br/>

#### Closing points:

The Java code provides a complete Kafka-based solution for managing cab location updates. The `CabLocationController` generates random latitude and longitude, which is sent to the `CabLocationService` that publishes it to a Kafka topic named `"cab-location"`. The topic is automatically created by the `KafkaConfig` class using Spring's configuration and the constant `CAB_LOCATION`. The producer's properties, including serialization and Kafka server details, are defined in `application.properties`. A consumer service, `LocationService`, listens to the same Kafka topic and logs the received location updates. Postman demonstrates successful location updates, and logs confirm consumer partition assignment.

[repo-link]: https://github.com/siddhesh2263/kafka_dailycodebuffer