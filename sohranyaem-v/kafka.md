# Как залить данные в Kafka?

Эта возможность появилась только в Spark 2.2, который на момент написания статьи доступен через ночные билды.

Поправим немного build.sbt 

```scala
name := "Spark Streaming"

version := "1.0"

scalaVersion := "2.11.8"

resolvers += "ASF repository" at "http://repository.apache.org/snapshots"

libraryDependencies += "org.apache.spark" % "spark-core_2.11" % "2.2.0-SNAPSHOT"
libraryDependencies += "org.apache.spark" % "spark-sql_2.11" % "2.2.0-SNAPSHOT"
libraryDependencies += "org.apache.spark" % "spark-streaming_2.11" % "2.2.0-SNAPSHOT"
libraryDependencies += "org.apache.spark" % "spark-streaming-kafka-0-10_2.11" % "2.2.0-SNAPSHOT"
libraryDependencies += "org.apache.spark" % "spark-sql-kafka-0-10_2.11" % "2.2.0-SNAPSHOT"
```

Наша цель построить цепочку: 

* запись в топик Messages =&gt; 
* чтение из этого топика =&gt; 
* обработка данных {join со статическим датасетом, группировка данных, сортировка,} =&gt; 
* загрузка данных в топик secondaryMessages =&gt; 
* чтение из этого топика в другом задании для Spark Cluster =&gt; 
* вывод на экран



Начнем с чтения из топика

```scala
import org.apache.spark.sql.SparkSession
import org.apache.spark.sql.streaming.OutputMode
import org.apache.spark.sql.types.StringType


object Kafka_to_Kafka {
  def main(args: Array[String]): Unit = {


    val spark = SparkSession.builder
      .master("local[2]")
      .appName("SparkKafka")
      .getOrCreate()

    spark.sparkContext.setLogLevel("ERROR")

    val stream = spark
      .readStream
      .format("kafka")
      .option("kafka.bootstrap.servers", "localhost:9092")
      .option("subscribe", "messages")
      .option("failOnDataLoss", "false") // полезная опция при работе с Kafka
      .load()
      
    
```

Обработаем данные

```scala
    import spark.implicits._

    val dictionary = Seq(Country("1", "Russia"), Country("2", "Germany"), Country("3", "USA")).toDS()

    val join = stream.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")
      .selectExpr("CAST(key as STRING)", "CAST(value AS INT)")
      .join(dictionary, "key")

    val result = join.select($"country", $"value")
      .groupBy($"country")
      .sum("value")
      .select($"country".alias("key"), $"sum(value)".alias("value").cast(StringType))
      .orderBy($"key".desc)
```

Настроим запись в Kafka

```scala
    val writer = result.writeStream
      .format("kafka")
      .option("kafka.bootstrap.servers", "localhost:9092")
      .option("topic", "secondaryMessages")
      .option("checkpointLocation", "/home/zaleslaw/checkpoints")
      .outputMode(OutputMode.Complete())
      .queryName("kafkaStream")
      .start()


    writer.awaitTermination()
```

Нам также понадобится указать checkpointLocation \(выбирайте всегда отдельные директории для отдельных задач\), что позволит нам приблизиться к заветной Fault Tolerance

Ну и запустим параллельную таску, которая будет черпать уже агрегированные данные из Kafka. Это мы уже делали в главе про [быстрый старт](https://zaleslaw.gitbooks.io/data-processing-book/content/basic-structured-streaming-bistrii-strat.html).

```scala
import org.apache.spark.sql.SparkSession
import org.apache.spark.sql.streaming.ProcessingTime


object Kafka_to_Console {
  def main(args: Array[String]): Unit = {


    val spark = SparkSession.builder
      .master("local[2]")
      .appName("SparkKafka")
      .getOrCreate()

    spark.sparkContext.setLogLevel("ERROR")

    val stream = spark
      .readStream
      .format("kafka")
      .option("kafka.bootstrap.servers", "localhost:9092")
      .option("subscribe", "secondaryMessages")
      .load()

    import spark.implicits._

    val result = stream.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")
      .as[(String, String)]

    val writer = result.writeStream
      .trigger(ProcessingTime(3000))
      .format("console")
      .start()

    writer.awaitTermination()

  }
}

```

Понятно, что теперь ваши возможности по конструированию цепочек \[pipelines\] из кубиков Kafka и Spark - практически ничем не ограничены.

