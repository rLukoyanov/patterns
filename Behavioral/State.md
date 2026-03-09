# 1️⃣ State

---

# 🧩 Проблема

Поведение объекта сильно зависит от его состояния,
и код превращается в большой набор `if/switch` по статусам.

---

# 💡 Идея

Каждое состояние оформить как отдельный объект,
а контекст делегирует ему поведение.

Переходы между состояниями управляются самими состояниями.

---

# 🏗 Структура

```
Context
  ↓
State interface
  ↓
Concrete states
```

---

# 💻 Реализация на Go

```go
package main

import "fmt"

type State interface {
	Handle(o *Order)
}

type Order struct {
	state State
}

func (o *Order) SetState(s State) {
	o.state = s
}

func (o *Order) Process() {
	o.state.Handle(o)
}

type CreatedState struct{}

func (CreatedState) Handle(o *Order) {
	fmt.Println("Order created -> paid")
	o.SetState(PaidState{})
}

type PaidState struct{}

func (PaidState) Handle(o *Order) {
	fmt.Println("Order paid -> shipped")
	o.SetState(ShippedState{})
}

type ShippedState struct{}

func (ShippedState) Handle(o *Order) {
	fmt.Println("Order already shipped")
}

func main() {
	order := &Order{state: CreatedState{}}
	order.Process()
	order.Process()
	order.Process()
}
```

---

# 📌 Когда использовать

Когда:

* объект проходит через фиксированные состояния
* в каждом состоянии поведение разное
* нужно убрать длинные ветвления по статусам

---

# 🚫 Когда не использовать

Не нужен если:

* состояний мало и логика простая
* переходы почти не меняются

---

# 🏭 Реальный пример

Статусы заказа:

```go
created -> paid -> shipped -> delivered
```

---

# 🧠 Задание

Реализуй State для:

```
Player
```

состояния:

```
Stopped
Playing
Paused
```

методы:

```
Play()
Pause()
Stop()
```
