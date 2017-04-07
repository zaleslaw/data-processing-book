# Читаем из Kafka

Для того, чтобы прочитать из Kafka и направить эти данные в Spark, надо сконфигурировать чтеца:

```scala
    val kafkaParams = Map[String, Object](
      "bootstrap.servers" -> "localhost:9092",
      "key.deserializer" -> classOf[StringDeserializer],
      "value.deserializer" -> classOf[StringDeserializer],
      "group.id" -> "test",
      "auto.offset.reset" -> "latest",
      "enable.auto.commit" -> (false: Boolean)
    )

```

Здесь присутствуют уже знакомые читателю классы для десериализации данных, адрес кластера **Kafka**, а также настройки, отвечающие за правила потребления из топика.

Затем задается конфигурация и поднимается контекст \(справедливо для Spark 1.\*\).

```scala
    val conf = new SparkConf().setAppName("DStream counter").setMaster("local[2]")
    val ssc = new StreamingContext(conf, Seconds(3))
```

После задается сам стриминг, которые будет запускать свой микробатчинг каждые 3 секунды. Стриминг будет запущен локально \(ресивер и драйвер на одной машине\).

```scala
    ssc.start()
    ssc.awaitTermination()
```

Дело за малым: создать прямой стрим прямо в недра Kafka.

```scala
    val topicName = "messages"
    val topics = Array(topicName)

    val stream = KafkaUtils.createDirectStream(ssc, PreferConsistent,
      Subscribe[String, String](topics, kafkaParams))
```

И описать логику обработки каждой RDD, получаемой после каждого вздоха нашего стриминга.

```scala
    stream.foreachRDD {
      rdd => println("Amount of elems " + rdd.count)
    }
```

В примере на консоль выводится количество элементов в каждой RDD.



