---
layout: post
title:  "Kubernetes - A brief overview"
date:   2024-10-16 00:02:29 -0400
categories: jekyll update
---

<em>8/31/2024</em>

Kubernetes is an open-source container orchestration tool that helps manage containerized applications across various deployment environments. The rise of microservices architecture has led to increased usage of containers. As applications grew to consist of thousands of containers, managing them using manual scripts became complex and inefficient. This complexity drove the need for container orchestration tools like Kubernetes.

### Key features offered by orchestration:
* High availability, ensuring no downtime.
* Scalability, allowing high performance.
* Disaster recovery, enabling backup and restore processes.

<br/>

#### Kubernetes basic components:

Pod is the smallest unit in Kubernetes (k8s). It is an abstraction layer over containers. For example, there might be a pod for a database. Typically, a pod is designed to run a single application container within it.

Kubernetes offers a virtual network, and each pod is assigned its own IP address, enabling inter-container communication. Pods, however, are ephemeral, meaning they can terminate unexpectedly, and when recreated, they will be assigned new IP addresses. To maintain consistency in communication, a service is used.

Each application pod and database pod has its own service. The lifecycle of a service is independent of the pod's lifecycle, meaning even if a pod dies, the service will retain its IP and continue functioning. Thus, there's no need to change the endpoint.

To make an application accessible via a browser, an external service is used. However, for sensitive components like the database, we can restrict access using an internal service. The external service's URL may not be user-friendly, which is where Ingress comes into play. Ingress manages routing by forwarding requests to the appropriate services.

<br/>

#### Configmap and secret:

Pods communicate using services, but how is the configuration managed? Typically, configurations might reside in the application's properties file. However, if the database URL changes, the application would need to be rebuilt with the new settings. ConfigMap solves this issue by allowing external configuration management.

Sensitive information such as database usernames and passwords is stored in Secrets, encoded in base64 format for security. Applications can access these configurations and secrets through environment variables, providing flexibility and reducing the need to rebuild applications for simple changes.

Key components: Pods, Services, Ingress, ConfigMaps, Secrets.

<br/>

#### Volumes:

Data storage is critical, especially for persistent applications like databases. If a database pod gets restarted, its data will be lost unless persistence is ensured. Volumes in Kubernetes attach physical storage either on the local machine within the Kubernetes cluster or on remote storage outside of the cluster. This ensures that data is persisted even when a pod is restarted. Kubernetes doesn’t manage data persistence explicitly—developers are responsible for managing it.

<br/>

#### Deployement and stateful set:

Users typically access the application through a browser. But what happens if the application pod dies? Without intervention, users would lose access. To solve this, Kubernetes allows replication across multiple servers. A Deployment ensures there are multiple clones of an application running on different servers, each accessible through a service with a static IP and DNS name.

Pods are an abstraction on top of containers, while Deployments are an abstraction on top of pods. Deployments allow specifying how many replicas should run at any given time.

If one pod dies, the service automatically forwards requests to another. However, database pods require a different approach because databases maintain a state. Replicating a database using a deployment would cause inconsistencies if multiple database pods write to the same storage. This is where StatefulSet comes in. StatefulSets manage the replication of stateful applications (like databases), ensuring data consistency by controlling which pod reads and writes at any given time. StatefulSets are more complex to set up than deployments, which is why many developers choose to host databases outside the Kubernetes cluster.

Even if an entire node fails, the application will remain accessible through other nodes in the cluster.

<br/>

#### Summary:

* Containers are abstracted into pods.
* Communication between pods happens through services.
* Ingress handles routing traffic into the cluster.
* ConfigMaps and Secrets manage external configurations.
* Volumes ensure data persistence.
* Deployments handle stateless pod replication, while StatefulSets manage stateful applications.

![image tooltip here]({{ "assets/kubernetes/image_1.png" | relative_url }})

<br/>

#### Kubernetes architecture:

Node Processes:
Each node hosts multiple pods. Three processes must be installed on every node:

1. Container Runtime (e.g., Docker): Responsible for running containers.
2. Kubelet: Communicates with the container runtime and node. It receives pod specifications and starts the pods by allocating resources such as CPU and memory. A Kubernetes cluster can have hundreds of such worker nodes, which communicate using services.
3. Kube Proxy: Ensures efficient and low-overhead communication within the cluster by forwarding requests to the correct pod intelligently.

Master Node:
Master nodes control the worker nodes and run four key processes:

1. API Server: Acts as the cluster gateway and gatekeeper, handling authentication and facilitating interactions with the cluster.
2. Scheduler: Decides where to place new pods based on resource availability. However, the scheduler only makes the decision; the Kubelet on the node starts the pod.
3. Controller Manager: Detects state changes, such as when a pod crashes, and ensures the cluster remains in its desired state by interacting with the scheduler to request pod restarts.
4. etcd: Serves as the cluster’s key-value store, maintaining the cluster's state. It stores information about resource availability, state changes, and more. However, no application data is stored here.

![image tooltip here]({{ "assets/kubernetes/001_master_node_api.png" | relative_url }})

In practice, a Kubernetes cluster is composed of multiple master nodes for high availability. Master nodes are critical but don't require as many resources as worker nodes. It's easy to scale the cluster by adding new master or node servers.

![image tooltip here]({{ "assets/kubernetes/002_master_node_all_processed.png" | relative_url }})

![image tooltip here]({{ "assets/kubernetes/003_example_cluster.png" | relative_url }})

<br/>

#### Minikube
Minikube allows you to run both master and node processes on a single machine, which is ideal for local development and testing. It provides an environment for testing Kubernetes features without needing a full multi-node cluster.

This command starts the Minikube environment, setting up a local Kubernetes cluster on your machine:
```
minikube start
````

This command displays the nodes in your Minikube cluster, verifying that the cluster has been set up successfully.
````
kubectl get nodes
```

![image tooltip here]({{ "assets/kubernetes/004_minikube.png" | relative_url }})

In Kubernetes, individual pods are not created manually. Instead, this process is abstracted by using deployments, which manage the lifecycle of the pods.

This command creates a deployment named nginx-depl that uses the nginx image, managing the pod(s) for this deployment automatically:
```
kubectl create deployment nginx-depl --image=nginx
```

This command retrieves information about deployments in the cluster, showing the status of each deployment.
```
kubectl get deployment
```

This command lists all the pods in the cluster, allowing you to see the individual instances running:
```
kubectl get pod
```

This command displays the ReplicaSet objects, which ensure that a specified number of pod replicas are running at all times:
```
kubectl get replicaset
```

<br/>

#### To Edit a Deployment:
This command opens the configuration of the nginx-depl deployment for editing, allowing you to make changes:
```
kubectl edit deployment nginx-depl
```

After editing, this command helps you verify that any changes are reflected in the ReplicaSet associated with the deployment:
```
kubectl get replicaset
```

This command retrieves the logs of a specific pod, which is useful for troubleshooting or monitoring pod activity:
```
kubectl logs <pod name>
```

This command provides detailed information about the specified pod, including events, conditions, and resource usage:
```
kubectl describe pod nginx-depl-5dfbd7dbbd-j6k84
```

This command fetches the logs for a MongoDB pod, which can help in checking MongoDB-related operations or troubleshooting:
```
kubectl logs mongo-depl-887485654-v8xdm
```

<br/>

#### Open an Interactive Terminal for a Database-Related Pod
To open an interactive terminal for the mongo pod, use:
```
kubectl exec -it mongo-depl-887485654-v8xdm -- bin/bash
```
This command opens a terminal inside the specified MongoDB pod, allowing you to run commands directly within the container.

Alternative Command for Windows (if the above command doesn't work):
```
winpty kubectl exec -it mongo-depl-887485654-v8xdm -- bin/bash
```
This command opens a terminal session on Windows systems (using winpty) if the previous command does not work as expected.

<br/>

#### To Delete a Deployment
This command deletes the specified deployment, along with all its associated pods and ReplicaSets:
```
kubectl delete deployment <deploymnent name>
```

<br/>

#### Creating Deployments Using a YAML File
This command applies the configuration from nginx-deployment.yaml, creating the deployment if it doesn’t exist or updating it if it does:
```
kubectl apply -f nginx-deployment.yaml
```
If changes are made to the YAML file, run the same command to apply the updates to the cluster.

![image tooltip here]({{ "assets/kubernetes/005_config_file_for_deployment.png" | relative_url }})

<!-- << 1:02:32 >> -->

<br/>

#### YAML Configuration File for Kubernetes
Each configuration file in Kubernetes consists of three main parts:

1. Metadata: Provides identifying information about the object, such as name and labels.
2. Specification: Describes the desired state of the object, such as the container image to use and the number of replicas.
3. Status: Automatically generated by Kubernetes. It reflects the actual state of the object and is compared with the desired state. If there is a mismatch, Kubernetes attempts to reconcile the two, demonstrating Kubernetes' self-healing property. This status information is retrieved from etcd, which holds the current status of all Kubernetes components.
Use a YAML validator to check the syntax of the YAML file.


<!-- ### RETRY -->

Store the configuration file with the application code.

First, apply the deployment and service YAML files:
```
kubectl apply -f nginx-deployment.yaml
kubectl apply -f nginx-service.yaml
```

<br/>

#### To Get Details of Pods and Services
This command lists all services in the cluster, showing essential details like type, cluster IP, and external IP:
```
kubectl get service
```

This command lists all pods in the cluster, including their status and readiness.
```
kubectl get pods
```

<br/>

#### To Verify Service Association with a Pod
This command provides detailed information about the nginx-service, including its endpoints, which contain the IP addresses of the associated pods:
```
kubectl describe service nginx-service
```

<br/>

#### To Get Pod IP Addresses
This command displays the IP addresses of the pods and additional information such as the node on which each pod is running:
```
kubectl get pod -o wide
```

`etcd` continuously monitors the status of the pods. This status is specified in the deployment YAML file and stored in the `etcd` location.

This command retrieves the full YAML content of the nginx-deployment, including status information:
```
kubectl get deployment nginx-deployment -o yaml
```

<br/>

#### To Save the YAML Configuration to a Backup File
This command exports the deployment YAML, along with its current status, to a file for backup:
```
kubectl get deployment nginx-deployment -o yaml > nginx-deployment-result.yaml
```

<br/>

#### Deleting Pods and Services Using YAML Files
Run these commands to delete the resources defined in the YAML files:
```
kubectl delete -f nginx-deployment.yaml
kubectl delete -f nginx-service.yaml
```

Lists all secrets in the namespace, showing details about sensitive data stored in Kubernetes:
```
kubectl get secret
```

Watches the status of all pods in real-time, providing live updates as the status changes:
```
kubectl get pod --watch
```

<!-- <<1:33:36>>
### RETRY

<< Restart from 1:16:21 >> -->


<br/>

<br/>

#### Deploying a Sample MongoDB Application
<em>10/25/2024</em>

#### Step 1: Create a mongodb deployement.
The MongoDB username and password should not be stored directly in the deployment YAML file. Instead, they are stored in a Secrets file, which Kubernetes manages securely. This keeps sensitive information out of the project repository. The Secrets data is stored in Base64 format.

Encoding username and password in Base64:

```
$ echo -n 'username' | base64
dXNlcm5hbWU=

$ echo -n 'password' | base64
cGFzc3dvcmQ=
```

<br/>

#### Create the Secrets File:
```
$ kubectl apply -f mongo-secret.yaml
secret/mongodb-secret created
```
This command creates a Kubernetes secret for MongoDB with the credentials.

<br/>

#### Update the MongoDB Deployment File
```
$ kubectl apply -f mongo.yaml
deployment.apps/mongodb-deployment created
```
This command creates the MongoDB deployment and includes the credentials from the secrets file.

<br/>

#### To Check the Deployment:
This command lists all the MongoDB pods:
```
kubectl get pod
```

This command provides details about the specific MongoDB pod:
```
kubectl describe pod mongodb-deployment-69b8c595fb-h29tl
```

<br/>

#### Creating an Internal Service

An internal service allows other pods to communicate with the MongoDB pod.

You can include multiple resources in one YAML file by using `---` as a separator. The deployment and service configurations are often in the same file, as they are closely related.

Ensure that the `containerPort` in the deployment file matches the `targetPort` in the service file.

This command applies the configuration, creating or updating the MongoDB deployment and its internal service:
```
$ kubectl apply -f mongo.yaml
deployment.apps/mongodb-deployment unchanged
service/mongodb-service created
```

<br/>

#### Creating a ConfigMap and Mongo Express Deployment
The database URL is sourced from a ConfigMap. First, create the ConfigMap, then create the Mongo Express deployment.

This command creates a ConfigMap with MongoDB configuration details:
```
$ kubectl apply -f mongo-configmap.yaml
configmap/mongodb-configmap created
```

This command deploys Mongo Express, a web-based MongoDB client:
```
$ kubectl apply -f mongo-express.yaml
deployment.apps/mongo-express created
```

<br/>

#### Setting Up an External Service for Mongo Express
To make Mongo Express accessible from outside the cluster, set the service type to LoadBalancer, which assigns an external IP address. Specify the nodePort property, within the range 30000-32767, for external access.
```
$ kubectl apply -f mongo-express.yaml
deployment.apps/mongo-express unchanged
service/mongo-express-service created
```

<br/>

#### Accessing the External Service in Minikube
In Minikube, an external IP may show as pending, as Minikube does not automatically assign external IPs. Use the following command to access the service in Minikube.
```
minikube service mongo-express-service
```
<br/>

#### Checking Credentials in the Logs
If prompted for credentials, retrieve them using:
```
kubectl logs <pod_name>
```

<!-- << 1:46:19 >> -->