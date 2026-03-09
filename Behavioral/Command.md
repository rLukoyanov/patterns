# 1️⃣ Command

---

# 🧩 Проблема

Нужно отделить объект, который инициирует действие,
от объекта, который это действие выполняет.

Без этого UI или API слой знает слишком много о бизнес-логике.

---

# 💡 Идея

Инкапсулировать запрос в отдельный объект-команду.

Команда хранит получателя и параметры операции,
а вызывающий просто запускает `Execute()`.

---

# 🏗 Структура

```
Invoker
  ↓
Command interface (Execute)
  ↓
Concrete Command
  ↓
Receiver
```

---

# 💻 Реализация на Go

```go
package main

import "fmt"

type Command interface {
	Execute()
}

type Light struct{}

func (Light) On() {
	fmt.Println("Light is ON")
}

type LightOnCommand struct {
	light Light
}

func (c LightOnCommand) Execute() {
	c.light.On()
}

type Remote struct {
	command Command
}

func (r *Remote) SetCommand(c Command) {
	r.command = c
}

func (r *Remote) PressButton() {
	r.command.Execute()
}

func main() {
	remote := &Remote{}
	remote.SetCommand(LightOnCommand{light: Light{}})
	remote.PressButton()
}
```

---

# 📌 Когда использовать

Когда:

* нужно отделить отправителя от получателя
* хочется добавлять очередь, логирование, retry
* нужен undo/redo

---

# 🚫 Когда не использовать

Не нужен если:

* действие одно и простое
* добавление команд создаёт лишние структуры

---

# 🏭 Реальный пример

Очередь задач:

```go
queue.Push(SendEmailCommand{...})
queue.Push(RebuildCacheCommand{...})
```

---

# 🧠 Задание

Реализуй Command для:

```
BankAccount
```

команды:

```
DepositCommand
WithdrawCommand
```

метод:

```
Execute()
```
