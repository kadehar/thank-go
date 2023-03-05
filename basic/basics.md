## О курсе

Привет! «Go на практике» — курс для практикующих программистов, которые хотят быстро освоить и начать применять новый
язык. Знание Go и знакомство с «Tour of Go» не требуется. Единственное входное требование — уметь писать код и делать
это с удовольствием.

**Источники**:

* [A Tour of Go](https://github.com/golang/tour/) The Go Authors
* [Go Documentation](https://go.dev/doc/) The Go Authors
* [Go talks](https://go.dev/talks/) The Go Authors
* [Go by Example](https://github.com/mmcgrana/gobyexample) Mark McGranaghan
* [Exercism Go Track](https://github.com/exercism/go) Exercism

## Базовые конструкции

Каждый учебник «для начинающих» считает своим долгом посвятить 200 страниц самой скучной части языка. Мы обойдемся одним
уроком.

### Всем привет

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