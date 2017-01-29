# Spark 2.1

Это раньше вам нужен был Hadoop, чтобы запустить Spark, а теперь и его не нужно.

Вам нужны три вещи: базовое знание Linux, Java и свеженький релиз Spark.

Для начала, нам надо убедиться, что у нас стоит свеженькая Java при помощи команды

```
$ java -version
```

Если вы видите похожие строки, то все в порядке, иначе надо установить Java 8

```
$ java version "1.8.0_111"
$ Java(TM) SE Runtime Environment (build 1.8.0_111-b14)
$ Java HotSpot(TM) 64-Bit Server VM (build 25.111-b14, mixed mode)
```

#### Абзац для тех, у кого нет Java

Не мучайтесь со свободными сборками, поставьте Java от Oracle.

```
$ sudo apt-get-repository ppa:webupd8team/java
$ sudo add-apt-repository ppa:webupd8team/java
$ sudo apt-get update
$ sudo apt-get install oracle-java8-installer
```

Чтобы Java была доступна нам в CLI, добавим строку **JAVA\_HOME="/usr/lib/jvm/java-8-oracle" **в файл /etc/environment

```
$ sudo nano /etc/environment

... добавляем строку JAVA_HOME="/usr/lib/jvm/java-8-oracle"

... сохраняем файл и применяем изменения

$ source /etc/environment

... проверяем, что JAVA_HOME доступна

$ echo $JAVA_HOME
```

Java установлена и доступна для будущего Spark-а, а Scala пока устанавливать нет нужды.

#### Установка Spark

Linux вам понадобится, чтобы установить в одну из директорий Spark. Ну как установить, просто скопировать и прописать несколько переменных окружения.

Итак, начнем с создания директории, где будет лежать Spark, при помощи простой команды

```
$ mkdir Spark
```

Зайдем в директорию и скопируем свеженький релиз со страницы [http://spark.apache.org/downloads.html](http://spark.apache.org/downloads.html "свеженький релиз") , который при этом будет собран для Hadoop 2.7 и выше \(если у вас нет Hadoop на машине, не переживайте, он не нужен для экспериментов\)

```
$ cd Spark
$ wget http://d3kbcqa49mib13.cloudfront.net/spark-2.1.0-bin-hadoop2.7.tgz

```

Затем скачанный архив в формате tgz \(или _тарбол_, как говорят в народе\) необходимо распаковать

```
$ tar xvf spark-2.1.0-bin-hadoop2.7.tgz
```

На самом деле Spark у вас уже есть, и можно было бы на этом остановиться, но чтобы была возможность запускать его через CLI в любом месте, нам надо изменить несколько строк в файле /etc/environment.

```
$ sudo nano /etc/environment

... добавляем строку SPARK_HOME="<Ваш путь до Spark>/Spark/spark-2.1.0-bin-hadoop2.7"

... изменяем переменную PATH c PATH="/usr/local/sbin: ..." на PATH="${SPARK_HOME}/bin:/usr/local/sbin:

... сохраняем файл и применяем изменения

$ source /etc/environment

... проверяем, что SPARK_HOME доступна

$ echo $SPARK_HOME

```

Ну вот и все! Вы можете запустить Spark локально на своей машине при помощи команды

```
$ spark-shell

```

Если все хорошо, то вы попадете в CLI Spark-а, где вам доступен SparkContext как sc и SparkSession как Spark

```
scala> spark
res1: org.apache.spark.sql.SparkSession = org.apache.spark.sql.SparkSession@282240

```

В заключение, хочу отметить, что данная установка не эквивалентна развертыванию Spark в кластере, вместе с Hadoop. Данной установки должно хватить для небольших экспериментов и изучения Spark API.




