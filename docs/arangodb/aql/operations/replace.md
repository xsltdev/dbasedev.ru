# REPLACE

Каждая операция `REPLACE` ограничена одной коллекцией, и имя коллекции не должно быть динамическим. В одном запросе AQL допускается только один оператор `REPLACE` для коллекции, и за ним не могут следовать операции чтения или записи, обращающиеся к той же коллекции, операции обхода или функции AQL, которые могут читать документы.

Нельзя заменить системные атрибуты `_id`, `_key` и `_rev`, но можно заменить атрибуты `_from` и `_to`.

Замена документа изменяет номер ревизии документа (атрибут `_rev`) на генерируемое сервером значение.

## Синтаксис

Для операции замены существует два синтаксиса:

<pre><code>REPLACE <em>document</em> IN <em>collection</em>
REPLACE <em>keyExpression</em> WITH <em>document</em> IN <em>collection</em></code></pre>

Оба варианта могут опционально заканчиваться предложением `OPTIONS { ... }`.

`collection` должно содержать имя коллекции, в которой должен быть заменен документ.

`document` должен быть объектом и содержать атрибуты и значения для установки. **Все существующие атрибуты** в хранимом документе **удаляются** из него и **устанавливаются только предоставленные атрибуты** (за исключением неизменяемых атрибутов `_id` и `_key` и управляемого системой атрибута `_rev`). Это отличает операцию `REPLACE` от операции `UPDATE`, которая влияет только на атрибуты, указанные в операции, и не изменяет другие атрибуты хранимого документа.

### `REPLACE <document> IN <collection>`

При использовании первого синтаксиса объект `document` должен иметь атрибут `_key` с ключом документа. Существующий документ с этим ключом заменяется атрибутами, предоставленными объектом `document` (за исключением системных атрибутов `_id`, `_key` и `_rev`).

Следующий запрос заменяет документ, идентифицированный ключом `my_key` в коллекции `users`, устанавливая только атрибуты `name` и `status`. Ключ передается через атрибут `_key` наряду с другими атрибутами:

<!-- 0001.part.md -->

<!-- 0002.part.md -->

```aql
REPLACE { _key: "my_key", name: "Jon", status: "active" } IN users
```

<!-- 0003.part.md -->

Следующий запрос недействителен, поскольку объект не содержит атрибута `_key`, и поэтому невозможно определить заменяемый документ:

<!-- 0004.part.md -->

```aql
REPLACE { name: "Jon" } IN users
```

<!-- 0005.part.md -->

Вы можете объединить операцию `REPLACE` с циклом `FOR` для определения необходимых ключевых атрибутов, как показано ниже:

<!-- 0006.part.md -->

```aql
FOR u IN users
  REPLACE { _key: u._key, name: CONCAT(u.firstName, " ", u.lastName), status: u.status } IN users
```

<!-- 0007.part.md -->

Обратите внимание, что операции `REPLACE` и `FOR` независимы друг от друга, и `u` не определяет автоматически документ для оператора `REPLACE`. Таким образом, следующий запрос является некорректным:

<!-- 0008.part.md -->

```aql
FOR u IN users
  REPLACE { name: CONCAT(u.firstName, " ", u.lastName), status: u.status } IN users
```

<!-- 0009.part.md -->

### `REPLACE <keyExpression> WITH <document> IN <collection>`

При использовании второго синтаксиса документ для замены определяется `keyExpression`. Это может быть либо строка с ключом документа, либо объект, содержащий атрибут `_key` с ключом документа, либо выражение, которое оценивается как одно из этих двух. Существующий документ с этим ключом заменяется атрибутами, предоставленными объектом `document` (за исключением системных атрибутов `_id`, `_key` и `_rev`).

Следующий запрос заменяет документ, идентифицированный ключом `my_key` в коллекции `users`, устанавливая только атрибуты `name` и `status`. Ключ передается в виде строки в `keyExpression`. Атрибуты для установки передаются отдельно в виде объекта `document`:

<!-- 0010.part.md -->

```aql
REPLACE "my_key" WITH { name: "Jon", status: "active" } IN users
```

<!-- 0011.part.md -->

Объект `document` может содержать атрибут `_key`, но он игнорируется.

Вы не можете определить документ для замены с помощью атрибута `_id` или передать идентификатор документа в виде строки (например, `"users/john"`). Однако вы можете использовать `PARSE_IDENTIFIER(<id>).key` в качестве `keyExpression`, чтобы получить ключ документа в виде строки:

<!-- 0012.part.md -->

```aql
LET key = PARSE_IDENTIFIER("users/john").key
REPLACE key WITH { ... } IN users
```

<!-- 0013.part.md -->

### Сравнение синтаксисов

Оба синтаксиса операции `REPLACE` позволяют вам определить документ для изменения и атрибуты для установки. Документ для обновления эффективно идентифицируется ключом документа в сочетании с указанной коллекцией.

Операция `REPLACE` поддерживает различные способы указания ключа документа. Вы можете выбрать наиболее удобный для вас вариант синтаксиса.

Следующие запросы эквивалентны:

<!-- 0014.part.md -->

```aql
FOR u IN users
  REPLACE u WITH { name: CONCAT(u.firstName, " ", u.lastName), status: u.status } IN users
```

<!-- 0015.part.md -->

<!-- 0016.part.md -->

```aql
FOR u IN users
  REPLACE u._key WITH { name: CONCAT(u.firstName, " ", u.lastName), status: u.status } IN users
```

<!-- 0017.part.md -->

<!-- 0018.part.md -->

```aql
FOR u IN users
  REPLACE { _key: u._key } WITH { name: CONCAT(u.firstName, " ", u.lastName), status: u.status } IN users
```

<!-- 0019.part.md -->

<!-- 0020.part.md -->

```aql
FOR u IN users
  REPLACE { _key: u._key, name: CONCAT(u.firstName, " ", u.lastName), status: u.status } IN users
```

<!-- 0021.part.md -->

## Выражения динамических ключей

Операция `REPLACE` может заменять произвольные документы, используя любой из двух синтаксисов:

<!-- 0022.part.md -->

```aql
FOR i IN 1..1000
  REPLACE { _key: CONCAT("test", i), name: "Paula", status: "active" } IN users
```

<!-- 0023.part.md -->

<!-- 0024.part.md -->

```aql
FOR i IN 1..1000
  REPLACE CONCAT("test", i) WITH { name: "Paula", status: "active" } IN users
```

<!-- 0025.part.md -->

## Нацелить на другую коллекцию

Документы, которые изменяет операция `REPLACE`, могут находиться в другой коллекции, чем те, которые были созданы предшествующей операцией `FOR`:

<!-- 0026.part.md -->

```aql
FOR u IN users
  FILTER u.active == false
  REPLACE u WITH { status: "inactive", name: u.name } IN backup
```

<!-- 0027.part.md -->

Обратите внимание, как документы считываются из коллекции `users`, но заменяются в другой коллекции `backup`. Для того чтобы это сработало, обе коллекции должны использовать совпадающие ключи документов.

Хотя переменная `u` содержит целый документ, она используется только для определения целевого документа. Атрибут `_key` объекта извлекается, и целевой документ определяется только значением строки ключа документа и указанной коллекцией операции `REPLACE` (`backup`). Ссылка на исходную коллекцию (`users`) отсутствует.

## Параметры запроса

Вы можете опционально задать параметры запроса для операции `REPLACE`:

<!-- 0028.part.md -->

```aql
REPLACE ... IN users OPTIONS { ... }
```

<!-- 0029.part.md -->

### `ignoreErrors`

Вы можете использовать `ignoreErrors` для подавления ошибок запроса, которые могут возникнуть при попытке заменить несуществующие документы или при нарушении ограничений уникального ключа:

<!-- 0030.part.md -->

```aql
FOR i IN 1..1000
  REPLACE CONCAT("test", i)
  WITH { foobar: true } IN users
  OPTIONS { ignoreErrors: true }
```

<!-- 0031.part.md -->

Вы не можете изменять системные атрибуты `_id`, `_key` и `_rev`, но попытки изменить их игнорируются и не считаются ошибками.

### `waitForSync`

Для обеспечения долговечности данных при возврате запроса замены существует опция запроса `waitForSync`:

<!-- 0032.part.md -->

```aql
FOR i IN 1..1000
  REPLACE CONCAT("test", i)
  WITH { foobar: true } IN users
  OPTIONS { waitForSync: true }
```

<!-- 0033.part.md -->

### `ignoreRevs`

Чтобы случайно не перезаписать документы, которые были изменены с момента последнего извлечения, вы можете использовать опцию `ignoreRevs`, чтобы либо позволить ArangoDB сравнивать значение `_rev` и добиваться успеха, только если они совпадают, либо позволить ArangoDB игнорировать их (по умолчанию):

<!-- 0034.part.md -->

```aql
FOR i IN 1..1000
  REPLACE { _key: CONCAT("test", i), _rev: "1287623" }
  WITH { foobar: true } IN users
  OPTIONS { ignoreRevs: false }
```

<!-- 0035.part.md -->

### `exclusive`

Движок RocksDB не требует блокировок на уровне коллекции. Различные операции записи в одну и ту же коллекцию не блокируют друг друга, если нет конфликтов _запись-запись_ на одних и тех же документах. С точки зрения разработки приложений может быть желательным иметь исключительный доступ на запись в коллекции, чтобы упростить разработку. Обратите внимание, что записи не блокируют чтения в RocksDB. Исключительный доступ также может ускорить запросы на модификацию, поскольку мы избегаем проверки конфликтов.

Используйте опцию `exclusive` для достижения этого эффекта на основе каждого запроса:

<!-- 0036.part.md -->

```aql
FOR doc IN collection
  REPLACE doc
  WITH { replaced: true } IN collection
  OPTIONS { exclusive: true }
```

<!-- 0037.part.md -->

### `refillIndexCaches`

Нужно ли обновлять существующие записи в кэше границ в памяти, если документы границ заменяются.

<!-- 0038.part.md -->

```aql
REPLACE { _key: "123", _from: "vert/C", _to: "vert/D" } IN edgeColl
  OPTIONS { refillIndexCaches: true }
```

<!-- 0039.part.md -->

## Возвращение измененных документов

При желании можно вернуть документы, измененные запросом. В этом случае за операцией `REPLACE` должна следовать операция `RETURN`. Допускаются также промежуточные операции `LET`. Эти операции могут ссылаться на псевдопеременные `OLD` и `NEW`. Псевдопеременная `OLD` ссылается на ревизии документа до замены, а `NEW` - на ревизии документа после замены.

И `OLD`, и `NEW` содержат все атрибуты документа, даже те, которые не указаны в выражении replace.

<!-- 0040.part.md -->

```aql
REPLACE document IN collection options RETURN OLD
REPLACE document IN collection options RETURN NEW
REPLACE keyExpression WITH document IN collection options RETURN OLD
REPLACE keyExpression WITH document IN collection options RETURN NEW
```

<!-- 0041.part.md -->

Ниже приведен пример с использованием переменной `previous` для возврата исходных документов до их модификации. Для каждого замененного документа возвращается ключ документа:

<!-- 0042.part.md -->

```aql
FOR u IN users
  REPLACE u WITH { value: "test" } IN users
  LET previous = OLD
  RETURN previous._key
```

<!-- 0043.part.md -->

Следующий запрос использует псевдо-значение `NEW`, чтобы вернуть замененные документы без некоторых их системных атрибутов:

<!-- 0044.part.md -->

```aql
FOR u IN users
  REPLACE u WITH { value: "test" } IN users
  LET replaced = NEW
  RETURN UNSET(replaced, "_key", "_id", "_rev")
```

<!-- 0045.part.md -->

## Транзакционность

На одном сервере операции замены выполняются транзакционно по принципу "все или ничего".

Если используется движок RocksDB и включены промежуточные фиксации, запрос может выполнять промежуточные фиксации транзакций в случае, если запущенная транзакция (AQL-запрос) достигает заданных пороговых значений размера. В этом случае операции запроса, выполненные до этого момента, фиксируются и не откатываются в случае последующего отмены/отката. Это поведение можно контролировать, изменяя настройки промежуточной фиксации для движка RocksDB.

Для коллекций с шардированием вся операция запроса и/или замены может не быть транзакционной, особенно если она затрагивает разные шарды и/или DB-серверы.

<!-- 0046.part.md -->
