# Определяемые типы

На предыдущем шаге мы создали структурный тип и методы для него. Но новый тип не обязательно создавать на основе структуры — можно использовать любые базовые типы.

Создадим тип «ИНН» на основе строки:

```go
type inn string
```

Тип `inn` (он называется *определяемым типом*, defined type) получил свойства базового типа `string`. Добавим ему новое поведение с помощью метода:

```go
func (id inn) isValid() bool {
    if len(id) != 12 {
        return false
    }
    for _, char := range id {
        if !unicode.IsDigit(char) {
            return false
        }
    }
    return true
}
```

```go
inn1 := inn("111201284667")
fmt.Println("inn", inn1, "is valid:", inn1.isValid())
// inn 111201284667 is valid: true

inn2 := inn("ohmyinn12345")
fmt.Println("inn", inn2, "is valid:", inn2.isValid())
// inn ohmyinn12345 is valid: false
```

Это чем-то похоже на наследование, но механизм более примитивный. Если создать новый определяемый тип на основе `inn` — он унаследует структуру и свойства `inn`, но не методы:

```go
type otherid inn

other := otherid("111201284667")
fmt.Println("other inn", other, "is valid:", other.isValid())
// ОШИБКА: other.isValid undefined
```

[Задание 1. Футбольный турнир](tasks/task1.md)

[<< Назад](methods.md) [На главную](content.md) [Вперед >>](composition.md)