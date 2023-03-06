# Внешние зависимости

Допустим, мы решили отказаться от проверки по шаблону и сравнивать строки по похожести. Воспользуемся для этого сторонним модулем [github.com/sahilm/fuzzy](https://github.com/sahilm/fuzzy).

Скачаем пакет:

```bash
$ go get github.com/sahilm/fuzzy
go: downloading github.com/sahilm/fuzzy v0.1.0
go get: added github.com/sahilm/fuzzy v0.1.0
```

>*`go get` помещает скачанный модуль в домашний каталог Go (`~/go`). В каталог с исходниками нашего модуля он не попадает. Таким образом, все внешние зависимости хранятся в общей куче, а не в конкретных проектах (в противоположность `venv` в питоне и `node_modules` в js)*.

Подключим и используем в `main.go`:

```go
import (
    // ...
    "github.com/sahilm/fuzzy"
)

func main() {
    // ...
    matches := fuzzy.Find(pattern, []string{src})
    isMatch := len(matches) > 0
    // ...
}
```

[Полный исходный код](https://github.com/gothanks/match/tree/a1e73bb8c9779bcb57dad8b92277e9ccee2c470c)

Соберем и проверим:

```bash
$ go build
$ go mod tidy

$ ./match -p awsome 'go is awesome'
go is awesome

$ ./match -p php 'go is awesome'
(пусто)
```

> *Go компилирует зависимости вместе с основными исходниками в единый бинарный файл. Никаких внешних бинарных зависимостей, все включено в общий бинарник `match`. Поэтому модули в Go распространяются в виде исходников на гитхабе. Не существует общего репозитория с собранными артефактами (как `PyPI` в питоне или `npm` в js).*

Зависимость зафиксирована в `go.mod`:

```text
module github.com/gothanks/match

go 1.18

require github.com/sahilm/fuzzy v0.1.0

require github.com/kylelemons/godebug v1.1.0 // indirect
```

Модуль `github.com/kylelemons/godebug` — транзитивная зависимость (от него зависит `github.com/sahilm/fuzzy`), поэтому записан как *indirect*.

# Установка зависимостей из импортов

Если вы скачали готовый проект с кучей зависимостей, не обязательно устанавливать их по одной. Достаточно запустить `go get` без параметров. Он проанализирует зависимости, перечисленные в `import`, скачает их и актуализирует `go.mod`:

```bash
$ go get 
go: added github.com/sahilm/fuzzy v0.1.0
```

`go build` тоже автоматически проверяет зависимости, перечисленные в `import`, но не скачивает их. Он сообщит, если чего-то не хватает. Например, если удалить строчки `require` из `go.mod` и повторить сборку, будет ошибка:

```bash
$ go build
main.go:6:2: no required module provides package github.com/sahilm/fuzzy; to add it:
	go get github.com/sahilm/fuzzy
```

# Обновление зависимостей

Вот несколько полезных команд.

Показать основной модуль и его зависимости:

```bash
$ go list -m all
github.com/gothanks/match
github.com/kylelemons/godebug v1.1.0
github.com/sahilm/fuzzy v0.1.0
```

Посмотреть все версии конкретного модуля:

```bash
$ go list -m -versions github.com/sahilm/fuzzy
github.com/sahilm/fuzzy v0.0.1 v0.0.2 v0.0.3 v0.0.4 v0.0.5 v0.1.0
```

Обновить конкретный модуль на последнюю patch-версию в пределах действующей minor-версии (например, `1.1.0 → 1.1.5`):

```bash
$ go get -u=patch github.com/sahilm/fuzzy
```

Обновить конкретный модуль на последнюю minor-версию в пределах действующей major-версии (`1.1.0 → 1.2.1`):

```bash
$ go get -u github.com/sahilm/fuzzy
```

> *Go предполагает, что авторы модулей следуют правилам семантического версионирования, при котором `minor-` и `patch-`версии остаются обратно совместимыми — поэтому `go get -u` обновляет на последнюю выпущенную версию.
> Если меняется `major-`версия (например, `1.2.1 → 2.0.0`), у модуля меняется идентификатор (`github.com/sahilm/fuzzy → github.com/sahilm/fuzzy/v2`). `go get -u` автоматически на такую версию не обновит. Это сделано специально, потому что новая `major-`версия может быть несовместима с предыдущей.*

Удалить неактуальные зависимости из `go.mod`:

```bash
$ go mod tidy
```

[<< Назад](packages.md) [На главную](content.md)