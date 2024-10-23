# Signalix

Signalix is a lightweight event handling library for Java that allows you to develop event-driven applications using a simple event logging and consumption mechanism.

## Getting Started

### Maven
Add the following dependency to your project:
```xml
<dependency>
    <groupId>org.exploit</groupId>
    <artifactId>signalix</artifactId>
    <version>0.1</version>
</dependency>
```

### Gradle
```groovy
implementation("org.exploit:signalix:0.1")
```

### Usage
#### Creating an Event
To get started, we first need to create the events that we will handle and a listener class for them.

```java
import org.exploit.signalix.marker.Event;

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

#### Listener class
After creating the events, we need to declare a listener class where our events will be processed. The class should implement the `Listener` interface.

Each listener class can have an unlimited number of event handlers regardless of their type.
```java
import org.exploit.signalix.marker.Listener;
import org.exploit.signalix.annotation.EventHandler;

public class MyListener implements Listener {
    @EventHandler
    public void onMyEvent(MyEvent e) {
        System.out.println("Handling my event: " + e.getSampleData());
    }
}
```

You can also define event listener in a functional way:
```java
public static void main(String[] args) {
    EventScope scope = new EventScope();
    
    scope.registerEvent(MyEvent.class, (e) -> {
        System.out.println("Handling event: " + e.getSampleData());
    });
}
```

#### Calling events
Finally, we need to create an instance of `EventScope`, register the listener, and call our event.
```java
import org.exploit.signalix.manager.EventScope;

public class Main {
    public static void main(String[] args) {
        var scope = new EventScope();
        scope.registerListener(new MyListener());
        
        scopa.call(new MyEvent("Hello World!"));
    }
}
```

To call event in a async way, use `callAsync`
```java
CompletableFuture<Void> call = scope.callAsync(new MyEvent(""));
```
___

You can handle the same event in multiple methods.
Event Handler's order can be controlled by priority, an event can extend `Cancellable` to stop further handlers execution.

```java
import org.exploit.signalix.marker.Event;
import org.exploit.signalix.event.Cancellable;

public class MyEvent implements Event, Cancellable {
    private boolean cancelled = false;
    
    private final String sampleData;

    public MyEvent(String sampleData) {
        this.sampleData = sampleData;
    }

    public String getSampleData() {
        return sampleData;
    }

    @Override
    public boolean isCancelled() {
        return cancelled;
    }

    @Override
    public void setCancelled(boolean cancelled) {
        this.cancelled = cancelled;
    }
}
```

Now let's try to create 2 event handlers where one cancels the next:
We will set the `EventPriority` for the handlers.

```java
import org.exploit.signalix.marker.Listener;
import org.exploit.signalix.annotation.EventHandler;

public class MyListener implements Listener {
    @EventHandler(priority = EventPriority.HIGHEST, ignoreCacnelled = true) // Will be executed last
    public void onMyEvent0(MyEvent e) {
        System.out.println("Will be executed although event is cancelled");
    }

    @EventHandler(priority = EventPriority.NORMAL) // will be executed second
    public void onMyEvent1(MyEvent e) {
        System.out.println("Will not be executed, as event is cancelled");
    }

    @EventHandler(priority = EventPriority.LOWEST) // Will be executed first
    public void onMyEvent2(MyEvent e) {
        System.out.println("Handling my event: " + e.getSampleData);
        e.setCancelled(true);
    }
}
```

## License
Signalix is distributed under the [BSD 2-Clause License](LICENSE.md)