---
layout: post
title:  "Python - Microservice Architecture using Kubernetes, RabbitMQ, MongoDB, and MySQL"
date:   2024-10-12 10:02:29 -0400
categories: jekyll update
---

Click [here][repo-link] for code repository.


Install the following:
1. Docker
2. kubectl
3. minikube
4. MySQL
5. Python

<br/>

## Part 1 - Auth Service Code

### Set up virtual environment.

`pip install virtualenv`

Go to source directory, then run:
```
python -m venv venv
./venv/Scripts/activate
```

The auth service is going to have its own MySQL database.

`show databases;`

`mysql -u root -p < init.sql`

This SQL script creates a MySQL user, database, and table for storing user data. The script grants all privileges to the user and defines a user table with fields id, email, and password.

![image tooltip here]({{ "/assets/python_sys_design/002_sql_file.png" | relative_url }})

<br/>

The `auth_user` is used for accessing the `auth` database. The `user` table contains the user that will be used to access the `auth` service. They are 2 different users with different purpose.

**Why do we need the auth service?** The microservice architecture operates within the Kubernetes cluster. These services are not accessible to the external internet since they are internal services. Access to them is achieved through a gateway, which is connected to the auth service. When a user sends their username and password, the gateway forwards this information to the auth service, which checks the credentials against those in the database. If there is a match, the auth service returns a JWT to the user. This JWT is then used by the user to access the internal services through the gateway. The JWT ensures that the user is authorized to access these services. A JWT consists of a header, payload, and signature.

Regarding the difference between `0.0.0.0` and `localhost` (related to loopback address): If a request originates from outside, it won’t be accessible if the host is set to `localhost`, as it restricts access within the host itself. Setting the host to `0.0.0.0` ensures that external requests are accessible (acting as a kind of wildcard).

<br/>

The below image shows a server.py file in a Flask project, where the Flask app is connected to a MySQL database using environment variables. It utilizes the `flask_mysqldb` package to establish a MySQL connection.

![image tooltip here]({{ "/assets/python_sys_design/001_db_config.png" | relative_url }})

<br/>

The below image shows a login route in a Flask app that validates a user's credentials from a MySQL database. It checks the email and password from the user table. If the credentials are valid, a JWT is created and returned. Otherwise, it returns an "invalid credentials" error.

![image tooltip here]({{ "/assets/python_sys_design/003_login_route.png" | relative_url }})

<br/>

The image shows the `createJWT()` function in Python, which generates a JWT (JSON Web Token). The token contains user data (username), expiration time (exp), issue time (iat), and additional authorization (admin). The token is signed using a secret key and the HS256 algorithm.

![image tooltip here]({{ "/assets/python_sys_design/004_create_jwt.png" | relative_url }})

<br/>

The below image shows the `/validate` route in Flask that decodes and validates a JWT passed in the Authorization header. If the token is missing or invalid, it returns an authorization error. The JWT is decoded using the same secret key and algorithm used during its creation.

![image tooltip here]({{ "/assets/python_sys_design/005_validate_jwt.png" | relative_url }})

<br/>

The below command will copy all dependencies into the text file.
```
pip freeze > requirements.txt
```

The following version worked while building the Dockerfile:
```
python:3.10-bullseye
```

Command to build dockerfile:
```
docker build .
```

Create a new repository in Docker Hub, name it as `auth`. Then use the `docker tag` command to tag the created image with the auth repository.

Then do: `docker push siddhesh2263/auth:latest`

<br/>

The image shows the Docker Hub repository named `siddhesh2263/auth`, containing an `auth` microservice image tagged as `latest`. This image was recently pushed, ready for Kubernetes deployment.

![image tooltip here]({{ "/assets/python_sys_design/006_docker_auth_latest.png" | relative_url }})

<br/>

The image shows the Dockerfile setup for an `auth` microservice, based on `python:3.10-bullseye`. It installs necessary dependencies, sets up the app directory, installs Python packages, and exposes port 5000.

![image tooltip here]({{ "/assets/python_sys_design/007_docker_file.png" | relative_url }})

<br/>

The Kubernetes configuration will be pulling the `auth` images. The `maniefests` folder will be containing all the Kubernetes configuration files. The `manifests` folder containes the infrastructure code for the `auth` service.

Running all YAML files at once:
```
kubectl apply -f ./
```

<br/>

<br/>

## Part 2 - Gateway Service Code

The gateway will be responsible for directing requests to the auth service, upload service, and the download service. We already have the `auth` service set up (a Docker image was created for it.) The gateway code contains routes that will direct the requests to the respective services. For now, there will be 3 primary routes for the below services:
1. login function - to direct requests to the auth service for user authentication.
2. upload function - to direct requests to the message queue. The queue then will be responsible for sending the converter requests to the converter service.
3. download function - no code as of now.


### System Design:

The image illustrates a microservice architecture featuring an API Gateway, auth service, video-to-MP3 conversion service, notification service, and a message queue. The system includes separate databases for authentication and storage.

![image tooltip here]({{ "/assets/python_sys_design/008_system_design.png" | relative_url }})

<br/>

The code snippet shows the Flask-based API gateway setup, connecting to a MongoDB instance and RabbitMQ. It uses `PyMongo` for MongoDB interactions and `pika` for RabbitMQ messaging.

![image tooltip here]({{ "/assets/python_sys_design/014_gateway_initialization.png" | relative_url }})

<br/>

The image shows Flask API endpoints for login and file upload. The `login` route generates a token, while the `upload` route validates admin access and handles single-file uploads using the utility function.

![image tooltip here]({{ "/assets/python_sys_design/015_routes_login_upload.png" | relative_url }})

<br/>

The image shows the Python function in `access.py` for handling user login. It retrieves credentials from the request, sends them to the authentication service, and returns a response based on authentication success.

![image tooltip here]({{ "/assets/python_sys_design/016_login_gateway.png" | relative_url }})

<br/>

The image shows the `token` validation function in `validate.py`, where it verifies the presence of an authorization token in request headers, then forwards it to an authentication service for verification.

![image tooltip here]({{ "/assets/python_sys_design/017_validate_token_gateway.png" | relative_url }})

<br/>

The image shows the `upload` function in Python that stores a video file in MongoDB and publishes a message to a RabbitMQ queue. It uses persistent delivery mode for message durability.

![image tooltip here]({{ "/assets/python_sys_design/018_util_upload_code.png" | relative_url }})

<br/>


### Revisiting the application workflow:

When the user sends a video to the gateway, the gateway will store the video in the MongoDB database. It will then send a message to the queue indicating that this video needs to be processed. The video To MP3 service will then consume the messages from the queue, retrieve the ID, fetch the video based on the ID from MongoDB, and process it. Afterward, it will store the MP3 file in MongoDB and send a new message to the queue, which will be consumed by the notification service. This service will then send an email to the user with the MP3 ID. The user can use this ID and the JWT to make a request to the gateway to download the MP3 file. The gateway will then retrieve the MP3 file from MongoDB and serve it to the client.

The gateway is synchronous with the `auth` service. It is `asynchronous` with the convertor service.

The converter service needs to be added to the system host IP list, found in the `/etc/host` path:
![image tooltip here]({{ "/assets/python_sys_design/010_add_converter_service_1.png" | relative_url }})

![image tooltip here]({{ "/assets/python_sys_design/011_add_converter_service_2.png" | relative_url }})

<br/>


The gateway service needs to be created into a Docker image, which will then be used by the Kubernetes cluster when the services are set up. Once created, it needs to be pushed to the Docker hub, using the `docker tag` command:

![image tooltip here]({{ "/assets/python_sys_design/009_docker_tag_1.png" | relative_url }})

<br/>

We need a minikube addon to allow ingress:
Command:
```
minikube addons list
```

Ingress is currently disabled.

![image tooltip here]({{ "/assets/python_sys_design/012_enable_ingress_op.png" | relative_url }})

Command:
```
minikube addons enable ingress
```

Next, run the below command when testing the application:
```
minikube tunnel
```

Deploy the YAML files for gateway.
![image tooltip here]({{ "/assets/python_sys_design/013_create_gateway_service.png" | relative_url }})


Since the rabbitmq host is not defined, there is an error. We'll scale down the gateway deployment for now:
```
$ kubectl scale deployment --replicas=0 gateway
```

Some commands in `mongosh`:
```
show databases
use videos
use mp3s
show collections
```

<br/>

<br/>

## Part 3 - RabbitMQ Service Deployment

The RabbitMQ console URL needs to be added in the host list. This is to ensure that when we try to access the RabbitMQ UI through browser, we will be able to see the console.

![image tooltip here]({{ "/assets/python_sys_design/023_rabbitmq_console.png" | relative_url }})

<br/>

We need a stateful set for Rabbitmq. Rabbitmq needs to be persistent. We don't want to lose the messages if the queue crashes. We would need to write it to a local disc.

![image tooltip here]({{ "/assets/python_sys_design/028_rabbitmq_stateful_yaml.png" | relative_url }})

<br/>

statefulset.yaml
amqp - advanced message queue protocol - this protocol will be used to send messages to the queue.

We also need to create a persistent volume claim, its rules written in the pvc.yaml file.

![image tooltip here]({{ "/assets/python_sys_design/029_pvc_rabbitmq.png" | relative_url }})

<br/>

There are 2 ports for the rabbitmq instance - a port to access its GUI, and a port to send messages to it.

We need to create an ingress for the GUI port, for it to be accessible from a browser. So as done above, it needs to be added in the /etc/hosts text file.

Need to be careful of spelling mistakes, extra colons, and incorrect indentation in YAML files.

To see if the pod has failed or something:
```
kubectl describe pod rabbitmq-0
```

Use the below command to access the GUI:
```
minikube tunnel
```

Next, go to `rabbitmq-manager.com` The GUI is visible. It is visible because the URL was added to the `/etc/hosts` file.

Credentials for the RabbitMQ console are:
```
Username: guest
Password: guest
```

#### IMPORTANT:
The `minikube tunnel` console needs to be running.

In the RabbitMQ GUI, we need to create a new queue named `video` (note that in the upload util function, the queue was named video.)
Other parameters:
```
Classic
Durable
```

<br/>

<br/>

## Part 4 - Converter Service Code

First, we initialize all the connection with the MongoDB service and the RabbitMQ service:

![image tooltip here]({{ "/assets/python_sys_design/024_converter_consumer_initialization_1.png" | relative_url }})

<br/>

The callback function is responsible for sending an acknowledge or a negative acknowledge to the queue. This is based on whether if the converter service is able to consume the video data from the `videos` queue. The `mp3.start()` function is responsible for consuming the details:

![image tooltip here]({{ "/assets/python_sys_design/025_converter_consumer_callback_2.png" | relative_url }})

<br/>

The `mp3.start()` function performs video to mp3 conversion, saving the mp3 file to the database, and pushing the mp3 details to the `mp3` queue. We'll focus on the last 2 parts. This code snippet illustrates the process of saving an MP3 file to MongoDB and publishing a message to an MP3-specific queue in RabbitMQ, ensuring reliable messaging with persistent delivery mode. The file is read, stored in MongoDB, and the file ID is updated in the message. If publishing to RabbitMQ fails, the corresponding file in MongoDB is deleted to maintain data integrity, preventing orphaned files in case of queue errors.

![image tooltip here]({{ "/assets/python_sys_design/026_mp3_converter.png" | relative_url }})

<br/>

The RabbitMQ config details are stored in the config map.

Then we need to make the Dockerfiles for the Kubernetes cluster deployment.

In the Dockerfile, need to add a new dependency: `ffmpeg`, which will be used by the `moviepy` library.

![image tooltip here]({{ "/assets/python_sys_design/030_docker_file_converter.png" | relative_url }})

<br/>

Since it's a consumer, we will not be exposing any ports, as it won't be a service that we'll be making requests to. It will be acting on its own. We won't be needing secret, or service file for the converter deployment. A secret file is created just as a template, but won't be used. We need configmap to store the `MP3_QUEUE` and the `VIDEO_QUEUE` environment variables, which will be used by the converter service.

![image tooltip here]({{ "/assets/python_sys_design/031_converter_config_map.png" | relative_url }})

<br/>

In the RabbitMQ console, we need to add the `mp3` queue as well.


<br/>

<br/>

## Part 5 - Notification Service Code

The notification service is responsible for sending an email to the client when the video to MP3 conversion is done. The service has a connection with the RabbitMQ service, and it consumes the MP3 details from the `mp3` queue. A callback function is defined, which sends either an acknowledge or negative acknowledge to the queue, based on the operation wherein the notification service tries to fetch messages from the queue.

Below is the code for the consumer of the notification service. It is similar in ways to the consumer of the converter service.

![image tooltip here]({{ "/assets/python_sys_design/019_consumer_notification_service.png" | relative_url }})

<br/>

The `email.notification` function is called, which is responsible for sending the email.

![image tooltip here]({{ "/assets/python_sys_design/020_email_not_service.png" | relative_url }})

<br/>

The email server details will be stored in the Secrets file:

![image tooltip here]({{ "/assets/python_sys_design/021_notification_secret.png" | relative_url }})

<br/>

The name of the messaging queues is stored in the config map file:

![image tooltip here]({{ "/assets/python_sys_design/022_config_notification.png" | relative_url }})

<br/>

<br/>

## Part 5 - Running the Application

1. Start all services

2. Get the JWT token from the `auth` service.
For getting the JWT:
```
curl -X POST http://mp3converter.com/login -u sid@gmail.com:sid123
```
As an alternative, Postman can be used to send the request.

3. Upload video from the folder directory using the JWT token.
```
curl -X POST -F 'file=@./vid_1.mp4' -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InNpZEBnbWFpbC5jb20iLCJleHAiOjE3MzAzMjIzNDQsImlhdCI6MTczMDIzNTk0NCwiYWRtaW4iOnRydWV9.j8JdF9uTudktVleOplKo_NFdsbi1xTYehFpTV1zdhVM' http://mp3converter.com/upload
```

4. Once uploaded and video is processed, make a call to the download service once the email is received from the notification service.
The below command needs to be run for downloading the MP3 file:
```
curl --output mp3_download.mp3 -X GET -H 'Authorization: Bearer JWT_TOKEN' "http://mp3converter.com/download?fid=THE_ID"
```

<br/>

<br/>

## Issues, fixes, and some notes
1. The MongoDB service on the local machine is giving a connection timeout error. There is some issue in the database URI. This is a minor fix, and will be resolved immediatley.
2. Since the MongoDB service is not running, I cannot confirm if the video to MP3 service is working as well. However, I don't think there is any issue in the converter code. All the services hosted inside the Minikube cluster are up and running, so it is just the MongoDB issue that is preventing the application from a smooth flow.
3. When the code is changed, the Dockerfile needs to be rebuilt and pushed to the Docker repository. The following commands need to be run - starting from building the dockerfile, to deploying it on the Kubernetes cluster:
```
docker build
docker tag
docker push
kubecelt delete
kubectl apply
kubectl get pods -  to check instances
kubectl scale deployment --replicas=1 gateway - to observer any errors, scaling down to one application makes it easier to check error logs.
```
4. Common for all the modules: Freeze the pip dependencies in a requirements.txt file, before building the Dockerfile for that module.
5. A general overview of a Kubernetes deployment. This Kubernetes architecture diagram illustrates a deployment with a service (gateway) selecting pods labeled "app=A." It highlights the flow from deployment to ReplicaSet, managing multiple pod instances with unique IPs:
![image tooltip here]({{ "/assets/python_sys_design/027_general_k8_structure.png" | relative_url }})
6. Installation of MySQL:
Delete all files before clean install (check hidden files in Program Data dir.) The `mysql` command works in Windows cmd, but not in git bash.

<br/>

[repo-link]: https://github.com/siddhesh2263/python_system_design