# Cogroup и его особенности

Эта операция похожа на FULL OUTER JOIN, но при этом вместо того, чтобы сделать "расплющивание" \[flattern\]

В документации написано

> **def cogroup\[W\]\(other: RDD\[\(K, W\)\]\): RDD\[\(K, \(Iterable\[V\], Iterable\[W\]\)\)\]**
>
> For each key k in \`this\` or \`other\`, return a resulting RDD that contains a tuple with the list of values for that key in \`this\` as well as \`other\`.

В итоге, мы получаем для каждого ключа, в качестве значения, список значений, соответствующих данному ключу в обоих соединяемых множествах.

```
    programmerProfiles.cogroup(codeRows).sortByKey(false).collect().foreach(println)


(Petr,(CompactBuffer(Scala),CompactBuffer(39)))
(Ivan,(CompactBuffer(Java),CompactBuffer(240)))
(Elena,(CompactBuffer(Scala),CompactBuffer(-15, 290)))
```

> Забавный факт, в Spark 2.2, join реализован через композицию cogroup и flatMapValues.



