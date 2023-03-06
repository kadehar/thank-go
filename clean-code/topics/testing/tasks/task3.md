# Тестируем погоду

Предположим у нас есть сервис `WeatherService`, который предсказывает погоду. Он работает по загадочному и непредсказуемому алгоритму, полному искусственного интеллекта, и возвращает прогноз дневной температуры на завтра, в градусах Цельсия:

```go
// WeatherService предсказывает погоду.
type WeatherService struct{}

// Forecast сообщает ожидаемую дневную температуру на завтра.
func (ws *WeatherService) Forecast() int {
    // магия
    return value
}
```

Мы сделали структуру `Weather`, которая запрашивает `WeatherService` и возвращает прогноз, но не числом, а текстом:

```go
// Weather выдает текстовый прогноз погоды.
type Weather struct {
    service *WeatherService
}

// Forecast сообщает текстовый прогноз погоды на завтра.
func (w Weather) Forecast() string {
    deg := w.service.Forecast()
    switch {
    case deg < 10:
        return "холодно"
    case deg >= 10 && deg < 15:
        return "прохладно"
    case deg >= 15 && deg < 20:
        return "идеально"
    case deg >= 20:
        return "жарко"
    }
    return "инопланетно"
}
```

Неплохо бы теперь проверить `Weather`. Проблема в том, что мы никак не контролируем поведение `WeatherService`, поэтому тесты не проходят:

```go
type testCase struct {
    deg  int
    want string
}

var tests []testCase = []testCase{
    {-10, "холодно"},
    {0, "холодно"},
    {5, "холодно"},
    {10, "прохладно"},
    {15, "идеально"},
    {20, "жарко"},
}

func TestForecast(t *testing.T) {
    service := &WeatherService{}
    weather := Weather{service}
    for _, test := range tests {
        name := fmt.Sprintf("%v", test.deg)
        t.Run(name, func(t *testing.T) {
            got := weather.Forecast()
            if got != test.want {
                t.Errorf("%s: got %s, want %s", name, got, test.want)
            }
        })
    }
}
```

```bash
$ go test
--- FAIL: TestForecast (0.00s)
    --- FAIL: TestForecast/10 (0.00s)
        weather_test.go:60: 10: got холодно, want прохладно
    --- FAIL: TestForecast/15 (0.00s)
        weather_test.go:60: 15: got холодно, want идеально
    --- FAIL: TestForecast/20 (0.00s)
        weather_test.go:60: 20: got холодно, want жарко
FAIL
```

Исправьте `Weather`, чтобы его можно было нормально протестировать. Сделайте заглушку `WeatherService` и используйте ее в `TestForecast`.

Не меняйте `WeatherService`, метод `Weather.Forecast()` и набор тестов `tests`.

В результате `Weather` должен подходить как для обычного использования (с типом `WeatherService`), так и для тестов (с заглушкой). Если вы видите сообщение об ошибке `cannot use stub (variable of type *WeatherServiceStub_) as type...` — значит, пока вам не удалось этого добиться.

```go
package main

import (
    "fmt"
    "math/rand"
    "testing"
    "time"
)

// WeatherService предсказывает погоду.
type WeatherService struct{}

// Forecast сообщает ожидаемую дневную температуру на завтра.
func (ws *WeatherService) Forecast() int {
    rand.Seed(time.Now().Unix())
    value := rand.Intn(31)
    sign := rand.Intn(2)
    if sign == 1 {
        value = -value
    }
    return value
}

// Weather выдает текстовый прогноз погоды.
type Weather struct {
    service *WeatherService
}

// Forecast сообщает текстовый прогноз погоды на завтра.
func (w Weather) Forecast() string {
    deg := w.service.Forecast()
    switch {
    case deg < 10:
        return "холодно"
    case deg >= 10 && deg < 15:
        return "прохладно"
    case deg >= 15 && deg < 20:
        return "идеально"
    case deg >= 20:
        return "жарко"
    }
    return "инопланетно"
}

type testCase struct {
    deg  int
    want string
}

var tests []testCase = []testCase{
    {-10, "холодно"},
    {0, "холодно"},
    {5, "холодно"},
    {10, "прохладно"},
    {15, "идеально"},
    {20, "жарко"},
}

func TestForecast(t *testing.T) {
    service := &WeatherService{}
    weather := Weather{service}
    for _, test := range tests {
        name := fmt.Sprintf("%v", test.deg)
	    t.Run(name, func(t *testing.T) {
		    got := weather.Forecast()
		    if got != test.want {
			    t.Errorf("%s: got %s, want %s", name, got, test.want)
		    }
	    })
    }
}
```

[<< Назад](../stubs-and-external-calls.md)