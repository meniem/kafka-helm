# Apache Kafka on K8s using Helm

## About this repo

This repository is deploying Apache Kafka and other third party integrations on Kubernetes using a Helm chart.


## What's going to be deployed

- **Namespace**: a new namespace called ```kafka``` that will have all the deployed resources.

- **RBAC**: a new ```ServiceAccount```, ```ClusterRole```, and ```ClusterRoleBinding``` to manage access between different resources.

- **Kafka-broker**: the Apache Kafka broker will be deployed as a ```StatefulSet``` using the latest kafka docker image. A ```headless``` service will be used to expose it.
- **Kafka configuration**: all Kafka  settings can be controlled from the ```broker-config``` ConfigMap file, including the init script "init.sh" and both: ```server.properties``` and ```log4j.properties```.

- **Zoo Keeper**: will be dployed as a ```StatefulSet```, along with ```ClusterIP``` servcie (expose both **2888** and **3888** ports), headless service  (expose **2181** port), and a ```ConfigMap``` settings file.

- **Persistent Volumes**: tow ```PersistentVolume``` and ```PersistentVolumeClaim``` will be provisioned to persist the data within for both kafka-broker and zoo keeper.

- **AKHQ**: Kafka GUI for Apache Kafka to manage topics, topics data, consumers group, schema registry, connect and more...

- **ZooNavigator**: it's a web-based ZooKeeper UI and editor/browser with many features.

- **kafka-cli**: a kafka-based k8s pod to interact with both kafka-broker and zookeeper programatically.


## How to deploy the chart:

* In order to deploy the above resourtce, you will need to install ```Helm```, a package manager for K8s. More info and installation guide can be found below:
[https://helm.sh/docs/intro/install/](https://helm.sh/docs/intro/install/)


* Once Helm is installed,m the above chart can be deployed using the below commands:

``` bash
git clone git@github.com:meniem/kafka-helm.git
helm upgrade --install kafka-chart ./helm
```

## How to deploy the Kafka cluster locally:

* You can create a local k8s environment to test the resopurces using Minikube, which runs a single-node Kubernetes cluster on your personal computer. To install and start a new minikube cluster, please follwo the steps below:

[https://kubernetes.io/docs/tasks/tools/](https://kubernetes.io/docs/tasks/tools/)

[https://minikube.sigs.k8s.io/docs/start/](https://minikube.sigs.k8s.io/docs/start/)

* Once the local k8s lcouster is up and running, you casn follwo the same steps above to apply the helm chart.

## Exposing the web-based tools locally:

* Both AKHQ and ZooNavigator tools are deployed using NodePort service type to easily expose them locally.

* To expsoe both services using Minikube, run the below commands:

```bash
minikube service --url akhq -n kafka
minikube service --url zoonavigator -n kafka
```

Note: for ZooNavigator, when asking for the connection string, type: ```zookeeper-svc.kafka.svc.cluster.local:2181``` as this is the FQDN of the service on the cluster.

## Interacting with the cluster from the CLI:

* The kafka-cli pod is used to interact with the cluster from CLI, you can open an SSH session with the pod and test the basic kafka opreations as shown below:

```bash
# ssh into the pod:
kubectl exec -it kafka-cli -n kafka -- bash
```
```bash
# create a new topic
./bin/kafka-topics.sh --create --zookeeper zookeeper-svc:2181 --replication-factor 1 --partitions 1 --topic testtopic
>Created topic "testtopic".
```
```bash
# push a created topic:
./bin/kafka-console-producer.sh --broker-list kafka-broker:9092 --topic testtopic
>Message 1 in testtopic
```
```bash
# Open another ssh session to access the consumer and check the pushed message:
kubectl exec -it kafka-cli -n kafka -- bash
./bin/kafka-console-consumer.sh --bootstrap-server kafka-broker:9092 --topic testtopic --partition 0 --from-beginning
>Message 1 in testtopic
```
