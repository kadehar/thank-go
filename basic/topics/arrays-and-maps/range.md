# Обход коллекции

`range` обходит элементы коллекций. Посмотрим, как использовать его на срезах, картах и строках.

Просуммировать элементы среза (или массива):

```go
nums := []int{2, 3, 4}
sum := 0
for _, num := range nums {
    sum += num
}
fmt.Println("sum:", sum)
// sum: 9
```

`range` на массивах и срезах возвращает индекс и значение для каждого элемента. В примере выше мы не использовали индекс, поэтому заглушили его пустым идентификатором `_`. Но иногда индекс может и пригодиться:

```go
nums := []int{2, 3, 4}
for idx, num := range nums {
    if num == 3 {
        fmt.Println("index:", idx)
    }
}
// index: 1
```

`range` на карте итерирует по записям:

```go
m := map[string]string{"a": "apple", "b": "banana"}
for key, val := range m {
    fmt.Printf("%s -> %s\n", key, val)
}
// a -> apple
// b -> banana
```

Или только по ключам:

```go
m := map[string]string{"a": "apple", "b": "banana"}
for key := range m {
    fmt.Println("key:", key)
}
// key: a
// key: b
```

`range` на строках итерирует по unicode-символам (рунам). Первое значение — порядковый номер байта, с которого начинается руна (руна может занимать несколько байт). Второе значение — числовой код самой руны:

```go
for idx, char := range "ого" {
    fmt.Println(idx, char, string(char))
}
// 0 1086 о
// 2 1075 г
// 4 1086 о
```

[Задание 3. Аббревиатура](tasks/task3.md)

[<< Назад](maps.md) [На главную](content.md)