# Интерфейсы

*Интерфейс* в Go — это набор сигнатур методов (то есть список методов без реализации).

Интерфейс геометрической фигуры:

```go
type geometry interface {
    area() float64
    perim() float64
}
```

Реализуем интерфейс в типе «прямоугольник». Реализовать интерфейс = реализовать его методы. Действует «утиный» принцип, как в питоне: если у типа есть перечисленные в интерфейсе методы — значит, он реализовал интерфейс. Явно указывать, что `rect` реализует `geometry`, не требуется:

```go
type rect struct {
    width, height float64
}

func (r rect) area() float64 {
    return r.width * r.height
}

func (r rect) perim() float64 {
    return 2*r.width + 2*r.height
}
```

Аналогично для типа «круг»:

```go
type circle struct {
    radius float64
}

func (c circle) area() float64 {
    return math.Pi * c.radius * c.radius
}

func (c circle) perim() float64 {
    return 2 * math.Pi * c.radius
}
```

Если у переменной интерфейсный тип, она поддерживает все методы, заданные на интерфейсе. Благодаря этому функция `measure()` работает с любой фигурой, реализующей интерфейс `geometry`:

```go
func measure(g geometry) {
    fmt.Printf("%T: %+v\n", g, g)
    fmt.Println("area:", g.area())
    fmt.Println("perimiter:", g.perim())
}
```

Раз типы `circle` и `rect` реализуют интерфейс `geometry`, мы можем передать их экземпляры в функцию `measure()`:

```go
r := rect{width: 3, height: 4}
c := circle{radius: 5}

measure(r)
// main.rect: {width:3 height:4}
// area: 12
// perimiter: 14

measure(c)
// main.circle: {radius:5}
// area: 78.53981633974483
// perimiter: 31.41592653589793
```

# Встраивание интерфейса

Иногда при композиции хочется дать доступ к поведению, но скрыть структуру. В этом поможет *встраивание интерфейса* (interface embedding).

Есть тип «счетчик»:

```go
type counter struct {
    val uint
}
func (c *counter) increment() {
    c.val++
}
func (c *counter) value() uint {
    return c.val
}
```

Мы хотим встраивать счетчик в другие типы, но не давать прямой доступ к полю `val` — чтобы менять значение счетчика можно было только через методы.

Определим интерфейс счетчика:

```go
type Counter interface {
    increment()
    value() uint
}
```

И вместо конкретного типа `counter` встроим интерфейс `Counter`, который он реализует:

```go
type usage struct {
    service string
    Counter
}
```

В конструкторе будем создавать конкретное значение типа `counter`, но потребителям об этом знать не обязательно:

```go
func newUsage(service string) *usage {
    return &usage{service, &counter{}}
}
```

Поскольку мы встроили интерфейс, прямого доступа к `counter.val` больше нет. Можно использовать только методы интерфейса:

```go
usage := newUsage("find")
usage.increment()
usage.increment()
usage.increment()
fmt.Printf("%s usage: %d\n", usage.service, usage.value())
// find usage: 3
```

# Пустой интерфейс

Если у интерфейса нет методов, его называют *пустым* (empty):

```go
interface{}
```

Пустой интерфейс может содержать значение любого типа (ведь у каждого типа есть как минимум 0 методов). Пустые интерфейсы используют, если тип значения заранее не известен. Например, функция из пакета `fmt`:

```go
func Println(a ...interface{}) (n int, err error)
```

`fmt.Println()` умеет печатать что угодно, поэтому принимает значения типа `interface{}`.

Начиная с Go 1.18 для `interface{}` ввели псевдоним `any`. Разницы между ними нет (псевдоним — это буквально тот же самый тип), но многие теперь предпочитают `any` за его краткость и выразительность.

```go
func repr(val any) string {
    return fmt.Sprintf("%#v", val)
}

func main() {
    var num int = 42
    fmt.Println(repr(num))
    // 42

    var thing interface{} = "shy string"
    fmt.Println(repr(thing))
    // "shy string"
}
```

[<< Назад](content.md) [Вперед >>](type-switch-and-assertion.md)