# Today I learned how to test Kafka integration in Quarkus app


When you start working with Quarkus, you discover the magic of dev services! 
If you have an application that use Kafka, a database, RabbitMQ, Redis, ... (full list  [here](https://quarkus.io/guides/dev-services) ), then if you don't have some properties set for dev and test profile like broker, url, ... Quarkus will start a dev service using  [Test Containers](https://www.testcontainers.org/).

You get crazy, code as much as integration test as you can, it is so easy and finally push to your favourite CI/CD pipeline.

And now everything is red in your test job!!! Whyyyyy??? Well, your sys admin removed the dind from the runners... Yes using dind can create a security breach in your pipeline...

Of course they tell you that you can start a service (gitlab services for example) and plug your tests on it but you want it to work locally so having to start (maintain) a container, inject the correct urls, be sure to clean / setup the content properly,... no thanks.

**We want to have something inside of the code. **

We will see how to setup your Kafka cluster inside of your integration tests.

## Create a Quarkus project

Let's go to https://code.quarkus.io/, create a new project with those dependencies:

- SmallRye Reactive Messaging
- SmallRye Reactive Messaging - Kafka Connector

Now let's remove the classes generated (in main and test) and the META-INF

Create a new service KafkaService:

```java
import io.smallrye.mutiny.Multi;
import java.time.Duration;
import javax.enterprise.context.ApplicationScoped;
import org.eclipse.microprofile.reactive.messaging.Incoming;
import org.eclipse.microprofile.reactive.messaging.Message;
import org.eclipse.microprofile.reactive.messaging.Outgoing;
import org.jboss.logging.Logger;

@ApplicationScoped
public class KafkaService {

  public static final Logger LOGGER = Logger.getLogger(KafkaService.class);

  @Outgoing("demo-kafka-out")
  public Multi<Message<String>> produceData() {
    return Multi.createFrom().ticks().every(Duration.ofSeconds(1))
        .map(tick -> Message.of("SmallRye Hello " + tick));
  }

  @Incoming("demo-kafka-in")
  public void consumeData(String message) {
    LOGGER.infov("Message received {0}", message);
  }
}
``` 
We have a simple example with one producer that will push every second a message and one consumer that will log this message.

We have to set the properties:

```properties
mp.messaging.outgoing.demo-kafka-out.connector=smallrye-kafka
mp.messaging.incoming.demo-kafka-in.connector=smallrye-kafka
mp.messaging.outgoing.demo-kafka-out.topic=demo-kafka
mp.messaging.incoming.demo-kafka-in.topic=demo-kafka
mp.messaging.outgoing.demo-kafka-out.value.serializer=org.apache.kafka.common.serialization.StringSerializer
mp.messaging.incoming.demo-kafka-in.value.deserializer=org.apache.kafka.common.serialization.StringDeserializer

``` 
Create a new empty test from KafkaService:


```java
package com.snoirot;

import io.quarkus.test.junit.QuarkusTest;
import org.junit.jupiter.api.Test;

@QuarkusTest
class KafkaServiceTest {

  @Test
  void testKafkaService() {

  }
}
``` 
So if we run this test, our test will be green and we can see some docker logs:
![Docker trace](https://cdn.hashnode.com/res/hashnode/image/upload/v1638258755591/uHMrqXhcz.png)

## Test without dev service

First thing, we have to tell Quarkus to not use dev services but only for test profile:
```properties
%test.quarkus.kafka.devservices.enabled=false
```
If I run the test again, there is no docker logs anymore and we have a message saying that the connection to broker could not be etablished.

![No docker trace](https://cdn.hashnode.com/res/hashnode/image/upload/v1638258948558/CgVu9TO2p.png)

## Setup QuarkusTestResourceLifecycleManager

Now we have to start a Kafka at the beginning of our test.

To do that, we will use  [QuarkusTestResourceLifecycleManager](https://quarkus.io/guides/getting-started-testing#quarkus-test-resource) 

We will create a KafkaTestResource that will implement QuarkusTestResourceLifecycleManager and start a Kafka broker there.

We need two dependencies that will contains the Kafka broker:

```xml
    <dependency>
      <groupId>com.salesforce.kafka.test</groupId>
      <artifactId>kafka-junit5</artifactId>
      <version>3.2.3</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.apache.kafka</groupId>
      <artifactId>kafka_2.12</artifactId>
      <version>2.8.0</version>
      <scope>test</scope>
    </dependency>
``` 
Now we can create KafkaTestResource:

```java

public class KafkaTestResource implements QuarkusTestResourceLifecycleManager {

  public static final Logger LOGGER = Logger.getLogger(KafkaTestResource.class);
  
  private SharedKafkaTestResource sharedKafkaTestResource;

  @Override
  public Map<String, String> start() {
    sharedKafkaTestResource = new SharedKafkaTestResource().withBrokers(3)
        .withBrokerProperty("auto.create.topics.enable", "true");

    try {
      sharedKafkaTestResource.beforeAll(null);
    } catch (Exception e) {
      LOGGER.error("Error starting kafka broker", e);
    }
    return Map.of( "kafka.bootstrap.servers", sharedKafkaTestResource.getKafkaConnectString());
  }

  @Override
  public void stop() {
    if (sharedKafkaTestResource != null) {
      sharedKafkaTestResource.afterAll(null);
    }
  }
}

``` 
This SharedKafkaTestResource would normally be used as an extension in Unit Tests with JUnit 5:

```java
@RegisterExtension
    static final SharedKafkaTestResource sharedKafkaTestResource = new SharedKafkaTestResource()
        .withBrokers(1)
        .withBrokerProperty("auto.create.topics.enable", "true");
``` 
and it will call the beforeAll to start the cluster and afterAll to stop it.

As I want to have an Integration Test, I have to call those methods myself.

This KafkaTestResource will return a map that will override the config map:
```java
return Map.of( "kafka.bootstrap.servers", sharedKafkaTestResource.getKafkaConnectString());
```
It is really useful as the port of the broker is not fixed.

Now i just have to call this resource from my test:

```java
@QuarkusTest
@QuarkusTestResource(value = KafkaTestResource.class, restrictToAnnotatedClass = true)
class KafkaServiceTest {
````

I recommend to use restrictToAnnotatedClass in case you have several Integration Tests.

Let's run the test. I see some exception about some module. I will just add this arguments to my test:

![Module error stack](https://cdn.hashnode.com/res/hashnode/image/upload/v1638259890806/LEtSjBmJ6.png)

````
--add-opens java.base/java.lang=ALL-UNNAMED
````
Let's run the test again and see that the Kafka is started.

![Start of kafka](https://cdn.hashnode.com/res/hashnode/image/upload/v1638260044431/QoDUnXCWD.png)

## Create a test

To create a test, I will use Mockito and Awaitility:

```xml
     <dependency>
      <groupId>org.awaitility</groupId>
      <artifactId>awaitility</artifactId>
      <version>4.1.1</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>io.quarkus</groupId>
      <artifactId>quarkus-junit5-mockito</artifactId>
      <scope>test</scope>
    </dependency>
``` 

NB : We can remove quarkus-junit5 dependency as it is included into quarkus-junit5-mockito

For my test, I will inject a Spy of my KafkaService and see if there is some call of consumeData which will mean my integration is working.


```java
import static org.awaitility.Awaitility.await;

import io.quarkus.test.common.QuarkusTestResource;
import io.quarkus.test.junit.QuarkusTest;
import io.quarkus.test.junit.mockito.InjectSpy;
import java.time.Duration;
import org.junit.jupiter.api.Test;
import org.mockito.Mockito;

@QuarkusTest
@QuarkusTestResource(value = KafkaTestResource.class, restrictToAnnotatedClass = true)
class KafkaServiceTest {

  @InjectSpy
  KafkaService kafkaService;

  @Test
  void testKafkaService() {

    var startTime = System.currentTimeMillis();
    await().atMost(Duration.ofSeconds(10)).until(() -> System.currentTimeMillis() - startTime > 9000);

    Mockito.verify(kafkaService, Mockito.atLeastOnce()).consumeData(Mockito.anyString());
  }
}
``` 

I use the await to wait 9 seconds as it can take time before the consumer get registered from the cluster.

I run the test, everything is green!

![Test passing](https://cdn.hashnode.com/res/hashnode/image/upload/v1638260585133/f9nCtw7Ys.png)

## Conclusion

We have seen that we can start a Kafka cluster inside our Integration Tests using  [kafka-junit5](https://github.com/salesforce/kafka-junit/tree/master/kafka-junit5) and a bit of imagination.

I recommend you to go read their github so you can create more complex Integration Tests in the future!




 