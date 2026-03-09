# 1️⃣ Future / Promise

---

# 🧩 Проблема

Нужно запустить долгую задачу асинхронно
и получить результат позже без блокировки основного потока.

---

# 💡 Идея

Вернуть объект future (в Go обычно канал),
который позже отдаст результат вычисления.

---

# 🏗 Структура

```
Client starts async task
  ↓
Future (channel)
  ↓
Result when ready
```

---

# 💻 Реализация на Go

```go
package main

import (
	"fmt"
	"time"
)

type Result struct {
	Value int
	Err   error
}

func AsyncSum(a, b int) <-chan Result {
	future := make(chan Result, 1)
	go func() {
		defer close(future)
		time.Sleep(500 * time.Millisecond)
		future <- Result{Value: a + b}
	}()
	return future
}

func main() {
	future := AsyncSum(10, 20)
	fmt.Println("doing other work...")

	res := <-future
	fmt.Println("sum:", res.Value)
}
```

---

# 📌 Когда использовать

Когда:

* операция длительная
* нужно продолжать работу, не блокируясь
* требуется композиция async-задач

---

# 🚫 Когда не использовать

Не нужен если:

* задача выполняется мгновенно
* асинхронность усложняет код больше, чем помогает

---

# 🏭 Реальный пример

Параллельный запрос к нескольким внешним API с ожиданием результата позже.

Из реального проекта:

- `sourcegraph/conc` — structured concurrency и futures-подобные паттерны: https://github.com/sourcegraph/conc/blob/main/pool/pool.go

Ещё примеры из реальных проектов:

- `golang/sync/errgroup` — конкурентный запуск задач и ожидание результата/ошибки: https://github.com/golang/sync/blob/master/errgroup/errgroup.go
- Temporal Go SDK — async activity execution и ожидание completion: https://github.com/temporalio/sdk-go/blob/master/workflow/future.go
- `cskr/pubsub` — async publish/subscribe доставка событий: https://github.com/cskr/pubsub/blob/master/pubsub.go
- NATS Go client — async запросы/подписки с последующим ожиданием ответов: https://github.com/nats-io/nats.go/blob/main/nats.go
- `go-redis` — конкурентные/pipeline команды к Redis: https://github.com/redis/go-redis/blob/master/pipeline.go
- gRPC Go — async streaming и обработка ответов по мере готовности: https://github.com/grpc/grpc-go/blob/master/stream.go

Короткий пример async-вычисления:

```go
future := Async(func() (User, error) {
	return repo.GetUser(ctx, id)
})

// ... другая работа
user, err := future.Await()
```

---

# 🧠 Задание

Реализуй Future для:

```
ReportGenerator
```

метод:

```
GenerateAsync(id string) <-chan Result
```

---

# ⚠️ Типичные ошибки

* **Deadlock** — future никогда не получает значение, а consumer бесконечно ждет `<-future`.
* **Goroutine leak** — async-задача зависает без timeout/cancel и остается в памяти.
* **Race condition** — несколько async-веток пишут в общий объект результата без синхронизации.

Как избегать:

* всегда завершать producer future: отправка результата + `close(future)`
* использовать `context.WithTimeout` для внешних I/O операций
* объединять результаты через каналы или защищенный aggregator

---

# ✅ Мини-чеклист перед продом

* Для async-операций задан timeout/deadline.
* Канал future всегда получает результат или ошибку (без «вечного ожидания»).
* Ошибки и отмены явно пробрасываются вызывающему коду.
* Ограничено число параллельных async-задач.
* Добавлены retry/backoff только там, где они безопасны идемпотентно.

