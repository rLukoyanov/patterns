# 1️⃣ Strategy

---

# 🧩 Проблема

Есть несколько алгоритмов решения одной задачи,
и выбор алгоритма зависит от условий во время выполнения.

Если всё зашить в `if/switch`, код быстро разрастается.

---

# 💡 Идея

Вынести каждый алгоритм в отдельную **стратегию**
с общим интерфейсом.

Контекст хранит ссылку на стратегию и делегирует ей работу.

---

# 🏗 Структура

```
Client
  ↓
Context
  ↓
Strategy interface
  ↓
Concrete strategies
```

---

# 💻 Реализация на Go

```go
package main

import "fmt"

type SortStrategy interface {
	Sort([]int) []int
}

type BubbleSort struct{}

func (BubbleSort) Sort(data []int) []int {
	fmt.Println("Bubble sort")
	return data
}

type QuickSort struct{}

func (QuickSort) Sort(data []int) []int {
	fmt.Println("Quick sort")
	return data
}

type Sorter struct {
	strategy SortStrategy
}

func (s *Sorter) SetStrategy(strategy SortStrategy) {
	s.strategy = strategy
}

func (s *Sorter) Execute(data []int) []int {
	return s.strategy.Sort(data)
}

func main() {
	sorter := &Sorter{}
	sorter.SetStrategy(BubbleSort{})
	sorter.Execute([]int{3, 2, 1})

	sorter.SetStrategy(QuickSort{})
	sorter.Execute([]int{3, 2, 1})
}
```

---

# 📌 Когда использовать

Когда:

* есть несколько взаимозаменяемых алгоритмов
* нужно переключать поведение в runtime
* хочется убрать большие `if/switch`

---

# 🚫 Когда не использовать

Не нужен если:

* алгоритм всегда один
* добавление интерфейсов только усложняет код

---

# 🏭 Реальный пример

Выбор способа оплаты:

```go
checkout.SetPaymentStrategy(CardPayment{})
checkout.SetPaymentStrategy(SBPayment{})
```

---

# 🧠 Задание

Реализуй Strategy для:

```
CompressionStrategy
```

реализации:

```
Zip
Gzip
Brotli
```

метод:

```
Compress(data []byte)
```
