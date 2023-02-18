# Запросы данных

Существует два основных типа запросов AQL:

- запросы, которые обращаются к данным (читают документы)
- запросы, которые изменяют данные (создают, обновляют, заменяют, удаляют документы)

## Запросы доступа к данным

Извлечение данных из базы данных с помощью AQL всегда включает операцию `RETURN`. Его можно использовать для возврата статического значения, например строки:

```aql
RETURN "Hello ArangoDB!"
```

Результатом запроса всегда является массив элементов, даже если был возвращен один элемент, и в этом случае он содержит один элемент: `["Hello ArangoDB!"]`.

Функцию `DOCUMENT()` можно вызвать для получения одного документа через его дескриптор документа, например:

```aql
RETURN DOCUMENT("users/phil")
```

`RETURN` обычно сопровождается циклом `FOR` для перебора документов коллекции. Следующий запрос выполняет тело цикла для всех документов коллекции, называемой пользователями. В этом примере каждый документ возвращается без изменений:

```aql
FOR doc IN users
  RETURN doc
```

Вместо того, чтобы возвращать необработанный документ, можно легко создать проекцию:

```aql
FOR doc IN users
  RETURN { user: doc, newAttribute: true }
```

Для каждого пользовательского документа возвращается объект с двумя атрибутами. Значением атрибута `user` установлено содержимое пользовательского документа, а `newAttribute` — это статический атрибут с логическим значением `true`.

В тело цикла можно добавить такие операции, как `FILTER`, `SORT` и `LIMIT`, чтобы сузить и упорядочить результат. Вместо показанного выше вызова `DOCUMENT()` можно также получить документ, который описывает пользователя `phil` следующим образом:

```aql
FOR doc IN users
  FILTER doc._key == "phil"
  RETURN doc
```

В этом примере используется ключ документа, но любой другой атрибут может также использоваться для фильтрации. Поскольку ключ документа гарантированно уникален, этому фильтру может соответствовать не более одного документа. Для других атрибутов это может быть не так. Чтобы вернуть подмножество активных пользователей (определяемых атрибутом, называемым `status`), отсортированных по имени в порядке возрастания, вы можете сделать:

```aql
FOR doc IN users
  FILTER doc.status == "active"
  SORT doc.name
  LIMIT 10
```

Обратите внимание, что операции не обязательно должны выполняться в фиксированном порядке и что их порядок может существенно повлиять на результат. Ограничение количества документов перед фильтром, как правило, не то, что вам нужно, потому что оно легко упускает множество документов, которые удовлетворяют критерию фильтра, но игнорируются из-за преждевременного предложения `LIMIT`. По вышеупомянутым причинам `LIMIT` обычно ставится в самом конце, после `FILTER`, `SORT` и других операций.

См. «[Операции высокого уровня](operations.md)» для получения более подробной информации.

## Запросы на изменение данных

AQL поддерживает следующие операции модификации данных:

- **INSERT**: вставлять новые документы в коллекцию
- **UPDATE**: частично обновить существующие документы в коллекции
- **REPLACE**: полностью заменить существующие документы в коллекции
- **REMOVE**: удалить существующие документы из коллекции
- **UPSERT**: условно вставить или обновить документы в коллекции

Вы можете использовать их для изменения данных одного или нескольких документов с помощью одного запроса. Это лучше, чем выборка и обновление документов по отдельности с помощью нескольких запросов. Однако, если необходимо изменить только один документ, специализированные операции модификации данных ArangoDB для отдельных документов могут выполняться быстрее.

Ниже вы найдете несколько простых примеров запросов, в которых используются эти операции. Операции подробно описаны в главе «[Операции высокого уровня](operations/index.md)».

### Изменение одного документа

Начнем с основ: операции `INSERT`, `UPDATE` и `REMOVE` для отдельных документов. Вот пример, который вставляет документ в коллекцию `users` с помощью операции [`INSERT`](operations/insert.md):

```aql
INSERT {
  firstName: "Anna",
  name: "Pavlova",
  profession: "artist"
} INTO users
```

Коллекция должна существовать до выполнения запроса. Запросы AQL не могут создавать коллекции.

Если вы запустите приведенный выше запрос, результатом будет пустой массив, потому что мы не указали, что возвращать, используя ключевое слово `RETURN`. Это необязательно в запросах на изменение, но обязательно в запросах на доступ к данным. Несмотря на пустой результат, приведенный выше запрос по-прежнему создает новый пользовательский документ.

Вы можете предоставить ключ для нового документа; если он не указан, ArangoDB создаст его для вас.

```aql
INSERT {
  _key: "GilbertoGil",
  firstName: "Gilberto",
  name: "Gil",
  city: "Fortalezza"
} INTO users
```

Поскольку ArangoDB не содержит схем, атрибуты документов могут различаться:

```aql
INSERT {
  _key: "PhilCarpenter",
  firstName: "Phil",
  name: "Carpenter",
  middleName: "G.",
  status: "inactive"
} INTO users
```

```aql
INSERT {
  _key: "NatachaDeclerck",
  firstName: "Natacha",
  name: "Declerck",
  location: "Antwerp"
} INTO users
```

Операция [`UPDATE`](operations/update.md) позволяет добавлять или изменять атрибуты существующих документов. Следующий запрос изменяет ранее созданного пользователя, изменяя атрибут `status` и добавляя атрибут `location`:

```aql
UPDATE "PhilCarpenter" WITH {
  status: "active",
  location: "Beijing"
} IN users
```

Операция [`REPLACE`](operations/replace.md) является альтернативой операции `UPDATE`, которая позволяет заменить все атрибуты документа (кроме атрибутов, которые нельзя изменить, например `_key`):

```aql
REPLACE {
  _key: "NatachaDeclerck",
  firstName: "Natacha",
  name: "Leclerc",
  status: "active",
  level: "premium"
} IN users
```

Вы можете удалить документ с помощью операции [`REMOVE`](operations/remove.md), требуя только ключ документа для его идентификации:

```aql
REMOVE "GilbertoGil" IN users
```

### Изменение нескольких документов

Операции модификации данных обычно сочетаются с циклами `FOR` для перебора заданного списка документов. При желании их можно комбинировать с операторами `FILTER` и т. п.

Чтобы создать несколько новых документов, используйте операцию `INSERT` вместе с `FOR`. Вы также можете использовать `INSERT` для создания копий существующих документов из других коллекций или для создания синтетических документов (например, в целях тестирования). Следующий запрос создает 1000 тестовых пользователей с некоторыми атрибутами и сохраняет их в коллекции `users`:

```aql
FOR i IN 1..1000
  INSERT {
    id: 100000 + i,
    age: 18 + FLOOR(RAND() * 25),
    name: CONCAT('test', TO_STRING(i)),
    status: i % 2 == 0 ? "active" : "not active",
    active: false,
    gender: i % 3 == 0 ? "male" : i % 3 == 1 ? "female" : "diverse"
  } IN users
```

Давайте изменим существующие документы, соответствующие некоторому условию:

```aql
FOR u IN users
  FILTER u.status == "not active"
  UPDATE u WITH { status: "inactive" } IN users
```

Вы также можете обновить существующие атрибуты на основе их предыдущего значения:

```aql
FOR u IN users
  FILTER u.active == true
  UPDATE u WITH { numberOfLogins: u.numberOfLogins + 1 } IN users
```

Приведенный выше запрос работает только в том случае, если в документе уже присутствует атрибут `numberOfLogins`. Если неясно, есть ли в документе атрибут `numberOfLogins`, увеличение необходимо сделать условным:

```aql
FOR u IN users
  FILTER u.active == true
  UPDATE u WITH {
    numberOfLogins: HAS(u, "numberOfLogins") ? u.numberOfLogins + 1 : 1
  } IN users
```

Обновления нескольких атрибутов могут быть объединены в один запрос:

```aql
FOR u IN users
  FILTER u.active == true
  UPDATE u WITH {
    lastLogin: DATE_NOW(),
    numberOfLogins: HAS(u, "numberOfLogins") ? u.numberOfLogins + 1 : 1
  } IN users
```

Обратите внимание, что запрос на обновление может завершиться ошибкой во время выполнения, например, из-за того, что документ, который нужно обновить, не существует. В этом случае запрос прерывается при первой ошибке. В режиме одного сервера все изменения, сделанные запросом, откатываются, как если бы они никогда не происходили.

Вы можете копировать документы из одной коллекции в другую, читая из одной коллекции и записывая в другую. Скопируем содержимое коллекции `users` в коллекцию `backup`:

```aql
FOR u IN users
  INSERT u IN backup
```

Note that both collections must already exist when the query is executed.
The query might fail if the `backup` collection already contains documents,
as executing the insert might attempt to insert the same document (identified
by the `_key` attribute) again. This triggers a unique key constraint violation
and aborts the query. In single server mode, all changes made by the query
are also rolled back.
To make such a copy operation work in all cases, the target collection can
be emptied beforehand, using a `REMOVE` query or by truncating it by other means.

To not just partially update, but completely replace existing documents, use
the `REPLACE` operation.
The following query replaces all documents in the `backup` collection with
the documents found in the `users` collection. Documents common to both
collections are replaced. All other documents remain unchanged.
Documents are compared using their `_key` attributes:

```aql
FOR u IN users
  REPLACE u IN backup
```

The above query fails if there are documents in the `users` collection that are
not in the `backup` collection yet. In this case, the query would attempt to replace
documents that do not exist. If such case is detected while executing the query,
the query is aborted. In single server mode, all changes made by the query are
rolled back.

To make the query succeed regardless of the errors, use the `ignoreErrors`
query option:

```aql
FOR u IN users
  REPLACE u IN backup OPTIONS { ignoreErrors: true }
```

This continues the query execution if errors occur during a `REPLACE`, `UPDATE`,
`INSERT`, or `REMOVE` operation.

Finally, let's find some documents in collection `users` and remove them
from collection `backup`. The link between the documents in both collections is
established via the documents' keys:

```aql
FOR u IN users
  FILTER u.status == "deleted"
  REMOVE u IN backup
```

The following example removes all documents from both `users` and `backup`:

```aql
LET r1 = (FOR u IN users  REMOVE u IN users)
LET r2 = (FOR u IN backup REMOVE u IN backup)
RETURN true
```

### Altering substructures

To modify lists in documents, for example, to update specific attributes of
objects in an array, you can compute a new array and then update the document
attribute in question. This may involve the use of subqueries and temporary
variables.

Create a collection named `complexCollection` and run the following query:

```aql
FOR doc IN [
  {
    "topLevelAttribute": "a",
    "subList": [
      {
        "attributeToAlter": "value to change",
        "filterByMe": true
      },
      {
        "attributeToAlter": "another value to change",
        "filterByMe": true
      },
      {
        "attributeToAlter": "keep this value",
        "filterByMe": false
      }
    ]
  },
  {
    "topLevelAttribute": "b",
    "subList": [
      {
        "attributeToAlter": "keep this value",
        "filterByMe": false
      }
    ]
  }
] INSERT doc INTO complexCollection
```

The following query updates the `subList` top-level attribute of documents.
The `attributeToAlter` values in the nested object are changed if the adjacent
`filterByMe` attribute is `true`:

```aql
FOR doc in complexCollection
  LET alteredList = (
    FOR element IN doc.subList
       RETURN element.filterByMe
              ? MERGE(element, { attributeToAlter: "new value" })
              : element
  )
  UPDATE doc WITH { subList: alteredList } IN complexCollection
  RETURN NEW
```

```json
[
  {
    "_key": "2607",
    "_id": "complexCollection/2607",
    "_rev": "_fWb_iOO---",
    "topLevelAttribute": "a",
    "subList": [
      {
        "attributeToAlter": "new value",
        "filterByMe": true
      },
      {
        "attributeToAlter": "new value",
        "filterByMe": true
      },
      {
        "attributeToAlter": "keep this value",
        "filterByMe": false
      }
    ]
  },
  {
    "_key": "2608",
    "_id": "complexCollection/2608",
    "_rev": "_fWb_iOO--_",
    "topLevelAttribute": "b",
    "subList": [
      {
        "attributeToAlter": "keep this value",
        "filterByMe": false
      }
    ]
  }
]
```

To improve the query's performance, you can only update documents if there is
a change to the `subList` to be saved. Instead of comparing the current and the
altered list directly, you may compare their hash values using the
[`HASH()` function](functions-miscellaneous.html#hash), which is faster for
larger objects and arrays. You can also replace the subquery with an
[inline expression](advanced-array-operators.html#inline-expressions):

```aql
FOR doc in complexCollection
  LET alteredList = doc.subList[*
    RETURN CURRENT.filterByMe
    ? MERGE(CURRENT, { attributeToAlter: "new value" })
    : CURRENT
  ]
  FILTER HASH(doc.subList) != HASH(alteredList)
  UPDATE doc WITH { subList: alteredList } IN complexCollection
  RETURN NEW
```

### Returning documents

Data modification queries can optionally return documents. In order to reference
the inserted, removed or modified documents in a `RETURN` statement, data modification
statements introduce the `OLD` and/or `NEW` pseudo-values:

```aql
FOR i IN 1..100
  INSERT { value: i } IN test
  RETURN NEW
```

```aql
FOR u IN users
  FILTER u.status == "deleted"
  REMOVE u IN users
  RETURN OLD
```

```aql
FOR u IN users
  FILTER u.status == "not active"
  UPDATE u WITH { status: "inactive" } IN users
  RETURN NEW
```

`NEW` refers to the inserted or modified document revision, and `OLD` refers
to the document revision before update or removal. `INSERT` statements can
only refer to the `NEW` pseudo-value, and `REMOVE` operations only to `OLD`.
`UPDATE`, `REPLACE` and `UPSERT` can refer to either.

In all cases, the full documents are returned with all their attributes,
including the potentially auto-generated attributes, such as `_id`, `_key`, and `_rev`,
and the attributes not specified in the update expression of a partial update.

#### Projections of OLD and NEW

It is possible to return a projection of the documents with `OLD` or `NEW` instead of
returning the entire documents. This can be used to reduce the amount of data returned
by queries.

For example, the following query returns only the keys of the inserted documents:

```aql
FOR i IN 1..100
  INSERT { value: i } IN test
  RETURN NEW._key
```

#### Using OLD and NEW in the same query

For `UPDATE`, `REPLACE`, and `UPSERT` operations, both `OLD` and `NEW` can be used
to return the previous revision of a document together with the updated revision:

```aql
FOR u IN users
  FILTER u.status == "not active"
  UPDATE u WITH { status: "inactive" } IN users
  RETURN { old: OLD, new: NEW }
```

#### Calculations with OLD or NEW

It is also possible to run additional calculations with `LET` statements between the
data modification part and the final `RETURN` of an AQL query. For example, the following
query performs an upsert operation and returns whether an existing document was
updated, or a new document was inserted. It does so by checking the `OLD` variable
after the `UPSERT` and using a `LET` statement to store a temporary string for
the operation type:

```aql
UPSERT { name: "test" }
  INSERT { name: "test" }
  UPDATE { } IN users
LET opType = IS_NULL(OLD) ? "insert" : "update"
RETURN { _key: NEW._key, type: opType }
```

### Restrictions

The name of the modified collection (`users` and `backup` in the above cases)
must be known to the AQL executor at query-compile time and cannot change at
runtime. Using a bind parameter to specify the
[collection name](../appendix-glossary.html#collection-name) is allowed.

It is not possible to use multiple data modification operations for the same
collection in the same query, or follow up a data modification operation for a
specific collection with a read operation for the same collection. Neither is
it possible to follow up any data modification operation with a traversal query
(which may read from arbitrary collections not necessarily known at the start of
the traversal).

That means you may not place several `REMOVE` or `UPDATE` statements for the same
collection into the same query. It is however possible to modify different collections
by using multiple data modification operations for different collections in the
same query.
In case you have a query with several places that need to remove documents from the
same collection, it is recommended to collect these documents or their keys in an array
and have the documents from that array removed using a single `REMOVE` operation.

Data modification operations can optionally be followed by `LET` operations to
perform further calculations and a `RETURN` operation to return data.

### Transactional Execution

On a single server, data modification operations are executed transactionally.
If a data modification operation fails, any changes made by it are rolled
back automatically as if they never happened.

If the RocksDB engine is used and intermediate commits are enabled, a query may
execute intermediate transaction commits in case the running transaction (AQL
query) hits the specified size thresholds. In this case, the query's operations
carried out so far are committed and not rolled back in case of a later abort/rollback.
That behavior can be controlled by adjusting the intermediate commit settings for
the RocksDB engine.

In a cluster, AQL data modification queries are not executed transactionally.
Additionally, AQL queries with `UPDATE`, `REPLACE`, `UPSERT`, or `REMOVE`
operations require the `_key` attribute to be specified for all documents that
should be modified or removed, even if a shard key attribute other than `_key`
is chosen for the collection.
