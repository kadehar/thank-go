# Структуры

*Структура* (struct) группирует поля в единую запись. В Go нет классов и объектов, так что структура — наиболее близкий аналог объекта в питоне и js.

Объявим тип `person` на основе структуры с полями `name` и `age`:

```go
type person struct {
    name string
    age  int
}
```

Так создается новая структура типа `person`:

```go
bob := person{"Bob", 20}
fmt.Println(bob)
// {Bob 20}
```

Можно явно указать названия полей:

```go
alice := person{name: "Alice", age: 30}
fmt.Println(alice)
// {Alice 30}
```

Если не указать поле, оно получит нулевое значение:

```go
fred := person{name: "Fred"}
fmt.Println(fred)
// {Fred 0}
```

Оператор `&` возвращает указатель на структуру:

```go
annptr := &person{name: "Ann", age: 40}
fmt.Println(annptr)
// &{Ann 40}
```

В Go иногда создают новые структуры через функцию-конструктор с префиксом `new`:

```go
func newPerson(name string) *person {
    p := person{name: name}
    p.age = 42
    return &p
}
```

Функция возвращает указатель на локальную переменную — это нормально. Go распознает такие ситуации, и выделяет память под структуру в *куче* (heap) вместо *стека* (stack), так что структура продолжит существовать после выхода из функции.

```go
john := newPerson("John")
fmt.Println(john)
// &{John 42}
```

Если функция-конструктор возвращает саму структуру, а не указатель — удобно использовать префикс `make` вместо `new`:

```go
func makePerson(name string) person {
    p := person{name: name}
    p.age = 42
    return p
}
```

> *В реальности чаще не заморачиваются и всегда используют префикс `new` вне зависимости от того, что возвращает конструктор — значение или указатель на него. Но на курсе я буду соблюдать это разделение: `make` — значение, `new` — указатель.*

Доступ к полям структуры — через точку:

```go
sean := person{name: "Sean", age: 50}
fmt.Println(sean.name)
// Sean
```

Чтобы получить доступ к полям структуры через указатель, не обязательно разыменовывать его через `*`. Эти два варианта эквивалентны:

```go
sven := &person{name: "Sven", age: 50}
fmt.Println((*sven).age)
// 50
fmt.Println(sven.age)
// 50
```

Поля структуры можно изменять:

```go
sven.age = 51
fmt.Println(sven.age)
// 51
```

# Составные структуры

Структуры могут включать другие структуры:

```go
type person struct {
    firstName string
    lastName  string
}

type book struct {
    title  string
    author person
}

b := book{
    title: "The Majik Gopher",
    author: person{
        firstName: "Christopher",
        lastName:  "Swanson",
    },
}

fmt.Println(b)
// {The Majik Gopher {Christopher Swanson}}
```

Если вложенная структура не представляет самостоятельной ценности, можно даже не объявлять отдельный тип:

```go
type user struct {
    name  string
    karma struct {
        value int
        title string
    }
}

u := user{
    name: "Chris",
    karma: struct {
        value int
        title string
    }{
        value: 100,
        title: "^-^",
    },
}
fmt.Printf("%+v\n", u)
// {name:Chris karma:{value:100 title:^-^}}
```

Благодаря шаблону `%+v`, `Printf()` печатает структуру вместе с названиями полей.

Поле структуры может ссылаться на другую структуру:

```go
type comment struct {
    text   string
    author *user
}

chris := user{
    name: "Chris",
}
c := comment{
    text:   "Gophers are awesome!",
    author: &chris,
}
fmt.Printf("%+v\n", c)
// {text:Gophers are awesome! author:0xc0000981e0}
```

[<< Назад](content.md) [Вперед >>](methods.md)