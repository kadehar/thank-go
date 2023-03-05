# Всем привет

Наша первая программа печатает в консоль сообщение «всем привет»:

```go
package main

import "fmt"

func main() {
    fmt.Println("hello everyone")
}
```

Чтобы запустить программу, установите Go, сохраните код в `hello.go` и запустите из консоли с помощью `go run`:

```bash
$ go run hello.go
hello everyone
```

Чтобы скомпилировать исходный код в бинарник, воспользуемся `go build`:

```bash 
$ go build hello.go
$ ls
hello	hello.go
```

Запустим скомпилированный бинарник:

```bash 
$ ./hello
hello everyone
```

[<< Назад](base.md) [Вперед >>](values.md)