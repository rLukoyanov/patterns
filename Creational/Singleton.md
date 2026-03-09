# 1️⃣ Singleton

---

# 🧩 Проблема

Нужен **один общий экземпляр** сервиса
для всего приложения.

Если создавать много экземпляров:

```go
logger1 := NewLogger()
logger2 := NewLogger()
```

состояние может расходиться, а ресурсы — тратиться лишне.

---

# 💡 Идея

Ограничить создание объекта до **одного экземпляра**
и дать глобальную точку доступа к нему.

---

# 🏗 Структура

```
Client
  ↓
GetInstance()
  ↓
Single shared object
```

---

# 💻 Реализация на Go

```go
package main

import (
	"fmt"
	"sync"
)

type Logger struct{}

func (l *Logger) Log(msg string) {
	fmt.Println("LOG:", msg)
}

var (
	instance *Logger
	once     sync.Once
)

func GetLogger() *Logger {
	once.Do(func() {
		instance = &Logger{}
	})
	return instance
}

func main() {
	a := GetLogger()
	b := GetLogger()

	fmt.Println(a == b)
	a.Log("singleton works")
}
```

---

# 📌 Когда использовать

Когда:

* нужен **единый shared-ресурс**
* важна централизованная конфигурация
* создание экземпляра дорогое

пример:

```
logger
config manager
metrics collector
```

---

# 🚫 Когда не использовать

Не нужен если:

* можно передать зависимость явно (DI)
* усложняет тестирование
* появляется скрытое глобальное состояние

---

# 🏭 Реальный пример

Глобальная конфигурация приложения:

```go
cfg := GetConfig()
fmt.Println(cfg.Env)
```

---

# 🧠 Задание

Реализуй Singleton для:

```
DBConnectionPool
```

требования:

```
thread-safe initialization
GetPool()
```

добавь метод:

```
Stats()
```
