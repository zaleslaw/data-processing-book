# Соединение двух RDD

В целом, нам доступны операции теории множеств, такие как объединение, пересечение, разность, и операции join, cogroup, пришедшие к нам из миря реляционной алгебры.

Рассмотрим пример с набором операций из теории множеств, удобных для взаимодействия множеств значений \(в первую очередь\)

```Scala
    //For windows only: don't forget to put winutils.exe to c:/bin folder
    System.setProperty("hadoop.home.dir", "c:\\")

    val spark = SparkSession.builder
      .master("local[*]")
      .appName("Set_theory")
      .getOrCreate()

    spark.sparkContext.setLogLevel("ERROR")
    val sc = spark.sparkContext

    // Set Theory in Spark
    val jvmLanguages = sc.parallelize(List("Scala", "Java", "Groovy", "Kotlin", "Ceylon"))
    val functionalLanguages = sc.parallelize(List("Scala", "Kotlin", "JavaScript", "Haskell"))
    val webLanguages = sc.parallelize(List("PHP", "Ruby", "Perl", "PHP", "JavaScript"))

    val result = webLanguages.distinct.union(jvmLanguages)
    println(result.toDebugString)
    result.collect.foreach(println)
```

На выходе мы получаем

```
(8) UnionRDD[6] at union at Ex_3_Set_theory.scala:24 []
 |  MapPartitionsRDD[5] at distinct at Ex_3_Set_theory.scala:24 []
 |  ShuffledRDD[4] at distinct at Ex_3_Set_theory.scala:24 []
 +-(4) MapPartitionsRDD[3] at distinct at Ex_3_Set_theory.scala:24 []
    |  ParallelCollectionRDD[2] at parallelize at Ex_3_Set_theory.scala:22 []
 |  ParallelCollectionRDD[0] at parallelize at Ex_3_Set_theory.scala:20 []


PHP
JavaScript
Ruby
Perl
Scala
Java
Groovy
Kotlin
Ceylon
```

Распечатываемая строка _.toDebugString_ весьма любопытна. По сути, мы видим перед собой весь план вычислений нашего финального результата.

Т.к. оба набора получены из коллекций при помощи оператора _.parallelize_, то мы видим две секции ParallelCollectionRDD, затем происходит .distinct на одном из множеств и их объединение, вовлекающее "перемешивание" \[shuffling\]

Аналогичный код может быть приведен для таких операций как разность, пересечение и прямое или декартово произведение \[Cartesian Product\].

```Scala
    println("----Intersection----")
    val intersection = jvmLanguages.intersection(functionalLanguages)
    println(intersection.toDebugString)
    intersection.collect.foreach(println)

----Intersection----

(4) MapPartitionsRDD[12] at intersection at Ex_3_Set_theory.scala:31 []
 |  MapPartitionsRDD[11] at intersection at Ex_3_Set_theory.scala:31 []
 |  MapPartitionsRDD[10] at intersection at Ex_3_Set_theory.scala:31 []
 |  CoGroupedRDD[9] at intersection at Ex_3_Set_theory.scala:31 []
 +-(4) MapPartitionsRDD[7] at intersection at Ex_3_Set_theory.scala:31 []
 |  |  ParallelCollectionRDD[0] at parallelize at Ex_3_Set_theory.scala:20 []
 +-(4) MapPartitionsRDD[8] at intersection at Ex_3_Set_theory.scala:31 []
    |  ParallelCollectionRDD[1] at parallelize at Ex_3_Set_theory.scala:21 []

Kotlin
Scala



    println("----Subtract----")
    val substraction = webLanguages.distinct.subtract(functionalLanguages)
    println(substraction.toDebugString)
    substraction.collect.foreach(println)

----Subtract----

(4) MapPartitionsRDD[19] at subtract at Ex_3_Set_theory.scala:36 []
 |  SubtractedRDD[18] at subtract at Ex_3_Set_theory.scala:36 []
 +-(4) MapPartitionsRDD[16] at subtract at Ex_3_Set_theory.scala:36 []
 |  |  MapPartitionsRDD[15] at distinct at Ex_3_Set_theory.scala:36 []
 |  |  ShuffledRDD[14] at distinct at Ex_3_Set_theory.scala:36 []
 |  +-(4) MapPartitionsRDD[13] at distinct at Ex_3_Set_theory.scala:36 []
 |     |  ParallelCollectionRDD[2] at parallelize at Ex_3_Set_theory.scala:22 []
 +-(4) MapPartitionsRDD[17] at subtract at Ex_3_Set_theory.scala:36 []
    |  ParallelCollectionRDD[1] at parallelize at Ex_3_Set_theory.scala:21 []
PHP
Ruby
Perl



    println("----Cartesian----")
    val cartestian = webLanguages.distinct.cartesian(jvmLanguages)
    println(cartestian.toDebugString)
    cartestian.collect.foreach(println)

----Cartesian----
(16) CartesianRDD[23] at cartesian at Ex_3_Set_theory.scala:41 []
 |   MapPartitionsRDD[22] at distinct at Ex_3_Set_theory.scala:41 []
 |   ShuffledRDD[21] at distinct at Ex_3_Set_theory.scala:41 []
 +-(4) MapPartitionsRDD[20] at distinct at Ex_3_Set_theory.scala:41 []
    |  ParallelCollectionRDD[2] at parallelize at Ex_3_Set_theory.scala:22 []
 |   ParallelCollectionRDD[0] at parallelize at Ex_3_Set_theory.scala:20 []
(PHP,Scala)
(PHP,Java)
(PHP,Groovy)
(PHP,Kotlin)
(PHP,Ceylon)
(JavaScript,Scala)
(JavaScript,Java)
(JavaScript,Groovy)
(JavaScript,Kotlin)
(JavaScript,Ceylon)
(Ruby,Scala)
(Ruby,Java)
(Ruby,Groovy)
(Ruby,Kotlin)
(Ruby,Ceylon)
(Perl,Scala)
(Perl,Java)
(Perl,Groovy)
(Perl,Kotlin)
(Perl,Ceylon)
```

#### Join для самых маленьких

Кроме операций над рядами значений, с парой PairedRDD можно сотворить обычный join.

Предположим, у нас имеется два набора данных о программистах, которые пишут код в проекте на разных языках и нам захотелось получить отчет с учетом двух наборов данных. Один набор данных мы зачем-то сохранили как последовательный файл, но сие лишь для фана и энджоймента.

```Scala
    // A few developers decided to commit something
    // Define pairs <Developer name, amount of commited core lines>
    val codeRows = sc.parallelize(Seq(("Ivan", 240), ("Elena", -15), ("Petr", 39), ("Elena", 290)))
    
    val programmerProfiles = sc.parallelize(Seq(("Ivan", "Java"), ("Elena", "Scala"), ("Petr", "Scala")))
    programmerProfiles.saveAsSequenceFile("/data/profiles")
    
    
    val joinResult = sc.sequenceFile("/data/profiles", 
    classOf[org.apache.hadoop.io.Text], classOf[org.apache.hadoop.io.Text])
      .map { case (x, y) => (x.toString, y.toString) } // transform from Hadoop Text to String
      .join(codeRows)


    joinResult.collect.foreach(println)
    
    
(Ivan,(Java,240))
(Elena,(Scala,-15))
(Elena,(Scala,290))
(Petr,(Scala,39))
```

#### Cogroup и его особенности

Эта операция похожа на FULL OUTER JOIN, но при этом вместо того, чтобы сделать "расплющивание" \[flattern\]

В документации написано

> **def cogroup\[W\]\(other: RDD\[\(K, W\)\]\): RDD\[\(K, \(Iterable\[V\], Iterable\[W\]\)\)\]**
>
> For each key k in \`this\` or \`other\`, return a resulting RDD that contains a tuple with the list of values for that key in \`this\` as well as \`other\`.

В итоге, мы получаем для каждого ключа, в качестве значения, список значений, соответствующих данному ключу в обоих соединяемых множествах.

```Scala
    programmerProfiles.cogroup(codeRows).sortByKey(false).collect().foreach(println)
    
(Petr,(CompactBuffer(Scala),CompactBuffer(39)))
(Ivan,(CompactBuffer(Java),CompactBuffer(240)))
(Elena,(CompactBuffer(Scala),CompactBuffer(-15, 290)))
```

> Забавный факт, в Spark 2.2, join реализован через композицию cogroup и flatMapValues.

#### Думаем головой при написании join

Даже если вы надеетесь на различные внутренние оптимизации Spark, не оставляйте попыток выразить свое намерение через код.

Например в ситуации, когда у нас есть два больших источника данных, которые мы соединяем, а потом фильтруем по какой-то общей колонке, лучше поменять порядок операций.

![](/assets/p1.png)

В современных версиях Spark, где даже для RDD применяются умные оптимизации, скорее всего будет задействована техника "push-down-predicate" и первый план превратится во второй.

Но зачастую, в очень сложных ансамблях запросов, оптимизации происходят не всегда удачным образом с точки зрения минимизации выделения памяти и CPU операций. Впрочем, этим в основном страдает код, использующий RDDю

Catalyst избавляет нас от многих бед в работе с DataFrames.

