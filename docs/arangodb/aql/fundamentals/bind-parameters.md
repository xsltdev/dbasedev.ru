# Связывание параметров

AQL поддерживает использование параметров привязки, что позволяет отделить текст запроса от литеральных значений, используемых в запросе. Рекомендуется отделять текст запроса от литеральных значений, поскольку это предотвратит (злонамеренное) внедрение ключевых слов и других имен коллекций в существующий запрос. Эта инъекция была бы опасной, потому что она может изменить смысл существующего запроса.

Используя параметры связывания, нельзя изменить значение существующего запроса. Параметры привязки можно использовать везде в запросе, где можно использовать литералы.

## Синтаксис

Общий синтаксис для параметров привязки: `@name`, где `@` означает, что это параметр привязки значения, а `name` — фактическое имя параметра. Его можно использовать для замены значений в запросе.

```aql
RETURN @value
```

Для коллекций используется несколько иной синтаксис `@@coll`, где `@@` означает, что это параметр привязки коллекции, а `coll` — это имя параметра.

```aql
FOR doc IN @@coll
  RETURN doc
```

Ключевые слова и другие языковые конструкции не могут быть заменены значениями привязки, например `FOR`, `FILTER`, `IN`, `INBOUND` или вызовами функций.

Имена параметров привязки должны начинаться с любой из букв от `a` до `z` (верхний или нижний регистр) или цифры (от `0` до `9`), за ними может следовать любая буква, цифра или символ подчеркивания.

Их нельзя заключать в кавычки в коде запроса:

```aql
FILTER u.name == "@name" // wrong
FILTER u.name == @name   // correct
```

```aql
FOR doc IN "@@collection" // wrong
FOR doc IN @@collection   // correct
```

Если вам нужно выполнить обработку строк (конкатенацию и т. д.) в запросе, вам нужно использовать [строковые функции](string.md) для этого:

```aql
FOR u IN users
  FILTER u.id == CONCAT('prefix', @id, 'suffix') && u.name == @name
  RETURN u
```

## Использование

### Простое использование

Значения параметра связывания необходимо передавать вместе с запросом при его выполнении, но не как часть самого текста запроса. В веб-интерфейсе рядом с редактором запросов есть панель, где можно ввести параметры привязки. Для приведенного ниже запроса появятся два поля ввода для ввода значений параметров `id` и `name`.

```aql
FOR u IN users
  FILTER u.id == @id && u.name == @name
  RETURN u
```

При использовании `db._query()` (например, в arangosh) в качестве параметров можно передать объект пар ключ-значение. Такой объект также можно передать в конечную точку HTTP API `_api/cursor` в качестве значения атрибута для ключа `bindVars`:

```json
{
  "query": "FOR u IN users FILTER u.id == @id && u.name == @name RETURN u",
  "bindVars": {
    "id": 123,
    "name": "John Smith"
  }
}
```

Параметры привязки, объявленные в запросе, также должны иметь значение параметра, иначе запрос завершится ошибкой. Указание параметров, не объявленных в запросе, также приведет к ошибке.

### Вложенные атрибуты

Параметры привязки можно использовать как для записи через точку, так и для записи в квадратных скобках для доступа к податрибутам. Они также могут быть связаны:

```aql
LET doc = { foo: { bar: "baz" } }

RETURN doc.@attr.@subattr
// or
RETURN doc[@attr][@subattr]
```

```json
{
  "attr": "foo",
  "subattr": "bar"
}
```

Оба варианта в приведенном выше примере возвращают `[ "baz" ]` в качестве результата запроса.

Полный путь к атрибуту, в частности, для сильно вложенных данных, также можно указать с помощью записи через точку и одного параметра привязки, передав массив строк в качестве значения параметра. Элементы массива представляют ключи атрибутов пути:

```aql
LET doc = { a: { b: { c: 1 } } }
RETURN doc.@attr
```

```json
{ "attr": ["a", "b", "c"] }
```

Пример запроса возвращает `[ 1 ]` в качестве результата. Обратите внимание, что `{ "attr": "a.b.c" }` вернет значение атрибута с именем `a.b.c`, а не значение атрибута `c` с родительскими элементами `a` и `b`, как `[ "a", "b", "c" ]`.

### Параметры привязки коллекции

Для внедрения имен коллекций существует специальный тип параметра привязки. Этот тип параметра связывания имеет имя с префиксом дополнительного символа `@`, поэтому `@@name` в запросе.

```aql
FOR u IN @@collection
  FILTER u.active == true
  RETURN u
```

Второй `@` будет частью имени параметра привязки, что важно помнить при указании `bindVars` (обратите внимание на начальный `@`):

```json
{
  "query": "FOR u IN @@collection FILTER u.active == true RETURN u",
  "bindVars": {
    "@collection": "users"
  }
}
```
