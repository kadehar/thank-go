# Методы

Go позволяет определять *методы* на типах.

Метод отличается от обычной функции специальным параметром — *получателем* (receiver). В определении метода получатель указывается сразу после ключевого слова `func`. В данном случае — получатель типа `rect`:

```go
type rect struct {
    width, height int
}

func (r rect) area() int {
    return r.width * r.height
}
```

Метод вызывается для получателя через точку, как в питоне или js:

```go
r := rect{width: 10, height: 5}
fmt.Println("rect area:", r.area())
// rect area: 50
```

Получателем может быть не значение заданного типа, а указатель на это значение:

```go
type circle struct {
    radius int
}

func (c *circle) area() float64 {
    return math.Pi * math.Pow(float64(c.radius), 2)
}

cptr := &circle{radius: 5}
fmt.Println("circle area:", cptr.area())
// circle area: 78.54
```

При вызове метода Go автоматически преобразует значение получателя в указатель или указатель в значение, как того требует определение метода. Любой из перечисленных вариантов будет работать:

```go
rptr := &r
r.area()
rptr.area()

c := *cptr
c.area()
cptr.area()
```

Считается хорошим тоном во всех методах использовать или только значение, или только указатель, но не смешивать одно с другим. Обычно используют указатель: так Go не приходится копировать всю структуру, а метод может ее изменить.

```go
// Если метод принимает получателя как значение, изменить его не получится
func (r rect) scale(factor int) {
    r.width *= factor
    r.height *= factor
}

fmt.Println("rect before scaling:", r)
// rect before scaling: {10 5}

r.scale(2)

fmt.Println("rect after scaling:", r)
// rect after scaling: {10 5}
```

```go
// Если метод принимает получателя как указатель, его можно изменить
func (c *circle) scale(factor int) {
    c.radius *= factor
}

fmt.Println("circle before scaling:", c)
// circle before scaling: {5}

c.scale(2)

fmt.Println("circle after scaling:", c)
// circle after scaling: {10}
```

Вопрос «значение или указатель для получателя» так популярен, что у сообщества есть [отдельный гайдлайн](https://github.com/golang/go/wiki/CodeReviewComments#receiver-type) на эту тему.

# Метод-значение

Есть тип «счетчик»:

```go
type counter struct {
    value uint
}

func (c *counter) increment() {
    c.value++
}
```

Можно создать значение `c` типа `counter` и вызвать метод `c.increment`:

```go
c := new(counter)

c.increment()
c.increment()
c.increment()

fmt.Println(c.value)
// 3
```

А можно вызвать метод `c.increment` как функцию:

```go
c := new(counter)
inc := c.increment

inc()
inc()
inc()

fmt.Println(c.value)
// 3
```

Здесь функция `inc` работает как замыкание — она имеет доступ к внутренним полям `c`, хотя снаружи они не видны.

Такая штука называется *метод-значение* (method value). Используется редко. Но может пригодиться, чтобы разрешить клиенту вызывать метод структуры, не давая при этом доступ к ее полям.

# Метод-выражение

Можно сделать еще более причудливый финт. Использовать метод вообще без привязки к конкретному значению, как обычную функцию:

```go
inc := (*counter).increment

first := new(counter)
inc(first)
inc(first)
inc(first)

second := new(counter)
inc(second)

fmt.Println(first.value)
// 3
fmt.Println(second.value)
// 1
```

Здесь функция `inc` принимает первым аргументом получателя метода — значение `x` типа `*counter` — и дальше работает как метод, увеличивая значение `x.value`.

Такая штука называется *метод-выражение* (method expression). На практике встречается еще реже, чем метод-значение.

[<< Назад](struct.md) [На главную](content.md) [Вперед >>](defined-types.md)