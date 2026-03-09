# 1️⃣ Prototype

---

# 🧩 Проблема

Создание объекта дорого или сложно,
а нужно быстро получать похожие экземпляры.

```go
report := BuildHugeReportFromScratch()
```

Повторная полная инициализация занимает время.

---

# 💡 Идея

Создать новый объект через **копирование существующего**
(прототипа), а затем изменить нужные поля.

---

# 🏗 Структура

```
Client
  ↓
Prototype interface (Clone)
  ↓
Concrete Prototype
```

---

# 💻 Реализация на Go

```go
package main

import "fmt"

type Config interface {
	Clone() Config
	SetName(name string)
	Print()
}

type AppConfig struct {
	Name    string
	Timeout int
	Retry   int
}

func (c AppConfig) Clone() Config {
	copy := c
	return &copy
}

func (c *AppConfig) SetName(name string) {
	c.Name = name
}

func (c *AppConfig) Print() {
	fmt.Printf("Name=%s Timeout=%d Retry=%d\n", c.Name, c.Timeout, c.Retry)
}

func main() {
	base := &AppConfig{Name: "base", Timeout: 30, Retry: 3}

	prod := base.Clone()
	prod.SetName("prod")

	dev := base.Clone()
	dev.SetName("dev")

	base.Print()
	prod.Print()
	dev.Print()
}
```

---

# 📌 Когда использовать

Когда:

* создание объекта **дорого по времени**
* нужно много похожих объектов
* хочется скрыть детали инициализации

пример:

```
configs
documents/templates
game objects
```

---

# 🚫 Когда не использовать

Не нужен если:

* объект проще создать заново
* копирование сложнее конструктора
* не решен вопрос deep/shallow copy

---

# 🏭 Реальный пример

Шаблоны писем:

```go
template := EmailTemplate{Subject: "Welcome", Body: "Hi, {{name}}"}
email := template.Clone().(*EmailTemplate)
```

---

# 🧠 Задание

Реализуй Prototype для:

```
Document
```

поля:

```
Title
Content
Tags []string
```

метод:

```
Clone()
```

Добавь корректное **deep copy** для `Tags`.

