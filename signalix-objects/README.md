# Signalix Objects
Signalix objects is a factory for creating event-driven objects, where event executing is controlled by `EventScope`.

## Getting started
First add following dependency to your project:

### Maven
```xml
<dependency>
    <groupId>org.exploit</groupId>
    <artifactId>signalix-objects</artifactId>
    <version>0.1.3</version>
</dependency>
```

### Gradle
```groovy
implementation("org.exploit:signalix-objects:0.1.3")
```
## Usage
First you need to have a created `EventScope` instance (either default or KafkaEventScope) and `SignalixObjectFactory`:

```java
EventScope eventScope = new EventScope();
SignalixObjectFactory factory = new SignalixObjectFactory(eventScope);
```

Now let's define several events to imagine some logic we'll create:

- `ActionPerformedEvent` - we will call it every time some action is performed
- `SessionTimeoutEvent` - we will call it wen session is timed out
- `SessionErrorEvent` - will be called every time some error occurs

```java
public class ActionPerformedEvent implements Event {
    private String sessionId;
    private String action;

    public String getSessionId() {
        return sessionId;
    }

    public String getAction() {
        return action;
    }
}
```

```java
public class SessionErrorEvent implements Event {
    private String sessionId;
    private Exception exception;

    public String getSessionId() {
        return sessionId;
    }

    public Exception getException() {
        return exception;
    }
}
```

```java
public class SessionTimeoutEvent implements Event {
    private String sessionId;

    public void setSessionId(String id) { this.sessionId = id; }

    public String getSessionId() {
        return sessionId;
    }
}
```

After we have defined our events, let's define a listener for them:

```java
public class MyListener implements Listener {
    @EventHandler
    public void onActionPerformedEvent(ActionPerformedEvent e) {
        System.out.println("Handling action: " + e.getAction());
    }

    @EventHandler
    public void onSessionErrorEvent(SessionErrorEvent e) {
        System.out.println("Handling error: " + e.getException());
    }

    @EventHandler
    public void onSessionTimeoutEvent(SessionTimeoutEvent e) {
        System.out.println("Handling timeout: " + e.getSessionId());
    }
}
```

Now we will define some `UserSession`, that will be wrapped by Signalix object factory:
```java
@Temporary(eventClass = SessionTimeoutEvent.class)
public class UserSession {
    @LiveTime
    private final long delay = 3000;

    private String sessionId;
    
    @OnCall(eventClass = ActionPerformedEvent.class)
    public void performAction(@MethodParam("action") String action) {

    }

    @OnException(eventClass = SessionErrorEvent.class)
    public void exceptional() {
        throw new RuntimeException("Something went wrong!");
    }

    public void setSessionId(String sessionId) {
        this.sessionId = sessionId;
    }

    public String getSessionId() { return sessionId; }
}
```

**Some key points**:
- `@Temporary` annotation is used to define event, that will be called after some "live" time.
- `@LiveTime` shows the live time in milliseconds of the object. After this time, event specified in @Temporary annotation will be called.
- `@OnCall` - is used to define event, that will be called when the annotated method is called.
- `@OnException` - is used to define event, that will be called when the annotated method throws an exception.
- `@MethodParam` - is used to define names of parameters in method arguments, to be injected in event object.

**Event fields**:
Events can inject fields from the class, from which the event is called and from the @MethodParam arguments for annotated methods.
To extend class field, **name the field with the same name as the class**.

For @OnException, you can inject an exception into event simply naming the field as **`exception`**.

Simply talking, `sessionId` field from `UserSession` will be injected into `ActionPerformedEvent` and `SessionTimeoutEvent` objects, if specified.
