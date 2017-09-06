# DataSet API за 5 минут

Предположим вы сидите под Windows и создали проект в IDEA, установив Scala Plugin.

File-&gt; New-&gt;Project-&gt;Scala+Sbt-&gt;Выберите версию Scala/Sbt\(например Sbt 0.13.15 и Scala 2.11.11\)

В файле build.sbt нужно добавить зависимости

```Scala
name := "Spark-sample"

version := "1.0"

scalaVersion := "2.11.11"

libraryDependencies += "org.apache.spark" % "spark-core_2.11" % "2.2.0"
libraryDependencies += "org.apache.spark" % "spark-sql_2.11" % "2.2.0"

```

#### Код-каркас для исполнения

На самом деле лучше всего стартовать с Zeppelin, описанном в следующей главе.

Но если вы решили заодно освоить проектостроение со Scala/Sbt, то заготовка класса вам пригодится.

```Scala
object Skeleton {
  def main(args: Array[String]): Unit = {
    //For windows only: don't forget to put winutils.exe to c:/bin folder
    System.setProperty("hadoop.home.dir", "c:\\")

    val spark = SparkSession.builder
      .master("local")
      .appName("RDD_Intro")
      .getOrCreate()

 
  }

}
```

В этом примере ищется несуществующий Hadoop под Windows \(это рудимент Spark\), затем поднимается контекст Spark-а и настраивается конфигурация.

Затем мы объявим case-класс, аналог Java Bean и объявим Scala-коллекцию с данными, которые будут использоваться для инициализации нашего DataSet.

```
   case class Engineer(name: String, programmingLang: String, level: Long)
   ...
  
   import spark.implicits._
   val engineers = Seq(Engineer("Petja", "Java", 3), 
                       Engineer("Vasja", "Java", 2), 
                       Engineer("Masha", "Scala", 2)).toDS()
```

Благодаря тому, что мы имеем перед глазами схему данных прямо на этапе компиляции \[in compile-time\], нам доступны разные операции, имеющие представление о внутренней структуре данных.



```Scala
    val result = engineers
      .map(x => (x.programmingLang, x.level, x.name)) // 1
      .filter(z => z._1 == "Java")                    // 2
      .groupBy("_1")                                  // 3
      .avg("_2")                                      // 4
      .sort($"avg(_2)".asc)                           // 5    

    result.show()                                     // 6
    
+----+-------+
|  _1|avg(_2)|
+----+-------+
|Java|    2.5|
+----+-------+
```

Давайте разберем по строкам

1. Все записи в Dataset имеют тип Engineer, поэтому мы можем обращаться к полям по имени.
2. Полученные в результате предыдущей операции 3-элементные кортежи фильтруются по языку программирования.
3. Затем группируются по этому языку.
4. Тут идет завершение операции группировки указанием агрегата: вычисления среднего вдоль каждого ключа.
5. А тут мы видим сортировку с обращением по имени при помощи $ \(это возможно при импорте spark.implicits.\_\) и возможностью вызывать на колонке дополнительные операции, такие как направление сортировки.
6. А здесь выводится результат \(и на самом деле начинаются все те ленивые вычисления, накиданные ранее\)

В результате мы получаем одну строку \(спасибо, фильтрация\).

Надеюсь, что это быстрое Intro только возбудит вас интерес к дальнейшему прочтению этой книги.



