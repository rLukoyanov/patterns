# 1️⃣ Observer

---

# 🧩 Проблема

Несколько объектов должны узнавать об изменении состояния одного объекта.

Если подписчиков вызывать вручную, код становится хрупким.

---

# 💡 Идея

Субъект хранит список подписчиков и отправляет им уведомления
при изменении состояния.

---

# 🏗 Структура

```
Subject
  ├─ Subscribe()
  ├─ Unsubscribe()
  └─ Notify()
      ↓
Observer interface
      ↓
Concrete observers
```

---

# 💻 Реализация на Go

```go
package main

import "fmt"

type Observer interface {
	Update(event string)
}

type User struct {
	Name string
}

func (u User) Update(event string) {
	fmt.Printf("%s got: %s\n", u.Name, event)
}

type Publisher struct {
	observers []Observer
}

func (p *Publisher) Subscribe(o Observer) {
	p.observers = append(p.observers, o)
}

func (p *Publisher) Notify(event string) {
	for _, o := range p.observers {
		o.Update(event)
	}
}

func main() {
	pub := &Publisher{}
	pub.Subscribe(User{Name: "Alice"})
	pub.Subscribe(User{Name: "Bob"})
	pub.Notify("new article")
}
```

---

# 📌 Когда использовать

Когда:

* есть отношение один-ко-многим
* подписчики динамически добавляются/удаляются
* нужна слабая связанность между источником и получателями

---

# 🚫 Когда не использовать

Не нужен если:

* получатель всегда один
* простого прямого вызова достаточно

---

# 🏭 Реальный пример

Уведомления в блоге:

```go
publisher.Notify("post published")
```

---

# 🧠 Задание

Реализуй Observer для:

```
OrderService
```

события:

```
created
paid
cancelled
```

наблюдатели:

```
EmailNotifier
AuditLogger
```
