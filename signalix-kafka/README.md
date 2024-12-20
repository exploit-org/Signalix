# Signalix Kafka

## Getting started
First add following dependency to your project:

### Maven
```xml
<dependency>
    <groupId>org.exploit</groupId>
    <artifactId>signalix-kafka</artifactId>
    <version>0.1.3</version>
</dependency>
```

### Gradle
```groovy
implementation("org.exploit:signalix-kafka:0.1.3")
```

## Usage
First you need to create corresponding `KafkaEventScope` instance:

```java
Map<String, String> kafkaProperties = new HashMap<>();
KafkaEventScope kafkaEventScope = new KafkaEventScope(kafkaProperties);
```

### Defining KafkaEventListener
```java
@KafkaEventListener
public class MyKafkaEventListener implements Listener {
    @EventHandler
    public void onEvent(MyEvent event) {
        System.out.println("Received event: " + event.getSampleData());
    }
}
```

Here you can configure:
```java
public @interface KafkaEventListener {
    String[] topics() default {}; // Subscribed topics

    String groupId() default "event-group"; // Group ID

    int concurrency() default 1; // Concurrency of Kafka consumer

    long pollMillis() default 300L; // Milliseconds to poll kafka

    long terminateTimeout() default 1000L; // Milliseconds to terminate the consumer

    String autoOffsetReset() default "earliest"; // Auto offset reset: earliest, latest, none

    String[] properties() default {}; // Additional kafka properties related to consumer
}
```

Now simply register it as default listener:

```java
public static void main(String[] args) {
    Map<String, String> kafkaProperties = new HashMap<String, String>();
    var kafkaEventScope = new KafkaEventScope(kafkaProperties);
    
    kafkaEventScope.registerListener(new MyKafkaListener());
}
```

### Calling Kafka Event
Defining Kafka Event is as simple as creating a class that implements `Event` interface and annotate it with `@KafkaEvent` annotation:

```java
@KafkaEvent
public class MyEvent implements Event {
    private final String sampleData;

    public MyEvent(String sampleData) {
        this.sampleData = sampleData;
    }

    public String getSampleData() {
        return sampleData;
    }
}
```

In @KafkaEvent annotation you can configure:
```java
public @interface KafkaEvent {
    String topic() default DEFAULT_TOPIC; // Topic name

    String[] headers() default {}; // Producer headers

    int partition() default 0; // Partition number

    String DEFAULT_TOPIC = "events";
}
```

Now you can call your event from KafkaEventScope:
```java
public static void main(String[] args) {
    Map<String, String> kafkaProperties = new HashMap<String, String>();
    var kafkaEventScope = new KafkaEventScope(kafkaProperties);
    
    kafkaEventScope.call(new MyEvent("Hello World!"));
}
```

Alternatively, you can specify topicId externally:
```java
kafkaEventScope.call("my-topic", new MyEvent("Hello World!"));
```