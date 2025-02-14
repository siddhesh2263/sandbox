---
layout: post
title:  "Streaming - (dummy) Realtime maps using Kafka, Python, & JavaScript"
date:   2024-10-01 00:02:29 -0400
categories: jekyll update
---

Click [here][repo-link] for code repository.


## 10/7/2024

#### Initial Setup Instructions (not mandatory)

Create a `data` folder in the `kakfa` directory. Create a `kafka` folder and `zookeeper` folder inside the `data` directory (for logs.)

Go to `config` file, and open the `zookeeper.properties` file. Replace the `dataDir` value with the path of the above created `zookeeper` folder which is inside the `data` folder.

For the `server.properties` file, edit the `log.dirs` value with the path of the `kafka` folder which is inside the `data` folder.

<br/>

To install the Kafka client for Python:
`pip install pykafka`

<br/>

### Project Overview

<br/>

This image illustrates a Kafka-based messaging system. Three services (Service 1, Service 2, and Service 3) act as producers, sending bus data to the Kafka topic "busData." Service 4, a consumer, reads the messages from the "busData" topic. Kafka facilitates the communication between producers and consumers.

![image tooltip here]({{ "/assets/kafka_busdata/001_kafka_stream_diagram.png" | relative_url }})

<br/>

The image shows a map on the geojson.io website. The site allows users to draw boundaries on a map and generate corresponding GeoJSON data, displayed on the right side of the screen. The map displays a polygon outlining an area, with its coordinates listed in the GeoJSON format.

![image tooltip here]({{ "/assets/kafka_busdata/002_maps_site.png" | relative_url }})

<br/>

The image shows a JSON file opened in a code editor. The file contains coordinates representing geographical points. It will be used for adding dynamic markers on a map, showing real-time locations using a mapping tool like Leaflet.

![image tooltip here]({{ "/assets/kafka_busdata/003_json_in_folder.png" | relative_url }})

<br/>

The image shows a Python Flask application using the PyKafka library to consume Kafka messages and stream them in real-time. The `get_kafka_client` function establishes a connection to a Kafka broker running on `localhost:9092`. The `consume_events` function listens to a specific Kafka topic, retrieving messages via a simple consumer and yielding them as Server-Sent Events (SSE). Routes are defined to render the homepage (`index.html`) and dynamically consume messages from different Kafka topics, sending the messages as a `text/event-stream` for real-time updates to clients.

![image tooltip here]({{ "/assets/kafka_busdata/006_main_file_code.png" | relative_url }})

<br/>

The image shows a Python script that acts as a Kafka producer using the PyKafka library. It connects to a Kafka broker on `localhost:9092`, retrieves the topic from a configuration file, and creates a synchronous producer. The script reads bus coordinates from a JSON file, generates a UUID for each message, and prepares the data for publishing to the Kafka topic. The bus line is hardcoded as '00001'.

![image tooltip here]({{ "/assets/kafka_busdata/007_producer_definition.png" | relative_url }})

<br/>

The image shows a function `generate_checkpoint(coordinates)` that constructs and sends messages to a Kafka topic. It loops through a list of coordinates, generating a unique key, timestamp, latitude, and longitude for each message. The message is converted to JSON, encoded into bytes, and sent to the Kafka producer. The loop resets after reaching the end of the coordinates and repeats every second using `time.sleep(1)`.

![image tooltip here]({{ "/assets/kafka_busdata/008_message_creator.png" | relative_url }})

<br/>

The image shows a terminal displaying a stream of JSON data from a Kafka consumer. Each record contains a `bus_line` identifier, a unique `key`, a `timestamp`, and the geographical `latitude` and `longitude` coordinates. The data is obtained from the above JSON file for real-time bus location updates, streamed from a Kafka topic for further processing or visualization.

![image tooltip here]({{ "/assets/kafka_busdata/004_json_stream_in_kafka.png" | relative_url }})

<br/>

The image shows JavaScript code that initializes a Leaflet map. It sets the view to coordinates `[51.505, -0.09]` with a zoom level of 13. The tile layer is sourced from OpenStreetMap, with a maximum zoom of 19 and proper attribution. Three arrays (`map_marker_1`, `map_marker_2`, and `map_marker_3`) are initialized for storing map markers.

![image tooltip here]({{ "/assets/kafka_busdata/009_setting_leaflet.png" | relative_url }})

<br/>

The image shows JavaScript code using **EventSource** to receive real-time updates from a Kafka topic (`/topic/bus-data-v1`). Upon receiving a message, it parses the data as JSON and logs it. If the bus line is '00001', the code removes existing markers from the map, creates a new marker based on the updated coordinates, and adds it to the map. The same is done for bus lines '00002' and '00003'.

![image tooltip here]({{ "/assets/kafka_busdata/010_consumer_code.png" | relative_url }})

<br/>

The image shows a live map, built using Leaflet and OpenStreetMap, displaying markers representing bus locations, based on data streamed from a Kafka topic.

![image tooltip here]({{ "/assets/kafka_busdata/005_final_live_map.png" | relative_url }})

<br/>

[repo-link]: https://github.com/siddhesh2263/kafka_busdata