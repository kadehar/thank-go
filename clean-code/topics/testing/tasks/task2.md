# Табличные тесты на сумму

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

Помогите мне написать на нее табличные тесты вместо пачки обычных.

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

func TestSum(t *testing.T) {
	tests := []struct {
		name string
		nums ____
		want int
	}{
		{"zero", []int{}, ____},
		{"one", ____, 1},
		{____, []int{1, 2}, 3},
		{"many", ____, 15},
	}
	for _, test := range ____ {
		t.Run(test.name, func(____) {
			got := ____
			if got != test.want {
				t.Errorf("%s: got %d, want %d", ____, ____, ____)
			}
		})
	}
}
```

[<< Назад](../table-driven-tests.md)