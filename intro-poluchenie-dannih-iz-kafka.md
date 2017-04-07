# Получение данных из Kafka

Для того, чтобы начать получать данные из топика, надо на него подписаться, а также задать еще ряд важных параметров, таких как классы для десериализации. 

```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("session.timeout", 10000);
props.put("group.id", "test");
props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
```

Подписка в один клик:

```java
KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);

consumer.subscribe(Arrays.asList("messages"));
```

Хотелось бы иметь потенциально бесконечный consumer, посему **while\(true\) **- наше все.

```java
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(100);
    
    records.forEach(e -> System.out.println("Offset " + e.offset() 
                                           + " key= " + e.key() 
                                         + " value= " + e.value()));
    }
```

В бесконечном цикле производится опрос, а извлеченные записи препарируются на предмет offset/key/value.

