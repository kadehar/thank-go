# Композиция

В Go нет наследования. Вместо него активно используется композиция — когда новое поведение собирают из кирпичиков существующего.

Есть тип «счетчик»:

```go 
type counter struct {
    value uint
}
```

Его можно увеличивать на единицу:

```go 
func (c *counter) increment() {
    c.value++
}
```

Или на указанное число:

```go 
func (c *counter) incrementDelta(delta uint) {
    c.value += delta
}
```

Мы хотим замерять использование сервисов. Чтобы не дублировать существующие функции, добавим счетчик в тип «использование сервиса»:

```go 
type usage struct {
    service string
    counter counter
}

func makeUsage(service string) usage {
    return usage{service, counter{}}
}
```

Будем мерить использование сервиса, увеличивая его счетчик:

```go 
usage := makeUsage("find")
usage.counter.increment()
usage.counter.increment()
usage.counter.increment()
fmt.Printf("%s usage: %d\n", usage.service, usage.counter.value)
// find usage: 3
```

Для типа «просмотры страницы» тоже добавим счетчик:

```go 
type pageviews struct {
    url *url.URL
    counter counter
}

func makePageviews(uri string) pageviews {
    u, err := url.Parse(uri)
    if err != nil {
        log.Fatal(err)
    }
    return pageviews{u, counter{}}
}
```

И будем мерить просмотры:

```go 
pv := makePageviews("/doc/find")
pv.counter.incrementDelta(100)
fmt.Printf("%s views: %d\n", pv.url, pv.counter.value)
// /doc/find views: 100
```

[<< Назад](defined-types.md) [На главную](content.md) [Вперед >>](embedding.md)