# Битовые функции

<small>Введено в: v3.7.7</small>

AQL предлагает некоторые функции побитового манипулирования и интерпретации для побитовой арифметики.

Эти функции могут работать с целыми числовыми значениями в диапазоне от 0 до 4294967295 (2<sup>32</sup> - 1), включая оба значения. Это позволяет рассматривать числа как наборы битов, включающие до 32 членов. Использование любой из битовых функций для чисел вне поддерживаемого диапазона заставит функцию вернуть `null` и зарегистрировать предупреждение.

Диапазон значений для битовых функций консервативно мал, поэтому при передаче входных/выходных значений битовых функций клиентским приложениям с числами неизвестной точности не должно происходить потери точности или ошибок округления.

## `BIT_AND()`

`BIT_AND(numbersArray) → result`

And- объединяет числовые значения в _numbersArray_ в одно числовое значение результата.

-   **numbersArray** (массив): массив с числовыми входными значениями
-   возвращает **результат** (число|null): результат объединения.

Функция ожидает массив с числовыми значениями в качестве входных данных. Значения в массиве должны быть числами, которые не должны быть отрицательными. Максимальное поддерживаемое значение входного числа 2<sup>32</sup> - 1. Значения входного числа вне допустимого диапазона заставят функцию вернуть `null` и выдать предупреждение. Любые значения `null` во входном массиве игнорируются.

---

`BIT_AND(value1, value2) → result`

Если два числа переданы как отдельные параметры функции `BIT_AND()`, она вернет побитовое значение и значение двух своих операндов. В качестве входных значений допускаются только числа в диапазоне от 0 до 2<sup>32</sup> - 1.

-   **значение1** (число): первый операнд
-   **значение2** (число): второй операнд
-   возвращает **результат** (число|null): и-комбинированный результат

<!-- конец списка -->

<!-- 0001.part.md -->

```aql
BIT_AND([1, 4, 8, 16]) // 0
BIT_AND([3, 7, 63]) // 3
BIT_AND([255, 127, null, 63]) // 63
BIT_AND(127, 255) // 127
BIT_AND("foo") // null
```

<!-- 0002.part.md -->

## `BIT_CONSTRUCT()`

`BIT_CONSTRUCT(positionsArray) → result`

Конструирует числовое значение, биты которого установлены в позициях, заданных в массиве.

-   **positionArray** (массив): массив с позициями битов для установки (с нулевым базисом)
-   возвращает **результат** (number|null): сгенерированное число.

В качестве входных данных функция ожидает массив с числовыми значениями. Значения в массиве должны быть числами, которые не должны быть отрицательными. Максимальное поддерживаемое значение входного числа - 31. При вводе значений чисел вне допустимого диапазона функция вернет `null` и выдаст предупреждение.

<!-- 0003.part.md -->

```aql
BIT_CONSTRUCT([1, 2, 3]) // 14
BIT_CONSTRUCT([0, 4, 8]) // 273
BIT_CONSTRUCT([0, 1, 10, 31]) // 2147484675
```

<!-- 0004.part.md -->

## `BIT_DECONSTRUCT()`

`BIT_DECONSTRUCT(number) → positionsArray`

Деконструирует числовое значение в массив с позициями его установленных битов.

-   **number** (число): входное значение для деконструкции
-   возвращает **positionArray** (array|null): массив с установленными позициями битов (на основе нулей).

Функция превращает числовое значение в массив с позициями всех установленных битов. Позиции в выходном массиве базируются на нулях. Входное значение должно быть числом от 0 до 2<sup>32</sup> - 1 (включительно). Для любых других входных данных функция вернет `null` и выдаст предупреждение.

<!-- 0005.part.md -->

```aql
BIT_DECONSTRUCT(14) // [1, 2, 3]
BIT_DECONSTRUCT(273) // [0, 4, 8]
BIT_DECONSTRUCT(2147484675) // [0, 1, 10, 31]
```

<!-- 0006.part.md -->

## `BIT_FROM_STRING()`

`BIT_FROM_STRING(bitstring) → число`

Преобразует битовую строку (состоящую из цифр `0` и `1`) в число.

Чтобы преобразовать число в битовую строку, смотрите [`BIT_TO_STRING()`](#bit_to_string).

-   **bitstring** (строка): последовательность строк, состоящая из символов `0` и `1`.
-   возвращает **number** (number|null): разобранное число.

Входное значение должно быть битовой строкой, состоящей только из символов `0` и `1`. Битовая строка может содержать до 32 значащих бит, включая все старшие нули. Обратите внимание, что битовая строка не должна начинаться с `0b`. Если битовая строка имеет неправильный формат, эта функция возвращает `null` и выдает предупреждение.

<!-- 0007.part.md -->

```aql
BIT_FROM_STRING("0111") // 7
BIT_FROM_STRING("000000000000010") // 2
BIT_FROM_STRING("11010111011101") // 13789
BIT_FROM_STRING("100000000000000000000") // 1048756
```

<!-- 0008.part.md -->

## `BIT_NEGATE()`

`BIT_NEGATE(число, биты) → результат`

Побитно отрицает биты в **number**, и сохраняет до **bits** битов в результате.

-   **number** (число): число для отрицания
-   **bits** (число): количество битов, которые нужно сохранить в результате (от 0 до 32)
-   возвращает **result** (number|null): результирующее число, содержащее до **bits** значащих битов.

Входное значение должно быть числом от 0 до 2<sup>32</sup> - 1 (включительно). Количество битов должно быть от 0 до 32. Для любых других входных данных функция вернет `null` и выдаст предупреждение.

<!-- 0009.part.md -->

```aql
BIT_NEGATE(0, 8) // 255
BIT_NEGATE(0, 10) // 1023
BIT_NEGATE(3, 4) // 12
BIT_NEGATE(446359921, 32) // 3848607374
```

<!-- 0010.part.md -->

## `BIT_OR()`

`BIT_OR(numbersArray) → result`

Or- объединяет числовые значения в _numbersArray_ в одно числовое значение результата.

-   **numbersArray** (массив): массив с числовыми входными значениями
-   возвращает **результат** (число|null): результат объединения в числовой результат.

Функция ожидает массив с числовыми значениями в качестве входных данных. Значения в массиве должны быть числами, которые не должны быть отрицательными. Максимальное поддерживаемое значение входного числа 2<sup>32</sup> - 1. Значения входного числа вне допустимого диапазона заставят функцию вернуть `null` и выдать предупреждение. Любые значения `null` во входном массиве игнорируются.

---

`BIT_OR(value1, value2) → result`

Если два числа переданы в качестве отдельных параметров функции `BIT_OR()`, она вернет побитовое или значение двух своих операндов. В качестве входных значений допускаются только числа в диапазоне от 0 до 2<sup>32</sup> - 1.

-   **значение1** (число): первый операнд
-   **значение2** (число): второй операнд
-   возвращает **результат** (число|null): или-комбинированный результат

<!-- конец списка -->

<!-- 0011.part.md -->

```aql
BIT_OR([1, 4, 8, 16]) // 29
BIT_OR([3, 7, 63]) // 63
BIT_OR([255, 127, null, 63]) // 255
BIT_OR(255, 127) // 255
BIT_OR("foo") // null
```

<!-- 0012.part.md -->

## `BIT_POPCOUNT()`

`BIT_POPCOUNT(number) → result`

Подсчитывает количество битов, установленных во входном значении.

-   **number** (число): массив с числовыми входными значениями
-   возвращает **result** (number|null): количество битов, установленных во входном значении.

Входное значение должно быть числом от 0 до 2<sup>32</sup> - 1 (включительно). Для любых других входных данных функция вернет `null` и выдаст предупреждение.

<!-- 0013.part.md -->

```aql
BIT_POPCOUNT(0) // 0
BIT_POPCOUNT(255) // 8
BIT_POPCOUNT(69399252) // 12
BIT_POPCOUNT("foo") // null
```

<!-- 0014.part.md -->

## `BIT_SHIFT_LEFT()`

`BIT_SHIFT_LEFT(число, сдвиг, биты) → результат`.

Побитовый сдвиг битов в **числе** влево и сохранение до **битов** битов в результате. Если биты переполняются из-за сдвига, они отбрасываются.

-   **number** (число): число для сдвига
-   **shift** (число): количество битов для сдвига (от 0 до 32)
-   **bits** (число): количество битов, которые нужно оставить в результате (от 0 до 32)
-   возвращает **result** (number|null): результирующее число, со значащими битами до **bits**.

Входное значение должно быть числом от 0 до 2<sup>32</sup> - 1 (включительно). Количество битов должно быть от 0 до 32. Для любых других входных данных функция вернет `null` и выдаст предупреждение.

<!-- 0015.part.md -->

```aql
BIT_SHIFT_LEFT(0, 1, 8) // 0
BIT_SHIFT_LEFT(7, 1, 16) // 14
BIT_SHIFT_LEFT(2, 10, 16) // 2048
BIT_SHIFT_LEFT(878836, 16, 32) // 1760821248
```

<!-- 0016.part.md -->

## `BIT_SHIFT_RIGHT()`

`BIT_SHIFT_RIGHT(число, сдвиг, биты) → результат`

Побитовый сдвиг битов в **числе** вправо и сохранение до **битов** битов в результате. Если биты переполняются из-за сдвига, они отбрасываются.

-   **number** (число): число для сдвига
-   **shift** (число): количество битов для сдвига (от 0 до 32)
-   **bits** (число): количество битов, которые нужно оставить в результате (от 0 до 32)
-   возвращает **result** (number|null): результирующее число, со значащими битами до **bits**.

Входное значение должно быть числом от 0 до 2<sup>32</sup> - 1 (включительно). Количество битов должно быть от 0 до 32. Для любых других входных данных функция вернет `null` и выдаст предупреждение.

<!-- 0017.part.md -->

```aql
BIT_SHIFT_RIGHT(0, 1, 8) // 0
BIT_SHIFT_RIGHT(33, 1, 16) // 16
BIT_SHIFT_RIGHT(65536, 13, 16) // 8
BIT_SHIFT_RIGHT(878836, 4, 32) // 54927
```

<!-- 0018.part.md -->

## `BIT_TEST()`

`BIT_TEST(number, index) → result`

Проверяет, установлено ли в позиции _index_ число **number**.

-   **number** (число): число для проверки
-   **index** (число): индекс проверяемого бита (от 0 до 31)
-   возвращает **результат** (boolean|null): установлен ли бит или нет.

Входное значение должно быть числом от 0 до 2<sup>32</sup> - 1 (включительно). Значение **index** должно быть от 0 до 31. Для любых других входных данных функция вернет `null` и выдаст предупреждение.

<!-- 0019.part.md -->

```aql
BIT_TEST(0, 3) // false
BIT_TEST(255, 0) // true
BIT_TEST(7, 2) // true
BIT_TEST(255, 8) // false
```

<!-- 0020.part.md -->

## `BIT_TO_STRING()`

`BIT_TO_STRING(number) → bitstring`

Преобразует числовое входное значение в битовую строку, состоящую из `0` и `1`.

Чтобы преобразовать битовую строку в число, смотрите [`BIT_FROM_STRING()`](#bit_from_string).

-   **number** (число): число для преобразования в строку
-   возвращает **bitstring** (string|null): битовую строку, сгенерированную из входного значения.

Входное значение должно быть числом от 0 до 2<sup>32</sup> - 1 (включительно). Для любых других входных данных функция вернет `null` и выдаст предупреждение.

<!-- 0021.part.md -->

```aql
BIT_TO_STRING(7, 4) // "0111"
BIT_TO_STRING(255, 8) // "11111111"
BIT_TO_STRING(60, 8) // "00011110"
BIT_TO_STRING(1048576, 32) // "00000000000100000000000000000000"
```

<!-- 0022.part.md -->

## `BIT_XOR()`

`BIT_XOR(numbersArray) → result`

Исключительно-или-объединяет числовые значения в _numbersArray_ в одно числовое значение результата.

-   **numbersArray** (массив): массив с числовыми входными значениями
-   возвращает **результат** (число|null): результат xor-комбинации

Функция ожидает массив с числовыми значениями в качестве входных данных. Значения в массиве должны быть числами, которые не должны быть отрицательными. Максимальное поддерживаемое значение входного числа 2<sup>32</sup> - 1. Значения входного числа вне допустимого диапазона заставят функцию вернуть `null` и выдать предупреждение. Любые значения `null` во входном массиве игнорируются.

---

`BIT_XOR(value1, value2) → result`.

Если два числа переданы в качестве отдельных параметров функции `BIT_XOR()`, она вернет побитовое исключающее или значение двух своих операндов. В качестве входных значений допускаются только числа в диапазоне от 0 до 2<sup>32</sup> - 1.

-   **значение1** (число): первый операнд
-   **значение2** (число): второй операнд
-   возвращает **результат** (число|null): результат xor-комбинации

<!-- конец списка -->

<!-- 0023.part.md -->

```aql
BIT_XOR([1, 4, 8, 16]) // 29
BIT_XOR([3, 7, 63]) // 59
BIT_XOR([255, 127, null, 63]) // 191
BIT_XOR(255, 257) // 510
BIT_XOR("foo") // null
```

<!-- 0024.part.md -->

<!-- 0025.part.md -->
