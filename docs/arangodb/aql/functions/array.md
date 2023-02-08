# Функции массива

AQL provides functions for higher-level array manipulation. Also see the
[numeric functions](functions-numeric.html) for functions that work on number arrays.
If you want to concatenate the elements of an array equivalent to `join()`
in JavaScript, see [CONCAT()](functions-string.html#concat) and
[CONCAT_SEPARATOR()](functions-string.html#concat_separator) in the string functions chapter.

Apart from that, AQL also offers several language constructs:

- simple [array access](fundamentals-data-types.html#arrays--lists) of individual elements,
- [array operators](advanced-array-operators.html) for array expansion and contraction,
  optionally with inline filter, limit and projection,
- [array comparison operators](operators.html#array-comparison-operators) to compare
  each element in an array to a value or the elements of another array,
- loop-based operations on arrays using [FOR](operations-for.html),
  [SORT](operations-sort.html),
  [LIMIT](operations-limit.html),
  as well as [COLLECT](operations-collect.html) for grouping,
  which also offers efficient aggregation.

## APPEND()

`APPEND(anyArray, values, unique) → newArray`

Add all elements of an array to another array. All values are added at the end of the
array (right side).

It can also be used to append a single element to an array. It is not necessary to wrap
it in an array (unless it is an array itself). You may also use [PUSH()](#push) instead.

- **anyArray** (array): array with elements of arbitrary type
- **values** (array\|any): array, whose elements shall be added to _anyArray_
- **unique** (bool, _optional_): if set to _true_, only those _values_ will be added
  that are not already contained in _anyArray_. The default is _false_.
- returns **newArray** (array): the modified array

**Examples**

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayAppend_1
@EXAMPLE_AQL{aqlArrayAppend_1}
RETURN APPEND([ 1, 2, 3 ], [ 5, 6, 9 ])
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayAppend_1
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayAppend_2
@EXAMPLE_AQL{aqlArrayAppend_2}
RETURN APPEND([ 1, 2, 3 ], [ 3, 4, 5, 2, 9 ], true)
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayAppend_2
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

## CONTAINS_ARRAY()

This is an alias for [POSITION()](#position).

## COUNT()

This is an alias for [LENGTH()](#length).

## COUNT_DISTINCT()

`COUNT_DISTINCT(anyArray) → number`

Get the number of distinct elements in an array.

- **anyArray** (array): array with elements of arbitrary type
- returns **number**: the number of distinct elements in _anyArray_.

**Examples**

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayCountDistinct_1
@EXAMPLE_AQL{aqlArrayCountDistinct_1}
RETURN COUNT_DISTINCT([ 1, 2, 3 ])
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayCountDistinct_1
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayCountDistinct_2
@EXAMPLE_AQL{aqlArrayCountDistinct_2}
RETURN COUNT_DISTINCT([ "yes", "no", "yes", "sauron", "no", "yes" ])
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayCountDistinct_2
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

## COUNT_UNIQUE()

This is an alias for [COUNT_DISTINCT()](#count_distinct).

## FIRST()

`FIRST(anyArray) → firstElement`

Get the first element of an array. It is the same as `anyArray[0]`.

- **anyArray** (array): array with elements of arbitrary type
- returns **firstElement** (any\|null): the first element of _anyArray_, or _null_ if
  the array is empty.

**Examples**

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayFirst_1
@EXAMPLE_AQL{aqlArrayFirst_1}
RETURN FIRST([ 1, 2, 3 ])
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayFirst_1
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayFirst_2
@EXAMPLE_AQL{aqlArrayFirst_2}
RETURN FIRST([])
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayFirst_2
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

## FLATTEN()

`FLATTEN(anyArray, depth) → flatArray`

Turn an array of arrays into a flat array. All array elements in _array_ will be
expanded in the result array. Non-array elements are added as they are. The function
will recurse into sub-arrays up to the specified depth. Duplicates will not be removed.

Also see [array contraction](advanced-array-operators.html#array-contraction).

- **array** (array): array with elements of arbitrary type, including nested arrays
- **depth** (number, _optional_): flatten up to this many levels, the default is 1
- returns **flatArray** (array): a flattened array

**Examples**

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayFlatten_1
@EXAMPLE_AQL{aqlArrayFlatten_1}
RETURN FLATTEN( [ 1, 2, [ 3, 4 ], 5, [ 6, 7 ], [ 8, [ 9, 10 ] ] ] )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayFlatten_1
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

To fully flatten the example array, use a _depth_ of 2:

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayFlatten_2
@EXAMPLE_AQL{aqlArrayFlatten_2}
RETURN FLATTEN( [ 1, 2, [ 3, 4 ], 5, [ 6, 7 ], [ 8, [ 9, 10 ] ] ], 2 )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayFlatten_2
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

## INTERLEAVE()

<small>Introduced in: v3.7.1</small>

`INTERLEAVE(array1, array2, ... arrayN) → newArray`

Accepts an arbitrary number of arrays and produces a new array with the elements
interleaved. It iterates over the input arrays in a round robin fashion, picks one element
from each array per iteration, and combines them in that sequence into a result array.
The input arrays can have different amounts of elements.

- **arrays** (array, _repeatable_): an arbitrary number of arrays as multiple
  arguments (at least 2)
- returns **newArray** (array): the interleaved array

**Examples**

```
@startDocuBlockInline aqlArrayInterleave_1
@EXAMPLE_AQL{aqlArrayInterleave_1}
RETURN INTERLEAVE( [1, 1, 1], [2, 2, 2], [3, 3, 3] )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayInterleave_1
```

```
@startDocuBlockInline aqlArrayInterleave_2
@EXAMPLE_AQL{aqlArrayInterleave_2}
RETURN INTERLEAVE( [ 1 ], [2, 2], [3, 3, 3] )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayInterleave_2
```

```
@startDocuBlockInline aqlArrayInterleave_3
@EXAMPLE_AQL{aqlArrayInterleave_3}
@DATASET{kShortestPathsGraph}
FOR v, e, p IN 1..3 OUTBOUND 'places/Toronto' GRAPH 'kShortestPathsGraph'
RETURN INTERLEAVE(p.vertices[*].\_id, p.edges[*].\_id)
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayInterleave_3
```

## INTERSECTION()

`INTERSECTION(array1, array2, ... arrayN) → newArray`

Return the intersection of all arrays specified. The result is an array of values that
occur in all arguments.

Other set operations are [UNION()](#union),
[MINUS()](#minus) and
[OUTERSECTION()](#outersection).

- **arrays** (array, _repeatable_): an arbitrary number of arrays as multiple arguments
  (at least 2)
- returns **newArray** (array): a single array with only the elements, which exist in all
  provided arrays. The element order is random. Duplicates are removed.

**Examples**

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayIntersection_1
@EXAMPLE_AQL{aqlArrayIntersection_1}
RETURN INTERSECTION( [1,2,3,4,5], [2,3,4,5,6], [3,4,5,6,7] )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayIntersection_1
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayIntersection_2
@EXAMPLE_AQL{aqlArrayIntersection_2}
RETURN INTERSECTION( [2,4,6], [8,10,12], [14,16,18] )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayIntersection_2
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

## JACCARD()

<small>Introduced in: v3.7.0</small>

`JACCARD(array1, array2) → jaccardIndex`

Calculate the [Jaccard index](https://en.wikipedia.org/wiki/Jaccard_index){:target="\_blank"}
of two arrays.

This similarity measure is also known as _Intersection over Union_ and could
be computed (less efficient and more verbose) as follows:

```aql
COUNT(a) == 0 && COUNT(b) == 0
? 1 // two empty sets have a similarity of 1 by definition
: COUNT(INTERSECTION(array1, array2)) / COUNT(UNION_DISTINCT(array1, array2))
```

- **array1** (array): array with elements of arbitrary type
- **array2** (array): array with elements of arbitrary type
- returns **jaccardIndex** (number): calculated Jaccard index of the input
  arrays _array1_ and _array2_

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayJaccard_1
@EXAMPLE_AQL{aqlArrayJaccard_1}
RETURN JACCARD( [1,2,3,4], [3,4,5,6] )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayJaccard_1
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayJaccard_2
@EXAMPLE_AQL{aqlArrayJaccard_2}
RETURN JACCARD( [1,1,2,2,2,3], [2,2,3,4] )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayJaccard_2
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayJaccard_3
@EXAMPLE_AQL{aqlArrayJaccard_3}
RETURN JACCARD( [1,2,3], [] )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayJaccard_3
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayJaccard_4
@EXAMPLE_AQL{aqlArrayJaccard_4}
RETURN JACCARD( [], [] )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayJaccard_4
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

## LAST()

`LAST(anyArray) → lastElement`

Get the last element of an array. It is the same as `anyArray[-1]`.

- **anyArray** (array): array with elements of arbitrary type
- returns **lastElement** (any\|null): the last element of _anyArray_ or _null_ if the
  array is empty.

**Example**

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayLast_1
@EXAMPLE_AQL{aqlArrayLast_1}
RETURN LAST( [1,2,3,4,5] )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayLast_1
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

## LENGTH()

`LENGTH(anyArray) → length`

Determine the number of elements in an array.

- **anyArray** (array): array with elements of arbitrary type
- returns **length** (number): the number of array elements in _anyArray_.

_LENGTH()_ can also determine the [number of attribute keys](functions-document.html#length)
of an object / document, the [amount of documents](functions-miscellaneous.html#length) in a
collection and the [character length](functions-string.html#length) of a string.

| Input  | Length                                                 |
| ------ | ------------------------------------------------------ |
| String | Number of Unicode characters                           |
| Number | Number of Unicode characters that represent the number |
| Array  | Number of elements                                     |
| Object | Number of first level elements                         |
| true   | 1                                                      |
| false  | 0                                                      |
| null   | 0                                                      |

**Examples**

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayLength_1
@EXAMPLE_AQL{aqlArrayLength_1}
RETURN LENGTH( "🥑" )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayLength_1
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayLength_2
@EXAMPLE_AQL{aqlArrayLength_2}
RETURN LENGTH( 1234 )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayLength_2
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayLength_3
@EXAMPLE_AQL{aqlArrayLength_3}
RETURN LENGTH( [1,2,3,4,5,6,7] )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayLength_3
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayLength_4
@EXAMPLE_AQL{aqlArrayLength_4}
RETURN LENGTH( false )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayLength_4
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayLength_5
@EXAMPLE_AQL{aqlArrayLength_5}
RETURN LENGTH( {a:1, b:2, c:3, d:4, e:{f:5,g:6}} )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayLength_5
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

## MINUS()

`MINUS(array1, array2, ... arrayN) → newArray`

Return the difference of all arrays specified.

Other set operations are [UNION()](#union),
[INTERSECTION()](#intersection) and
[OUTERSECTION()](#outersection).

- **arrays** (array, _repeatable_): an arbitrary number of arrays as multiple
  arguments (at least 2)
- returns **newArray** (array): an array of values that occur in the first array,
  but not in any of the subsequent arrays. The order of the result array is undefined
  and should not be relied on. Duplicates will be removed.

**Example**

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayMinus_1
@EXAMPLE_AQL{aqlArrayMinus_1}
RETURN MINUS( [1,2,3,4], [3,4,5,6], [5,6,7,8] )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayMinus_1
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

## NTH()

`NTH(anyArray, position) → nthElement`

Get the element of an array at a given position. It is the same as `anyArray[position]`
for positive positions, but does not support negative positions.

- **anyArray** (array): array with elements of arbitrary type
- **position** (number): position of desired element in array, positions start at 0
- returns **nthElement** (any\|null): the array element at the given _position_.
  If _position_ is negative or beyond the upper bound of the array,
  then _null_ will be returned.

**Examples**

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayNth_1
@EXAMPLE_AQL{aqlArrayNth_1}
RETURN NTH( [ "foo", "bar", "baz" ], 2 )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayNth_1
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayNth_2
@EXAMPLE_AQL{aqlArrayNth_2}
RETURN NTH( [ "foo", "bar", "baz" ], 3 )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayNth_2
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayNth_3
@EXAMPLE_AQL{aqlArrayNth_3}
RETURN NTH( [ "foo", "bar", "baz" ], -1 )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayNth_3
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

## OUTERSECTION()

`OUTERSECTION(array1, array2, ... arrayN) → newArray`

Return the values that occur only once across all arrays specified.

Other set operations are [UNION()](#union),
[MINUS()](#minus) and
[INTERSECTION()](#intersection).

- **arrays** (array, _repeatable_): an arbitrary number of arrays as multiple arguments
  (at least 2)
- returns **newArray** (array): a single array with only the elements that exist only once
  across all provided arrays. The element order is random.

**Example**

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayOutersection_1
@EXAMPLE_AQL{aqlArrayOutersection_1}
RETURN OUTERSECTION( [ 1, 2, 3 ], [ 2, 3, 4 ], [ 3, 4, 5 ] )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayOutersection_1
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

## POP()

`POP(anyArray) → newArray`

Remove the last element of _array_.

To append an element (right side), see [PUSH()](#push).<br>
To remove the first element, see [SHIFT()](#shift).<br>
To remove an element at an arbitrary position, see [REMOVE_NTH()](#remove_nth).

- **anyArray** (array): an array with elements of arbitrary type
- returns **newArray** (array): _anyArray_ without the last element. If it's already
  empty or has only a single element left, an empty array is returned.

**Examples**

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayPop_1
@EXAMPLE_AQL{aqlArrayPop_1}
RETURN POP( [ 1, 2, 3, 4 ] )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayPop_1
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayPop_2
@EXAMPLE_AQL{aqlArrayPop_2}
RETURN POP( [ 1 ] )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayPop_2
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

## POSITION()

`POSITION(anyArray, search, returnIndex) → position`

Return whether _search_ is contained in _array_. Optionally return the position.

- **anyArray** (array): the haystack, an array with elements of arbitrary type
- **search** (any): the needle, an element of arbitrary type
- **returnIndex** (bool, _optional_): if set to _true_, the position of the match
  is returned instead of a boolean. The default is _false_.
- returns **position** (bool\|number): _true_ if _search_ is contained in _anyArray_,
  _false_ otherwise. If _returnIndex_ is enabled, the position of the match is
  returned (positions start at 0), or _-1_ if it's not found.

To determine if or at which position a string occurs in another string, see the
[CONTAINS() string function](functions-string.html#contains).

**Examples**

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayPosition_1
@EXAMPLE_AQL{aqlArrayPosition_1}
RETURN POSITION( [2,4,6,8], 4 )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayPosition_1
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayPosition_2
@EXAMPLE_AQL{aqlArrayPosition_2}
RETURN POSITION( [2,4,6,8], 4, true )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayPosition_2
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

## PUSH()

`PUSH(anyArray, value, unique) → newArray`

Append _value_ to _anyArray_ (right side).

To remove the last element, see [POP()](#pop).<br>
To prepend a value (left side), see [UNSHIFT()](#unshift).<br>
To append multiple elements, see [APPEND()](#append).

- **anyArray** (array): array with elements of arbitrary type
- **value** (any): an element of arbitrary type
- **unique** (bool): if set to _true_, then _value_ is not added if already
  present in the array. The default is _false_.
- returns **newArray** (array): _anyArray_ with _value_ added at the end
  (right side)

Note: The _unique_ flag only controls if _value_ is added if it's already present
in _anyArray_. Duplicate elements that already exist in _anyArray_ will not be
removed. To make an array unique, use the [UNIQUE()](#unique) function.

**Examples**

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayPush_1
@EXAMPLE_AQL{aqlArrayPush_1}
RETURN PUSH([ 1, 2, 3 ], 4)
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayPush_1
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayPush_2
@EXAMPLE_AQL{aqlArrayPush_2}
RETURN PUSH([ 1, 2, 2, 3 ], 2, true)
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayPush_2
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

## REMOVE_NTH()

`REMOVE_NTH(anyArray, position) → newArray`

Remove the element at _position_ from the _anyArray_.

To remove the first element, see [SHIFT()](#shift).<br>
To remove the last element, see [POP()](#pop).

- **anyArray** (array): array with elements of arbitrary type
- **position** (number): the position of the element to remove. Positions start
  at 0. Negative positions are supported, with -1 being the last array element.
  If _position_ is out of bounds, the array is returned unmodified.
- returns **newArray** (array): _anyArray_ without the element at _position_

**Examples**

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayRemoveNth_1
@EXAMPLE_AQL{aqlArrayRemoveNth_1}
RETURN REMOVE_NTH( [ "a", "b", "c", "d", "e" ], 1 )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayRemoveNth_1
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayRemoveNth_2
@EXAMPLE_AQL{aqlArrayRemoveNth_2}
RETURN REMOVE_NTH( [ "a", "b", "c", "d", "e" ], -2 )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayRemoveNth_2
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

## REPLACE_NTH()

<small>Introduced in: v3.7.0</small>

`REPLACE_NTH(anyArray, position, replaceValue, defaultPaddingValue) → newArray`

Replace the element at _position_ in _anyArray_ with _replaceValue_.

- **anyArray** (array): array with elements of arbitrary type
- **position** (number): the position of the element to replace. Positions start
  at 0. Negative positions are supported, with -1 being the last array element.
  If a negative _position_ is out of bounds, then it is set to the first element (0)
- **replaceValue** the value to be inserted at _position_
- **defaultPaddingValue** to be used for padding if _position_ is two or more
  elements beyond the last element in _anyArray_
- returns **newArray** (array): _anyArray_ with the element at _position_
  replaced by _replaceValue_, or appended to _anyArray_ and possibly padded by
  _defaultPaddingValue_

It is allowed to specify a position beyond the upper array boundary:

- _replaceValue_ is appended if _position_ is equal to the array length
- if it is higher, _defaultPaddingValue_ is appended to _anyArray_ as many
  times as needed to place _replaceValue_ at _position_
- if no _defaultPaddingValue_ is supplied in above case, then a query error
  is raised

**Examples**

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayReplaceNth_1
@EXAMPLE_AQL{aqlArrayReplaceNth_1}
RETURN REPLACE_NTH( [ "a", "b", "c" ], 1 , "z")
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayReplaceNth_1
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayReplaceNth_2
@EXAMPLE_AQL{aqlArrayReplaceNth_2}
RETURN REPLACE_NTH( [ "a", "b", "c" ], 3 , "z")
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayReplaceNth_2
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayReplaceNth_4
@EXAMPLE_AQL{aqlArrayReplaceNth_4}
RETURN REPLACE_NTH( [ "a", "b", "c" ], 6, "z", "y" )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayReplaceNth_4
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayReplaceNth_5
@EXAMPLE_AQL{aqlArrayReplaceNth_5}
RETURN REPLACE_NTH( [ "a", "b", "c" ], -1, "z" )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayReplaceNth_5
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayReplaceNth_6
@EXAMPLE_AQL{aqlArrayReplaceNth_6}
RETURN REPLACE_NTH( [ "a", "b", "c" ], -9, "z" )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayReplaceNth_6
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

Trying to access out of bounds, without providing a padding value will result in an error:

{% arangoshexample examplevar="examplevar" script="script" result="result" %}
@startDocuBlockInline aqlArrayReplaceNth_3
@EXAMPLE_ARANGOSH_OUTPUT{aqlArrayReplaceNth_3}
db.\_query('RETURN REPLACE_NTH( [ "a", "b", "c" ], 6 , "z")'); // xpError(ERROR_QUERY_FUNCTION_ARGUMENT_TYPE_MISMATCH)
@END_EXAMPLE_ARANGOSH_OUTPUT
@endDocuBlock aqlArrayReplaceNth_3
{% endarangoshexample %}
{% include arangoshexample.html id=examplevar script=script result=result %}

## REMOVE_VALUE()

`REMOVE_VALUE(anyArray, value, limit) → newArray`

Remove all occurrences of _value_ in _anyArray_. Optionally with a _limit_
to the number of removals.

- **anyArray** (array): array with elements of arbitrary type
- **value** (any): an element of arbitrary type
- **limit** (number, _optional_): cap the number of removals to this value
- returns **newArray** (array): _anyArray_ with _value_ removed

**Examples**

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayRemoveValue_1
@EXAMPLE_AQL{aqlArrayRemoveValue_1}
RETURN REMOVE_VALUE( [ "a", "b", "b", "a", "c" ], "a" )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayRemoveValue_1
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayRemoveValue_2
@EXAMPLE_AQL{aqlArrayRemoveValue_2}
RETURN REMOVE_VALUE( [ "a", "b", "b", "a", "c" ], "a", 1 )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayRemoveValue_2
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

## REMOVE_VALUES()

`REMOVE_VALUES(anyArray, values) → newArray`

Remove all occurrences of any of the _values_ from _anyArray_.

- **anyArray** (array): array with elements of arbitrary type
- **values** (array): an array with elements of arbitrary type, that shall
  be removed from _anyArray_
- returns **newArray** (array): _anyArray_ with all individual _values_ removed

**Example**

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayRemoveValues_1
@EXAMPLE_AQL{aqlArrayRemoveValues_1}
RETURN REMOVE_VALUES( [ "a", "a", "b", "c", "d", "e", "f" ], [ "a", "f", "d" ] )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayRemoveValues_1
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

## REVERSE()

`REVERSE(anyArray) → reversedArray`

Return an array with its elements reversed.

- **anyArray** (array): array with elements of arbitrary type
- returns **reversedArray** (array): a new array with all elements of _anyArray_ in
  reversed order

**Example**

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayReverse_1
@EXAMPLE_AQL{aqlArrayReverse_1}
RETURN REVERSE ( [2,4,6,8,10] )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayReverse_1
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

## SHIFT()

`SHIFT(anyArray) → newArray`

Remove the first element of _anyArray_.

To prepend an element (left side), see [UNSHIFT()](#unshift).<br>
To remove the last element, see [POP()](#pop).<br>
To remove an element at an arbitrary position, see [REMOVE_NTH()](#remove_nth).

- **anyArray** (array): array with elements with arbitrary type
- returns **newArray** (array): _anyArray_ without the left-most element. If _anyArray_
  is already empty or has only one element left, an empty array is returned.

**Examples**

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayShift_1
@EXAMPLE_AQL{aqlArrayShift_1}
RETURN SHIFT( [ 1, 2, 3, 4 ] )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayShift_1
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayShift_2
@EXAMPLE_AQL{aqlArrayShift_2}
RETURN SHIFT( [ 1 ] )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayShift_2
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

## SLICE()

`SLICE(anyArray, start, length) → newArray`

Extract a slice of _anyArray_.

- **anyArray** (array): array with elements of arbitrary type
- **start** (number): start extraction at this element. Positions start at 0.
  Negative values indicate positions from the end of the array.
- **length** (number, _optional_): extract up to _length_ elements, or all
  elements from _start_ up to _length_ if negative (exclusive)
- returns **newArray** (array): the specified slice of _anyArray_. If _length_
  is not specified, all array elements starting at _start_ will be returned.

**Examples**

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArraySlice_1
@EXAMPLE_AQL{aqlArraySlice_1}
RETURN SLICE( [ 1, 2, 3, 4, 5 ], 0, 1 )
@END_EXAMPLE_AQL
@endDocuBlock aqlArraySlice_1
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArraySlice_2
@EXAMPLE_AQL{aqlArraySlice_2}
RETURN SLICE( [ 1, 2, 3, 4, 5 ], 1, 2 )
@END_EXAMPLE_AQL
@endDocuBlock aqlArraySlice_2
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArraySlice_3
@EXAMPLE_AQL{aqlArraySlice_3}
RETURN SLICE( [ 1, 2, 3, 4, 5 ], 3 )
@END_EXAMPLE_AQL
@endDocuBlock aqlArraySlice_3
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArraySlice_4
@EXAMPLE_AQL{aqlArraySlice_4}
RETURN SLICE( [ 1, 2, 3, 4, 5 ], 1, -1 )
@END_EXAMPLE_AQL
@endDocuBlock aqlArraySlice_4
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArraySlice_5
@EXAMPLE_AQL{aqlArraySlice_5}
RETURN SLICE( [ 1, 2, 3, 4, 5 ], 0, -2 )
@END_EXAMPLE_AQL
@endDocuBlock aqlArraySlice_5
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArraySlice_6
@EXAMPLE_AQL{aqlArraySlice_6}
RETURN SLICE( [ 1, 2, 3, 4, 5 ], -3, 2 )
@END_EXAMPLE_AQL
@endDocuBlock aqlArraySlice_6
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

## SORTED()

`SORTED(anyArray) → newArray`

Sort all elements in _anyArray_. The function will use the default comparison
order for AQL value types.

- **anyArray** (array): array with elements of arbitrary type
- returns **newArray** (array): _anyArray_, with elements sorted

**Example**

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArraySorted_1
@EXAMPLE_AQL{aqlArraySorted_1}
RETURN SORTED( [ 8,4,2,10,6 ] )
@END_EXAMPLE_AQL
@endDocuBlock aqlArraySorted_1
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

## SORTED_UNIQUE()

`SORTED_UNIQUE(anyArray) → newArray`

Sort all elements in _anyArray_. The function will use the default comparison
order for AQL value types. Additionally, the values in the result array will
be made unique.

- **anyArray** (array): array with elements of arbitrary type
- returns **newArray** (array): _anyArray_, with elements sorted and duplicates
  removed

**Example**

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArraySortedUnique_1
@EXAMPLE_AQL{aqlArraySortedUnique_1}
RETURN SORTED_UNIQUE( [ 8,4,2,10,6,2,8,6,4 ] )
@END_EXAMPLE_AQL
@endDocuBlock aqlArraySortedUnique_1
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

## UNION()

`UNION(array1, array2, ... arrayN) → newArray`

Return the union of all arrays specified.

Other set operations are [MINUS()](#minus),
[INTERSECTION()](#intersection) and
[OUTERSECTION()](#outersection).

- **arrays** (array, _repeatable_): an arbitrary number of arrays as multiple
  arguments (at least 2)
- returns **newArray** (array): all array elements combined in a single array,
  in any order

**Examples**

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayUnion_1
@EXAMPLE_AQL{aqlArrayUnion_1}
RETURN UNION(
[ 1, 2, 3 ],
[ 1, 2 ]
)
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayUnion_1
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

Note: No duplicates will be removed. In order to remove duplicates, please use
either [UNION_DISTINCT()](#union_distinct)
or apply [UNIQUE()](#unique) on the
result of _UNION()_:

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayUnion_2
@EXAMPLE_AQL{aqlArrayUnion_2}
RETURN UNIQUE(
UNION(
[ 1, 2, 3 ],
[ 1, 2 ]
)
)
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayUnion_2
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

## UNION_DISTINCT()

`UNION_DISTINCT(array1, array2, ... arrayN) → newArray`

Return the union of distinct values of all arrays specified.

- **arrays** (array, _repeatable_): an arbitrary number of arrays as multiple
  arguments (at least 2)
- returns **newArray** (array): the elements of all given arrays in a single
  array, without duplicates, in any order

**Example**

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayUnionDistinct_1
@EXAMPLE_AQL{aqlArrayUnionDistinct_1}
RETURN UNION_DISTINCT(
[ 1, 2, 3 ],
[ 1, 2 ]
)
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayUnionDistinct_1
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

## UNIQUE()

`UNIQUE(anyArray) → newArray`

Return all unique elements in _anyArray_. To determine uniqueness, the
function will use the comparison order.

- **anyArray** (array): array with elements of arbitrary type
- returns **newArray** (array): _anyArray_ without duplicates, in any order

**Example**

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayUnique_1
@EXAMPLE_AQL{aqlArrayUnique_1}
RETURN UNIQUE( [ 1,2,2,3,3,3,4,4,4,4,5,5,5,5,5 ] )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayUnique_1
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

## UNSHIFT()

`UNSHIFT(anyArray, value, unique) → newArray`

Prepend _value_ to _anyArray_ (left side).

To remove the first element, see [SHIFT()](#shift).<br>
To append a value (right side), see [PUSH()](#push).

- **anyArray** (array): array with elements of arbitrary type
- **value** (any): an element of arbitrary type
- **unique** (bool): if set to _true_, then _value_ is not added if already
  present in the array. The default is _false_.
- returns **newArray** (array): _anyArray_ with _value_ added at the start
  (left side)

Note: The _unique_ flag only controls if _value_ is added if it's already present
in _anyArray_. Duplicate elements that already exist in _anyArray_ will not be
removed. To make an array unique, use the [UNIQUE()](#unique) function.

**Examples**

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayUnshift_1
@EXAMPLE_AQL{aqlArrayUnshift_1}
RETURN UNSHIFT( [ 1, 2, 3 ], 4 )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayUnshift_1
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}

{% aqlexample examplevar="examplevar" type="type" query="query" bind="bind" result="result" %}
@startDocuBlockInline aqlArrayUnshift_2
@EXAMPLE_AQL{aqlArrayUnshift_2}
RETURN UNSHIFT( [ 1, 2, 3 ], 2, true )
@END_EXAMPLE_AQL
@endDocuBlock aqlArrayUnshift_2
{% endaqlexample %}
{% include aqlexample.html id=examplevar type=type query=query bind=bind result=result %}
