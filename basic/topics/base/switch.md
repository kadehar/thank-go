# switch

`switch` описывает условие с множеством веток.

В отличие от многих других языков, не нужно указывать `break`. Go выполняет только подходящую ветку и не проваливается в следующую:

```go
i := 2
fmt.Print("Write ", i, " as ")
switch i {
case 1:
    fmt.Println("one")
case 2:
    fmt.Println("two")
case 3:
    fmt.Println("three")
}
// Write 2 as two
```

Чтобы провалиться в следующую ветку, есть специальное ключевое слово `fallthrough`. Его можно использовать только в качестве последней инструкции в блоке:

```go
i := 2
fmt.Print("Write ", i, " as ")
switch i {
case 1:
    fmt.Println("one")
case 2:
    fmt.Println("two")
    fallthrough
case 3:
    fmt.Println("Bye-bye")
}
// Write 2 as two
// Bye-bye
```

В одной ветке можно указать несколько выражений. Ветка `default` сработает, если остальные не подошли:

```go
// одной строкой инициализировали day
// и сразу сделали switch по нему
switch day := time.Now().Weekday(); day {
case time.Saturday, time.Sunday:
    fmt.Println(day, "is a weekend")
default:
    fmt.Println(day, "is a weekday")
}
// Tuesday is a weekday
```

Выражения в ветках не обязательно должны быть константами. `switch` может работать как `if`:

```go
t := time.Now()
switch {
case t.Hour() < 12:
    fmt.Println("It's before noon")
default:
    fmt.Println("It's after noon")
}
// It's before noon
```

[Задание 4. Язык по коду](tasks/task4.md)

[<< Назад](./conditions.md) [На главную](../base.md)