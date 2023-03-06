# О разделе

В Go есть все необходимое для хороших тестов.

Проверим функцию, которая возвращает минимальное из двух чисел:

```go
func IntMin(a, b int) int {
    if a < b {
        return a
    }
    return b
}
```

Тесты пишут в отдельном файле с суффиксом `_test`. Например, если `IntMin()` определена в файле `ints.go`, то тесты будут в том же пакете, но в отдельном файле `ints_test.go`.

Тест — это функция с префиксом `Test`:

```go
func TestIntMin(t *testing.T) {
    got := IntMin(2, -2)
    want := -2
    if got != want {
        t.Errorf("got %d; want %d", got, want)
    }
}
```

Тест всегда принимает указатель на `testing.T` — это такой сборник полезных функций. Например, `t.Errorf()` выводит сообщение об ошибке, не прекращая выполнение теста.

Выполним тест с подробными результатами (опция `-v`):

```bash
$ go test -v
=== RUN   TestIntMin
--- PASS: TestIntMin (0.00s)
PASS
ok      ints  0.004s
```

[Sandbox](https://go.dev/play/p/-dfEkOHXEhl)

[Задание 1. Тесты на сумму](tasks/task1.md)

[<< Назад](content.md) [Вперед >>](table-driven-tests.md)