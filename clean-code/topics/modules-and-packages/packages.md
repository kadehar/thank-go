# Пакет из нескольких файлов

В первой версии логика функции `match()` довольно примитивная:

```go
func match(pattern string, src string) bool {
    return strings.Contains(src, pattern)
}
```

Доработаем утилиту. Чтобы не перегружать файл `main.go`, вынесем логику сравнения в отдельный файл `match.go` в том же пакете:

```text
match
├── go.mod
├── main.go
└── match.go
```

```go
// -- match/match.go --
package main

import "strings"

// match returns true if src matches pattern,
// false otherwise.
func match(pattern string, src string) bool {
    return strings.Contains(src, pattern)
}
```

```go
// -- match/main.go --
// ...

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
```

В `main.go` ничего не изменилось — поскольку фукция `match()` объявлена в том же пакете, ее можно вызывать напрямую.

Теперь разрешим использовать в шаблоне специальные символы `?` и `*` по аналогии с командной строкой:

```go
// -- match/match.go --
// ...

// match returns true if src matches pattern,
// false otherwise.
//
// Supports wildcards:
//  ?  matches any single character
//  *  matches everything
func match(pattern string, src string) (bool, error) {
    pat := translate(pattern)
    re, err := regexp.Compile(pat)
    if err != nil {
        return false, err
    }
    isMatch := re.MatchString(src)
    return isMatch, nil
}
```

```go
// -- main.go --
// ...

func main() {
    // ...
    isMatch, err := match(pattern, src)
    if err != nil {
        fail(err)
    }
    // ...
}
```

[Полный исходный код](https://github.com/gothanks/match/tree/22f315c4be45da419f0625dafc723ebadeb325e1)

Проверим:

```bash
$ go build
$ ./match -p 'a??some' 'go is awesome'
go is awesome
```

# Пакеты и области видимости

Допустим, логика сопоставления строки с шаблоном усложнилась настолько, что плохо помещается в один файл. Вынесем ее в отдельный пакет `glob`:

```text
match
├── glob
│   └── glob.go
├── go.mod
└── main.go
```

```go
// -- match/glob/glob.go --
/*
glob package matches strings against patterns.
Supports wildcards:
  ?  matches any single character
  *  matches everything
*/
package glob

// ...

// match returns true if src matches pattern,
// false otherwise.
func match(pattern string, src string) (bool, error) {
    // ...
}

// ...
```

Теперь программа откажется компилироваться:

```bash
$ go build
# github.com/gothanks/match
./main.go:19:18: undefined: match
```

Функция `match()` переехала из пакета `main` в пакет `glob`, поэтому из `main` больше не доступна. В Go действует правило:

* Если функция, тип, переменная, константа или поле структуры называется со строчной буквы (`match`) — она видна только в пределах родного пакета (*не экспортирована* в терминах Go).
* Если называется с прописной буквы (`Match`) — видна и в других пакетах (*экспортирована*).

Экспортируем `match()` и импортируем пакет `glob` в `main.go`:

```go
// -- match/glob/glob.go --
// ...

// Match returns true if src matches pattern,
// false otherwise.
func Match(pattern string, src string) (bool, error) {
    // ...
}

// ...
```

```go
// -- match/main.go --
package main

import (
    "errors"
    "flag"
    "fmt"
    "os"
    "strings"

    "github.com/gothanks/match/glob"
)

func main() {
    // ...
    isMatch, err := glob.Match(pattern, src)
    // ...
}
```

[Полный исходный код](https://github.com/gothanks/match/tree/a08c300589c6a69b7e1ded3c28447fdd2cdb2403)

Пакеты импортируются по полному имени с идентификатором модуля (`github.com/gothanks/match/glob`), а в коде доступны по локальному имени (`glob`).

В Go не принято «мельчить» с пакетами. В нашей ситуации создание пакета `glob` не было оправдано, я сделал его только в учебных целях.

[<< Назад](new-module.md) [На главную](content.md) [Вперед >>](dependencies.md)