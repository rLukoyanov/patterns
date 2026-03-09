# 1️⃣ Abstract Factory

---

# 🧩 Проблема

Нужно создавать **связанные объекты** (например, кнопка + чекбокс),
но не хочется зависеть от конкретной платформы.

Если создавать объекты напрямую:

```go
button := MacButton{}
checkbox := WindowsCheckbox{} // несовместимое семейство
```

можно случайно смешать несовместимые реализации.

---

# 💡 Идея

Создавать объекты через **фабрику семейства**.

Клиент выбирает одну фабрику (например, `MacFactory`),
и получает из нее только совместимые продукты.

---

# 🏗 Структура

```
Client
  ↓
AbstractFactory
  ├─ CreateButton() -> Button
  └─ CreateCheckbox() -> Checkbox

ConcreteFactory (Mac / Windows)
  ↓
Concrete products (MacButton, MacCheckbox ...)
```

---

# 💻 Реализация на Go

```go
package main

import "fmt"

type Button interface {
	Render()
}

type Checkbox interface {
	Check()
}

type GUIFactory interface {
	CreateButton() Button
	CreateCheckbox() Checkbox
}

type MacButton struct{}

func (MacButton) Render() {
	fmt.Println("Render Mac button")
}

type MacCheckbox struct{}

func (MacCheckbox) Check() {
	fmt.Println("Check Mac checkbox")
}

type WindowsButton struct{}

func (WindowsButton) Render() {
	fmt.Println("Render Windows button")
}

type WindowsCheckbox struct{}

func (WindowsCheckbox) Check() {
	fmt.Println("Check Windows checkbox")
}

type MacFactory struct{}

func (MacFactory) CreateButton() Button {
	return MacButton{}
}

func (MacFactory) CreateCheckbox() Checkbox {
	return MacCheckbox{}
}

type WindowsFactory struct{}

func (WindowsFactory) CreateButton() Button {
	return WindowsButton{}
}

func (WindowsFactory) CreateCheckbox() Checkbox {
	return WindowsCheckbox{}
}

func BuildUI(factory GUIFactory) {
	button := factory.CreateButton()
	checkbox := factory.CreateCheckbox()
	button.Render()
	checkbox.Check()
}

func main() {
	BuildUI(MacFactory{})
}
```

---

# 📌 Когда использовать

Когда:

* есть **семейства связанных объектов**
* нужно гарантировать **совместимость** продуктов внутри семейства
* хочется переключать целое семейство одной конфигурацией

пример:

```
UI themes (Light / Dark)
OS widgets (Mac / Windows)
Database drivers (Postgres / MySQL families)
```

---

# 🚫 Когда не использовать

Не нужен если:

* есть только **один тип продукта**
* нет требований к совместимости между объектами
* добавление новых семейств не планируется

---

# 🏭 Реальный пример

Кроссплатформенный UI:

```go
factory := WindowsFactory{}
button := factory.CreateButton()
checkbox := factory.CreateCheckbox()
```

---

# 🧠 Задание

Реализуй Abstract Factory для:

```
NotificationFactory
```

семейства:

```
Email + EmailTemplate
SMS + SMSTemplate
```

методы:

```
CreateSender()
CreateTemplate()
```
