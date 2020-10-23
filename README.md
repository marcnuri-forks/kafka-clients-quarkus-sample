# Kafka Clients Quarkus Edition

## :rocket: :sparkles: :rotating_light: QUARKUS EDITION :rotating_light: :sparkles: :rocket:

This repo is a fork of the original one based in [Spring Boot](https://github.com/rmarting/kafka-clients-sb-sample)
but refactored to be a full-compliant [Quarkus](https://quarkus.io/) application.

The following components were refactored from Spring Boot to Quarkus Extensions:

| Spring Boot Dependency | Quarkus Extension |
|------------------------|--------------------|
| spring-boot-starter-web | [REST Services](https://quarkus.io/guides/rest-json) |
| spring-boot-starter-actuator | [Microprofile Health](https://quarkus.io/guides/microprofile-health) |
| springdoc-openapi-ui | [OpenAPI and Swagger UI](https://quarkus.io/guides/openapi-swaggerui) |
| spring-kafka | [Kafka with Reactive Messaging](https://quarkus.io/guides/kafka) |
| avro | Not documented (Experimental extension) |
| apicurio-registry-utils-serde | [Apicurio Registry](https://github.com/Apicurio/apicurio-registry) |

This new version is really fast (less than 2 seconds) ... like a :rocket: 

```text
2020-09-21 20:48:44,789 INFO  [io.quarkus] (main) kafka-clients-quarkus-sample 1.0.0-SNAPSHOT on JVM (powered by Quarkus 1.8.1.Final) started in 1.572s. Listening on: http://0.0.0.0:8181
2020-09-21 20:48:44,789 INFO  [io.quarkus] (main) Profile prod activated. 
```

## :rocket: :sparkles: :rotating_light: QUARKUS EDITION :rotating_light: :sparkles: :rocket: 

This sample project demonstrates how to use [Kafka Clients](https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients)
and [SmallRye Reactive Messaging](https://smallrye.io/smallrye-reactive-messaging) on Quarkus to send and consume messages from an
[Apache Kafka](https://kafka.apache.org/) cluster. The Apache Kafka cluster is operated by [Strimzi](https://strimzi.io/)
operator deployed on a Kubernetes or OpenShift Platform. These messages will be validated by a Schema Registry or Service Registry
operated by [Apicurio](https://www.apicur.io/registry/docs/apicurio-registry/index.html#intro-to-the-registry) operator.

Apache Kafka is an open-sourced distributed event streaming platform for high-performance data pipelines,
streaming analytics, data integration, and mission-critical applications.

Service Registry is a datastore for sharing standard event schemas and API designs across API and event-driven architectures.
You can use Service Registry to decouple the structure of your data from your client applications, and to share and
manage your data types and API descriptions at runtime using a REST interface.

The example includes a simple REST API with the following operations:

* Send messages to a Topic
* Consume messages from a Topic

## Environment

This project requires a Kubernetes or OpenShift platform available. If you do not have one, you could use 
one of the following resources to deploy locally a Kubernetes or OpenShift Cluster:

* [Red Hat CodeReady Containers - OpenShift 4 on your Laptop](https://github.com/code-ready/crc)
* [Minikube - Running Kubernetes Locally](https://kubernetes.io/docs/setup/minikube/)

> Notes for Minikube:
> * In older versions you may hit an [issue](https://github.com/kubernetes/minikube/issues/8330)
> with Persistent Volume Claims stuck in Pending status
> * Operator Lifecycle Manager is needed to deploy operators. To enable it in Minikube: 
>   - Option 1: ```minikube start --addons olm```
>   - Option 2: ```minikube addons enable olm```
> * Others addons needed: ```ingress```, ```registry```

> Note: Whatever the platform you are using (Kubernetes or OpenShift), you could use the 
> Kubernetes CLI (```kubectl```) or OpenShift CLI (```oc```) to execute the commands described in this repo.
> To reduce the length of this document, the commands displayed will use the Kubernetes CLI. When a specific
> command is only valid for Kubernetes or OpenShift it will be identified.

To deploy the resources, we will create a new ```amq-streams-demo``` namespace in the cluster in the case of Kubernetes:

```shell script
❯ kubectl create namespace amq-streams-demo
```

If your are using OpenShift, then we will create a project:

```shell script
❯ oc new-project amq-streams-demo
```

> Note: All the commands should be executed in this namespace. You could permanently save the namespace for
> all subsequent ```kubectl``` commands in that context.
>
> In Kubernetes:
>
> ```shell script
> ❯ kubectl config set-context --current --namespace=amq-streams-demo
> ```
> 
> In OpenShift:
>
> ```shell script
> ❯ oc project amq-streams-demo
> ```

### Deploying Strimzi and Apicurio Operators

> **NOTE**: Only *cluster-admin* users could deploy Kubernetes Operators. This section must
> be executed with one of them.

To deploy the Strimzi and Apicurio Operators only to inspect our namespace, we need to use
an ```OperatorGroup```. An OperatorGroup is an OLM resource that provides multitenant configuration to
OLM-installed Operators. For more information about this object, please review the
official documentation [here](https://docs.openshift.com/container-platform/4.5/operators/understanding_olm/olm-understanding-operatorgroups.html).

```shell script
❯ kubectl apply -f src/main/olm/operator-group.yml
operatorgroup.operators.coreos.com/amq-streams-demo-og created
```

Now we are ready to deploy the Strimzi and Apicurio Operators:

For Kubernetes use the following subscriptions:

```shell script
❯ kubectl apply -f src/main/strimzi/operator/subscription-k8s.yml
subscription.operators.coreos.com/strimzi-kafka-operator created
❯ kubectl apply -f src/main/apicurio/operator/subscription-k8s.yml
subscription.operators.coreos.com/apicurio-registry created
```

For OpenShift use the following subscriptions:

```shell script
❯ oc apply -f src/main/strimzi/operator/subscription.yml
subscription.operators.coreos.com/strimzi-kafka-operator created
❯ oc apply -f src/main/apicurio/operator/subscription.yml 
subscription.operators.coreos.com/apicurio-registry created
```

You could check that operators are successfully registered with:

```shell script
❯ kubectl get csv
NAME                                    DISPLAY                      VERSION              REPLACES   PHASE
apicurio-registry.v0.0.3-v1.2.3.final   Apicurio Registry Operator   0.0.3-v1.2.3.final              Succeeded
strimzi-cluster-operator.v0.19.0        Strimzi                      0.19.0                          Succeeded
```

or verify the pods are running:

```shell script
❯ kubectl get pod
NAME                                                READY   STATUS    RESTARTS   AGE
apicurio-registry-operator-cbf6fcf57-d6shn          1/1     Running   0          3m2s
strimzi-cluster-operator-v0.19.0-7555cff6d9-vlgwp   1/1     Running   0          3m7s
```

For more information about how to install Operators using the CLI command, please review this [article](
https://docs.openshift.com/container-platform/4.5/operators/olm-adding-operators-to-cluster.html#olm-installing-operator-from-operatorhub-using-cli_olm-adding-operators-to-a-cluster)

### Deploying Kafka

```src/main/strimzi``` folder includes a set of custom resource definitions to deploy a Kafka Cluster
and some Kafka Topics using the Strimzi Operators.

To deploy the Kafka Cluster:

```shell script
❯ kubectl apply -f src/main/strimzi/kafka/kafka.yml
kafka.kafka.strimzi.io/my-kafka created
```

> If you want to deploy a Kafka Cluster with HA capabilities, there is a definition
> in [kafka-ha.yml](./src/main/strimzi/kafka/kafka-ha.yml) file.

To deploy the Kafka Topics:

```shell script
❯ kubectl apply -f src/main/strimzi/topics/kafkatopic-messages-offline.yml
kafkatopic.kafka.strimzi.io/messages created
❯ kubectl apply -f src/main/strimzi/topics/kafkatopic-messages-online.yml
kafkatopic.kafka.strimzi.io/messages created
```

> If you want to use a Kafka Topic with HA capabilities, there is a definition
> in [kafkatopic-messages-ha.yml](./src/main/strimzi/topics/kafkatopic-messages-offline-ha.yml) file.

There is a set of different users to connect to Kafka Cluster. We will deploy here to be used later.

```shell script
❯ kubectl apply -f src/main/strimzi/users/
kafkauser.kafka.strimzi.io/application created
kafkauser.kafka.strimzi.io/service-registry-scram created
kafkauser.kafka.strimzi.io/service-registry-tls created
```

After some minutes Kafka Cluster will be deployed:

```shell script
❯ kubectl get pod
NAME                                                READY   STATUS    RESTARTS   AGE
apicurio-registry-operator-cbf6fcf57-d6shn          1/1     Running   0          4m32s
my-kafka-entity-operator-6d9596458b-rl5w9           3/3     Running   0          59s
my-kafka-kafka-0                                    2/2     Running   0          2m6s
my-kafka-zookeeper-0                                1/1     Running   0          3m20s
strimzi-cluster-operator-v0.19.0-7555cff6d9-vlgwp   1/1     Running   0          4m37s
```

### Service Registry

Service Registry needs a set of Kafka Topics to store schemas and metadata of them. We need to execute the following
commands to create the KafkaTopics and to deploy an instance of Service Registry:

```shell script
❯ kubectl apply -f src/main/apicurio/topics/
kafkatopic.kafka.strimzi.io/global-id-topic created
kafkatopic.kafka.strimzi.io/storage-topic created
❯ kubectl apply -f src/main/apicurio/service-registry.yml
apicurioregistry.apicur.io/service-registry created
```

A new DeploymentConfig is created with the prefix ```service-registry-deployment-``` and a new route with
the prefix ```service-registry-ingres-```. We must inspect it to get the route created to expose the Service Registry API.

In Kubernetes we will use an ingress entry based with ```NodePort```. To get the ingress entry:

```shell script
❯ kubectl get deployment | grep service-registry-deployment
service-registry-deployment-m57cq
❯ kubectl expose deployment service-registry-deployment-m57cq --type=NodePort --port=8080
service/service-registry-deployment-m57cq exposed
❯ kubectl get service/service-registry-deployment-m57cq
NAME                                TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service-registry-deployment-m57cq   NodePort   10.101.44.116   <none>        8080:31216/TCP   39s 
❯ minikube service service-registry-deployment-m57cq --url -n amq-streams-demo
http://192.168.39.227:31216
```

In OpenShift, we only need to check the ```host``` attribute from the OpenShift Route:

```shell script
❯ oc get route
NAME                                   HOST/PORT                                      PATH   SERVICES                         PORT    TERMINATION   WILDCARD
service-registry-ingress-rjmr4-8gxsd   service-registry.amq-streams.apps-crc.testing  /      service-registry-service-bfj4n   <all>                 None
```

While few minutes until your Service Registry has deployed.

The Service Registry Web Console and API endpoints will be available from: 

* **Web Console**: http://<KUBERNETES_OPENSHIFT_SR_ROUTE_SERVICE_HOST>/ui/
* **API REST**: http://KUBERNETES_OPENSHIFT_SR_ROUTE_SERVICE_HOST/api/

Set up the ```apicurio.registry.url``` property in the ```pom.xml``` file the Service Registry url before to publish the
schemas used by this application:

```shell script
❯ oc get route service-registry-ingress-rjmr4-8gxsd -o jsonpath='{.spec.host}'
```

To register the schemas in Service Registry execute:

```shell script
❯ mvn clean generate-sources -Papicurio
```

The next screenshot shows the schemas registed in the Web Console:

![Artifacts registered in Apicurio Registry](./img/apicurio-registry-artifacts.png) 

# Build and Deploy

Before we build the application we need to set up some values in ```src/main/resources/application.properties``` file.

Review and set up the right values from your Kafka Cluster 

* **Kafka Bootstrap Servers**: Kafka brokers are defined by a Kubernetes or OpenShift service created by Strimzi when
the Kafka cluster is deployed. This service, called *cluster-name*-kafka-bootstrap exposes 9092 port for plain
traffic and 9093 for encrypted traffic. 

```text
app.kafka.bootstrap-servers = my-kafka-kafka-bootstrap:9092
```

* **Kafka User Credentials**: Kafka Cluster requires authentication, we need to set up the Kafka User credentials
in our application (```app.kafka.user.*``` properties in ```application.properties``` file). Each KafkaUser has its own
secret to store the password. This secret must be checked to extract the password for our user.

To extract the password of the KafkaUser and declare as Environment Variable:

```shell script
❯ export KAFKA_USER_PASSWORD=$(kubectl get secret application -o jsonpath='{.data.password}' | base64 -d)
```

It is a best practice use directly the secret as variable in our deployment in Kubernetes or OpenShift. We could do
it declaring the variable in the container spec as:

```yaml
spec:
  template:
    spec:
      containers:
        - env:
          - name: KAFKA_USER_PASSWORD
            valueFrom:
              secretKeyRef:
                key: password
                name: application
```

There is a deployment definition in [deployment.yml](./src/main/jkube/deployment.yml) file. This file will be used
by JKube to deploy our application in Kubernetes or OpenShift.

* **Service Registry API Endpoint**: Avro Serde classes need to communicate with the Service Registry API to check and
validate the schemas. 

```text
apicurio.registry.url = http://service-registry.amq-streams-demo.apps-crc.testing/api
```

To build the application:

```shell script
❯ mvn clean package
```

To run locally:

```shell script
❯ export APP_BG_VERSION=local
❯ export APP_BG_MODE=local
❯ export KAFKA_USER_PASSWORD=$(kubectl get secret application -o jsonpath='{.data.password}' | base64 -d)
❯ mvn compile quarkus:dev
```

## Blue:large_blue_circle:/Green:large_green_circle: Deployment

To deploy into Kubernetes or OpenShift platform using [Eclipse JKube](https://github.com/eclipse/jkube) Maven Plug-ins:

To deploy the blue application version using the Kubernetes Maven Plug-In:

```shell script
❯ eval $(minikube docker-env)
❯ mvn clean package k8s:build k8s:resource k8s:apply -Pkubernetes,blue
```

To deploy the blue application version using the OpenShift Maven Plug-In (only valid for OpenShift Platform):

```shell script
❯ mvn clean package oc:build oc:resource oc:apply -Popenshift,blue
```

To deploy the green application version using the Kubernetes Maven Plug-In:

```shell script
❯ eval $(minikube docker-env)
❯ mvn clean package k8s:build k8s:resource k8s:apply -Pkubernetes,green
```

To deploy the green application version using the OpenShift Maven Plug-In:

```shell script
❯ mvn clean package oc:build oc:resource oc:apply -Popenshift,green
```

Now we deploy the generic resources to identify the *online* and *offline* modes of the application:

```shell script
❯ oc apply -f src/main/blue-green
```

Rolling Blue to Green in Kubernetes will be:

```shell script
❯ kubectl set env deployment/kafka-clients-quarkus-sample-blue APP_BG_MODE=offline
❯ kubectl set env deployment/kafka-clients-quarkus-sample-green APP_BG_MODE=online
❯ kubectl patch svc kafka-clients-quarkus-sample-online --type=merge -p '{"spec": {"selector": {"deploymentconfig": "kafka-clients-quarkus-sample-green"}}}'
```

Rolling Blue to Green in OpenShift will be:

```shell script
❯ oc set env dc/kafka-clients-quarkus-sample-blue APP_BG_MODE=offline
❯ oc set env dc/kafka-clients-quarkus-sample-green APP_BG_MODE=online
❯ oc patch route kafka-clients-quarkus-sample --type=merge -p '{"spec": {"to": {"name": "kafka-clients-quarkus-sample-green"}}}'
```

Rolling Green to Blue in Kubernetes will be:

```shell script
❯ kubectl set env deployment/kafka-clients-quarkus-sample-green APP_BG_MODE=offline
❯ kubectl set env deployment/kafka-clients-quarkus-sample-blue APP_BG_MODE=online
❯ kubectl patch svc kafka-clients-quarkus-sample-online --type=merge -p '{"spec": {"selector": {"deploymentconfig": "kafka-clients-quarkus-sample-blue"}}}'
```

Rolling Green to Blue in OpenShift will be:

```shell script
❯ oc set env dc/kafka-clients-quarkus-sample-green APP_BG_MODE=offline
❯ oc set env dc/kafka-clients-quarkus-sample-blue APP_BG_MODE=online
❯ oc patch route kafka-clients-quarkus-sample --type=merge -p '{"spec": {"to": {"name": "kafka-clients-quarkus-sample-blue"}}}'
```

# REST API

REST API is available from a Swagger UI at:

```text
http://<KUBERNETES_OPENSHIFT_ROUTE_SERVICE_HOST>/swagger-ui
```

**KUBERNETES_OPENSHIFT_ROUTE_SERVICE_HOST** will be the route create on Kubernetes or OpenShift to expose outside the
service. 

To get the route the following command in OpenShift give you the host:

```shell script
❯ oc get route kafka-clients-quarkus-sample -o jsonpath='{.spec.host}'
```

There are two groups to manage a topic from a Kafka Cluster.

* **Producer**: Send messageDTOS to a topic 
* **Consumer**: Consume messageDTOS from a topic

## Producer REST API

Sample REST API to send messages to a Kafka Topic.
Parameters:

* **topicName**: Topic Name
* **messageDTO**: Message content based in a custom messageDTO:

Model:

```text
MessageDTO {
  key (integer, optional): Key to identify this message,
  timestamp (string, optional, read only): Timestamp,
  content (string): Content,
  partition (integer, optional, read only): Partition number,
  offset (integer, optional, read only): Offset in the partition
}
```

Simple Sample:

```shell script
❯ curl -X POST http://$(oc get route kafka-clients-quarkus-sample -o jsonpath='{.spec.host}')/producer/kafka/messages \
-H "Content-Type:application/json" -d '{"content": "Simple message"}' | jq
{
  "content": "Simple message",
  "offset": 3,
  "partition": 0,
  "timestamp": 1581087543362
}
```

With Minikube:

```shell script
❯ curl $(minikube ip):$(kubectl get svc kafka-clients-quarkus-sample -o jsonpath='{.spec.ports[].nodePort}')/producer/kafka/messages \
-H "Content-Type:application/json" -d '{"content": "Simple message from Minikube"}' | jq
{
  "content": "Simple message from Minikube",
  "offset": 4,
  "partition": 0,
  "timestamp": 1596203271368
}
```

## Consumer REST API

Sample REST API to consume messages from a Kafka Topic.
Parameters:

* **topicName**: Topic Name (Required)
* **partition**: Number of the partition to consume (Optional)
* **commit**: Commit messaged consumed. Values: true|false

Simple Sample:

```shell script
❯ curl -v "http://$(oc get route kafka-clients-quarkus-sample -o jsonpath='{.spec.host}')/consumer/kafka/messages?commit=true&partition=0" | jq
{
  "messages": [
    {
      "content": "Simple message",
      "offset": 0,
      "partition": 0,
      "timestamp": 1581087539350
    },
    ...
    {
      "content": "Simple message",
      "offset": 3,
      "partition": 0,
      "timestamp": 1581087584266
    }
  ]
}
```

With Minikube:

```shell script
❯ curl $(minikube ip):$(kubectl get svc kafka-clients-quarkus-sample -o jsonpath='{.spec.ports[].nodePort}')"/consumer/kafka/messages?commit=true&partition=0" | jq
{
  "messages":[
    {
      "content": "Simple message from Minikube",
      "offset": 4,
      "partition": 0,
      "timestamp": 1596203271368
    }
  ]
}
```

That is! You have been deployed a full stack of components to produce and consume checked and valid messages using
a schema declared. Congratulations!.

## Main References

* [Strimzi](https://strimzi.io/)
* [Apicurio](https://www.apicur.io/)
* [OperatorHub - Strimzi](https://operatorhub.io/operator/strimzi-kafka-operator)
* [OperatorHub - Apicurio Registry](https://operatorhub.io/operator/apicurio-registry)
* [Quarkus - Building applications with Maven](https://quarkus.io/guides/maven-tooling)
* [Quarkus - Writing JSON REST Services](https://quarkus.io/guides/rest-json)
* [Quarkus - Using OpenAPI and Swagger UI](https://quarkus.io/guides/openapi-swaggerui)
* [Quarkus - Contexts and Dependency Injection](https://quarkus.io/guides/cdi-reference)
* [Quarkus - Using Apache Kafka with Reactive Messaging](https://quarkus.io/guides/kafka)
* [Quarkus - Configuring your application](https://quarkus.io/guides/config)
* [Eclipse JKube](https://www.eclipse.org/jkube/)
