---
title: Kafka Learning
date: 2021-01-11 18:02:25
tags:
- Kafka
---

# Start Kafka
```
$ ./bin/zookeeper-server-start.sh ./config/zookeeper.properties
$ ./bin/kafka-server-start.sh ./config/server.properties
$ ./bin/kafka-topics.sh --create --topic test-topic --replication-factor 1 --partitions 1 --zookeeper localhost:2181
$ ./bin/kafka-topics.sh --bootstrap-server localhost:9092 --alter --topic test-topic --partitions 10
```

# Create New Project

Run:
```
$ mvn archetype:generate -DgroupId=org.torapture.test -DartifactId=kafka-test -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4 -DinteractiveMode=false
```

To use the producer and consumer, you can use the following maven dependency:
```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>2.7.0</version>
</dependency>
```

After modified, the file structure is:
```
$ tree kafka-test
kafka-test
├── kafka-test.iml
├── pom.xml
└── src
    ├── main
    │   └── java
    │       └── org
    │           └── torapture
    │               └── test
    │                   ├── Consumer.java
    │                   └── Producer.java
    └── test
        └── java
            └── org
                └── torapture
                    └── test
                        └── AppTest.java

```

`Producer`:
```java
package org.torapture.test;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;

import java.util.Properties;

public class Producer {
    public static void onCompletion(RecordMetadata meta, Exception e) {
        if (e != null) {
            e.printStackTrace();
        } else {
            System.out.printf("offset: %d, partition: %d, topic: %s%n", meta.offset(), meta.partition(), meta.topic());
        }
    }

    public static void main(String[] args) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

        KafkaProducer<String, String> producer = new KafkaProducer<>(props);

        for (int i = 0; i < 10; i++) {
            String key = String.valueOf(i);
            String value = String.format("value is %d", i);
            ProducerRecord<String, String> record = new ProducerRecord<>("test-topic", key, value);
            producer.send(record, Producer::onCompletion);
        }
        producer.close();
    }
}
```

`Consumer`:
```java
package org.torapture.test;

import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.errors.WakeupException;

import java.util.Collections;
import java.util.Properties;

class Wakeup<K, V> implements Runnable {
    KafkaConsumer<K, V> consumer;

    Wakeup(KafkaConsumer<K, V> consumer) {
        this.consumer = consumer;
    }

    @Override
    public void run() {
        try {
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        consumer.wakeup();
    }
}

public class Consumer {
    public static void main(String[] args) throws InterruptedException {
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("group.id", "test-group");
        props.put("auto.offset.reset", "earliest");

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        consumer.subscribe(Collections.singletonList("test-topic"));

        Thread wakeup = new Thread(new Wakeup<>(consumer));
        wakeup.start();

        try {
            while (true) {
                ConsumerRecords<String, String> records = consumer.poll(100);
                for (ConsumerRecord<String, String> record : records) {
                    System.out.printf("topic = %s, partition = %d, offset = %d, key = %s, value = %s%n",
                            record.topic(), record.partition(), record.offset(), record.key(), record.value());
                }
            }
        } catch (WakeupException e) {

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            consumer.close();
        }

        wakeup.join();
    }
}
```

# Produce and Consume

Build the project first by:
```
mvn package
```

Then:
```
$ mvn exec:java -Dexec.mainClass=org.torapture.test.Producer                                        
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by com.google.inject.internal.cglib.core.$ReflectUtils$1 (file:/usr/share/maven/lib/guice.jar) to method java.lang.ClassLoader.defineClass(java.lang.String,byte[],int,int,java.security.ProtectionDomain)
WARNING: Please consider reporting this to the maintainers of com.google.inject.internal.cglib.core.$ReflectUtils$1
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
[INFO] Scanning for projects...
[INFO] 
[INFO] -------------------< org.torapture.test:kafka-test >--------------------
[INFO] Building kafka-test 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- exec-maven-plugin:3.0.0:java (default-cli) @ kafka-test ---
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
offset: 0, partition: 3, topic: test-topic
offset: 1, partition: 3, topic: test-topic
offset: 2, partition: 3, topic: test-topic
offset: 0, partition: 0, topic: test-topic
offset: 0, partition: 8, topic: test-topic
offset: 1, partition: 8, topic: test-topic
offset: 0, partition: 9, topic: test-topic
offset: 1, partition: 9, topic: test-topic
offset: 0, partition: 6, topic: test-topic
offset: 0, partition: 7, topic: test-topic
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  0.848 s
[INFO] Finished at: 2021-01-11T18:56:03+08:00
[INFO] ------------------------------------------------------------------------


$ mvn exec:java -Dexec.mainClass=org.torapture.test.Consumer
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by com.google.inject.internal.cglib.core.$ReflectUtils$1 (file:/usr/share/maven/lib/guice.jar) to method java.lang.ClassLoader.defineClass(java.lang.String,byte[],int,int,java.security.ProtectionDomain)
WARNING: Please consider reporting this to the maintainers of com.google.inject.internal.cglib.core.$ReflectUtils$1
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
[INFO] Scanning for projects...
[INFO] 
[INFO] -------------------< org.torapture.test:kafka-test >--------------------
[INFO] Building kafka-test 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- exec-maven-plugin:3.0.0:java (default-cli) @ kafka-test ---
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
topic = test-topic, partition = 3, offset = 0, key = 3, value = value is 3
topic = test-topic, partition = 3, offset = 1, key = 4, value = value is 4
topic = test-topic, partition = 3, offset = 2, key = 9, value = value is 9
topic = test-topic, partition = 0, offset = 0, key = 5, value = value is 5
topic = test-topic, partition = 8, offset = 0, key = 2, value = value is 2
topic = test-topic, partition = 8, offset = 1, key = 6, value = value is 6
topic = test-topic, partition = 9, offset = 0, key = 1, value = value is 1
topic = test-topic, partition = 9, offset = 1, key = 7, value = value is 7
topic = test-topic, partition = 6, offset = 0, key = 0, value = value is 0
topic = test-topic, partition = 7, offset = 0, key = 8, value = value is 8
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  10.663 s
[INFO] Finished at: 2021-01-11T18:56:17+08:00
[INFO] ------------------------------------------------------------------------
```
