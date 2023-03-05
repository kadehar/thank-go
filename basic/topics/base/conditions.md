# if/else

Конструкция `if-else` в Go ведет себя без особых сюрпризов.

Вокруг условия не нужны круглые скобки, но фигурные скобки для веток обязательны:

```go
f 7 % 2 == 0 {
    fmt.Println("7 is even")
} else {
    fmt.Println("7 is odd")
}
// 7 is odd
```

Можно использовать `if` без `else`:

```go
if 8 % 4 == 0 {
    fmt.Println("8 is divisible by 4")
}
// 8 is divisible by 4
```

Единственный нюанс: перед условием может идти выражение. Объявленные в нем переменные доступны во всех ветках:

```go
if num := 9; num < 0 {
    fmt.Println(num, "is negative")
} else if num < 10 {
    fmt.Println(num, "has 1 digit")
} else {
    fmt.Println(num, "has multiple digits")
}
// 9 has 1 digit
```

[<< Назад](loops.md) [На главную](base.md) [Вперед >>](switch.md)