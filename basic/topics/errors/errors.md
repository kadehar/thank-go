# Ошибки

В Go нет исключений и блока `try-catch`, как в питоне или js. Вместо этого функции явно возвращают ошибку отдельным значением. Благодаря этому ошибки невозможно проигнорировать, а разработчики продумывают поведение программы в случае проблем.

Ошибки принято возвращать последним значением с интерфейсным типом `error`:

```go
func sqrt(x float64) (float64, error) {
    if x < 0 {
        return 0, errors.New("expect x >= 0")
    }
    // `nil` в качестве ошибки указывает, что ошибок не было.
    return math.Sqrt(x), nil
}
```

Проверим работу `sqrt()` на положительном и отрицательном значениях. Обратите внимание, как мы получаем результат и проверяем ошибку внутри условия `if` — это стандартная практика в Go.

```go
for _, x := range []float64{49, -49} {
    if res, err := sqrt(x); err != nil {
        fmt.Printf("sqrt(%v) failed: %v\n", x, err)
    } else {
        fmt.Printf("sqrt(%v) = %v\n", x, res)
    }
}
// sqrt(49) = 7
// sqrt(-49) failed: expect x >= 0
```

# Собственный тип ошибки

Чтобы создать собственный тип ошибки, достаточно реализовать метод `Error()`.

```go
type error interface {
    Error() string
}
```

Создадим ошибку, которая описывает проблему поиска `substr` в строке `src`:

```go
type lookupError struct {
    src    string
    substr string
}

func (e lookupError) Error() string {
    return fmt.Sprintf("'%s' not found in '%s'", e.substr, e.src)
}
```

Напишем функцию `indexOf()`, которая возвращает индекс вхождения подстроки `substr` в строку `src`. Если вхождения нет, возвращает ошибку типа `lookupError`:

```go
func indexOf(src string, substr string) (int, error) {
    idx := strings.Index(src, substr)
    if idx == -1 {
        // Создаем и возвращаем ошибку типа `lookupError`.
        return -1, lookupError{src, substr}
    }
    return idx, nil
}
```

Проверим работу `indexOf()` для разных подстрок.

```go
src := "go is awesome"
for _, substr := range []string{"go", "js"} {
    if res, err := indexOf(src, substr); err != nil {
        fmt.Printf("indexOf(%#v, %#v) failed: %v\n", src, substr, err)
    } else {
        fmt.Printf("indexOf(%#v, %#v) = %v\n", src, substr, res)
    }
}
// indexOf("go is awesome", "go") = 0
// indexOf("go is awesome", "js") failed: 'js' not found in 'go is awesome'
```

Поскольку `indexOf()` возвращает общий тип `error`, чтобы получить доступ к конкретному объекту ошибки, придется использовать приведение типа:

```go
_, err := indexOf(src, "js")
if err, ok := err.(lookupError); ok {
    fmt.Println("err.src:", err.src)
    fmt.Println("err.substr:", err.substr)
}
// err.src: go is awesome
// err.substr: js
```

[Задание 1. Счет в банке](tasks/task1.md)

[<< Назад](content.md) [Вперед >>](defer-panic-recover.md)