# Курсоры

База данных возвращает результаты из метода `find`, используя курсор. Реализация курсоров на стороне клиента, как правило, позволяет в значительной степени контролировать конечный результат запроса. Вы можете ограничить количество результатов, пропустить некоторое количество результатов, отсортировать результаты по любой комбинации клавиш в любом направлении и выполнить ряд других мощных операций.

Чтобы создать курсор с помощью оболочки, поместите документы в коллекцию, выполните запрос к ним и присвойте результаты локальной переменной (переменные, определенные с помощью слова `var`, являются локальными). Здесь мы создаем очень простую коллекцию и выполняем запрос, сохраняя результаты в переменной `cursor`:

```
> for(i=0; i<100; i++) {
    db.collection.insertOne({x : i});
  }
> var cursor = db.collection.find();
```

Преимущество этих действий состоит в том, что вы можете смотреть на один результат за раз. Если вы сохраните результаты в глобальной переменной или вообще не будете ее использовать, оболочка MongoDB будет автоматически перебирать и отображать первую пару документов. Это то, что мы видели до этого момента, и часто именно такое поведение вам и нужно, чтобы увидеть, что находится в коллекции, но не заниматься программированием с помощью оболочки.

Чтобы перебрать результаты, можно использовать метод `next`. Можно воспользоваться методом `hasNext`, чтобы проверить, есть ли еще результат. Типичная итерация результата выглядит следующим образом:

```
> while (cursor.hasNext()) {
    obj = cursor.next();
    // do stuff
  }
```

Метод `cursor.hasNext()` проверяет, существует ли следующий результат, а метод `cursor.next()` выбирает его.

Класс `cursor` также реализует интерфейс итератора JavaScript, поэтому вы можете использовать его в цикле `forEach`:

```
> var cursor = db.people.find();
> cursor.forEach(function(x) {
    print(x.name);
  });
adam
matt
zak
```

Когда вы вызываете метод `find`, оболочка не запрашивает базу данных тотчас же. Она ждет, пока вы не начнете запрашивать результаты для отправки запроса, что позволит вам связать дополнительные параметры в запросе до его выполнения. Почти каждый метод в объекте cursor возвращает сам курсор, поэтому можно связывать опции в любом порядке. Например, все приведенное ниже равнозначно:

```
> var cursor = db.foo.find().sort({"x" : 1}).limit(1).skip(10);
> var cursor = db.foo.find().limit(1).sort({"x" : 1}).skip(10);
> var cursor = db.foo.find().skip(10).limit(1).sort({"x" : 1});
```

На данный момент запрос еще не выполнен. Все эти функции просто создают его. Теперь предположим, что мы вызываем следующий метод:

```
> cursor.hasNext()
```

В этот момент запрос будет отправлен на сервер. Оболочка извлекает первые 100 результатов или первые 4 МБ результатов (в зависимости от того, что меньше), чтобы при последующих вызовах `next` или `hasNext` не нужно было совершать путешествие к серверу. После того как клиент выполнит первый набор результатов, оболочка снова свяжется с базой данных и запросит дополнительные результаты с помощью запроса `getMore`.

Запросы `getMore` в основном содержат идентификатор курсора и спрашивают у базы данных, есть ли еще какие-либо результаты, возвращая следующий пакет, если таковые имеются. Этот процесс продолжается до тех пор, пока курсор не будет исчерпан и не будут возвращены все результаты.

## Ограничения, пропуск и сортировка

Наиболее распространенные параметры запроса – ограничение количества возвращаемых результатов, пропуск количества результатов и сортировка. Все эти параметры должны быть добавлены до отправки запроса в базу данных.

Чтобы установить предел, включите в свой вызов метода `find` функцию `limit`. Например, чтобы вернуть только три результата, используйте это:

```
> db.c.find().limit(3)
```

Если в коллекции меньше трех документов, соответствующих вашему запросу, будет возвращено только количество совпадающих документов; функция `limit` устанавливает верхний предел, а не нижний. Функция `skip` работает похожим образом:

```
> db.c.find().skip(3)
```

Первые три совпадающих документа будут пропущены, а оставшие-
ся совпадения будут возвращены. Если в вашей коллекции меньше трех
документов,
ни один из документов возвращен не будет.

Функция `sort` принимает объект: набор пар типа «ключ/значение», где ключи – это имена ключей, а значения – направления сортировки. Направление сортировки может быть `1` (по возрастанию) или `–1` (по убыванию). Если указано несколько ключей, результаты будут отсортированы в указанном порядке. Например, чтобы отсортировать результаты по `username` в порядке возрастания и по `age` в порядке убывания, мы делаем следующее:

```
> db.c.find().sort({username : 1, age : -1})
```

Эти три метода можно сочетать, что часто удобно для нумерации страниц. Например, предположим, что вы работаете в интернет-магазине и кто-то ищет mp3. Если вы хотите, чтобы 50 результатов на странице были отсортированы по цене от высокой к низкой, можно выполнить следующий запрос:

```
> db.stock.find({"desc" : "mp3"}).limit(50).sort({"price" : -1})
```

Если человек нажимает кнопку «Следующая страница», чтобы увидеть дополнительные результаты, можно просто добавить к запросу функцию `skip`, в результате чего первые 50 совпадений будут пропущены (которые пользователь уже видел на странице 1):

```
> db.stock.find({"desc" : "mp3"}).limit(50).skip(50).sort({"price" : -1})
```

Однако большие пропуски не очень производительны; предложения по поводу того, как избежать их, приводятся в следующем разделе.

### Порядок сравнения

В MongoDB существует иерархия относительно того, как типы сравниваются. Иногда у вас будет один ключ с несколькими типами: например, целые числа и логические типы данных или строки и нули. Если вы выполняете сортировку по ключу со смесью типов, существует предопределенный порядок, в котором они будут отсортированы. Вот как выглядит этот порядок от наименьшего значения к наибольшему:

1. минимальное значение;
2. ноль;
3. числа (целые, длинные, двойные, десятичные);
4. струны;
5. объект/документ;
6. массив;
7. двоичные данные;
8. идентификатор объекта;
9. логический тип данных;
10. дата;
11. временная отметка;
12. регулярное выражение;
13. максимальное значение.

## Избегайте больших пропусков

Использование функции `skip` для небольшого количества документов – это прекрасно. Но при большом количестве результатов она может работать медленно, поскольку ей нужно найти и затем отбросить все пропущенные результаты. Большинство СУБД хранят больше метаданных в индексе, чтобы было легче иметь дело с пропусками, но в MongoDB такой функционал пока отсутствует, поэтому больших пропусков следует избегать. Часто можно рассчитать результаты следующего запроса на основе предыдущего.

### Разбивка результатов на страницы без пропусков

Самый простой способ сделать нумерацию страниц – это вернуть первую страницу результатов, используя функцию `limit`, а затем вернуть каждую последующую страницу в виде смещения от начала:

```
> // Не используйте этот вариант: в случае с большими пропусками он будет
работать медленно;
> var page1 = db.foo.find(criteria).limit(100)
> var page2 = db.foo.find(criteria).skip(100).limit(100)
> var page3 = db.foo.find(criteria).skip(200).limit(100)
...
```

Однако в зависимости от вашего запроса обычно можно найти способ разбивки на страницы без пропусков. Например, предположим, что мы хотим отображать документы в порядке убывания на основе `date`. Мы можем получить первую страницу результатов:

```
> var page1 = db.foo.find().sort({"date" : -1}).limit(100)
```

Затем, предполагая, что дата уникальна, мы можем использовать значение "date" последнего документа в качестве критерия для получения следующей страницы:

```js
var latest = null;
//Отображаем первую страницу;
while (page1.hasNext()) {
  latest = page1.next();
  display(latest);
}
//Получаем следующую страницу;
var page2 = db.foo.find({ date: { $lt: latest.date } });
page2.sort({ date: -1 }).limit(100);
```

Теперь в запрос не нужно включать функцию `skip`.

### Поиск случайного документа

Одна из довольно распространенных проблем состоит в том, как получить случайный документ из коллекции. Наивное (и медленное) решение состоит в том, чтобы подсчитать количество документов, а затем использовать метод `find`, пропуская случайное количество документов от нуля до размера коллекции:

```
> // Не используйте этот вариант;
> var total = db.foo.count()
> var random = Math.floor(Math.random()*total)
> db.foo.find().skip(random).limit(1)
```

На самом деле получать случайный элемент таким способом крайне неэффективно: вам нужно сделать подсчет (что может быть затратно, если вы используете критерии), а пропуск большого количества элементов может занять много времени.

Это займет немного времени, но если вы знаете, что будете искать случайный элемент в коллекции, существует гораздо более эффективный способ сделать это. Хитрость заключается в том, чтобы добавлять дополнительный случайный ключ к каждому документу при его вставке. Например, если мы используем оболочку, то могли бы использовать функцию `Math.random()` (которая создает случайное число от 0 до 1):

```
> db.people.insertOne({"name" : "joe", "random" : Math.random()})
> db.people.insertOne({"name" : "john", "random" : Math.random()})
> db.people.insertOne({"name" : "jim", "random" : Math.random()})
```

Теперь, если хотим найти случайный документ из коллекции, мы можем вычислить случайное число и использовать его в качестве критерия запроса, вместо того чтобы применять функцию `skip`:

```
> var random = Math.random()
> result = db.people.findOne({"random" : {"$gt" : random}})
```

Существует небольшая вероятность того, что значение `random` будет больше, чем любое из «случайных» значений в коллекции, и результаты не будут возвращены. От этого можно защититься, просто вернув документ в другом направлении:

```
> if (result == null) {
... result = db.people.findOne({"random" : {"$lte" : random}})
... }
```

Если в коллекции нет никаких документов, этот метод в конечном итоге вернет значение `null`, что не лишено смысла.

Данный метод можно использовать с произвольно сложными запросами; просто убедитесь, что у вас есть индекс, который включает в себя случайный ключ. Например, если мы хотим найти случайного водопроводчика в Калифорнии, можно создать индекс для "profession", "state» и "random":

```
> db.people.ensureIndex({"profession" : 1, "state" : 1, "random" : 1})
```

Это позволит нам быстро найти случайный результат.

## Бесконечные курсоры

У курсора есть две стороны: клиентский курсор и курсор базы данных, представляющий клиентскую сторону. Мы уже говорили о клиентской стороне, но кратко рассмотрим, что происходит на сервере.

На стороне сервера курсор занимает память и ресурсы. Как только у курсора заканчиваются результаты или клиент отправляет сообщение, сообщающее ему о прекращении работы, база данных может освободить ресурсы, которые она использовала. Освобождение этих ресурсов позволяет базе данных использовать их для других целей, и это хорошо, поэтому мы хотим обеспечить быстрое освобождение курсоров (в пределах разумного).

Есть пара условий, которые могут вызвать остановку работы (и последующую очистку) курсора. Во-первых, когда курсор завершает итерацию результатов сопоставления, он очищается самостоятельно. Другой способ заключается в том, что когда курсор выходит из области видимости на стороне клиента, драйверы отправляют базе данных специальное сообщение, чтобы уведомить ее, что она может остановить курсор.

Наконец, даже если пользователь не прошел итерацию по всем результатам и курсор по-прежнему находится в области видимости, после 10 минут бездействия курсор базы данных остановится автоматически. Таким образом, если клиент аварийно завершает работу или у него какая-то неисправность, в MongoDB не будет тысяч открытых курсоров.

Эта «смерть по тайм-ауту», как правило, является желаемым поведением: очень немногие приложения ожидают, что их пользователи будут сидеть без дела несколько минут в ожидании результатов. Тем не менее иногда вы можете знать, что вам нужно, чтобы курсора хватило надолго. В этом случае многие драйверы реализовали функцию под названием `immortal`, или аналогичный механизм, который велит базе данных не превышать время ожидания курсора. Если вы отключите тайм-аут курсора, то должны выполнить итерацию всех его результатов или уничтожить его, дабы убедиться, что он закрыт. В противном случае он будет находиться в базе данных, пожирая ресурсы, до тех пор, пока сервер не будет перезагружен.
