# Удаление документов

Теперь, когда в нашей базе данных есть данные, давайте удалим их. Для этой цели CRUD API предоставляет методы `deleteOne` и `deleteMany`. Оба этих метода принимают документ фильтра в качестве первого параметра. Фильтр задает набор критериев для сопоставления при удалении документов. Чтобы удалить документ со значением `_id`, равным `4`, мы используем метод `deleteOne` в оболочке _mongo_, как показано здесь:

```
> db.movies.find()
{ "_id" : 0, "title" : "Top Gun"}
{ "_id" : 1, "title" : "Back to the Future"}
{ "_id" : 3, "title" : "Sixteen Candles"}
{ "_id" : 4, "title" : "The Terminator"}
{ "_id" : 5, "title" : "Scarface"}
> db.movies.deleteOne({"_id" : 4})
{ "acknowledged" : true, "deletedCount" : 1 }
> db.movies.find()
{ "_id" : 0, "title" : "Top Gun"}
{ "_id" : 1, "title" : "Back to the Future"}
{ "_id" : 3, "title" : "Sixteen Candles"}
{ "_id" : 5, "title" : "Scarface"}
```

В этом примере мы использовали фильтр, который может соответствовать только одному документу, поскольку значения `_id` уникальны в коллекции. Однако мы также можем указать фильтр, который соответствует нескольким документам в коллекции. В этом случае метод `deleteOne` удалит первый найденный документ, который соответствует фильтру. Какой документ будет найден первым, зависит от нескольких факторов, в том числе от порядка, в котором были вставлены документы, того, какие обновления были внесены в них (для некоторых подсистем хранения), и того, какие индексы указаны. Как и при любой операции с базой данных, убедитесь, что вы знаете, как использование метода `deleteOne` повлияет на ваши данные.

Чтобы удалить все документы, которые соответствуют фильтру, используйте метод `deleteMany`:

```
> db.movies.find()
{ "_id" : 0, "title" : "Top Gun", "year" : 1986 }
{ "_id" : 1, "title" : "Back to the Future", "year" : 1985 }
{ "_id" : 3, "title" : "Sixteen Candles", "year" : 1984 }
{ "_id" : 4, "title" : "The Terminator", "year" : 1984 }
{ "_id" : 5, "title" : "Scarface", "year" : 1983 }
> db.movies.deleteMany({"year" : 1984})
{ "acknowledged" : true, "deletedCount" : 2 }
> db.movies.find()
{ "_id" : 0, "title" : "Top Gun", "year" : 1986 }
{ "_id" : 1, "title" : "Back to the Future", "year" : 1985 }
{ "_id" : 5, "title" : "Scarface", "year" : 1983 }
```

В качестве более реалистичного варианта использования предположим, что вы хотите удалить каждого пользователя из коллекции `mailing.list`, где значение `opt-out` равно `true`:

```
> db.mailing.list.deleteMany({"opt-out" : true})
```

В версиях MongoDB до 3.0 `remove` был основным методом удаления документов. Драйверы MongoDB представили методы `deleteOne` и `deleteMany` одновременно с выпуском сервера MongoDB 3.0, и оболочка начала поддерживать эти методы в версии 3.2. Хотя метод `remove` все еще поддерживается по причине обратной совместимости, в своих приложениях следует использовать методы `deleteOne` и `deleteMany`. Текущий CRUD API предоставляет более чистый набор семантики и, особенно в случае с операциями с несколькими документами, помогает разработчикам приложений избежать распространенных ошибок с предыдущим API.

## drop

Можно использовать метод `deleteMany`, чтобы удалить все документы в коллекции:

```
> db.movies.find()
{ "_id" : 0, "title" : "Top Gun", "year" : 1986 }
{ "_id" : 1, "title" : "Back to the Future", "year" : 1985 }
{ "_id" : 3, "title" : "Sixteen Candles", "year" : 1984 }
{ "_id" : 4, "title" : "The Terminator", "year" : 1984 }
{ "_id" : 5, "title" : "Scarface", "year" : 1983 }
> db.movies.deleteMany({})
{ "acknowledged" : true, "deletedCount" : 5 }
> db.movies.find()
```

Удаление документов обычно является довольно быстрой операцией. Однако если вы хотите очистить всю коллекцию, ее быстрее удалить с помощью метода `drop`:

```
> db.movies.drop()
true
```

а затем воссоздать любые индексы в пустой коллекции.

Как только данные удаляются, они исчезают навсегда. Операции `drop` и `delete` невозможно отменить, равно как нельзя восстановить удаленные документы, разумеется, если только речь не идет о восстановлении предварительно сохраненной версии данных.
