# Logs from Quarkus to Kibana through Kafka

In this article, we'll see how we can produce logs and read them in a dashboard using the following technologies:

* Quarkus
* Log4j2
* Apache Kafka
* Kafka-Connect
* ElasticSearch
* Kibana
* Docker-Compose

## Quarkus

We'll first need to generate a Quarkus application.

In fact, any kind of application will do, at least if it uses Maven, but for conveniency, let's use the same app.

Let's start by navigating to [https://code.quarkus.io/](https://code.quarkus.io/) and generate an application
using [RESTEasy JAX-RS](https://docs.jboss.org/resteasy/docs/3.0.19.Final/userguide/html_single/index.html).

![quarkus-gen](./static/quarkus-generation.png)

## Log4j2

### Dependency

Now that the application is generated, we'll continue by adding the `log4j2` dependency

```
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.7</version>
</dependency>
```

### GreetingResource.java

Now we've done that, we'll modify the class `GreetingResource.java` file and add some logging to it

```
@Path("/hello-resteasy")
public class GreetingResource {

    private static final Logger logger = LogManager.getLogger(GreetingResource.class);

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public String hello() {
        logger.info("Hello called");
        return "Hello RESTEasy";
    }

    @GET
    @Path("error")
    @Produces(MediaType.TEXT_PLAIN)
    public String error() {
        logger.error("Error called");
        return "Error";
    }
}
```

This simply means that, after log4j2 is configured, whenever we'll call `/hello-resteasy`, `Hello called` will get
printed to the console.

And whenever we'll call `/hello-resteasy/error`, `Error called` will get printed to the console.

That might seem nice, but that's not exactly what we want.

We want to integrate with `ElasticSearch`, and it only accepts `JSON` as inputs.

### Log4j2's JsonLayout for Appenders

Luckily enough, `log4j2` comes with a type of layout producing `JSON`.

We'll only need to configure `log4j2` in the following way

```
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="DEBUG">
    <Appenders>
        <Kafka name="Kafka" topic="quarkus-logs">
            <JsonLayout complete="false" locationInfo="true" properties="true" propertiesAsList="true" eventEol="true"/>
            <Property name="bootstrap.servers">${env:KAFKA_BOOTSTRAP_SERVER}</Property>
            <Property name="acks">0</Property>
        </Kafka>
    </Appenders>
    <Loggers>
        <Root level="debug">
            <AppenderRef ref="Kafka"/>
        </Root>
        <Logger name="org.apache.kafka" level="INFO"/>
    </Loggers>
</Configuration>
```

We notice that the `log4j2` appender used is the `KafkaAppender` which will serve as a `Kafka Consumer` out-of-the-box.

The properties used are:

* `bootstrap.servers` : This serves to point to our broker. It will help create the topic and write to it
* `acks` : set at 0 (equivalent to shoot and forget), meaning there is no acknowledgment of message receival. For logs,
  as they're not crucial, it's not needed to have strong delivery guarantees.

We can also notice the usage of `<Logger name="org.apache.kafka" level="INFO"/>` to avoid recursive logging.

It comes from
the [official documentation](https://logging.apache.org/log4j/log4j-2.4/manual/appenders.html#KafkaAppender).

![kafka-appendar](./static/log4j2-kafka-appender-recursive-logging.png)

## Apache Kafka

## ElasticSearch

## Kibana

[https://code.quarkus.io/]: https://code.quarkus.io/