# Публикация данных в Kafka

Написать простой поставщик данных в Kafka несложно, но давайте определимся с тем, куда он будет писать, что он будет писать и какими порциями. Все эти параметры могут быть настроены программно при создании экземпляра **KafkaProducer**

```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("batch.size", 65536);
props.put("buffer.memory", 10000000);
props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
```

Так как во всех примерах Kafka поднята локально, на порту по умолчанию, то первый параметр очевиден. Далее мы конфигурируем размер батча и буфера в памяти, на который **Producer **может рассчитывать.

Кроме того, для пары ключ-значения нужно указать классы из пакета **Kafka **ответственных за сериализацию значений.

Самое время объявить и инициализировать нашу черную дыру, попутно отправив туда чумодан с пропертями:

```java
KafkaProducer<String, String> producer = new KafkaProducer<>(props);
```

Пришло время, используя **producer**, отправить наши данные \(используя очередное значение счетчика в качестве ключа и значения\)

```java
 String topicName = "messages";

 for (int i = 0; i < 100; i++) {
            producer.send(new ProducerRecord<>(topicName, Integer.toString(i), Integer.toString(i)));

 }
```

В качестве входного параметра нужно передать имя топика \(хотя данные будут писаться в конкретную партицию\), в нашем случае это имя "messages".

#### Расширенный контроль над происходящим

Когда  необходимо контролировать, что и куда было отправлено, какой offset был сгенерирован, стоит создать и передать в качестве параметра метода _.send\(\)_ свой собственный Callback с методом _.onCompletion\(\)._

А уже через геттеры параметра metadata можно достать много интерсной информации, среди которых есть и номер партиции, в которую реально были записаны данные, а также timestamp, который нам еще пригодится в будущем.

```java
 producer.send(new ProducerRecord<>(topicName, Integer.toString(i), Integer.toString(i)), 
                    (metadata, exception) -> System.out.println("Topic: " + metadata.topic() +
                            " offset: " + metadata.offset() +
                            " partition #: " + metadata.partition() +
                            " timestamp: " + metadata.timestamp()));
```

Можно обработать и исключительную ситуацию, но это уже на любителя.

#### Тотальный контроль над записью

Под контрольным тоталем предлагается понимать возможность жестко указывать в какую партицию топика нужно писать \(и тут мы подбираемся к использованию наших знаний о внутреннем устройстве Kafka\).

Для этого нам нужно добавить специальный класс, воплощающий интерфейс **Partitioner **и передать метаинформацию о нем в качестве параметра.

Причем этот класс может содержать в себе весьма сложную условную логику маршрутизации данных в зависимости от топика и самих данных, которые приходят в метод _.partition\(\). _

```java
public class UnfairPartitioner implements Partitioner {
    @Override
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
        if(topic.equals("messages")){
            if(Integer.valueOf((String) key) % 10 == 2) {
                return 0;
            } else {
                return 1;
            }
        }
        return 1;
    }

    @Override
    public void close() {

    }

    @Override
    public void configure(Map<String, ?> configs) {

    }
}
```

В этом примере 1/10 часть данных отправляется в первую партицию, 9/10 - во вторую, а в третью и последующие не отправляется ничего.

Параметр, в которым мы переопределим отвественного за маршрутизацию выглядит следующим образом:

```java
props.put("partitioner.class", UnfairPartitioner.class);
```

Запустив пример с выводом на экран метаданных, можно убедиться, что данные теперь иначе расбегаются по партициям. 





