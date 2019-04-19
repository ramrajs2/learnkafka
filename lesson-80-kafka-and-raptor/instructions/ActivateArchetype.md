# Activate the Archetype

## Introduction

The Kafka archetype provides an incomplete implementation of the message daemon.

The good news is that we don't need to do much to get the Kafka archetype to work.

By default, the Kafka archetype uses a test broker array that should be up and running at all time (in the case of YAM, we depended on a Stage II machine that has been setup to test the Raptor framework).

## Complete the Archetype

### Steps

#### 1. Uncomment the integration

The archetype defines a working integration, but it initially comes commented out.

You can find the integration files in the directory `src/main/resources/spring/` inside the daemon project.

There are two integration files.
The `integration.xml` simply imports the `integration-kafka.xml`.

Open the file `src/main/resources/spring/integration-kafka.xml`.

You'll see that the integration definition is commented out.

Uncomment the integration definition.

The file content should now look similar to this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:int="http://www.springframework.org/schema/integration"
       xmlns:int-kafka="http://www.springframework.org/schema/integration/kafka"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/integration
            http://www.springframework.org/schema/integration/spring-integration.xsd
            http://www.springframework.org/schema/integration/kafka
            http://www.springframework.org/schema/integration/kafka/spring-integration-kafka.xsd
    ">


<!-- Below is a sample Kafka consumer configuration.
     Please also update Kafka configuration in your application.properties and application-override.properties.
 -->
    <int-kafka:message-driven-channel-adapter id="kafkaListener" listener-container="container" channel="channelToConsume" error-channel="errorChannel"/>

    <int:publish-subscribe-channel id="errorChannel"/>

    <bean id="container" class="org.springframework.kafka.listener.KafkaMessageListenerContainer">
        <constructor-arg>
            <bean class="org.springframework.kafka.core.DefaultKafkaConsumerFactory">
                <constructor-arg ref="consumerPropertyMap"/>
            </bean>
        </constructor-arg>
        <constructor-arg>
            <bean class="org.springframework.kafka.listener.config.ContainerProperties" parent="paypalKafkaListenerContainerProperties">
                <constructor-arg name="topics" value="${kafka-generic.consumer.topics}"/>
            </bean>
        </constructor-arg>
    </bean>
    <bean id="consumerPropertyMap" class="com.paypal.kafka.spring.integration.ConsumerPropertyMap">
        <constructor-arg>
            <map>
                <entry key="bootstrap.servers" value-ref="consumerServers"/>
                <entry key="group.id" value="${kafka-generic.consumer.group.id}"/>
                <entry key="key.deserializer" value="${kafka-generic.consumer.key.deserializer}"/>
                <entry key="value.deserializer" value="${kafka-generic.consumer.value.deserializer}"/>
            </map>
        </constructor-arg>
    </bean>
    <bean id="consumerServers" class="com.paypal.kafka.spring.integration.BootstrapServersList">
        <constructor-arg name="hosts" value="${kafka-generic_host}"/>
        <constructor-arg name="port" value="${kafka-generic_port}"/>
    </bean>

    <int:channel id="channelToConsume">
        <int:interceptors>
            <bean class="com.paypal.kafka.spring.integration.KafkaSubInterceptor"/>
        </int:interceptors>
    </int:channel>

    <int:service-activator input-channel="channelToConsume" ref="kafkaSampleServiceActivator"/>

</beans>

```

#### 2. Setup a run configuration

The functional test uses JMX to verify that the server has received messages.

We have to make sure that the run configuration of the message daemon is configured in a way that allows the functional test to connect to the daemon.

For that to work, we have to add a few parameters to the run command of the daemon.

In the run configuration for the daemon, add the following parameters to the java :

```
-Dcom.sun.management.jmxremote
-Dcom.sun.management.jmxremote.port=32500
-Dcom.sun.management.jmxremote.authenticate=false
-Dcom.sun.management.jmxremote.ssl=false
```

E.g., in Eclipse, you may set this up in the run configuration (you can create the run configuration manually, or run the application once, stop the application and then modify the launch configuraiton).

<img src="../pics/Kafka_Launch_Config_JMX.png"/>

#### 4. Run the daemon

Start up the daemon (either in your IDE or using maven).

Look at the console (or the output) You should see the server starting up ending with something similar to this:

```
06:18:33.906 [main] INFO org.apache.kafka.common.utils.AppInfoParser Kafka version : 0.9.0.1
06:18:33.906 [main] INFO org.apache.kafka.common.utils.AppInfoParser Kafka commitId : 23c69d62a0cabf06
06:18:33.909 [main] INFO org.springframework.integration.kafka.inbound.KafkaMessageDrivenChannelAdapter started kafkaListener
06:18:33.949 [main] INFO org.springframework.boot.context.embedded.tomcat.TomcatEmbeddedServletContainer Tomcat started on port(s): 8083 (http) 8080 (http)
06:18:33.954 [main] INFO com.paypal.test.RaptorApplication Started RaptorApplication in 4.719 seconds (JVM running for 5.154)
06:18:34.179 [container-kafka-consumer-1] INFO org.springframework.kafka.listener.KafkaMessageListenerContainer partitions revoked:[]
06:18:36.489 [Sherlock-Worker-3-1] INFO org.ebayopensource.sherlock.client.frontier.FrontierChannel Closing connection to sherlock-ftrproxy-vip.qa.paypal.com/10.57.18.185:80; Reason: Idle channel timeout.
06:18:36.517 [Sherlock-Nio-Worker-1-1] INFO org.ebayopensource.sherlock.client.frontier.FrontierClient Disconnected from sherlock-ftrproxy-vip.qa.paypal.com/10.57.18.185:80.
```

We now have a running deamon connected to one of our Kafka test arrays.

#### 5. Run the functional test

Next, we will use TestNG to run a functional test.

The test simply sends a message on a specified topic on the Kafka broker array, then reads a message count on the message daemon using JMX.

The test fails if the message count does not increase.

Run the functional test `KafkaMessagingFunctionalTest` from the functional test project (Run as TestNG test).

## Understanding the starter project

### Functional test

Let's walk through the test project first. Open the `KafkaMessagingFunctionalTest.java` and navigate to the test method.

The first thing you'll see is that the method is reading a message count before the test starts:

```java
//Get the send count of the consumer message channel before publishing a message
Integer sendCountBeforePublishing =
  (Integer) mBeanServerConnection.getAttribute(mbeanName, "SendCount");
```

What happens in this line is that we are connecting to the message daemon and reading the present count of messages that it has received.

Next, it runs into a loop that is waiting checks every `CONSUMPTION_DELAY` to see if the message count has increased.
If we don't see a message count increase before `MAX_TEST_WAIT_TIME` has passed, the test fails.

The only line of code that is not related to test is the line:

```java
//Publish a message to the topic : TOPIC_NAME_FROM_CONSUMER
messagePublisher.publishMessage();
```

This is where we publish the message.

The `messagePublisher` is injected into the test:

```java
@Inject
private KafkaMessagePublisher messagePublisher;
```

The `KafkaMessagePublisher` is a simple utility class written for this test specifically.

If you study this class you'll see some generic Kafka code:

```java
/**
 * Publish a message to the topic : TOPIC_FROM_APPLICATION_DAEMON
 */
public void publishMessage() throws Exception {
    String message="Hello World";
    Producer producer = MakoKafkaProducerFactory.getDefaultProducerInstance();
    ProducerRecord<String, String> record = new ProducerRecord<>(TOPIC_FROM_APPLICATION_DAEMON,  message);
    producer.send(record);
    producer.close();
    logger.info("A message has been published. Its content is " + message);
}
```

Basically, all the code does is to open a connection to Kafka, create a producer and send the `Hello World` message on some the topic `TOPIC_FROM_APPLICATION_DAEMON`.

### The Daemon

The daemon is a bit more complex, but the complexity is that of Kafka.
There is very little PayPal specific code in Daemon, hence, as you lear Kafka, you'll have very little problem understanding the daemon.

Here is a guided tour.

#### Integration file(s)

The first thing we'll look at is the `integraion-kafka.xml` file.
This is the section we commented back in in the lab instruction.

Spring Itegration has a Kafa channel adapter that can connect to Kafka and forward the messages to a specified channel.

```xml
<int-kafka:message-driven-channel-adapter id="kafkaListener" listener-container="container" channel="channelToConsume" error-channel="errorChannel"/>
```

The integration framgment creates a channel adapter and forwards the messages to the `channelToConsume` channel.

You will see the channel is defined in the integration file:

```xml
<int:channel id="channelToConsume">
    <int:interceptors>
        <bean class="com.paypal.kafka.spring.integration.KafkaSubInterceptor"/>
    </int:interceptors>
</int:channel>
```

Next, let's look at the listener to the channel.

The listener is a simple service-activator:

```xml
<int:service-activator input-channel="channelToConsume" ref="kafkaSampleServiceActivator"/>
```

There are a set also some bean definitions.
The bean definitions are standard Spring-Kafka for the most part.
You can read about how to do this (here)[http://docs.spring.io/spring-kafka/reference/htmlsingle/#si-kafka].

We'll focus on the PayPal specific variations.

You'll see in the configuration that one of the constructor arguments is using the class `com.paypal.kafka.spring.integration.ConsumerPropertyMap`.

In a generic Spring/Kafka integration, the `ContainerPropertyMap` class would have been a simple map class.
The reason for using our own is that there may be a need for the framework to inject properties. By having a subclass of a simple map, the framework can introduce settings without you having to worry about updating your project.

You will also see a reference to the `com.paypal.kafka.spring.integration.BootstrapServersList`.
This is a necessity because of the way PayPal specifies server connections.

In Kafka you configure the connection to a broker pool. It is usually a simple list of host and port combinations.
At PayPal, however, we specify this a little bit different.

The `com.paypal.kafka.spring.integration.BootstrapServersList` class simply takes our PayPal format for connections and convert them into a format that Kafka understands.

Except for the two list types above, the Raptor integration with Kafka is completely generic. Hence, any documentation that you find online should be applicable.

#### The Service Activator

The only thing left to discuss is really the service activator.

```java
@Component
public class KafkaSampleServiceActivator {

    private static Logger logger = LoggerFactory.getLogger(KafkaSampleServiceActivator.class);

    /**
     * Integration point of business logic.
     */
    public Message onMessage(Message message) {
        logger.info("message received");
        return message;
    }
}
```

This is a generic Spring Integration service activator that implements `onMessage`.

All it does when it receives the message is to log it.
