# Обертывание ошибок

Допустим, есть функция, которая извлекает значение по ключу из карты и возвращает ошибку, если ключ не найден:

```go
var errNotFound error = errors.New("not found")

func getValue(m map[string]string, key string) (string, error) {
    val, ok := m[key]
    if !ok {
        return "", errNotFound
    }
    return val, nil
}
```

И есть структура `languages` с информацией о языках. Она возвращает описание языка по названию:

```go
type languages map[string]string

func (l languages) describe(lang string) (string, error) {
    descr, err := getValue(l, lang)
    if err != nil {
        return "", err
    }
    return descr, nil
}
```

```go
var langs languages = languages{
    "go":     "is awesome",
    "python": "is everywhere",
    "php":    "just is",
}

func main() {
    descr, err := langs.describe("java")
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Println(descr)
}
```

Внутри у `languages` карта, а метод `languages.describe()` использует `getValue()`, чтобы получить информацию о языке по названию. Получив ошибку, метод транслирует ее клиенту. В примере для `"java"` напечатается такой результат:

```text
not found
```

Формально все верно. Но `errNotFound` — низкоуровневая ошибка общего назначения. Она ничего не говорит о проблеме с поиском языка. Клиент хотел бы больше информации.

Можно создать новую ошибку в `describe()` с помощью `fmt.Errorf()`:

```go
func (l languages) describe(lang string) (string, error) {
    descr, err := getValue(l, lang)
    if err != nil {
        return "", fmt.Errorf("error describing %s: unknown language", lang)
    }
    return descr, nil
}
```

```text
error describing java: unknown language
```

Но создав новую ошибку, мы потеряли информацию о первоначальной `errNotFound`. Вдруг клиенту она важна?

Можно *обернуть* (wrap) исходную ошибку в новую с помощью `fmt.Errorf()` и спецификатора `%w`:

```go
func (l languages) describe(lang string) (string, error) {
    descr, err := getValue(l, lang)
    if err != nil {
        return "", fmt.Errorf("error describing %s: %w", lang, err)
    }
    return descr, nil
}
```

Теперь метод возвращает ошибку-матрешку: снаружи у нее информативная `error describing...`, а внутри исходная `errNotFound`. В сложных программах таких «обертываний» может быть много, пока ошибка поднимается от самых нижних слоев кода к уровню API или UI.

# Обертывание собственных ошибок

Если вместо `fmt.Errorf()` мы захотим использовать собственный тип ошибки — дело усложнится. Допустим, хотим записывать в ошибку название языка отдельным полем:

```go
type languageErr struct {
    lang string
}

func (le languageErr) Error() string {
    return fmt.Sprintf("%s language error", le.lang)
}
```

Чтобы `languageErr` могла выступать оберткой для других ошибок, придется сделать еще две вещи:

1. Добавить отдельное поле для внутренней ошибки (ее будем оборачивать).
2. Добавить метод `Unwrap()`, который возвращает внутреннюю ошибку («разворачивает»).

```go
type languageErr struct {
    lang string
    err  error
}

func (le languageErr) Error() string {
    return fmt.Sprintf("%s language error: %v", le.lang, le.err)
}

func (le languageErr) Unwrap() error {
    return le.err
}
```

Теперь можно создать новую `languageErr` как обертку над исходной ошибкой в методе `describe()`:

```go
func (l languages) describe(lang string) (string, error) {
    descr, err := getValue(l, lang)
    if err != nil {
        return "", languageErr{lang, err}
    }
    return descr, nil
}
```

Сама по себе слоеная ошибка — только половина дела. Вторая половина — научиться клиенту с ней работать. Сейчас посмотрим, как.

# errors.Is()

Получив ошибку-матрешку, клиент может проверить, есть ли на каком-то слое интересующая его проблема. Для этого используют функцию `errors.Is()`:

```go
func (l languages) describe(lang string) (string, error) {
    descr, err := getValue(l, lang)
    if err != nil {
        return "", languageErr{lang, err}
    }
    return descr, nil
}

// ...

func main() {
    descr, err := langs.describe("java")
    if errors.Is(err, errNotFound) {
        fmt.Println("this is an errNotFound error")
        // do something about it...
    }
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Println(descr)
}
```

```text
this is an errNotFound error
java language error: not found
```

Неважно, сколько в ошибке слоев. Если на каком-то из них встретилось значение `errNotFound` — `errors.Is()` вернет `true`.

# errors.As()

Раньше мы использовали приведение типа, чтобы получить доступ к ошибке конкретного типа вместо абстрактного `error`:

```go
descr, err := langs.describe("java")
if langErr, ok := err.(languageErr); ok {
    fmt.Println("Language error:", langErr.lang)
}
```

```text
Language error: java
```

Но это работает только для ошибки самого верхнего уровня. До ошибки из середины «матрешки» через приведение типа не добраться. А вот через `errors.As()` — можно:

```go
descr, err := langs.describe("java")
// обернем еще раз, чтобы languageErr
// оказалась внутрь матрешки
err = fmt.Errorf("wrap once more: %w", err)

var langErr languageErr
if errors.As(err, &langErr) {
    fmt.Println("Language error:", langErr.lang)
}
```

```text
Language error: java
wrap once more: java language error: not found
```

`errors.As()` проверяет каждый слой ошибки, и если видит там искомый тип `languageErr` — заполняет значение `langErr` по переданному указателю, и возвращает `true`. Если искомого типа нет — возвращает `false`.

Итого по обертыванию:

* Простой способ создать новую ошибку на основе существующей — `fmt.Errorf()` и спецификатор `%w`
* Если нужен собственный тип ошибки, придется добавить в него поле типа `error` и метод `Unwrap()`
* `errors.Is()` проверяет конкретную ошибку на каждом слое.
* `errors.As()` заполняет ошибку конкретного типа, если он встречается на одном из слоев.

[Задание 3. Игра без права на ошибку](tasks/task3.md)

[<< Назад](defer-panic-recover.md) [На главную](content.md)