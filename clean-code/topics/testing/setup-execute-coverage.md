# Подготовка и завершение

Бывает, хочется выполнить какой-то код *до старта тестов* (setup), а какой-то — *после их заверешения* (teardown). Например, перед тестами подготовить данные, а после — почистить их.

В других языках для этого часто используют отдельные функции. В Go же обходятся одной — `TestMain()`:

```go
func TestMain(m *testing.M) {
    fmt.Println("Setup tests...")
    start := time.Now()

    m.Run()

    fmt.Println("Teardown tests...")
    end := time.Now()
    fmt.Println("Tests took", end.Sub(start))
}
```

```bash
$ go test -v
Setup tests...
=== RUN   TestIntMin
--- PASS: TestIntMin (0.00s)
PASS
Teardown tests...
Tests took 137.999µs
ok      ints  0.005s
```

`m.Run()` запускает тесты (функции вида `Test*()`). Получается, что любой код до него — это `setup`, а после — `teardown`.

[Sandbox](https://go.dev/play/p/CnejrOMVqt1)

# Избирательный запуск

Иногда хочется запустить не все тесты, а только часть. Например, если некоторые тесты медленные, и каждый раз гонять их не хочется.

Наша старая знакомая — функция, которая суммирует целые числа:

```go
func Sum(nums ...int) int {
    total := 0
    for _, num := range nums {
        total += num
    }
    return total
}
```

И пара тестов:

```go
func TestSumFew(t *testing.T) {
    if Sum(1, 2, 3, 4, 5) != 15 {
        t.Errorf("Expected Sum(1, 2, 3, 4, 5) == 15")
    }
}

func TestSumN(t *testing.T) {
    n := 1_000_000_000
    nums := make([]int, n)
    for i := 0; i < n; i++ {
        nums[i] = i + 1
    }
    got := Sum(nums...)
    want := n * (n + 1) / 2
    if got != want {
        t.Errorf("Expected sum[i=1..n](i) == n*(n+1)/2")
    }
}
```

Прогоним тесты:

```bash
$ go test -v
=== RUN   TestSum
--- PASS: TestSum (0.00s)
=== RUN   TestSumN
--- PASS: TestSumN (3.78s)
```

Ни малейшего желания каждый раз ждать 4 секунды, пока выполняется `TestSumN` на миллиарде слагаемых. Воспользуемся так называемым `short`-режимом, который делит все тесты на «короткие» и «длинные». `TestSumFew` будет коротким тестом, а `TestSumN` — длинным:

```go
func TestSumN(t *testing.T) {
    if testing.Short() {
        t.Skip("skipping test in short mode.")
    }
    // сам тест
}
```

Теперь тесты в коротком режиме будут игнорировать `TestSumN` и отрабатывать моментально:

```bash
$ go test -v -short
=== RUN   TestSumFew
--- PASS: TestSumFew (0.00s)
=== RUN   TestSumN
    selective_test.go:21: skipping test in short mode.
--- SKIP: TestSumN (0.00s)
```

[Sandbox](https://go.dev/play/p/FzWJvGubMve)

Альтернативный вариант — указать маску названия теста. Так выполнится только `TestSumFew`:

```bash
$ go test -v -run Few
```

А так — только `TestSumN`:

```bash
$ go test -v -run N
```

# Тестовое покрытие

*Покрытие* (coverage) показывает, насколько полно тесты проверяют основной код. Чтобы его замерить, достаточно запустить `go test` с опцией `-cover`.

Вот наш модуль:

```text
ints/
├── go.mod
├── ints.go
└── ints_test.go
```

Основной код:

```go
// -- ints/ints.go --
package ints

func IntMin(a, b int) int {
    if a < b {
        return a
    }
    return b
}
```

Тесты:

```go
// -- ints/ints_test.go --
package ints

import (
    "testing"
)

func TestIntMin(t *testing.T) {
    got := IntMin(2, -2)
    want := -2
    if got != want {
        t.Errorf("got %d; want %d", got, want)
    }
}
```

Запускаем тесты:

```bash
$ go test -cover
PASS
coverage: 66.7% of statements
ok      ints    0.004s
```

Только 66%! Чтобы не гадать, какой код не покрыт, попросим Go собрать детальную *статистику* (profile):

```bash
$ go test -coverprofile=cover.prof
PASS
coverage: 66.7% of statements
ok      ints    0.004s
```

И посмотрим отчет в HTML:

```bash
go tool cover -html=cover.prof
```

[<< Назад](stubs-and-external-calls.md) [На главную](content.md)