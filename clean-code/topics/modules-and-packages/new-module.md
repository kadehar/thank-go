# Новый модуль

Модуль инициализируется командой `go mod init`:

```bash
$ mkdir match
$ cd match
$ go mod init github.com/gothanks/match
go: creating new go.mod: module github.com/gothanks/match
$ touch main.go
```

`github.com/gothanks/match` — это глобально уникальный идентификатор модуля и одновременно URL, по которому он должен быть доступен (за минусом префикса `https://`).

> *Если будете повторять шаги урока локально, репозиторий на гитхабе можно не создавать. Пока никто не использует ваш модуль, размещать его публично не требуется.*

Наш модуль:

```text
match/
├── go.mod
└── main.go
```

`go.mod` описывает модуль: идентификатор, минимальная версия Go, зависимости (их пока нет):

```bash
$ cat go.mod 
module github.com/gothanks/match

go 1.18
```

Наш модуль содержит пакет с единственным файлом `main.go`. Реализуем в нем логику программы:

```go
// match tool checks a string against a pattern.
// If it matches - prints the string, otherwise prints nothing.
package main

import (
    "errors"
    "flag"
    "fmt"
    "os"
    "strings"
)

func main() {
    pattern, src, err := readInput()
    if err != nil {
        fail(err)
    }
    isMatch := match(pattern, src)
    if !isMatch {
        os.Exit(0)
    }
    fmt.Println(src)
}

// match returns true if src matches pattern,
// false otherwise.
func match(pattern string, src string) bool {
    return strings.Contains(src, pattern)
}

// readInput reads pattern and source string
// from command line arguments and returns them.
func readInput() (pattern, src string, err error) {
    // ...
}

// fail prints the error and exits.
func fail(err error) {
    // ...
}
```

[Полный исходный код](https://github.com/gothanks/match/tree/bf94b3f8c2828eb2e076a5c1de36de8a7dbc9968)

Если программа компилируется в исполняемый файл, у нее должен быть пакет с названием `main`. Если бы мы делали библиотеку — назвали бы файл `match.go`, а пакет `match`.

# Сборка и установка

`go build` собирает модуль в исполняемый файл:

```bash
$ go build
$ ./match -p is 'go is awesome'
go is awesome
```

`go install` устанавливает модуль в домашний каталог Go (`$HOME/go` для Linux/macOS, `%USERPROFILE%\go` для Windows):

```bash
$ go install
$ ~/go/bin/match -p is 'go is awesome'
go is awesome
```

Go компилирует программу в единый исполняемый файл. Чтобы ее запустить, не требуется устанавливать сам Go или зависимости. Копируете бинарник на целевую машину и запускаете, это все.

Бинарник не кросс-платформенный: чтобы запустить Go-программу на конкретной целевой архитектуре, придется собрать специально под нее. Не будем разбирать это на курсе, но если интересно — вот статья с подробностями: [Building Go Applications for Different Operating Systems and Architectures.](https://www.digitalocean.com/community/tutorials/building-go-applications-for-different-operating-systems-and-architectures)

[<< Назад](about.md) [На главную](content.md) [Вперед >>](packages.md)