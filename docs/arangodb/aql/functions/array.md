# Функции массива

AQL предоставляет функции для манипулирования массивами на более высоком уровне. Также смотрите [numeric functions](numeric.md) для функций, работающих с массивами чисел. Если вы хотите объединить элементы массива, эквивалентно `join()` в JavaScript, смотрите [`CONCAT()`](string.md) и [`CONCAT_SEPARATOR()`](string.md) в главе о строковых функциях.

Помимо этого, AQL также предлагает несколько языковых конструкций:

-   простой [доступ к массивам](../fundamentals/data-types.md) отдельных элементов,
-   [операторы массива](../advanced/array-operators.md) для расширения и сжатия массива, опционально с встроенным фильтром, ограничением и проекцией,
-   [операторы сравнения массивов](../operators.md) для сравнения каждого элемента массива со значением или элементами другого массива,
-   операции над массивами с помощью циклов [`FOR`](../operations/for.md), [`SORT`](../operations/sort.md), [`LIMIT`](../operations/limit.md), а также [`COLLECT`](../operations/collect.md) для группировки, который также предлагает эффективную агрегацию.

## APPEND()

`APPEND(anyArray, values, unique) → newArray`

Добавляет все элементы массива в другой массив. Все значения добавляются в конец массива (правая часть).

Также может использоваться для добавления одного элемента в массив. Нет необходимости оборачивать его в массив (если только он сам не является массивом). Вместо него можно также использовать `PUSH()`.

-   **anyArray** (массив): массив с элементами произвольного типа
-   **values** (array|any): массив, элементы которого должны быть добавлены к _anyArray_.
-   **unique** (bool, _опционально_): если установлено значение _true_, будут добавлены только те _значения_, которые еще не содержатся в _anyArray_. По умолчанию используется значение _false_.
-   возвращает **newArray** (массив): измененный массив.

## CONTAINS_ARRAY()

Это псевдоним для `POSITION()`.

## COUNT()

Это псевдоним для `LENGTH()`.

## COUNT_DISTINCT()

`COUNT_DISTINCT(anyArray) → number`.

Получает количество отдельных элементов в массиве.

-   **anyArray** (массив): массив с элементами произвольного типа
-   возвращает **number**: количество отдельных элементов в _anyArray_.

## COUNT_UNIQUE()

Это псевдоним для `COUNT_DISTINCT()`.

## FIRST()

`FIRST(anyArray) → firstElement`

Получает первый элемент массива. Это то же самое, что и `anyArray[0]`.

-   **anyArray** (массив): массив с элементами произвольного типа
-   возвращает **firstElement** (any|null): первый элемент _anyArray_, или _null_, если массив пуст.

## FLATTEN()

`FLATTEN(anyArray, depth) → flatArray`

Превращает массив массивов в плоский массив. Все элементы массива _array_ будут расширены в результирующем массиве. Элементы, не входящие в массив, добавляются по мере их наличия. Функция выполняет перебор в подмассивы до указанной глубины. Дубликаты не удаляются.

Также смотрите [сокращение массива](advanced-array-operators.html#array-contraction).

-   **array** (массив): массив с элементами произвольного типа, включая вложенные массивы.
-   **depth** (число, _опционально_): сплющивание до этого количества уровней, по умолчанию 1.
-   возвращает **flatArray** (массив): сплющенный массив

## INTERLEAVE()

<small>Введено в: v3.7.1</small>

`INTERLEAVE(array1, array2, ... arrayN) → newArray`

Принимает произвольное количество массивов и создает новый массив с чередованием элементов. Он перебирает входные массивы по кругу, выбирает по одному элементу из каждого массива за итерацию и объединяет их в такой последовательности в результирующий массив. Входные массивы могут содержать разное количество элементов.

<!-- 0003.part.md -->

-   **arrays** (массив, _повторяемый_): произвольное количество массивов в качестве нескольких аргументов (не менее 2)
-   возвращает **newArray** (array): чередующийся массив

## INTERSECTION()

`INTERSECTION(array1, array2, ... arrayN) → newArray`

Возвращает пересечение всех указанных массивов. Результатом является массив значений, которые встречаются во всех аргументах.

Другие операции над множествами: `UNION()`, `MINUS()` и `OUTERSECTION()`.

-   **arrays** (array, _repeatable_): произвольное количество массивов в качестве нескольких аргументов (не менее 2).
-   возвращает **newArray** (массив): единый массив, содержащий только те элементы, которые есть во всех предоставленных массивах. Порядок элементов случайный. Дубликаты удаляются.

<!-- 0004.part.md -->

## JACCARD()

<small>Введено в: v3.7.0</small>

`JACCARD(array1, array2) → jaccardIndex`

Вычисляет [индекс Жаккара](https://en.wikipedia.org/wiki/Jaccard_index) двух массивов.

Эта мера сходства также известна как _Intersection over Union_ и может быть вычислена (менее эффективно и более многословно) следующим образом:

<!-- 0005.part.md -->

```aql
COUNT(a) == 0 && COUNT(b) == 0
? 1 // two empty sets have a similarity of 1 by definition
: COUNT(INTERSECTION(array1, array2)) / COUNT(UNION_DISTINCT(array1, array2))
```

<!-- 0006.part.md -->

-   **array1** (массив): массив с элементами произвольного типа
-   **array2** (массив): массив с элементами произвольного типа
-   возвращает **jaccardIndex** (число): вычисленный индекс Жаккарда входных массивов _array1_ и _array2_.

## LAST()

`LAST(anyArray) → lastElement`

Получает последний элемент массива. Это то же самое, что и `anyArray[-1]`.

-   **anyArray** (массив): массив с элементами произвольного типа
-   возвращает **lastElement** (any|null): последний элемент _anyArray_ или _null_, если массив пуст.

## LENGTH()

`LENGTH(anyArray) → length`

Определите количество элементов в массиве.

-   **anyArray** (массив): массив с элементами произвольного типа
-   возвращает **length** (число): количество элементов массива _anyArray_.

_LENGTH()_ также может определить [количество ключей атрибутов](document.md) объекта / документа, [количество документов](miscellaneous.md) в коллекции и [длину символов](string.md) строки.

| Вход   | Длина                                             |
| ------ | ------------------------------------------------- |
| Строка | Количество символов Юникода                       |
| Число  | Количество символов Юникода, представляющих число |
| Массив | Количество элементов                              |
| Object | Количество элементов первого уровня               |
| true   | 1                                                 |
| false  | 0                                                 |
| null   | 0                                                 |

## MINUS()

`MINUS(array1, array2, ... arrayN) → newArray`

Возвращает разность всех указанных массивов.

Другие операции над множествами: [`UNION()`](#union), [`INTERSECTION()`](#intersection) и [`OUTERSECTION()`](#outersection).

-   **arrays** (array, _repeatable_): произвольное количество массивов в качестве нескольких аргументов (не менее 2).
-   возвращает **newArray** (массив): массив значений, которые встречаются в первом массиве, но не встречаются ни в одном из последующих массивов. Порядок в результирующем массиве не определен, и на него не следует полагаться. Дубликаты будут удалены.

<!-- 0009.part.md -->

## NTH()

`NTH(anyArray, position) → nthElement`

Получение элемента массива в заданной позиции. Это то же самое, что и `anyArray[position]` для положительных позиций, но не поддерживает отрицательные позиции.

-   **anyArray** (массив): массив с элементами произвольного типа
-   **position** (число): позиция нужного элемента в массиве, позиции начинаются с 0.
-   возвращает **nthElement** (any|null): элемент массива в заданной _позиции_. Если _позиция_ отрицательна или выходит за верхнюю границу массива, то будет возвращен _null_.

## OUTERSECTION()

`OUTERSECTION(array1, array2, ... arrayN) → newArray`

Возвращает значения, которые встречаются только один раз во всех указанных массивах.

Другие операции над множествами: [UNION()](#union), [MINUS()](#minus) и [INTERSECTION()](#intersection).

-   **arrays** (array, _repeatable_): произвольное количество массивов в качестве нескольких аргументов (не менее 2).
-   возвращает **newArray** (массив): единый массив, содержащий только те элементы, которые существуют только один раз во всех предоставленных массивах. Порядок элементов является случайным.

<!-- 0010.part.md -->

## POP()

`POP(anyArray) → newArray`

Удалить последний элемент из _массива_.

Чтобы добавить элемент (правая сторона), смотрите [PUSH()](#push).

Чтобы удалить первый элемент, смотрите [SHIFT()](#shift).

Чтобы удалить элемент в произвольной позиции, смотрите [REMOVE_NTH()](#remove_nth).

-   **anyArray** (массив): массив с элементами произвольного типа
-   возвращает **newArray** (массив): _любойМассив_ без последнего элемента. Если он уже пуст или в нем остался только один элемент, возвращается пустой массив.

## POSITION()

`POSITION(anyArray, search, returnIndex) → position`

Возвращает, содержится ли _search_ в _array_. Опционально возвращает позицию.

-   **anyArray** (массив): стог сена, массив с элементами произвольного типа.
-   **search** (any): иголка, элемент произвольного типа.
-   **returnIndex** (bool, _опционально_): если установлено значение _true_, вместо булевого числа возвращается позиция совпадения. По умолчанию _false_.
-   возвращает **позицию** (bool|number): _true_, если _search_ содержится в _anyArray_, _false_ в противном случае. Если включен _returnIndex_, возвращается позиция совпадения (позиции начинаются с 0), или _-1_, если оно не найдено.

Чтобы определить, встречается ли строка в другой строке и в какой позиции, смотрите функцию [CONTAINS() string](string.md).

## PUSH()

`PUSH(anyArray, value, unique) → newArray`

Добавляет _значение_ к _любому массиву_ (правая часть).

Чтобы удалить последний элемент, смотрите [POP()](#pop).

Чтобы добавить значение (левая часть), смотрите [UNSHIFT()](#unshift).

Чтобы добавить несколько элементов, смотрите [APPEND()](#append).

-   **anyArray** (массив): массив с элементами произвольного типа.
-   **значение** (any): элемент произвольного типа
-   **unique** (bool): если установлено значение _true_, то _значение_ не добавляется, если уже присутствует в массиве. По умолчанию _false_.
-   возвращает **newArray** (массив): _любой массив_ с _значением_, добавленным в конец (правая часть).

Примечание: Флаг _unique_ контролирует только добавление _значения_, если оно уже присутствует в _anyArray_. Дублирующие элементы, которые уже существуют в _любом массиве_, не будут удалены. Чтобы сделать массив уникальным, используйте функцию [UNIQUE()](#unique).

## REMOVE_NTH()

`REMOVE_NTH(anyArray, position) → newArray`

Удаляет элемент в _позиции_ из _anyArray_.

Чтобы удалить первый элемент, смотрите [SHIFT()](#shift).

Чтобы удалить последний элемент, смотрите [POP()](#pop).

-   **anyArray** (массив): массив с элементами произвольного типа.
-   **position** (число): позиция элемента, который нужно удалить. Позиции начинаются с 0. Поддерживаются отрицательные позиции, при этом -1 является последним элементом массива. Если _position_ выходит за границы, массив возвращается без изменений.
-   возвращает **newArray** (массив): _любой массив_ без элемента в _позиции_.

## REPLACE_NTH()

<small>Введено в: v3.7.0</small>

`REPLACE_NTH(anyArray, position, replaceValue, defaultPaddingValue) → newArray`

Заменяет элемент в _позиции_ в _anyArray_ на _replaceValue_.

-   **anyArray** (массив): массив с элементами произвольного типа
-   **position** (число): позиция заменяемого элемента. Позиции начинаются с 0. Поддерживаются отрицательные позиции, при этом -1 является последним элементом массива. Если отрицательная _позиция_ выходит за границы, то она устанавливается на первый элемент (0).
-   **replaceValue** значение, которое будет вставлено в _позицию_.
-   **defaultPaddingValue** для вставки, если _position_ находится на два или более элемента дальше последнего элемента в _любом массиве_.
-   возвращает **newArray** (массив): _anyArray_ с элементом в _позиции_, замененным _replaceValue_, или добавленным к _anyArray_ и, возможно, заполненным _defaultPaddingValue_.

Допускается указывать позицию за верхней границей массива:

-   _replaceValue_ добавляется, если _position_ равно длине массива
-   если больше, то _defaultPaddingValue_ добавляется к _anyArray_ столько раз, сколько необходимо для размещения _replaceValue_ в _позиции_
-   если в вышеприведенном случае не указано _defaultPaddingValue_, то выдается ошибка запроса

## REMOVE_VALUE()

`REMOVE_VALUE(anyArray, value, limit) → newArray`

Удаляет все вхождения _value_ в _anyArray_. Опционально с _ограничением_ на количество удалений.

-   **anyArray** (массив): массив с элементами произвольного типа
-   **значение** (любое): элемент произвольного типа
-   **limit** (число, _опционально_): ограничение числа удалений этим значением.
-   возвращает **newArray** (массив): _любой массив_ с удаленным _значением_.

## REMOVE_VALUES()

`REMOVE_VALUES(anyArray, values) → newArray`

Удаляет все вхождения любого из _значений_ из _любого массива_.

-   **anyArray** (массив): массив с элементами произвольного типа
-   **values** (массив): массив с элементами произвольного типа, которые должны быть удалены из _anyArray_.
-   возвращает **newArray** (массив): _любойМассив_ с удаленными отдельными _значениями_.

## REVERSE()

`REVERSE(anyArray) → reversedArray`

Возвращает массив с перевернутыми элементами.

-   **anyArray** (массив): массив с элементами произвольного типа
-   возвращает **reversedArray** (массив): новый массив со всеми элементами _anyArray_ в обратном порядке.

## SHIFT()

`SHIFT(anyArray) → newArray`

Удаляет первый элемент из _anyArray_.

Чтобы добавить элемент (с левой стороны), смотрите [UNSHIFT()](#unshift).

Чтобы удалить последний элемент, смотрите [POP()](#pop).

Чтобы удалить элемент в произвольной позиции, смотрите [REMOVE_NTH()](#remove_nth).

<!-- 0016.part.md -->

-   **anyArray** (массив): массив с элементами произвольного типа
-   возвращает **newArray** (массив): _anyArray_ без крайнего левого элемента. Если _anyArray_ уже пуст или в нем остался только один элемент, возвращается пустой массив.

## SLICE()

`SLICE(anyArray, start, length) → newArray`

Извлечение фрагмента из _любого массива_.

-   **anyArray** (массив): массив с элементами произвольного типа
-   **start** (число): начать извлечение с данного элемента. Позиции начинаются с 0. Отрицательные значения указывают на позиции с конца массива.
-   **length** (число, _опционально_): извлечение до _length_ элементов, или всех элементов от _start_ до _length_, если отрицательно (исключающее).
-   возвращает **newArray** (массив): указанный фрагмент _любого массива_. Если _length_ не указано, будут возвращены все элементы массива, начиная с _start_.

## SORTED()

`SORTED(anyArray) → newArray`

Сортирует все элементы в _любом массиве_. Функция будет использовать порядок сравнения по умолчанию для типов значений AQL.

-   **anyArray** (массив): массив с элементами произвольного типа
-   возвращает **newArray** (массив): _anyArray_, с отсортированными элементами.

<!-- 0018.part.md -->

## SORTED_UNIQUE()

`SORTED_UNIQUE(anyArray) → newArray`

Сортирует все элементы в _любом массиве_. Функция будет использовать порядок сравнения по умолчанию для типов значений AQL. Кроме того, значения в результирующем массиве будут уникальными.

-   **anyArray** (массив): массив с элементами произвольного типа
-   возвращает **newArray** (массив): _любой массив_, элементы которого отсортированы и удалены дубликаты.

## UNION()

`UNION(array1, array2, ... arrayN) → newArray`

Возвращает объединение всех указанных массивов.

Другие операции с массивами: [MINUS()](#minus), [INTERSECTION()](#intersection) и [OUTERSECTION()](#outersection).

-   **arrays** (массив, _повторяемый_): произвольное количество массивов в качестве нескольких аргументов (не менее 2).
-   возвращает **newArray** (массив): все элементы массива, объединенные в один массив, в любом порядке

<!-- 0019.part.md -->

## UNION_DISTINCT()

`UNION_DISTINCT(array1, array2, ... arrayN) → newArray`.

Возвращает объединение отличительных значений всех указанных массивов.

-   **arrays** (array, _repeatable_): произвольное количество массивов в качестве нескольких аргументов (не менее 2).
-   возвращает **newArray** (массив): элементы всех заданных массивов в одном массиве, без дубликатов, в любом порядке.

## UNIQUE()

`UNIQUE(anyArray) → newArray`

Возвращает все уникальные элементы в _anyArray_. Для определения уникальности функция будет использовать порядок сравнения.

-   **anyArray** (массив): массив с элементами произвольного типа
-   возвращает **newArray** (массив): _любой массив_ без дубликатов, в любом порядке.

## UNSHIFT()

`UNSHIFT(anyArray, value, unique) → newArray`

Добавьте _значение_ к _любому массиву_ (левая часть).

Чтобы удалить первый элемент, смотрите [SHIFT()](#shift).

Чтобы добавить значение (правая часть), смотрите [PUSH()](#push).

-   **anyArray** (массив): массив с элементами произвольного типа.
-   **значение** (any): элемент произвольного типа
-   **unique** (bool): если установлено значение _true_, то _значение_ не добавляется, если уже присутствует в массиве. По умолчанию _false_.
-   возвращает **newArray** (массив): _любой массив_ с _значением_, добавленным в начало (левая часть).

<!-- 0020.part.md -->

Примечание: Флаг _unique_ контролирует только добавление _значения_, если оно уже присутствует в _anyArray_. Дублирующие элементы, которые уже существуют в _любом массиве_, не будут удалены. Чтобы сделать массив уникальным, используйте функцию [UNIQUE()](#unique).
