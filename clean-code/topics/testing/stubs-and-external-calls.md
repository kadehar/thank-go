# Тесты и внешние вызовы

Предположим, у нас есть модуль, который проверяет доступность произвольного URL. Возвращает `true`, если адрес доступен, и `false` в противном случае:

```text
ping/
├── go.mod
├── ping.go
└── ping_test.go
```

Основной код:

```go
// -- ping/ping.go --
// Пакет ping проверяет доступность URL.
package ping

import "net/http"

// Pinger проверяет доступность URL.
type Pinger struct {
    client *http.Client
}

// Ping запрашивает указанный URL.
// Возвращает true, если адрес доступен, и false в противном случае.
func (p Pinger) Ping(url string) bool {
    resp, err := p.client.Head(url)
    if err != nil {
        return false
    }
    if resp.StatusCode != 200 {
        return false
    }
    return true
}
```

Тесты:

```go
// -- ping/ping_test.go --
package ping

import (
    "net/http"
    "testing"
)

func TestPing(t *testing.T) {
    client := &http.Client{}
    pinger := Pinger{client}
    got := pinger.Ping("https://example.com")
    if !got {
        t.Errorf("Expected example.com to be available")
    }
    got = pinger.Ping("https://example.com/404")
    if got {
        t.Errorf("Expected example.com/404 to be unavailable")
    }
}
```

```bash
$ go test
PASS
ok      ping    0.728s
```

Тесты проходят, но занимают чуть ли не секунду, потому что делают реальные http-запросы. Это совсем не здорово, попробуем исправить.

# Заглушки

В питоне или js мы могли бы сделать *заглушку* (stub) http-клиента с единственным методом `.Head()` и подсунуть ее в `Pinger` вместо настоящего клиента. В Go так сделать не получится: он статически типизирован и ожидает переменную типа `http.Client`:

```go
type Pinger struct {
    client *http.Client
}
```

Заменим эту жесткую зависимость. Введем интерфейс с единственным нужным нам методом — `.Head()`, и будем использовать его в `Pinger` вместо `http.Client`:

```go
// HTTPClient - это усеченный http-клиент,
// который умеет делать HEAD-запросы.
type HTTPClient interface {
    Head(url string) (resp *http.Response, err error)
}

// Pinger проверяет доступность URL.
type Pinger struct {
    client HTTPClient
}
```

В основном коде можем спокойно использовать настоящий `http.Client`, потому что он реализует интерфейс `HTTPClient`:

```go
func main() {
    client := &http.Client{}
    pinger := Pinger{client}
    url := "https://ya.ru"
    alive := pinger.Ping(url)
    fmt.Println(url, "is alive =", alive)
}
```

```bash
$ go run main.go 
https://ya.ru is alive = true
```

А в тестах создадим заглушку, которая вместо обращения к сайту по http сразу возвращает ответ:

```go
// MockClient - это заглушка http-клиента для тестов.
type MockClient struct{}

// Head возвращает http-ответ со статусом, указанным в url.
// Например:
// url = https://ya.ru/200 -> статус = 200
// url = https://ya.ru/404 -> статус = 404
func (client *MockClient) Head(url string) (resp *http.Response, err error) {
    parts := strings.Split(url, "/")
    last := parts[len(parts)-1]
    statusCode, err := strconv.Atoi(last)
    if err != nil {
        return nil, err
    }
    resp = &http.Response{StatusCode: statusCode}
    return resp, nil
}
```

Используем заглушку в тестах:

```go
func TestPing(t *testing.T) {
    client := &MockClient{}
    pinger := Pinger{client}
    got := pinger.Ping("https://example.com/200")
    if !got {
        t.Errorf("Expected example.com/200 to be available")
    }
    got = pinger.Ping("https://example.com/404")
    if got {
        t.Errorf("Expected example.com/404 to be unavailable")
    }
}
```

[Sandbox](https://go.dev/play/p/03WeWfcCFCB)

Запускаем тесты:

```bash
$ go test
PASS
ok      ping    0.012s
```

12 миллисекунд вместо 728. Другое дело!

Вообще, для http-заглушек в Go есть отдельный пакет [net/http/httptest](https://pkg.go.dev/net/http/httptest). Здесь я его не использовал, чтобы продемонстрировать универсальный принцип создания заглушек для внешних или «тяжелых» зависимостей:

1. Заменяем конкретную зависимость на зависимость от интерфейса.
2. Реализуем интерфейс в тестах и используем его.

[Задание 3. Тестируем погоду](tasks/task3.md)

[<< Назад](table-driven-tests.md) [На главную](content.md) [Вперед >>](setup-execute-coverage.md)