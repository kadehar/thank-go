# Тесты на сумму

Есть функция, которая суммирует целые числа:

```go
func Sum(nums ...int) int {
    total := 0
    for _, num := range nums {
        total += num
    }
    return total
}
```

Помогите мне написать на нее тесты.

```go
package main

import (
	"testing"
)

func Sum(nums ...int) int {
	total := 0
	for _, num := range nums {
		total += num
	}
	return total
}

func TestSumZero(t *____.T) {
	if Sum() != 0 {
		t.Errorf("Expected Sum() == 0")
	}
}

func TestSumOne(t ____) {
	if Sum(1) != 1 {
		____("Expected Sum(1) == 1")
	}
}

func ____SumPair(____) {
	if Sum(1, 2) != 3 {
		t.Errorf("Expected Sum(1, 2) == 3")
	}
}

func TestSumMany(t *testing.T) {
	if ____ != ____ {
		____("Expected Sum(1, 2, 3, 4, 5) == 15")
	}
}
```

[<< Назад](../about.md)