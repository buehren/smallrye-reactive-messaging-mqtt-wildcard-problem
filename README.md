# Reproduce Error: MQTT subscription topic wildcard stopped working in smallrye-reactive-messaging 2.0.3 (and still does not work in 2.1.0)

When subscribing to an MQTT topic, "+" and "#" can be used as wildcards ([more details](https://mosquitto.org/man/mqtt-7.html)).

This works with smallrye-reactive-messaging 2.0.2 but stops working when switching to 2.0.3 or 2.0.4. 
It still fails in 2.1.0.

## Preparation

### Clone project for demonstrating the error:
```
git clone https://github.com/buehren/smallrye-reactive-messaging-mqtt-wildcard-problem.git
cd smallrye-reactive-messaging-mqtt-wildcard-problem
```

### Start MQTT server (Mosquitto)
```
docker-compose up -d
```


## See error with smallrye-reactive-messaging 2.1.0
```
mvn -Dorg.slf4j.simpleLogger.defaultLogLevel=debug clean package exec:java
```

The output contains lines like these:
```
Sending message on dynamic topic: hello
Sending message on dynamic topic: hello
```

**Problem:** There are no lines with ```received```.


## See it working in 2.1.0 without wildcard

Edit ```src/main/resources/META-INF/microprofile-config.properties``` and activate the line ```smallrye.messaging.source.my-topic.topic=hello``` instead of the line after that (disable the line with ```.topic=#```).

Start again:
```
mvn -Dorg.slf4j.simpleLogger.defaultLogLevel=debug clean package exec:java
```

The output contains lines like these:
```
Sending message on dynamic topic: hello
received: Hello 0 from topic hello
```

**Now it works. So the wildcard was the cause of the problem.**


## See it working in 2.0.2 with wildcard

Edit ```src/main/resources/META-INF/microprofile-config.properties``` and re-activate the line ```smallrye.messaging.source.my-topic.topic=#``` instead of the line before that (disable the line with ```.topic=hello```).

Edit ```pom.xml``` and change the version in ```<smallrye-reactive-messaging.version>2.x.x</smallrye-reactive-messaging.version>``` to ```2.0.2```.

Start again:
```
mvn -Dorg.slf4j.simpleLogger.defaultLogLevel=debug clean package exec:java
```

The output contains lines like these:
```
Sending message on dynamic topic: hello
received: Hello 0 from topic hello
```

Now it works even with a wildcard.

If you change the version from ```2.0.2``` to ```2.0.3``` it fails.

**So the problem must have been introduced in ```2.0.3```.**


## Cleanup

### Stop MQTT server (Mosquitto)
```
docker-compose down
```
