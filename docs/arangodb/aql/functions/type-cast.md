# Проверка типов и функции приведения

Some operators expect their operands to have a certain data type. For example,
logical operators expect their operands to be boolean values, and the arithmetic
operators expect their operands to be numeric values. If an operation is performed
with operands of other types, an automatic conversion to the expected types is
tried. This is called implicit type casting. It helps to avoid query
aborts.

Type casts can also be performed upon request by invoking a type cast function.
This is called explicit type casting. AQL offers several functions for this.
Each of the these functions takes an operand of any data type and returns a result
value with the type corresponding to the function name. For example, _TO_NUMBER()_
will return a numeric value.

## Type casting functions

### TO_BOOL()

`TO_BOOL(value) → bool`

Take an input _value_ of any type and convert it into the appropriate
boolean value.

- **value** (any): input of arbitrary type
- returns **bool** (boolean):
  - _null_ is converted to _false_
  - Numbers are converted to _true_, except for 0, which is converted to _false_
  - Strings are converted to _true_ if they are non-empty, and to _false_ otherwise
  - Arrays are always converted to _true_ (even if empty)
  - Objects / documents are always converted to _true_

It's also possible to use double negation to cast to boolean:

```aql
!!1 // true
!!0 // false
!!-0.0 // false
not not 1 // true
!!"non-empty string" // true
!!"" // false
```

`TO_BOOL()` is preferred however, because it states the intention clearer.

### TO_NUMBER()

`TO_NUMBER(value) → number`

Take an input _value_ of any type and convert it into a numeric value.

- **value** (any): input of arbitrary type
- returns **number** (number):
  - _null_ and _false_ are converted to the value _0_
  - _true_ is converted to _1_
  - Numbers keep their original value
  - Strings are converted to their numeric equivalent if the string contains a
    valid representation of a number. Whitespace at the start and end of the string
    is allowed. String values that do not contain any valid representation of a number
    will be converted to the number _0_.
  - An empty array is converted to _0_, an array with one member is converted into the
    result of `TO_NUMBER()` for its sole member. An array with two or more members is
    converted to the number _0_.
  - An object / document is converted to the number _0_.
  - A unary plus will also cast to a number, but `TO_NUMBER()` is the preferred way:
    ```aql
    +'5' // 5
    +[8] // 8
    +[8,9] // 0
    +{} // 0
    ```
  - A unary minus works likewise, except that a numeric value is also negated:
    ```aql
    -'5' // -5
    -[8] // -8
    -[8,9] // 0
    -{} // 0
    ```

### TO_STRING()

`TO_STRING(value) → str`

Take an input _value_ of any type and convert it into a string value.

- **value** (any): input of arbitrary type
- returns **str** (string):
  - _null_ is converted to an empty string `""`
  - _false_ is converted to the string _"false"_, _true_ to the string _"true"_
  - Numbers are converted to their string representations. This can also be a
    scientific notation (e.g. "2e-7")
  - Arrays and objects / documents are converted to string representations,
    which means JSON-encoded strings with no additional whitespace

```aql
TO_STRING(null) // ""
TO_STRING(true) // "true"
TO_STRING(false) // "false"
TO_STRING(123) // "123"
TO_STRING(+1.23) // "1.23"
TO_STRING(-1.23) // "-1.23"
TO_STRING(0.0000002) // "2e-7"
TO_STRING( [1, 2, 3] ) // "[1,2,3]"
TO_STRING( { foo: "bar", baz: null } ) // "{\"foo\":\"bar\",\"baz\":null}"
```

### TO_ARRAY()

`TO_ARRAY(value) → array`

Take an input _value_ of any type and convert it into an array value.

- **value** (any): input of arbitrary type
- returns **array** (array):
  - _null_ is converted to an empty array
  - Boolean values, numbers and strings are converted to an array containing
    the original value as its single element
  - Arrays keep their original value
  - Objects / documents are converted to an array containing their attribute
    **values** as array elements, just like [VALUES()](functions-document.html#values)

```aql
TO_ARRAY(null) // []
TO_ARRAY(false) // [false]
TO_ARRAY(true) // [true]
TO_ARRAY(5) // [5]
TO_ARRAY("foo") // ["foo"]
TO_ARRAY([1, 2, "foo"]) // [1, 2, "foo"]
TO_ARRAY({foo: 1, bar: 2, baz: [3, 4, 5]}) // [1, 2, [3, 4, 5]]
```

### TO_LIST()

`TO_LIST(value) → array`

This is an alias for [TO_ARRAY()](#to_array).

## Type check functions

AQL also offers functions to check the data type of a value at runtime. The
following type check functions are available. Each of these functions takes an
argument of any data type and returns true if the value has the type that is
checked for, and false otherwise.

### IS_NULL()

`IS_NULL(value) → bool`

Check whether _value_ is _null_. Identical to `value == null`.

To test if an attribute exists, see [HAS()](functions-document.html#has) instead.

- **value** (any): value to test
- returns **bool** (boolean): _true_ if _value_ is `null`,
  _false_ otherwise

### IS_BOOL()

`IS_BOOL(value) → bool`

Check whether _value_ is a _boolean_ value

- **value** (any): value to test
- returns **bool** (boolean): _true_ if _value_ is `true` or `false`,
  _false_ otherwise

### IS_NUMBER()

`IS_NUMBER(value) → bool`

Check whether _value_ is a number

- **value** (any): value to test
- returns **bool** (boolean): _true_ if _value_ is a number,
  _false_ otherwise

### IS_STRING()

`IS_STRING(value) → bool`

Check whether _value_ is a string

- **value** (any): value to test
- returns **bool** (boolean): _true_ if _value_ is a string,
  _false_ otherwise

### IS_ARRAY()

`IS_ARRAY(value) → bool`

Check whether _value_ is an array / list

- **value** (any): value to test
- returns **bool** (boolean): _true_ if _value_ is an array / list,
  _false_ otherwise

### IS_LIST()

`IS_LIST(value) → bool`

This is an alias for [IS_ARRAY()](#is_array)

### IS_OBJECT()

`IS_OBJECT(value) → bool`

Check whether _value_ is an object / document

- **value** (any): value to test
- returns **bool** (boolean): _true_ if _value_ is an object / document,
  _false_ otherwise

### IS_DOCUMENT()

`IS_DOCUMENT(value) → bool`

This is an alias for [IS_OBJECT()](#is_object)

### IS_DATESTRING()

`IS_DATESTRING(str) → bool`

Check whether _value_ is a string that can be used in a date function.
This includes partial dates such as _"2015"_ or _"2015-10"_ and strings
containing properly formatted but invalid dates such as _"2015-02-31"_.

- **str** (string): date string to test
- returns **bool** (boolean): _true_ if _str_ is a correctly formatted date string,
  _false_ otherwise including all non-string values, even if some of them may be usable
  in date functions (numeric timestamps)

### IS_IPV4()

See [String Functions](functions-string.html#is_ipv4).

### IS_KEY()

`IS_KEY(str) → bool`

Check whether _value_ is a string that can be used as a
document key, i.e. as the value of the _\_key_ attribute.
See [Naming Conventions for Document Keys](../data-modeling-naming-conventions-document-keys.html).

- **str** (string): document key to test
- returns **bool** (boolean): whether _str_ can be used as document key

### TYPENAME()

`TYPENAME(value) → typeName`

Return the data type name of _value_.

- **value** (any): input of arbitrary type
- returns **typeName** (string): data type name of _value_
  (`"null"`, `"bool"`, `"number"`, `"string"`, `"array"` or `"object"`)

|   Example Value | Data Type Name |
| --------------: | -------------- |
|          `null` | `"null"`       |
|          `true` | `"bool"`       |
|         `false` | `"bool"`       |
|           `123` | `"number"`     |
|         `-4.56` | `"number"`     |
|             `0` | `"number"`     |
|      `"foobar"` | `"string"`     |
|         `"123"` | `"string"`     |
|            `""` | `"string"`     |
|   `[ 1, 2, 3 ]` | `"array"`      |
|  `["foo",true]` | `"array"`      |
|           `[ ]` | `"array"`      |
| `{"foo":"bar"}` | `"object"`     |
| `{"foo": null}` | `"object"`     |
|           `{ }` | `"object"`     |
