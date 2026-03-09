# 1️⃣ Adapter

---

# 🧩 Проблема

Есть существующий код с ожидаемым интерфейсом,
но сторонний компонент имеет несовместимый API.

Прямое использование невозможно без переписывания клиента.

---

# 💡 Идея

Создать адаптер, который оборачивает несовместимый объект
и приводит его к нужному интерфейсу.

---

# 🏗 Структура

```
Client
  ↓
Target interface
  ↓
Adapter
  ↓
Adaptee
```

---

# 💻 Реализация на Go

```go
package main

import "fmt"

type PaymentProcessor interface {
	Pay(amount int)
}

type LegacyBankAPI struct{}

func (LegacyBankAPI) MakeTransfer(sum int) {
	fmt.Println("Transfer via legacy bank:", sum)
}

type BankAdapter struct {
	legacy LegacyBankAPI
}

func (a BankAdapter) Pay(amount int) {
	a.legacy.MakeTransfer(amount)
}

func Checkout(p PaymentProcessor) {
	p.Pay(100)
}

func main() {
	adapter := BankAdapter{legacy: LegacyBankAPI{}}
	Checkout(adapter)
}
```

---

# 📌 Когда использовать

Когда:

* нужно интегрировать старый или внешний API
* менять клиентский код нежелательно
* требуется совместимость интерфейсов

---

# 🚫 Когда не использовать

Не нужен если:

* проще изменить исходный компонент
* несовместимость минимальна и временная

---

# 🏭 Реальный пример

Интеграция legacy-сервиса оплаты в новый checkout.

---

# 🧠 Задание

Реализуй Adapter для:

```
SMSProvider
```

старый API:

```
SendSMS(phone, text)
```

новый интерфейс:

```
Notifier.Send(userID, message)
```
