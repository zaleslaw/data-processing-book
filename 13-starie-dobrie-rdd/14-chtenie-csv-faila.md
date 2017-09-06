# Чтение CSV-файла

Пусть у нас имеется файл с данными о новорожденных США за разные годы, сгруппированных по полу, штату и имени.

&lt;!--table  
	{mso-displayed-decimal-separator:"\.";  
	mso-displayed-thousand-separator:"\,";}  
@page  
	{margin:.75in .7in .75in .7in;  
	mso-header-margin:.3in;  
	mso-footer-margin:.3in;}  
tr  
	{mso-height-source:auto;}  
col  
	{mso-width-source:auto;}  
br  
	{mso-data-placement:same-cell;}  
td  
	{padding-top:1px;  
	padding-right:1px;  
	padding-left:1px;  
	mso-ignore:padding;  
	color:black;  
	font-size:11.0pt;  
	font-weight:400;  
	font-style:normal;  
	text-decoration:none;  
	font-family:Calibri, sans-serif;  
	mso-font-charset:0;  
	mso-number-format:General;  
	text-align:general;  
	vertical-align:bottom;  
	border:none;  
	mso-background-source:auto;  
	mso-pattern:auto;  
	mso-protection:locked visible;  
	white-space:nowrap;  
	mso-rotate:0;}  
--&gt;  


| Id | Name | Year | Gender | State | Count |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Mary | 1910 | F | AK | 14 |
| 2 | Annie | 1910 | F | AK | 12 |
| 3 | Anna | 1910 | F | AK | 10 |
| 4 | Margaret | 1910 | F | AK | 8 |
| 5 | Helen | 1910 | F | AK | 7 |
| 6 | Elsie | 1910 | F | AK | 6 |
| 7 | Lucy | 1910 | F | AK | 6 |
| 8 | Dorothy | 1910 | F | AK | 5 |
| 9 | Mary | 1911 | F | AK | 12 |
| 10 | Margaret | 1911 | F | AK | 7 |
| 11 | Ruth | 1911 | F | AK | 7 |
| 12 | Annie | 1911 | F | AK | 6 |
| 13 | Elizabeth | 1911 | F | AK | 6 |
| 14 | Helen | 1911 | F | AK | 6 |
| 15 | Mary | 1912 | F | AK | 9 |
| 16 | Elsie | 1912 | F | AK | 8 |
| 17 | Agnes | 1912 | F | AK | 7 |
| 18 | Anna | 1912 | F | AK | 7 |
| 19 | Helen | 1912 | F | AK | 7 |
| 20 | Louise | 1912 | F | AK | 7 |

Давайте распарсим эти данные, превратив в RDD, где записью будет словарь из 6 пар ключ-значение, где ключ - название колонки, а значение - значение в этой колонке. Да, избыточно с точки зрения метаинформации, но тоже вариант.



```Scala
    // read from file
    val stateNamesCSV = sc.textFile("/home/zaleslaw/data/StateNames.csv")      
    // split / clean data
    val headerAndRows = stateNamesCSV.map(line => line.split(",").map(_.trim))
    // get header
    val header = headerAndRows.first
    // filter out header (eh. just check if the first val matches the first header name)
    val data = headerAndRows.filter(_ (0) != header(0))
    // splits to map (header/value pairs)
    val stateNames = data.map(splits => header.zip(splits).toMap)
    // print top-5
    stateNames.take(5).foreach(println)
```

Данный код предполагает, что в файле имеется первая строка с заголовками, которую мы планируем использовать в качестве набора ключей в каждой записи-словаре.

#### Работа с RDD, содержащей строки CSV-файла

Вы можете задать резонный вопрос: "Как всем этим пользоваться?" 

Рассмотрим пример, где нужно отфильтровать все записи с Name==Anna и количеством рождений больше сотни, взяв TOP-5 записей



```Scala
    stateNames
      .filter(e => e("Name") == "Anna" && e("Count").toInt > 100)
      .take(5)
      .foreach(println)

```

Конечно, это не единственный спобос, да и далеко не самый лучший с точки зрения overhead по памяти, но данная схема позволяет работать нам в стиле, наиболее близком к стилю DataFrames/DataSet API.

> Для парсинга CSV-файлов лучше использовать встроенные возможности .read.csv\(..\) для последних версий Spark или библиотеку [https://github.com/databricks/spark-csv](https://github.com/databricks/spark-csv "spark-csv")  для более ранних версий.



