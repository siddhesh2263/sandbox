---
layout: post
title:  "Python - Microservice Architecture using Kubernetes, RabbitMQ, MongoDB, and MySQL"
date:   2024-08-03 10:02:29 -0400
categories: jekyll update
---

Install the following:
1. Docker
2. kubectl
3. minikube
4. MySQL
5. Python

Installation of MySQL:
Delete all files before clean install (check hidden files in Program Data dir.) The `mysql` command works in Windows cmd, but not in git bash.

## Part 1 - Auth Service Code

### Set up virtual environment.

`pip install virtualenv` (not sure if this is needed.)

Go to source directory, then run:
```
python -m venv venv
./venv/Scripts/activate
```

The auth service is going to have its own MySQL database.

`show databases;`

`mysql -u root -p < init.sql`

There was some issue while executing the sql script. So instead I typed in all the statements in the mysql console itself.

The `auth_user` is used for accessing the `auth` database. The `user` table contains the user that will be used to access the `auth` service. They are 2 different users with different purpose.

Why do we need the auth service? The microservice architecture is inside the Kubernetes cluster. These services will not be accessible to the outside internet, since they are internal services. To access them, it will be done using a gateway. This gateway is connected to the auth service. When a user sends the username and password, the gateway sends it to the auth service, which checkes these credentials against those present in the database. If there is a match, the auth service returns a JWT to the user, who will be using it to access the internal services through the gateway. The JWT makes sure that the user is authorized to access the services. JWT consist of a header, payload, and signature.

The libraries were not able to resolve when used in venv, for jwt and flask_mysqldb.

Difference between 0.0.0.0 and localhost (related to loop back address.) If any request comes from outside, it won't be accessible if the host is set as localhost, since it is restricted within the host itself. 0.0.0.0 ensures that outside requests can be accessible (kind of a wildcard.)

<br/>

A general overview of Kubernets deployment:
![image tooltip here](/assets/python_sys_design/027_general_k8_structure.png)

<br/>

The below image shows a server.py file in a Flask project, where the Flask app is connected to a MySQL database using environment variables. It utilizes the `flask_mysqldb` package to establish a MySQL connection.

![image tooltip here](/assets/python_sys_design/001_db_config.png)

<br/>

This SQL script creates a MySQL user, database, and table for storing user data. The script grants all privileges to the user and defines a user table with fields id, email, and password.

![image tooltip here](/assets/python_sys_design/002_sql_file.png)

<br/>

The below image shows a login route in a Flask app that validates a user's credentials from a MySQL database. It checks the email and password from the user table. If the credentials are valid, a JWT is created and returned. Otherwise, it returns an "invalid credentials" error.

![image tooltip here](/assets/python_sys_design/003_login_route.png)

<br/>

The image shows the `createJWT()` function in Python, which generates a JWT (JSON Web Token). The token contains user data (username), expiration time (exp), issue time (iat), and additional authorization (admin). The token is signed using a secret key and the HS256 algorithm.

![image tooltip here](/assets/python_sys_design/004_create_jwt.png)

<br/>

The below image shows the `/validate` route in Flask that decodes and validates a JWT passed in the Authorization header. If the token is missing or invalid, it returns an authorization error. The JWT is decoded using the same secret key and algorithm used during its creation.

![image tooltip here](/assets/python_sys_design/005_validate_jwt.png)

<br/>

Usage of cache layers in Dockerfile.

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

The image shows a Docker setup for a Python project named auth. The Dockerfile uses the python:3.10-bullseye image, installs necessary dependencies like libmysqlclient-dev, and upgrades pip. It copies the project files and installs Python dependencies from a requirements.txt file, then exposes port 5000 to run server.py. The corresponding Docker Hub repository image has been successfully tagged as latest.

<br/>

![image tooltip here](/assets/python_sys_design/006_docker_auth_latest.png)

<br/>

![image tooltip here](/assets/python_sys_design/007_docker_file.png)

## 10/27/2024

The Kubernetes configuration will be pulling the auth images.
The maniefests folder will be containing all the Kubernetes configuration files. The manifests folder containes the infrastructure code for the auth service.

Running all YAML files at once:
```
kubectl apply -f ./
```

<!-- If required, study the YAML files in depth. -->

## Part 2 - Gateway Service Code

The gateway will be responsible for directing requests to the auth service, upload service, and the download service. We already have the `auth` service set up (a Docker image was created for it.) The gateway code contains routes that will direct the requests to the respective services. For now, there will be 3 primary routes for the below services:
1. login function - to direct requests to the auth service for user authentication.
2. upload function - to direct requests to the message queue. The queue then will be responsible for sending the converter requests to the converter service.
3. download function - no code as of now.


System Design:

![image tooltip here](/assets/python_sys_design/008_system_design.png)

![image tooltip here](/assets/python_sys_design/014_gateway_initialization.png)

![image tooltip here](/assets/python_sys_design/015_routes_login_upload.png)

![image tooltip here](/assets/python_sys_design/016_login_gateway.png)

![image tooltip here](/assets/python_sys_design/017_validate_token_gateway.png)

![image tooltip here](/assets/python_sys_design/018_util_upload_code.png)



The below line abstracts the MongoDB connections:
```
mongo = PyMongo(server)
```

When the user sends a video to the gateway, the gateway will store the video in the mongo db database. It will then send a message to the queue that this video needs to be processed. Then the VTMP3 service will consume the messages from the queue, get the ID, fetch the video based on the ID from the mongo db, and process it. It will then store the MP3 on the mongo db, and send a new message in the queue, which will be consumed by the notification service. This service then will send an email to the user with the MP3 ID. Then the user will use this ID and the JWT to make a request to the gateway to download the MP3 file. The gateway will then pull the MP3 file from mongodb, and serve it to the client.

The gateway is synchronous with the auth service. It is asynchronous with the convertor service.

<!-- Eventual consistency. -->

<!-- Competing consumers pattern. -->

The converter service needs to be added to the system host IP list (I did not understand this part):
![image tooltip here](/assets/python_sys_design/010_add_converter_service_1.png)

![image tooltip here](/assets/python_sys_design/011_add_converter_service_2.png)


The gateway service needs to be created into a Docker image, which will then be used by the Kubernetes cluster when the services are set up. Once created, it needs to be pushed to the Docker hub, using the `docker tag` command:

![image tooltip here](/assets/python_sys_design/009_docker_tag_1.png)

We need a minikube addon to allow ingress - 2:39:39
Command:
```
minikube addons list
```

Ingress is currently disabled.

Command:
```
minikube addons enable ingress
```

Next, run the below command when testing the application:
```
minikube tunnel
```

![image tooltip here](/assets/python_sys_design/012_enable_ingress_op.png)

Deploy the YAML files for gateway.
![image tooltip here](/assets/python_sys_design/013_create_gateway_service.png)


Since the rabbitmq host is not defined, there is an error. We'll scale down the gateway deployment for now:
```
$ kubectl scale deployment --replicas=0 gateway
```

We need a stateful set for rabbitmq.


## 10/29/2024

rabbitmq needs to be persistent.
we don't want to lose the messages if the queue crashes. we would need to write it to a local disc.

statefulset.yaml
amqp - advanced message queue protocol - this protocol will be used to send messages to the queue.

We also need to create a persistent volume claim, its rules written in the pvc.yaml file.

There are 2 ports for the rabbitmq instance - a port to access its GUI, and a port to send messages to it.

We need to create an ingress for the GUI port, for it to be accessible from a browser. So as done above, it needs to be added in the /etc/hosts text file.

Need to be careful of spelling mistakes, extra colons, and incorrect indentation in YAML files.

Some error in rabbitmq pod, related to pvc.

To see if the pod has failed or something:
`kubectl describe pod rabbitmq-0`

Use the below command to access the GUI:
minikube tunnel

Go to rabbitmq-manager.com
The GUI is visible.

Credentials are:
guest
guest

The `minikube tunnel` console needs to be running.

Create a new queue named `video` (note that in the upload util function, the queue was named video.)
Other parameters:
Classic
Durable

Now we need to spin up the `gateway` service. All the services are up and running.

We then create the converter service. Actual comments about the working of the code is in the files.

Then we need to make the Dockerfiles for the Kubernetes cluster deployment.

Freeze the pip dependencies in a requirements.txt file.

In the Dockerfile, need to add a new dependency: ffmpeg, which will be used by the moviepy library.

Since it's a consumer, we will not be exposing any ports. Because it won't be a service that we'll be making requests to. It will be acting on its own.

Then do docker build, docker tag, then docker push.

We won't be needing secret, or service file for the converter deployment. A secret file is created just as a template, but won't be used.

We need configmap to store the MP3_QUEUE and the VIDEO_QUEUE environment variables, which will be used by the converter service.

In the RabbitMQ console, add the mp3 queue as well.

Error:
f"/{message["video_fid"]}.mp3" vs f"/{message['video_fid']}.mp3"

Since there was an error in the code, the Dockerfile needs to be rebuilt.

All services are running now.

Now, we'll try to upload a video, and see if it gets queued up in the message queue.

Review the credentials from the user table in the auth database. This will be used for the auth service.
Error:
in gateway/auth_svc/access file:
return response.txt, None needs to replaced with return response.text, None

There are some changes in the validate file as well.

We need to rebuild the Dockerfile.

After this, delete all the gateway resources. Use the kubectl apply command again to deploy the resources.

ERROR:
Changing server.config["MYSQL_PORT"] = os.environ.get("MYSQL_PORT") to server.config["MYSQL_PORT"] = int(os.environ.get("MYSQL_PORT")) in src/auth/server file.
Paster error message.

Scaling services down to 1 instance, so that checking for error logs can be easier.

JWT is generated now.

Issue at validate function inside gateway server code. Added a error block.

Issue resolved, but now it is giving a not authorized message.

In the auth/server file, renamed algorithms to algorithm.
After change, internal server error is coming.

The error is coming while inserting data into mongodb. I also realized I never installed mongodb.

I installed mongodb, created a database, and pasted the URI in the gateway server file. docker build, docker tag, docker push, kubecelt delete, kubectl apply, kubectl get pods to check instances, kubectl scale deployment --replicas=1 gateway.

Changing URI localhost to 0.0.0.0.

Tried multiple combinations for URI, and the problem is while connecting and accessing the mongodb database.
Err 111.

## I will resolve the mongodb issue later. moving on.

4:05:00 - issue about rabbitmq, but did not encounter.

---

Check the converter service logs for progress.

Currently, messages will be piled up on the mp3 queue, because there is not consumer for that queue.

Using mongosh, I'm able to create the vidoes database. But it disappers after closing console.

Also, when the console was left open, and the request was sent for upload, it still resulted in a timeout. - err 111.

There should be a mp3s and videos database.

I tried running the mongo database using the --bind_ip argument as well, still no progress.

Some commands in mongosh:
show databases
use videos
use mp3s
show collections

In mp3s database:
db.fs.files.find()

In rabbitmq console, can try to get tge get message button.

mongosh:
db.fs.files.find({"_id": ObjectId("<some ID>")})

4:16:00

Command to download the mp3 file:
mongofiles --db=mp3s get_id --local=test.mp3 '{"$oid": "<some ID>"}'

Now we need to create a service that consumes the messages. It will be a notification service.

We need to create 2 PyMongo instances for videos and mp3s databases.

Rebuild all.

Then we'll build the notification service. The code will be almost same as the converter consumer, with some changes - such as updating the queue name, adding a new function for sending mail.

The YAML files will be similar to that of converter manifests files.

We need to setup a smtp google server.

The less secure app access option is removed from google.

The email and password needs to be updated to that of a proper email id in the users table.

Port 587 is required for TLS.

Once the email is received, we can use the ID to download the file.

The below command needs to be run for downloading the MP3 file:
curl --output mp3_download.mp3 -X GET -H 'Authorization: Bearer JWT_TOKEN' "http://mp3converter.com/download?fid=THE_ID"


For uploading video:
```
curl -X POST -F 'file=@./vid_1.mp4' -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InNpZEBnbWFpbC5jb20iLCJleHAiOjE3MzAzMjIzNDQsImlhdCI6MTczMDIzNTk0NCwiYWRtaW4iOnRydWV9.j8JdF9uTudktVleOplKo_NFdsbi1xTYehFpTV1zdhVM' http://mp3converter.com/upload
```

For getting the JWT:
curl -X POST http://mp3converter.com/login -u sid@gmail.com:sid123

Write all the issues you're plagued with, and at the end write down what needs to be implemented.

Be very to the point, and truthful.


## Part 3 - RabbitMQ Service Deployment

The RabbitMQ console URL needs to be added in the host list. This is to ensure that when we try to access the RabbitMQ UI through browser, we will be able to see the console.

![image tooltip here](/assets/python_sys_design/023_rabbitmq_console.png)

## Part 4 - Converter Service Code

First, we initialize all the connection with the MongoDB service and the RabbitMQ service:

![image tooltip here](/assets/python_sys_design/024_converter_consumer_initialization_1.png)

The callback function is responsible for sending an acknowledge or a negative acknowledge to the queue. This is based on whether if the converter service is able to consume the video data from the `videos` queue. The `mp3.start()` function is responsible for consuming the details:

![image tooltip here](/assets/python_sys_design/025_converter_consumer_callback_2.png)

The `mp3.start()` function performs video to mp3 conversion, saving the mp3 file to the database, and pushing the mp3 details to the `mp3` queue. We'll focus on the last 2 parts:

![image tooltip here](/assets/python_sys_design/026_mp3_converter.png)

The RabbitMQ config details are stored in the config map, such as the name of the mp3 queue and the video queue.


## Part 5 - Notification Service Code

The notification service is responsible for sending an email to the client when the video to MP3 conversion is done. The service has a connection with the RabbitMQ service, and it consumes the MP3 details from the `mp3` queue. A callback function is defined, which sends either an acknowledge or negative acknowledge to the queue, based on the operation wherein the notification service tries to fetch messages from the queue.

Below is the code for the consumer of the notification service. It is similar in ways to the consumer of the converter service.

![image tooltip here](/assets/python_sys_design/019_consumer_notification_service.png)

<br/>

The `email.notification` function is called, which is responsible for sending the email.

![image tooltip here](/assets/python_sys_design/020_email_not_service.png)

<br/>

The email server details will be stored in the Secrets file:

![image tooltip here](/assets/python_sys_design/021_notification_secret.png)

<br/>

The name of the messaging queues is stored in the config map file:

![image tooltip here](/assets/python_sys_design/022_config_notification.png)

<br/>

## Part 5 - Running and using the application

1. Start all services
2. Get the JWT token from the auth service
3. Upload video from the folder directory using the JWT token
3. Once uploaded and video is processed, make a call to the download service once the email is received from the notification service.

## Issues and Fixes
1. The MongoDB service on the local machine is giving a connection timeout error. There is some issue in the database URI. This is a minor fix, and will be resolved immediatley. Check `converter/consumer.py` file for MongoClient.
2. Since the MongoDB service is not running, I cannot confirm if the video to MP3 service is working as well. However, I don't think there is any issue in the converter code. All the services hosted inside the Minikube cluster are up and running, so it is just the MongoDB issue that is preventing the application from a smooth flow.