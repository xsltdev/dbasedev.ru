# Порядок типов и значений

При проверке на равенство или неравенство или при определении порядка сортировки значений AQL использует детерминированный алгоритм, учитывающий как типы данных, так и фактические значения.

Сравниваемые операнды сначала сравниваются по их типам данных и только по их значениям данных, если операнды имеют одинаковые типы данных.

При сравнении типов данных используется следующий порядок типов:

```
null  <  bool  <  number  <  string  <  array/list  <  object/document
```

Это означает, что `null` — это наименьший тип в AQL, а `document` — это тип с наивысшим порядком. Если сравниваемые операнды имеют разный тип, то определяется результат сравнения и сравнение завершается.

Например, логическое `true` значение всегда будет меньше любого числового или строкового значения, любого массива (даже пустого массива) или любого объекта/документа. Кроме того, любое строковое значение (даже пустая строка) всегда будет больше любого числового значения, логического значения, _истинного_ или _ложного_.

```aql
    null  <  false
    null  <  true
    null  <  0
    null  <  ''
    null  <  ' '
    null  <  '0'
    null  <  'abc'
    null  <  [ ]
    null  <  { }

    false  <  true
    false  <  0
    false  <  ''
    false  <  ' '
    false  <  '0'
    false  <  'abc'
    false  <  [ ]
    false  <  { }

    true  <  0
    true  <  ''
    true  <  ' '
    true  <  '0'
    true  <  'abc'
    true  <  [ ]
    true  <  { }

    0  <  ''
    0  <  ' '
    0  <  '0'
    0  <  'abc'
    0  <  [ ]
    0  <  { }

    ''  <  ' '
    ''  <  '0'
    ''  <  'abc'
    ''  <  [ ]
    ''  <  { }

    [ ]  <  { }
```

Если два сравниваемых операнда имеют одинаковые типы данных, то сравниваются значения операндов. Для примитивных типов (null, boolean, number и string) результат определяется следующим образом:

- null: _null_ равен _null_
- boolean: _false_ меньше, чем _true_
- number: числовые значения упорядочены по кардинальному значению
- string: строковые значения упорядочиваются с использованием локализованного сравнения с использованием настроенного языка сервера для сортировки в соответствии с правилами алфавитного порядка этого языка.

Note: unlike in SQL, _null_ can be compared to any value, including _null_
itself, without the result being converted into _null_ automatically.

For compound, types the following special rules are applied:

Two array values are compared by comparing their individual elements position by
position, starting at the first element. For each position, the element types
are compared first. If the types are not equal, the comparison result is
determined, and the comparison is finished. If the types are equal, then the
values of the two elements are compared. If one of the arrays is finished and
the other array still has an element at a compared position, then _null_ will be
used as the element value of the fully traversed array.

If an array element is itself a compound value (an array or an object / document), then the
comparison algorithm will check the element's sub values recursively. The element's
sub-elements are compared recursively.

```aql
[ ]  <  [ 0 ]
[ 1 ]  <  [ 2 ]
[ 1, 2 ]  <  [ 2 ]
[ 99, 99 ]  <  [ 100 ]
[ false ]  <  [ true ]
[ false, 1 ]  <  [ false, '' ]
```

Two object / documents operands are compared by checking attribute names and value. The
attribute names are compared first. Before attribute names are compared, a
combined array of all attribute names from both operands is created and sorted
lexicographically. This means that the order in which attributes are declared
in an object / document is not relevant when comparing two objects / documents.

The combined and sorted array of attribute names is then traversed, and the
respective attributes from the two compared operands are then looked up. If one
of the objects / documents does not have an attribute with the sought name, its attribute
value is considered to be _null_. Finally, the attribute value of both
objects / documents is compared using the before mentioned data type and value comparison.
The comparisons are performed for all object / document attributes until there is an
unambiguous comparison result. If an unambiguous comparison result is found, the
comparison is finished. If there is no unambiguous comparison result, the two
compared objects / documents are considered equal.

```aql
{ }  ==  { "a" : null }

{ }  <  { "a" : 1 }
{ "a" : 1 }  <  { "a" : 2 }
{ "b" : 1 }  <  { "a" : 0 }
{ "a" : { "c" : true } }  <  { "a" : { "c" : 0 } }
{ "a" : { "c" : true, "a" : 0 } }  <  { "a" : { "c" : false, "a" : 1 } }

{ "a" : 1, "b" : 2 }  ==  { "b" : 2, "a" : 1 }
```
