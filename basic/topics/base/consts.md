# Константы

Go поддерживает *константы* для символов, строк, логических и числовых значений.

`const` объявляет константу:

```go
const s string = "constant"
fmt.Println(s)
// constant

const n = 500000000
fmt.Println(n)
// 500000000

const ch = 'a'
fmt.Println(ch)
// 97
// это числовой код латинского символа 'a'
```

Если указать выражение — Go рассчитает и сохранит результат. В выражении можно использовать объявленные ранее константы:

```go
const n = 500000000
const d = 3e20 / n
fmt.Println(d)
// 6e+11
```

Тип числовой константы определяется из контекста. Например, когда есть явное приведение типа:

```go
fmt.Println(int64(d))
// 600000000000
```

Или функция, которая требует значение определенного типа (`math.Sin()` ожидает `float64`):

```go
fmt.Println(math.Sin(n))
// -0.28470407323754404
```

[Задание 2. Расстояние между точками](tasks/task2.md)

[<< Назад](./variables.md) [На главную](content.md) [Вперед >>](./loops.md)