# LET

Оператор `LET` может быть использован для присвоения произвольного значения переменной. Затем переменная вводится в область видимости, в которой находится оператор `LET`.

## Синтаксис

<pre><code>LET <em>variableName</em> = <em>expression</em></code></pre>

_expression_ может быть простым выражением или подзапросом.

Для разрешенных имен переменных [Синтаксис AQL](../fundamentals/syntax.md).

## Использование

Переменные в AQL неизменяемы, что означает, что они не могут быть переназначены:

<!-- 0001.part.md -->

```aql
LET a = [1, 2, 3]  // initial assignment

a = PUSH(a, 4)     // syntax error, unexpected identifier
LET a = PUSH(a, 4) // parsing error, variable 'a' is assigned multiple times
LET b = PUSH(a, 4) // allowed, result: [1, 2, 3, 4]
```

<!-- 0002.part.md -->

Операторы `LET` в основном используются для объявления сложных вычислений и для того, чтобы избежать повторных вычислений одного и того же значения в нескольких частях запроса.

<!-- 0003.part.md -->

```aql
FOR u IN users
  LET numRecommendations = LENGTH(u.recommendations)
  RETURN {
    "user" : u,
    "numRecommendations" : numRecommendations,
    "isPowerUser" : numRecommendations >= 10
  }
```

<!-- 0004.part.md -->

В приведенном выше примере вычисление количества рекомендаций разложено на части с помощью оператора `LET`, что позволяет избежать двойного вычисления значения в операторе `RETURN`.

Еще один случай использования `LET` - объявление сложного вычисления в подзапросе, что делает весь запрос более читабельным.

<!-- 0005.part.md -->

```aql
FOR u IN users
  LET friends = (
  FOR f IN friends
    FILTER u.id == f.userId
    RETURN f
  )
  LET memberships = (
  FOR m IN memberships
    FILTER u.id == m.userId
      RETURN m
  )
  RETURN {
    "user" : u,
    "friends" : friends,
    "numFriends" : LENGTH(friends),
    "memberShips" : memberships
  }
```

<!-- 0006.part.md -->

<!-- 0007.part.md -->
