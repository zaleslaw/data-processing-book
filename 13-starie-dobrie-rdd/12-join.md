# Join для самых маленьких

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

Конечно, поддерживаются не только INNER JOIN, но и аналоги FULL OUTER, RIGHT OUTER и LEFT OUTER

