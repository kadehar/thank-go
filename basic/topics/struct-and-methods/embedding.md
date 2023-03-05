# Встраивание

Все хорошо, но несколько неудобно было писать `usage.counter.increment()` на предыдущем шаге. По-хорошему, `usage`  *расширяет* `counter` — отношение между ними больше похоже на наследование, чем на композицию. В Go в таких случаях используют *встраивание* (embedding). Посмотрим, как оно работает.

Есть тип «счетчик», такой же, как на предыдущем шаге:

```go
type counter struct {
    value uint
}
func (c *counter) increment() {
    c.value++
}
func (c *counter) incrementDelta(delta uint) {
    c.value += delta
}
```

Мы хотим замерять использование сервисов. *Встроим* счетчик в тип «использование сервиса»:

```go
type usage struct {
    service string
    counter
}

func makeUsage(service string) usage {
    return usage{service, counter{}}
}
```

Благодаря встраиванию, поля и методы счетчика доступны прямо на `usage`, без обращения к полю `counter`:

```go
usage := makeUsage("find")
usage.increment()
usage.increment()
usage.increment()
fmt.Printf("%s usage: %d\n", usage.service, usage.value)
// find usage: 3
```

Аналогично с типом «просмотры страниц»:

```go
type pageviews struct {
    url *url.URL
    counter
}

func makePageviews(uri string) pageviews {
    u, err := url.Parse(uri)
    if err != nil {
        log.Fatal(err)
    }
    return pageviews{u, counter{}}
}
```

Поля и методы счетчика доступны прямо на `pageviews`:

```go
pv := makePageviews("/doc/find")
pv.incrementDelta(100)
fmt.Printf("%s views: %d\n", pv.url, pv.value)
// /doc/find views: 100
```

[Задание 2. Валидатор пароля](tasks/task2.md)

[<< Назад](composition.md) [На главную](content.md)