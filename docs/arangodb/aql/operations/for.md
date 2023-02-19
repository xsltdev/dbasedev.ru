# FOR

Универсальное ключевое слово **`FOR`** можно использовать для перебора коллекции или представления, всех элементов массива или для обхода графа.

## Синтаксис

Общий синтаксис для перебора коллекций и массивов:

<pre><code>FOR <em>variableName</em> IN <em>expression</em></code></pre>

Существует также специальный вариант для [обхода графа](graphs/traversals.md):

<pre>
<code>FOR <em>vertexVariableName</em> [, <em>edgeVariableName</em> [, <em>pathVariableName</em> ] ]
  IN <em>traversalExpression</em></code>
</pre>

Для представлений есть специальное (необязательное) ключевое слово [`SEARCH`](search.md):

<pre><code>FOR <em>variableName</em> IN <em>viewName</em> SEARCH <em>searchExpression</em></code></pre>

!!!info ""

    Представления нельзя использовать в качестве коллекций ребер в обходах:

    ```aql
    FOR v IN 1..3 ANY startVertex viewName /* неверно! */
    ```

Все варианты могут дополнительно заканчиваться предложением `OPTIONS { … }`.

## Применение

Каждый элемент массива, возвращаемый _выражением_, посещается ровно один раз. Требуется, чтобы _выражение_ возвращало массив во всех случаях. Пустой массив также разрешен. Текущий элемент массива становится доступным для дальнейшей обработки в переменной, указанной в `variableName`.

```aql
FOR u IN users
  RETURN u
```

Это будет перебирать все элементы из массива `users` (обратите внимание: этот массив состоит из всех документов из коллекции с именем `users` в данном случае) и сделает текущий элемент массива доступным в переменной `u`. `u` не изменяется в этом примере, а просто вставляется в результат с помощью ключевого слова `RETURN`.

!!!note ""

    При переборе массивов на основе коллекций, как показано здесь, порядок документов не определен, если явный порядок сортировки не определен с помощью инструкции `SORT`.

Переменная, представленная `FOR`, доступна до тех пор, пока не будет закрыта область видимости `FOR`.

Другой пример, который использует статически объявленный массив значений для перебора:

```aql
FOR year IN [ 2011, 2012, 2013 ]
  RETURN {
	"year" : year,
	"isLeapYear" : year % 4 == 0 && (year % 100 != 0 || year % 400 == 0)
  }
```

Допускается также вложение нескольких операторов `FOR`. Когда операторы `FOR` являются вложенными, будет создано перекрестное произведение элементов массива, возвращаемых отдельными операторами `FOR`.

```aql
FOR u IN users
  FOR l IN locations
    RETURN { "user" : u, "location" : l }
```

В этом примере есть две итерации массива: внешняя итерация по массиву `users` плюс внутренняя итерация по массиву `locations`. Внутренний массив просматривается столько раз, сколько элементов во внешнем массиве. Для каждой итерации текущие значения `users` и `locations` становятся доступными для дальнейшей обработки в переменных `u` и `l`.

Вы также можете использовать подзапросы, например, для независимого перебора коллекции и получения результатов обратно в виде массива, к которому затем можно получить доступ во внешнем цикле `FOR`:

```aql
FOR u IN users
  LET subquery = (FOR l IN locations RETURN l.location)
  RETURN { "user": u, "locations": subquery }
```

Также см. [Объединение запросов с подзапросами](../fundamentals/subqueries.md).

## Параметры

Для коллекций и представлений конструкция `FOR` поддерживает необязательное предложение `OPTIONS` для изменения поведения. Общий синтаксис:

<pre>
<code>FOR <em>variableName</em> IN <em>expression</em> OPTIONS { <em>option</em>: <em>value</em>, <em>...</em> }</code>
</pre>

### `indexHint`

Для коллекций индексные подсказки могут быть переданы оптимизатору с помощью опции `indexHint`. Значение может быть одним именем индекса или списком имен индексов в порядке предпочтения:

```aql
FOR … IN … OPTIONS { indexHint: "byName" }
```

```aql
FOR … IN … OPTIONS { indexHint: ["byName", "byColor"] }
```

Всякий раз, когда есть возможность потенциально использовать индекс для этого цикла `FOR`, оптимизатор сначала проверит, можно ли использовать указанный индекс. В случае массива индексов оптимизатор проверит выполнимость каждого индекса в указанном порядке. Он будет использовать первый подходящий индекс, независимо от того, будет ли обычно использоваться другой индекс.

Если ни один из указанных индексов не подходит, он возвращается к своей обычной логике выбора другого индекса или терпит неудачу, если включен `forceIndexHint`.

### `forceIndexHint`

Подсказки индекса не применяются по умолчанию. Если для `forceIndexHint` задано значение `true`, то генерируется ошибка, если `indexHint` не содержит пригодный для использования индекс, вместо использования резервного индекса или вообще без использования индекса.

```aql
FOR … IN … OPTIONS { indexHint: … , forceIndexHint: true }
```

### `disableIndex`

<small>Добавлено в: v3.9.1</small>

In some rare cases it can be beneficial to not do an index lookup or scan,
but to do a full collection scan.
An index lookup can be more expensive than a full collection scan if
the index lookup produces many (or even all documents) and the query cannot
be satisfied from the index data alone.

Consider the following query and an index on the `value` attribute being
present:

```aql
FOR doc IN collection
  FILTER doc.value <= 99
  RETURN doc.other
```

In this case, the optimizer will likely pick the index on `value`, because
it will cover the query's `FILTER` condition. To return the value for the
`other` attribute, the query must additionally look up the documents for
each index value that passes the `FILTER` condition. If the number of
index entries is large (close or equal to the number of documents in the
collection), then using an index can cause more work than just scanning
over all documents in the collection.

The optimizer will likely prefer index scans over full collection scans,
even if an index scan turns out to be slower in the end.
You can force the optimizer to not use an index for any given `FOR`
loop by using the `disableIndex` hint and setting it to `true`:

```aql
FOR doc IN collection OPTIONS { disableIndex: true }
  FILTER doc.value <= 99
  RETURN doc.other
```

Using `disableIndex: false` has no effect on geo indexes or fulltext indexes.

Note that setting `disableIndex: true` plus `indexHint` is ambiguous. In
this case the optimizer will always prefer the `disableIndex` hint.

### `maxProjections`

<small>Introduced in: v3.9.1</small>

By default, the query optimizer will consider up to 5 document attributes
per FOR loop to be used as projections. If more than 5 attributes of a
collection are accessed in a `FOR` loop, the optimizer will prefer to
extract the full document and not use projections.

The threshold value of 5 attributes is arbitrary and can be adjusted
by using the `maxProjections` hint.
The default value for `maxProjections` is `5`, which is compatible with the
previously hard-coded default value.

For example, using a `maxProjections` hint of 7, the following query will
extract 7 attributes as projections from the original document:

```aql
FOR doc IN collection OPTIONS { maxProjections: 7 }
  RETURN [ doc.val1, doc.val2, doc.val3, doc.val4, doc.val5, doc.val6, doc.val7 ]
```

Normally it is not necessary to adjust the value of `maxProjections`, but
there are a few corner cases where it can make sense:

- It can be beneficial to increase `maxProjections` when extracting many small
  attributes from very large documents, and a full copy of the documents should
  be avoided.
- It can be beneficial to decrease `maxProjections` to _avoid_ using
  projections, if the cost of projections is higher than doing copies of the
  full documents. This can be the case for very small documents.

{% hint 'info' %}
Starting with version 3.10, `maxProjections` can be used in
[Graph Traversals](graphs-traversals.html#working-with-named-graphs) (Enterprise Edition only).
{% endhint %}

### `useCache`

<small>Introduced in: v3.10.0</small>

You can disable in-memory caches that you may have enabled for persistent indexes
on a case-by-case basis. This is useful for queries that access indexes with
enabled in-memory caches, but for which it is known that using the cache will
have a negative performance impact. In this case, you can set the `useCache`
hint to `false`:

```aql
FOR doc IN collection OPTIONS { useCache: false }
  FILTER doc.value == @value
  ...
```

You can set the hint individually per `FOR` loop.
If you do not set the `useCache` hint, it will implicitly default to `true`.

The hint does not have any effect on `FOR` loops that do not use indexes, or
on `FOR` loops that access indexes that do not have in-memory caches enabled.
It also does not affect queries for which an existing in-memory
cache cannot be used (i.e. because the query's filter condition does not contain
equality lookups for all index attributes). It cannot be used for `FOR`
operations that iterate over Views or perform graph traversals.

Also see [Caching of index values](../indexing-persistent.html#caching-of-index-values).

### `lookahead`

The multi-dimensional index type `zkd` supports an optional index hint for
tweaking performance:

```aql
FOR … IN … OPTIONS { lookahead: 32 }
```

See [Multi-dimensional indexes](../indexing-multi-dim.html#lookahead-index-hint).
