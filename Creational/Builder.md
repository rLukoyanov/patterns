# 1️⃣ Builder

---

# 🧩 Проблема

Сложный объект создается через длинный конструктор
с большим количеством параметров.

```go
server := NewServer("localhost", 8080, true, 100, 30, "/tmp", ...)
```

Код трудно читать и легко ошибиться в порядке аргументов.

---

# 💡 Идея

Строить объект **пошагово** через builder,
а в конце вызывать `Build()`.

Это делает создание объекта понятным и гибким.

---

# 🏗 Структура

```
Client
  ↓
Builder (setters)
  ↓
Build()
  ↓
Complex Object
```

---

# 💻 Реализация на Go

```go
package main

import "fmt"

type Server struct {
	Host    string
	Port    int
	TLS     bool
	Timeout int
}

type ServerBuilder struct {
	server Server
}

func NewServerBuilder() *ServerBuilder {
	return &ServerBuilder{
		server: Server{
			Host:    "127.0.0.1",
			Port:    80,
			TLS:     false,
			Timeout: 10,
		},
	}
}

func (b *ServerBuilder) Host(host string) *ServerBuilder {
	b.server.Host = host
	return b
}

func (b *ServerBuilder) Port(port int) *ServerBuilder {
	b.server.Port = port
	return b
}

func (b *ServerBuilder) EnableTLS() *ServerBuilder {
	b.server.TLS = true
	return b
}

func (b *ServerBuilder) Timeout(seconds int) *ServerBuilder {
	b.server.Timeout = seconds
	return b
}

func (b *ServerBuilder) Build() Server {
	return b.server
}

func main() {
	server := NewServerBuilder().
		Host("localhost").
		Port(8080).
		EnableTLS().
		Timeout(30).
		Build()

	fmt.Println(server)
}
```

---

# 📌 Когда использовать

Когда:

* объект имеет **много опциональных параметров**
* нужна **читаемая последовательная сборка**
* важно зафиксировать шаги создания

пример:

```
HTTP server config
SQL query builder
UI form builder
```

---

# 🚫 Когда не использовать

Не нужен если:

* у объекта 2-3 простых параметра
* структура почти не меняется
* builder добавляет лишнюю сложность

---

# 🏭 Реальный пример

Сборка HTTP-клиента:

```go
client := NewClientBuilder().
	BaseURL("https://api.example.com").
	Token("secret").
	Timeout(15).
	Build()
```

---

# 🧠 Задание

Реализуй Builder для:

```
Report
```

поля:

```
Title
Author
WithCharts
WithSummary
Format
```

метод:

```
Build()
```
