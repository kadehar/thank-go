# Продолжительность в минутах

Напишите программу, которая считает количество минут во временном отрезке.

```
1h30m = 90 min
300s = 5 min
10m = 10 min
```

Используйте для этого `time.Duration.Minutes()` из стандартной библиотеки.

<details>
    <summary><strong>Solution</strong></summary>

```go 
package main

import (
	"fmt"
	"time"
)

func main() {
    // считываем временной отрезок из os.Stdin
    // гарантируется, что значение корректное
    // не меняйте этот блок
    var s string
    fmt.Scan(&s)
    d, _ := time.ParseDuration(s)

    // выведите исходное значение
    // и количество минут в нем
    // в формате "исходное = X min"
    // используйте метод .Minutes() объекта d
    fmt.Println(s,"=", d.Minutes(), "min")
}
```

</details>

[<< Назад](../values.md)