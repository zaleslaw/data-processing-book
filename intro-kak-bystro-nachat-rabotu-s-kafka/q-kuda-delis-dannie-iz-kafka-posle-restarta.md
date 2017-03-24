# Куда делись данные из Kafka после рестарта?

Вы оставили работать Kafka на ночь, ушли домой, вернулись на следующее утро и обнаружили, что свет на работе вырубали.

Рестартуете Zookeeper, Kafka и хотите проверить, что ваши топики на месте, до того, как их пушить.

```
$ bin/kafka-topics.sh --list --zookeeper localhost:2181
```

Но список пуст. Пусто. Ничего. Вы в панике ищите вчерашние 100 Гб логов в системе. Но система похудела на 100 Гб. Что же произошло?

#### С вами произошли настройки Kafka по умолчанию

Помните, мы учились запускать Kafka c **config/server.properties. **Так вот если сходить в эти проперти, то можно там заметить такую строку: **log.dirs=/tmp/kafka-logs.**

А директория **tmp **не самое лучшее место для сохранения чего-либо в долгосрочной перспективе. Поменяйте этот путь на какой-то другой, хоть в /home направьте ваши ивенты, если вы в себе уверены. 

После пары рестартов Kafka данные начинают пухнуть в указанной вами директории и вроде бы все хорошо.

#### До следующего "конца света"

После рестарта системы данные на месте, но через несколько мгновений они начинают исчезать.

Более того, мы в логах Kafka мы видим какие-то следы удаления, вроде вот таких

```
Deleting index .../messages-1/00000000000000000000.timeindex.deleted (kafka.log.TimeIndex)
```

Что делать? Кто виноват?

Если предположить, что какие-то метаданные о нашем топике **messages **сохранились, то мы можем попробовать их изменить, например командой, меняющей количество партиций. Результат наводит на подозрения в сторону Zookeeper, не так ли?

```
$ bin/kafka-topics.sh --alter --zookeeper localhost:2181 --partitions 4 --topic messages
Error while executing topic command : Topic messages does not exist on ZK path localhost:2181
[2017-03-24 20:36:06,816] ERROR java.lang.IllegalArgumentException: Topic messages does not exist on ZK path localhost:2181
	at kafka.admin.TopicCommand$.alterTopic(TopicCommand.scala:119)
	at kafka.admin.TopicCommand$.main(TopicCommand.scala:62)
	at kafka.admin.TopicCommand.main(TopicCommand.scala)
 (kafka.admin.TopicCommand$)
```

А вдруг Zookeeper тоже сплоховал и решил поперсестировать в директорию **tmp**?

Так и есть: **zookeeper.properties **содержит в себе замечательную строку **dataDir=/tmp/zookeeper. **Поправим и ее.

Перезапустим все.

Насладимся.

