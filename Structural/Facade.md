# 1️⃣ Facade

---

# 🧩 Проблема

Подсистема состоит из множества сервисов,
и клиенту приходится знать детали их взаимодействия.

---

# 💡 Идея

Сделать фасад с простым интерфейсом,
который скрывает сложность работы с подсистемой.

---

# 🏗 Структура

```
Client
  ↓
Facade
  ↓
Subsystem A / B / C
```

---

# 💻 Реализация на Go

```go
package main

import "fmt"

type Inventory struct{}
func (Inventory) Reserve(item string) { fmt.Println("Reserve:", item) }

type Billing struct{}
func (Billing) Charge(user string) { fmt.Println("Charge:", user) }

type Shipping struct{}
func (Shipping) Ship(item string) { fmt.Println("Ship:", item) }

type OrderFacade struct {
	inventory Inventory
	billing   Billing
	shipping  Shipping
}

func (f OrderFacade) PlaceOrder(user, item string) {
	f.inventory.Reserve(item)
	f.billing.Charge(user)
	f.shipping.Ship(item)
}

func main() {
	facade := OrderFacade{}
	facade.PlaceOrder("ruslan", "book")
}
```

---

# 📌 Когда использовать

Когда:

* есть сложная подсистема
* нужно дать простой API для частых сценариев
* хочется снизить связанность клиента с модулями

---

# 🚫 Когда не использовать

Не нужен если:

* подсистема и так проста
* фасад превращается в «бог-объект»

---

# 🏭 Реальный пример

`OrderService.PlaceOrder()` внутри вызывает склад, оплату и доставку.

---

# 🧠 Задание

Реализуй Facade для:

```
VideoConverter
```

подсистемы:

```
Decoder
Encoder
AudioProcessor
```

метод фасада:

```
Convert(input, outputFormat)
```
