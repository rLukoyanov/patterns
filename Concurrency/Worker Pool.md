# 1️⃣ Worker Pool

---

# 🧩 Проблема

Есть много независимых задач,
которые нужно выполнять параллельно, но с ограничением по ресурсам.

Запускать горутину на каждую задачу без лимита — риск перегрузки CPU/RAM.

---

# 💡 Идея

Создать фиксированное количество воркеров,
которые читают задачи из общего канала и отправляют результаты в другой канал.

---

# 🏗 Структура

```
Jobs channel
  ↓
Worker 1..N (goroutines)
  ↓
Results channel
```

---

# 💻 Реализация на Go

```go
package main

import (
	"fmt"
	"sync"
)

func worker(id int, jobs <-chan int, results chan<- int, wg *sync.WaitGroup) {
	defer wg.Done()
	for job := range jobs {
		fmt.Printf("worker %d processing %d\n", id, job)
		results <- job * job
	}
}

func main() {
	jobs := make(chan int)
	results := make(chan int)

	var wg sync.WaitGroup
	workerCount := 3

	for i := 1; i <= workerCount; i++ {
		wg.Add(1)
		go worker(i, jobs, results, &wg)
	}

	go func() {
		for j := 1; j <= 5; j++ {
			jobs <- j
		}
		close(jobs)
	}()

	go func() {
		wg.Wait()
		close(results)
	}()

	for r := range results {
		fmt.Println("result:", r)
	}
}
```

---

# 📌 Когда использовать

Когда:

* много однотипных задач
* нужен контролируемый уровень параллелизма
* важно утилизировать CPU без перегрузки

---

# 🚫 Когда не использовать

Не нужен если:

* задач очень мало
* каждая задача требует уникальной сложной оркестрации

---

# 🏭 Реальный пример

Обработка очереди изображений или email-рассылки с лимитом воркеров.

Из реального проекта:

- `alitto/pond` — lightweight worker pool для Go: https://github.com/alitto/pond/blob/main/pool.go

Ещё примеры из реальных проектов:

- Kubernetes `client-go/workqueue` — очереди задач с воркерами в контроллерах: https://github.com/kubernetes/client-go/blob/master/util/workqueue/queue.go
- `panjf2000/ants` — высокопроизводительный пул горутин: https://github.com/panjf2000/ants/blob/dev/ants.go
- `gocraft/work` — background jobs с worker pool поверх Redis: https://github.com/gocraft/work/blob/master/worker_pool.go
- `hibiken/asynq` — распределённые воркеры и очереди задач: https://github.com/hibiken/asynq/blob/master/server.go
- `Jeffail/tunny` — goroutine pool для контролируемой параллельной обработки: https://github.com/Jeffail/tunny/blob/master/tunny.go
- `gammazero/workerpool` — минималистичный worker pool для Go: https://github.com/gammazero/workerpool/blob/main/workerpool.go

Короткий пример использования:

```go
pool := pond.NewPool(10)
defer pool.StopAndWait()

for _, url := range urls {
	u := url
	pool.Submit(func() {
		_ = fetch(u)
	})
}
```

---

# 🧠 Задание

Реализуй Worker Pool для:

```
URL fetcher
```

требования:

```
10 workers
jobs <- []string (URLs)
results <- status code
```

---

# ⚠️ Типичные ошибки

* **Deadlock** — забыли закрыть `jobs` или `results`, чтение/запись блокируется навсегда.
* **Goroutine leak** — воркеры не завершаются, если нет сигнала остановки при раннем выходе.
* **Race condition** — общий state между воркерами меняется без `mutex/atomic`.

Как избегать:

* закрывать каналы по четкому owner-правилу (кто пишет — тот закрывает)
* использовать `context.Context` для отмены долгих задач
* защищать разделяемые структуры через `sync.Mutex` или `sync/atomic`

---

# ✅ Мини-чеклист перед продом

* Ограничен размер пула воркеров и очереди задач.
* Все воркеры завершаются по `context`/stop-сигналу.
* Каналы `jobs/results` закрываются в одном понятном месте.
* Добавлены метрики: длина очереди, время задачи, число активных воркеров.
* Есть graceful shutdown и ожидание завершения через `WaitGroup`.
