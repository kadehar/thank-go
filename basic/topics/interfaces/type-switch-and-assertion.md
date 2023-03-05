# Приведение типа

*Приведение типа* (type assertion) извлекает конкретное значение из переменной интерфейсного типа:

```go
var value any = "hello"
str := value.(string)
fmt.Println(str)
// hello
```

Если тип конкретного значения отличается от указанного, произойдет ошибка:

```go
flo := value.(float64)
// ошибка
```

Чтобы проверить тип конкретного значения, используют опциональный флаг, который сигнализирует — правильный тип или нет:

```go
str, ok := value.(string)
fmt.Println(str, ok)
// hello true

flo, ok := value.(float64)
fmt.Println(flo, ok)
// 0 false
```

# Переключатель типа

Приведение типа можно использовать вместе со `switch`. Такая конструкция называется *переключателем типа* (type switch):

```go
var value any = "hello"

switch v := value.(type) {
case string:
    fmt.Printf("%#v is a string\n", v)
case float64:
    fmt.Printf("%#v is a float\n", v)
default:
    fmt.Printf("%#v is a mystery\n", v)
}
// "hello" is a string
```

[Задание 1. Универсальный итератор](tasks/task1.md)

[Задание 2. Максимальный элемент последовательности](tasks/task2.md)

[<< Назад](interfaces.md) [На главную](content.md)