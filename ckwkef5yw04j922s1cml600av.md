# Today I learned OpenTelemetry with SmallRye reactive messaging and KafkaStream

Finally my application is on production : several micro-services using event-driven architecture where several producers can trigger same behaviour.

Suddenly a bug! Some triggers are weird, but who is triggering them? Which micro-services?
Impossible to tell...so I put logs in the code trying to point the issue, but it is hard to isolate the root cause in a system that produces thousands of message per second.

Then one of my coworkers mention OpenTelemetry and Jaeger...

So I look a bit, and to make it simple, I understand that OpenTelemetry track a call by using headers and Jaeger is the tool to visualise them.

![spans-traces.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1638172656833/MTbJcAHtx.png)

Seems like it is the perfect solution!

But to every perfect solution comes some unexpected issues...

## The stack

The services are Quarkus apps where some use : 

- SmallRye Reactive Messaging annotations Incoming / Outgoing
- Apache KafkaStream

## SmallRye Reactive Messaging
To use OpenTelemetry with SmallRye Reactive Messaging, you can follow Quarkus guide  [here](https://quarkus.io/guides/opentelemetry) 
I will summarise it as what is interesting for us is at the beginning and the end of the guide.

I have to add those dependencies in the pom:
```xml
    <dependency>
      <groupId>io.quarkus</groupId>
      <artifactId>quarkus-opentelemetry-exporter-jaeger</artifactId>
    </dependency>
    <dependency>
      <groupId>io.quarkus</groupId>
      <artifactId>quarkus-opentelemetry</artifactId>
    </dependency>
```
And configure my exporter in applications.properties
````properties
quarkus.application.name=kafka-opentelemetry
quarkus.opentelemetry.enabled=true
quarkus.opentelemetry.tracer.exporter.otlp.endpoint=http://localhost:4317
````
To start the jaeger locally, I followed  [this chapter](https://quarkus.io/guides/opentelemetry#run-the-application) .
I add to fix the port setup to make it works:
````yaml
jaeger-all-in-one:
    image: jaegertracing/all-in-one:latest
    ports:
      - "16686:16686"
      - "14268:14268"
      - "14250:14250"
````
So if you have a method that only consume or produce data, then the lib does the job.
When you have a method that consume and produce data, then the trouble starts.

To show you the issue, I have done this example: 

1.  produceData produces a "SmallRye Hello X" messages every second (produceData) on channel "demo-kafka-tracing-string-out"

2. toUpperCase consume from this channel, transform the String to uppercase and send the new String to channel "demo-kafka-tracing-uppercase-out"
3. sortString will consume from this channel, sort the String and send the sorted value to channel "demo-kafka-tracing-sort-out"
4. displayMessage will consume from last channel and log the value.

````java
 @ApplicationScoped
public class KafkaTracingService {

  public static final Logger LOGGER = Logger.getLogger(KafkaTracingService.class);

  @Outgoing("demo-kafka-tracing-string-out")
  public Multi<Message<String>> produceData() {
    return Multi.createFrom().ticks().every(Duration.ofSeconds(1)).map(tick -> Message.of("SmallRye Hello " + tick));
  }

  @Incoming("demo-kafka-tracing-uppercase-in")
  @Outgoing("demo-kafka-tracing-uppercase-out")
  public Message<String> toUpperCase(ConsumerRecord<String, String> message) {
    return Message.of(message.value().toUpperCase(Locale.ROOT));
  }

  @Incoming("demo-kafka-tracing-sort-in")
  @Outgoing("demo-kafka-tracing-sort-out")
  public Message<String> sortString(ConsumerRecord<String, String> message) {
    var tempArray = message.value().toCharArray();
    Arrays.sort(tempArray);
    return Message.of(new String(tempArray));
  }

  @Incoming("demo-kafka-tracing-display-in")
  public void displayMessage(ConsumerRecord<String, String> message) {
    LOGGER.infov("Message received {0}", message.value());
  }
}
````
And the properties for that:
````properties
mp.messaging.outgoing.demo-kafka-tracing-string-out.connector=smallrye-kafka
mp.messaging.outgoing.demo-kafka-tracing-uppercase-out.connector=smallrye-kafka
mp.messaging.outgoing.demo-kafka-tracing-sort-out.connector=smallrye-kafka
mp.messaging.incoming.demo-kafka-tracing-uppercase-in.connector=smallrye-kafka
mp.messaging.incoming.demo-kafka-tracing-sort-in.connector=smallrye-kafka
mp.messaging.incoming.demo-kafka-tracing-display-in.connector=smallrye-kafka
mp.messaging.outgoing.demo-kafka-tracing-string-out.topic=string-out
mp.messaging.outgoing.demo-kafka-tracing-uppercase-out.topic=uppercase-out
mp.messaging.outgoing.demo-kafka-tracing-sort-out.topic=sort-out
mp.messaging.incoming.demo-kafka-tracing-uppercase-in.topic=string-out
mp.messaging.incoming.demo-kafka-tracing-sort-in.topic=uppercase-out
mp.messaging.incoming.demo-kafka-tracing-display-in.topic=sort-out
mp.messaging.outgoing.demo-kafka-tracing-string-out.value.serializer=org.apache.kafka.common.serialization.StringSerializer
mp.messaging.outgoing.demo-kafka-tracing-uppercase-out.value.serializer=org.apache.kafka.common.serialization.StringSerializer
mp.messaging.outgoing.demo-kafka-tracing-sort-out.value.serializer=org.apache.kafka.common.serialization.StringSerializer
mp.messaging.incoming.demo-kafka-tracing-uppercase-in.value.deserializer=org.apache.kafka.common.serialization.StringDeserializer
mp.messaging.incoming.demo-kafka-tracing-sort-in.value.deserializer=org.apache.kafka.common.serialization.StringDeserializer
mp.messaging.incoming.demo-kafka-tracing-display-in.value.deserializer=org.apache.kafka.common.serialization.StringDeserializer

````
When the application is running, the trace looks like that:

![Trace without chain](https://cdn.hashnode.com/res/hashnode/image/upload/v1638171015669/Oa2VCdYmP.png)

If we open one string-out for example, we only see two span
![Screenshot 2021-11-29 at 08.36.29.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1638171394134/g-uuyNnYC.png)

So the guide says to add:
```java
Metadata.of(TracingMetadata.withPrevious(Context.current()));
```
I tried, but it does not fix our issue.
The correct thing to add to our Consumer/Producer is :
````java
TracingMetadata.withCurrent(Context.current())
````
Now the code is:
```java
@Outgoing("demo-kafka-tracing-string-out")
  public Multi<Message<String>> produceData() {
    return Multi.createFrom().ticks().every(Duration.ofSeconds(1)).map(tick -> Message.of("SmallRye Hello " + tick));
  }

  @Incoming("demo-kafka-tracing-uppercase-in")
  @Outgoing("demo-kafka-tracing-uppercase-out")
  public Message<String> toUpperCase(ConsumerRecord<String, String> message) {
    return Message.of(message.value().toUpperCase(Locale.ROOT)).addMetadata(TracingMetadata.withCurrent(
        Context.current()));
  }

  @Incoming("demo-kafka-tracing-sort-in")
  @Outgoing("demo-kafka-tracing-sort-out")
  public Message<String> sortString(ConsumerRecord<String, String> message) {
    var tempArray = message.value().toCharArray();
    Arrays.sort(tempArray);
    return Message.of(new String(tempArray)).addMetadata(TracingMetadata.withCurrent(
        Context.current()));
  }

  @Incoming("demo-kafka-tracing-display-in")
  public void displayMessage(ConsumerRecord<String, String> message) {
    LOGGER.infov("Message received {0}", message.value());
  }
```
And if I run the application again, then I can see this trace:

![Trace with chain](https://cdn.hashnode.com/res/hashnode/image/upload/v1638171690088/HiQMda_sP.png)
And if i select one string-out, i will see the whole chain.
![Trace with spans](https://cdn.hashnode.com/res/hashnode/image/upload/v1638171722731/dW_YyE6YD.png)

## KafkaStream

For KafkaStream, there is no solution. I had to create my own implementation to make it work. 
I merged the TracingKafkaClientSupplier from opentracing-kafka-stream with KafkaSource / KafkaSink from smallrye-reactive-messaging-kafka.

Now I have my own KafkaClientSupplier that is used by KafkaStream to create Consumer and Producer.

So this is my KafkaClientSupplier. I decided to extends DefaultKafkaClientSupplier instead of duplicate it.
```java
import java.util.Map;
import org.apache.kafka.clients.consumer.Consumer;
import org.apache.kafka.clients.producer.Producer;
import org.apache.kafka.streams.processor.internals.DefaultKafkaClientSupplier;

public class KafkaStreamClientSupplier extends DefaultKafkaClientSupplier {

  @Override
  public Producer<byte[], byte[]> getProducer(Map<String, Object> config) {
    return new TracingKafkaProducer<>(super.getProducer(config));
  }

  @Override
  public Consumer<byte[], byte[]> getConsumer(Map<String, Object> config) {
    return new TracingKafkaConsumer<>(super.getConsumer(config));
  }
}
```
Now I have my two new classes:

- TracingKafkaProducer
- TracingKafkaConsumer
#### TracingKafkaProducer

TracingKafkaProducer is a copy of the same class in opentracing-kafka-client. I have changed the send method to add the tracing part using OpenTelemetry:
````java
@Override
  public Future<RecordMetadata> send(ProducerRecord<K, V> record, Callback callback) {
    createOutgoingTrace(record);
    return producer.send(record, callback);
  }

  private void createOutgoingTrace(ProducerRecord<K, V> message) {

    TracingMetadata tracingMetadata =TracingMetadata.empty();

    if (message.headers() != null) {
      // Read tracing headers
      Context context = GlobalOpenTelemetry.getPropagators().getTextMapPropagator()
          .extract(Context.current(), message.headers(), HeaderExtractAdapter.GETTER);
      tracingMetadata = TracingMetadata.withCurrent(context);
    }

      final SpanBuilder spanBuilder = TRACER.spanBuilder(message.topic() + " send")
          .setSpanKind(SpanKind.PRODUCER);

      if (tracingMetadata.getCurrentContext() != null) {
        // Handle possible parent span
        final Context parentSpanContext = tracingMetadata.getCurrentContext();
        if (parentSpanContext != null) {
          spanBuilder.setParent(parentSpanContext);
        } else {
          spanBuilder.setNoParent();
        }
      } else {
        spanBuilder.setNoParent();
      }

      final Span span = spanBuilder.startSpan();
      Scope scope = span.makeCurrent();

      // Set Span attributes
      if (message.partition() != null) {
        span.setAttribute(SemanticAttributes.MESSAGING_KAFKA_PARTITION, message.partition());
      }
      span.setAttribute(SemanticAttributes.MESSAGING_SYSTEM, "kafka");
      span.setAttribute(SemanticAttributes.MESSAGING_DESTINATION, message.topic());
      span.setAttribute(SemanticAttributes.MESSAGING_DESTINATION_KIND, "topic");

      // Set span onto headers
      GlobalOpenTelemetry.getPropagators().getTextMapPropagator()
          .inject(Context.current(), message.headers(), HeaderInjectAdapter.SETTER);

      span.end();
      scope.close();
  }
````
#### TracingKafkaConsumer
TracingKafkaProducer is a copy of the same class in opentracing-kafka-client. I have changed the poll method to add the tracing part using OpenTelemetry:
````java
 @Override
  public ConsumerRecords<K, V> poll(Duration duration) {
    ConsumerRecords<K, V> records = consumer.poll(duration);

    for (ConsumerRecord<K, V> record : records) {
      incomingTrace(record);
    }

    return records;
  }

  public void incomingTrace(ConsumerRecord<K, V> kafkaRecord) {
    if (TRACER != null) {
      TracingMetadata tracingMetadata =TracingMetadata.empty();
      if (kafkaRecord.headers() != null) {
        // Read tracing headers
        Context context = GlobalOpenTelemetry.getPropagators().getTextMapPropagator()
            .extract(Context.root(), kafkaRecord.headers(), HeaderExtractAdapter.GETTER);
        tracingMetadata = TracingMetadata.withPrevious(context);
      }

      final SpanBuilder spanBuilder = TRACER.spanBuilder(kafkaRecord.topic() + " receive")
          .setSpanKind(SpanKind.CONSUMER);

      // Handle possible parent span
      final Context parentSpanContext = tracingMetadata.getPreviousContext();

      if (parentSpanContext != null) {
        spanBuilder.setParent(parentSpanContext);
      } else {
        spanBuilder.setNoParent();
      }

      final Span span = spanBuilder.startSpan();

      // Set Span attributes
      span.setAttribute(SemanticAttributes.MESSAGING_KAFKA_PARTITION, kafkaRecord.partition());
      span.setAttribute("offset", kafkaRecord.offset());
      span.setAttribute(SemanticAttributes.MESSAGING_SYSTEM, "kafka");
      span.setAttribute(SemanticAttributes.MESSAGING_DESTINATION, kafkaRecord.topic());
      span.setAttribute(SemanticAttributes.MESSAGING_DESTINATION_KIND, "topic");

      final String groupId = consumer.groupMetadata().groupId();
      final String clientId = consumer.groupMetadata().groupInstanceId().orElse("");
      span.setAttribute("messaging.consumer_id", constructConsumerId(groupId, clientId));
      span.setAttribute(SemanticAttributes.MESSAGING_KAFKA_CONSUMER_GROUP, groupId);
      if (!clientId.isEmpty()) {
        span.setAttribute(SemanticAttributes.MESSAGING_KAFKA_CLIENT_ID, clientId);
      }

      span.makeCurrent();

      // Set span onto headers
      GlobalOpenTelemetry.getPropagators().getTextMapPropagator()
          .inject(Context.current(), kafkaRecord.headers(), HeaderInjectAdapter.SETTER);

      span.end();
    }
  }

  private String constructConsumerId(String groupId, String clientId) {
    String consumerId = groupId;
    if (!clientId.isEmpty()) {
      consumerId += " - " + clientId;
    }
    return consumerId;
  }
````
#### Usage

To inject it in our Quarkus apps, I just have to create a Produces bean:
```java
 @Produces
  public KafkaClientSupplier kafkaClientSupplier() {
    return new KafkaStreamClientSupplier();
  }
```

#### Example
For the demo, I have a KafkaStream that reproduce the workflow of the example I have in SmallRye part.
```java
@ApplicationScoped
public class KafkaStreamService {

  public static final Logger LOGGER = Logger.getLogger(KafkaStreamService.class);

  @Outgoing("demo-kafka-stream-string-out")
  public Multi<Message<String>> produceData() {
    return Multi.createFrom().ticks().every(Duration.ofSeconds(1))
      .map(tick -> Message.of("Stream Hello " + tick));
  }

  @Produces
  public Topology buildTopology() {

    var builder = new StreamsBuilder();
    builder.stream(
            "stream-string-out",
            Consumed.with(Serdes.String(), Serdes.String()))
        .mapValues(object -> object.toUpperCase(Locale.ROOT))
        .to("stream-uppercase-out",
            Produced.with(Serdes.String(), Serdes.String()));

    builder.stream(
            "stream-uppercase-out",
            Consumed.with(Serdes.String(), Serdes.String()))
        .mapValues(object -> {
          var tempArray = object.toCharArray();
          Arrays.sort(tempArray);
          return new String(tempArray);
        })
        .to("stream-sort-out",
            Produced.with(Serdes.String(), Serdes.String()));

    builder.stream(
            "stream-sort-out",
            Consumed.with(Serdes.String(), Serdes.String()))
        .process(() -> new Processor<String, String>() {
          @Override
          public void init(ProcessorContext processorContext) {

          }

          @Override
          public void process(String s, String s2) {
            LOGGER.infov("Message received from stream {0}", s2);
          }

          @Override
          public void close() {

          }
        });
    return builder.build();
  }

  @Produces
  public KafkaClientSupplier kafkaClientSupplier() {
    return new KafkaStreamClientSupplier();
  }

}
```
And if I run my app, I will see the trace:
![Stream trace](https://cdn.hashnode.com/res/hashnode/image/upload/v1638172535714/vjjoYnt9_.png)
And the chaining:
![Stream chaining](https://cdn.hashnode.com/res/hashnode/image/upload/v1638172584214/ZQkjRx7cx.png)

## Conclusion
It was not a simple journey, debugging inside of OpenTelemetry lib, to understand OpenTracing lib also and learning about Metadata but at the end, we have a nice tracing solution and now, I can finally go back to investigate my bug !!!
You can find the whole demo project on my  [github](https://github.com/seb-noirot/opentelemetry-smallrye-kafka) 
