# Быстрый старт со Structured Streaming

В Spark 2.0 появился экспериментальный API под названием Structured Streaming. В версии 2.1 появились новые фичи, которые требуют пристального внимания. Не смотря на это, данный API до сих пор находится в версии alpha.

Давайте вместе попробуем понять, насколько удобно с ним работать.

#### Проект SparkKafka и его зависимости

Создадим sbt проект SparkKafka, где и попробуем в режиме песочницы возможности экспериментального API.

Начнем с зависимостей.

```scala
name := "SparkKafka"

version := "1.0"

scalaVersion := "2.11.8"
libraryDependencies += "org.apache.spark" % "spark-core_2.11"  % "2.1.0"
libraryDependencies += "org.apache.spark" % "spark-sql_2.11"  % "2.1.0"
libraryDependencies += "org.apache.spark" % "spark-streaming_2.11"  % "2.1.0"
libraryDependencies += "org.apache.spark" % "spark-streaming-kafka-0-10_2.11" % "2.1.0"
libraryDependencies += "org.apache.spark" % "spark-sql-kafka-0-10_2.11" % "2.1.0"
```

Да, на момент написания статьи можно использовать и scala 2.12, но автор предпочел надежную\(sic!\) 2.11 и библиотечки, скомпилированные именно под эту версию.

`Обратите внимание на именование зависимостей в соотвествии с версией Scala. Если вы решите это изменить, то вам может понадобиться другая зависимость `**`spark-core_2.10`**`, например. Если же вы захотите перейти на другую версию Spark-а, то нужно будет поменять только конец строки:  "2.1.0" -> "2.2.0".`

#### Потребитель данных \(Consumer\) из Kafka

Предположим, что кто-то сейчас заливает данными наш топик **messages**. Нам остается только подписаться на данный топик и наслаждаться приемом данных.

Начнем с импорта ** SparkSession **- важного объекта, основной точки входа при работе со Spark-ом.

```scala
import org.apache.spark.sql.SparkSession

object KafkaConsumerWithStructuredStreaming {
  def main(args: Array[String]): Unit = {
```

Затем необходимо создать и сконфигурировать экземпляр ** SparkSession**.

```scala
  val spark  = SparkSession.builder
      .master("local")
      .appName("SparkKafka")
      .getOrCreate()
```

Теперь можно приступать к строительству трубопровода из Kafka, например, на консоль. Сконфигурируем для начала источник.

```scala
  val stream = spark
      .readStream
      .format("kafka")   // тип источника
      .option("kafka.bootstrap.servers", "localhost:9092, <your_other_host>:9092") // URL до Kafka
      .option("subscribe", "messages") // ну и наш топик
      .load()
```

Предположим, мы просто хотим извлечь пары key-value для всех записей, которые падают в нашу потенциально-бесконечную таблицу \(как бы состоящую из двух колонок key, value\).

Для этого нам надо применить к этой таблице оператор select и преобразовать наши ints в строки, для чего нам явно пригодятся [имплиситы](https://habrahabr.ru/post/209850/).

```scala
   import spark.implicits._

    val result = stream
      .selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")
      .as[(String, String)]
```

Теперь самое время "записать" куда-то этот стрим, например на консоль.

За "запись" у нас отвечает оператор **writeStream**

```scala
    val finalization = result
      .writeStream
      .format("console")
      .start() // старт нашего стриминга, без этого ничего не поедет

    finalization.awaitTermination() // а без этого ничего не будет длится
```

Вот и готов у нас первый трубопровод для данных. Если данных будет течь по нему слишком много, оператор **selectExpr **будет выбирать только top-20 записей, дабы не перегружать консоль.

