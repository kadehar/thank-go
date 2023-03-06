# Табличные тесты

Чтобы проверить граничные условия и различные сочетания параметров, нам пришлось бы написать целую кучу тестовых функций. Обычно вместо этого используют *табличные* (table-driven) тесты:

* отдельно описывают входные данные и ожидаемые значения;
* последовательно выполняют каждую проверку с помощью `t.Run()`.

```go
func TestIntMin(t *testing.T) {
    var tests = []struct {
        a, b int
        want int
    }{
        {0, 1, 0},
        {1, 0, 0},
        {1, 1, 1},
    }

    for _, test := range tests {
        name := fmt.Sprintf("case(%d,%d)", test.a, test.b)
        t.Run(name, func(t *testing.T) {
            got := IntMin(test.a, test.b)
            if got != test.want {
                t.Errorf("got %d, want %d", got, test.want)
            }
        })
    }
}
```

Срез `tests` — наша таблица с отдельными проверками. У каждой проверки есть входные параметры (`a`, `b`) и ожидаемый результат (`want`). `t.Run()` запускает конкретную проверку и выводит ошибку, если она не прошла.

```bash
$ go test -v
=== RUN   TestIntMin
=== RUN   TestIntMin/case(0,1)
=== RUN   TestIntMin/case(1,0)
=== RUN   TestIntMin/case(1,1)
--- PASS: TestIntMin (0.00s)
    --- PASS: TestIntMin/case(0,1) (0.00s)
    --- PASS: TestIntMin/case(1,0) (0.00s)
    --- PASS: TestIntMin/case(1,1) (0.00s)
PASS
ok      ints  0.005s
```

[Sandbox](https://go.dev/play/p/Os42_Au3c8c)

[Задание 2. Табличные тесты на сумму](tasks/task2.md)

[<< Назад](about.md) [На главную](content.md) [Вперед >>](stubs-and-external-calls.md)