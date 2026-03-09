# 1️⃣ Decorator

---

# 🧩 Проблема

Нужно добавлять функциональность объекту динамически,
не раздувая количество наследников.

---

# 💡 Идея

Обернуть объект в декоратор с тем же интерфейсом
и добавить поведение до/после вызова базового объекта.

---

# 🏗 Структура

```
Client
  ↓
Component interface
  ↓
Concrete Component
  ↑
Decorator(s)
```

---

# 💻 Реализация на Go

```go
package main

import "fmt"

type Notifier interface {
	Send(msg string)
}

type EmailNotifier struct{}

func (EmailNotifier) Send(msg string) {
	fmt.Println("Email:", msg)
}

type SlackDecorator struct {
	next Notifier
}

func (d SlackDecorator) Send(msg string) {
	d.next.Send(msg)
	fmt.Println("Slack:", msg)
}

type SMSDecorator struct {
	next Notifier
}

func (d SMSDecorator) Send(msg string) {
	d.next.Send(msg)
	fmt.Println("SMS:", msg)
}

func main() {
	var n Notifier = EmailNotifier{}
	n = SlackDecorator{next: n}
	n = SMSDecorator{next: n}
	n.Send("Server is down")
}
```

---

# 📌 Когда использовать

Когда:

* нужно комбинировать дополнительные поведения
* важно избегать множества подклассов
* поведение должно подключаться по конфигурации

---

# 🚫 Когда не использовать

Не нужен если:

* расширение всегда одно и фиксированное
* цепочка декораторов усложняет отладку

---

# 🏭 Реальный пример

Пайплайн уведомлений: email + slack + sms.

---

# 🧠 Задание

Реализуй Decorator для:

```
Storage
```

базовый компонент:

```
FileStorage
```

декораторы:

```
EncryptionDecorator
CompressionDecorator
```
