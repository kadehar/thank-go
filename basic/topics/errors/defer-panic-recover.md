# Defer

*Defer* позволяет отложить выполнение кода до момента завершения функции. Обычно его используют, чтобы освободить ресурсы, выделенные внутри функции (открытые файлы, соединения и тому подобное). В питоне в таких случаях применяют контекстные менеджеры, а в js конструкцию try-finally.

Допустим, мы хотим создать файл, записать в него что-то и закрыть. Вот как поможет `defer`:

```go
func main() {
    f, err := createFile("/tmp/defer.txt")
    if err != nil {
        fmt.Println("Error creating file:", err)
        return
    }

    defer closeFile(f)    // (1)

    if err := writeFile(f); err != nil {
        fmt.Println("Error writing to file:", err)
        return            // (2)
    }

    fmt.Println("Success!")
}
```

После того как файл открыт, мы с помощью `defer` указываем, что необходимо вызвать отложенную функцию `closeFile()`➊. Она выполнится в самом конце, при завершении функции `main()`. Причем отложенная функция отработает в любом случае — даже если во время записи в файл произошла ошибка и сработал досрочный `return`➋.

Допустим, создание файла пройдет успешно, а при записи случится ошибка:

```go
func createFile(name string) (*os.File, error) {
    fmt.Println("Creating file...")
    // ...
}

func writeFile(f *os.File) error {
    fmt.Println("Writing to file...")
    // эмулируем неминуемую ошибку
    return fmt.Errorf("oh no, it all went wrong!")
}

func closeFile(f *os.File) {
    fmt.Println("Closing file...")
    // ...
}
```

`closeFile()` все равно отработает:

```text
Creating file...
Writing to file...
Error writing to file: oh no, it all went wrong!
Closing file...
```

# Panic

Если во время выполнения программы происходит неисправимая ошибка, срабатывает *паника* (panic). Это аналог исключения в питоне или js.

Допустим, мы написали функцию, которая возвращает символ строки по индексу, но забыли проверить, что индекс попадает в границы:

```go
func getChar(str string, idx int) byte {
    return str[idx]
}
```

Если вызвать `getChar()` с некорректным индексом — сработает паника:

```go
c := getChar("hello", 10)
// panic: runtime error: index out of range [10] with length 5
```

Панику можно вызвать и вручную с помощью одноименной встроенной функции:

```go
panic("oops")
```

Так редко делают — в Go принято возвращать ошибку из функции, а не паниковать.

# Recover

Раз есть непредвиденные ошибки (паника), должен быть и способ их поймать. В Go для этого используется встроенная функция `recover()`. Посмотрим, как она работает.

Мы все так же забыли проверить, что индекс попадает в границы:

```go
func getChar(str string, idx int) byte {
    return str[idx]
}
```

Но зная свою забывчивость, решили отловить любые непредвиденные ошибки:

```go
func protect(fn func()) {
    defer func() {
        if err := recover(); err != nil {
            fmt.Println("ERROR:", err)
        } else {
            fmt.Println("Everything went smoothly!")
        }
    }()
    fn()
}
```

`protect()` первым делом объявляет анонимную отложенную функцию, которая сработает после того, как будет выполнена `fn()`. Если срабатывает паника, вызывается отложенная функция. Внутри нее `recover()` возвращает ошибку, которая вызвала панику. Если паники не было, отложенная функция тоже вызывается, но `recover()` внутри возвращает `nil`.

Здесь сработает паника:

```go
protect(func() {
    c := getChar("hello", 10)
    fmt.Println("hello[10] = ", c)
})
// ERROR: runtime error: index out of range [10] with length 5
```

А здесь функция отработает без ошибок:

```go
protect(func() {
    c := getChar("hello", 4)
    fmt.Println("hello[4] =", c)
})
// hello[4] = 111
// Everything went smoothly!
```

Возможно, вы заметили, что ручной вызов `panic()` в сочетании с `defer()` и `recover()` можно использовать, чтобы эмулировать конструкцию `try-catch`. В Go так не принято. Всегда старайтесь явно возвращать ошибки из функции, а на вызывающей стороне проверять их.

[Задание 2. От паники к ошибкам](tasks/task2.md)

[<< Назад](errors.md) [На главную](content.md) [Вперед >>](errors-wrap.md)