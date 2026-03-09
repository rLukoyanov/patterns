# 1️⃣ Factory Method

---

# 🧩 Проблема

Код зависит от **конкретных структур**.

```go
car := BMW{}
```

Если появится новая машина — код придется менять.

Это нарушает **Open/Closed Principle**.

---

# 💡 Идея

Создание объекта делегируется **фабричной функции**.

Клиент работает только с **интерфейсом**.

---

# 🏗 Структура

```
Client
  ↓
Factory
  ↓
Interface
  ↓
Concrete implementations
```

---

# 💻 Реализация на Go

```go
package main

import "fmt"

type Payment interface {
	Pay(amount int)
}

type Stripe struct{}

func (s Stripe) Pay(amount int) {
	fmt.Println("Paying with Stripe:", amount)
}

type PayPal struct{}

func (p PayPal) Pay(amount int) {
	fmt.Println("Paying with PayPal:", amount)
}

func NewPayment(method string) Payment {
	switch method {
	case "stripe":
		return Stripe{}
	case "paypal":
		return PayPal{}
	default:
		return nil
	}
}

func main() {
	payment := NewPayment("stripe")
	payment.Pay(100)
}
```

---

# 📌 Когда использовать

Когда:

* есть **несколько реализаций интерфейса**
* тип определяется **runtime конфигурацией**

пример:

```
database
cache
payment
storage
logger
```

---

# 🚫 Когда не использовать

Не нужен если:

* только **одна реализация**
* фабрика превращается в **огромный switch**

---

# 🏭 Реальный пример

Создание storage:

```go
storage := NewStorage("s3")
```

Реализации:

```
S3Storage
LocalStorage
GCSStorage
```

---

# 🔎 Идиоматичный Go-вариант

В Go часто используют **map factories** вместо switch.

```go
var factories = map[string]func() Payment{
	"stripe": func() Payment { return Stripe{} },
	"paypal": func() Payment { return PayPal{} },
}

func NewPayment(method string) Payment {
	if f, ok := factories[method]; ok {
		return f()
	}
	return nil
}
```

Это **расширяемо без изменения кода**.

---

# 🧠 Задание

Реализуй Factory для:

```
Storage interface
```

реализации:

```
LocalStorage
S3Storage
GCSStorage
```

метод:

```
Save(file string)
```
