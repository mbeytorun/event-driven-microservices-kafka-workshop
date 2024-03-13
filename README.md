# Event-Driven Microservices with Apache Kafka

Prepare Environment
The environment is completely based on docker containers. In order to easily start the multiple containers, we are going to use Docker Compose.

export DOCKER_HOST_IP=192.168.25.136
export PUBLIC_IP=192.168.25.136
Make sure to adapt the IP address according to your environment.

Creating the necessary Kafka Topics
The Kafka cluster is configured with auto.topic.create.enable set to false. Therefore we first have to create all the necessary topics, using the kafka-topics command line utility of Apache Kafka.

We can easily get access to the kafka-topics CLI by navigating into one of the containers for the 3 Kafka Brokers. Let's use broker-1

docker exec -ti broker-1 bash
First let's see all existing topics

kafka-topics --zookeeper zookeeper-1:2181 --list
And now create the topics customer and order both with cleanup policy set to compact in order to enable log compaction.

kafka-topics --zookeeper zookeeper-1:2181 --create --topic customer --partitions 8 --replication-factor 2 --config cleanup.policy=compact --config segment.ms=100 --config segment.bytes=1000 --config delete.retention.ms=100 --config min.cleanable.dirty.ratio=0.01

kafka-topics --zookeeper zookeeper-1:2181 --create --topic order --partitions 8 --replication-factor 2 --config cleanup.policy=compact --config segment.ms=100 -config segment.bytes=1000 --config delete.retention.ms=100 --config min.cleanable.dirty.ratio=0.01
Make sure to exit from the container after the topics have been created successfully.

Start the Customer Microservice:

docker run -ti -e PUBLIC_IP=$PUBLIC_IP -p 8080:8080 -d --name customer-management-ms gschmutz/customer-management-ms:1.0.2
to view the log and see that is has been started successfully, enter

docker logs -f customer-management-ms

Start the Order Microservice:

docker run -ti -e PUBLIC_IP=$PUBLIC_IP -p 8081:8080 -d --name order-management-ms gschmutz/order-management-ms:1.0.2
to view the log and see that is has been started successfully, enter

docker logs -f order-management-ms

Add a Customer:
The following curl command adds a new customer via the Customer Microservice:

```
curl -X POST -H 'Content-Type: application/json' -i http://streamingplatform:8080/api/customers --data '{
  "customerId" : 1,
  "firstName" : "James",
  "lastName" : "Black",
  "title" : "Mr",
  "emailAddress" : "james.black@someemail.com",
  "addresses" : [
  		{
  		  "street" : "Park Road",
         "number" : "121",
         "postcode" : "",
		  "city" : "London",
         "country" : "United Kingdom"
  		}
  ]
}'
curl -X POST -H 'Content-Type: application/json' -i http://streamingplatform:8080/api/customers --data '{
  "customerId" : 2,
  "firstName" : "John",
  "lastName" : "Snow",
  "title" : "Mr",
  "emailAddress" : "jsnow@testmail.com",
  "addresses" : [
  		{
  		  "street" : "Elalmis",
         "number" : "12",
         "postcode" : "",
		  "city" : "Umraniye",
         "country" : "Turkiye"
  		}
  ]
}'
```

Consuming messages with "Normal" consumer
In one of the Kafka broker

docker exec -ti docker_broker-1_1 bash
Use the Kafka Console Consumer to get the current messages

kafka-console-consumer --bootstrap-server broker-1:9092 --topic customer
Use the Kafka Console Consumer to get the all historical messages


