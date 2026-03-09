# 1️⃣ Fan-In / Fan-Out

---

# 🧩 Проблема

Нужно распараллелить обработку одного потока задач,
а затем собрать результаты в единый поток.

---

# 💡 Идея

**Fan-Out:** отправить задачи нескольким воркерам.

**Fan-In:** объединить выходы нескольких воркеров в один канал.

---

# 🏗 Структура

```
Input
  ↓ (fan-out)
Worker 1..N
  ↓ (fan-in)
Merged output
```

---

# 💻 Реализация на Go

```go
package main

import (
	"fmt"
	"sync"
)

func worker(id int, in <-chan int) <-chan int {
	out := make(chan int)
	go func() {
		defer close(out)
		for n := range in {
			out <- n * 10
			fmt.Printf("worker %d handled %d\n", id, n)
		}
	}()
	return out
}

func merge(chans ...<-chan int) <-chan int {
	out := make(chan int)
	var wg sync.WaitGroup

	forward := func(c <-chan int) {
		defer wg.Done()
		for v := range c {
			out <- v
		}
	}

	wg.Add(len(chans))
	for _, c := range chans {
		go forward(c)
	}

	go func() {
		wg.Wait()
		close(out)
	}()

	return out
}

func main() {
	in := make(chan int)
	go func() {
		defer close(in)
		for i := 1; i <= 6; i++ {
			in <- i
		}
	}()

	w1 := worker(1, in)
	w2 := worker(2, in)
	w3 := worker(3, in)

	for v := range merge(w1, w2, w3) {
		fmt.Println("out:", v)
	}
}
```

---

# 📌 Когда использовать

Когда:

* нужно масштабировать throughput
* важно собрать результаты в единый поток
* шаг обработки CPU-bound или I/O-bound и хорошо параллелится

---

# 🚫 Когда не использовать

Не нужен если:

* объём данных мал
* порядок результатов критичен без доп. логики упорядочивания

---

# 🏭 Реальный пример

Параллельный парсинг файлов с объединением итогового потока событий.

Из реального проекта:

- HashiCorp `go-plugin` — обмен сообщениями и мультиплексирование потоков между процессами: https://github.com/hashicorp/go-plugin/blob/main/mux_broker.go

Ещё примеры из реальных проектов:

- Kubernetes `client-go/workqueue` — fan-out обработка задач несколькими worker-ами: https://github.com/kubernetes/client-go/blob/master/util/workqueue/queue.go
- `hibiken/asynq` — распределение задач по воркерам (fan-out) через Redis queue: https://github.com/hibiken/asynq/blob/master/processor.go
- `segmentio/kafka-go` — consumer group: чтение из нескольких партиций и объединение обработки: https://github.com/segmentio/kafka-go/blob/main/consumergroup.go
- Go `x/sync/errgroup` — конкурентный запуск задач и fan-in ошибок/результатов: https://github.com/golang/sync/blob/master/errgroup/errgroup.go
- Temporal Go SDK — параллельные activity/workflow-паттерны с агрегацией результатов: https://github.com/temporalio/sdk-go/blob/master/workflow/future.go
- Apache Beam (Go SDK) — распределённые data pipelines с fan-out/fan-in этапами: https://github.com/apache/beam/blob/master/sdks/go/pkg/beam/pardo.go

Короткий пример fan-out + merge:

```go
jobs := genTasks(files)
w1 := parseWorker(jobs)
w2 := parseWorker(jobs)
w3 := parseWorker(jobs)

for evt := range merge(w1, w2, w3) {
	handle(evt)
}
```

---

# 🔁 Дополнительные примеры

### 1) Параллельный поиск по нескольким API

```go
queries := genQueries("golang patterns")

g := googleWorker(queries)
y := yandexWorker(queries)
b := bingWorker(queries)

for res := range merge(g, y, b) {
	collect(res)
}
```

### 2) Обработка логов из нескольких источников

```go
appLogs := parseWorker(appLogStream)
nginxLogs := parseWorker(nginxLogStream)
dbLogs := parseWorker(dbLogStream)

for evt := range merge(appLogs, nginxLogs, dbLogs) {
	if evt.Level == "ERROR" {
		alert(evt)
	}
}
```

### 3) Генерация превью изображений

```go
jobs := genImages(paths)

r1 := resizeWorker(jobs, 320)
r2 := resizeWorker(jobs, 640)
r3 := resizeWorker(jobs, 1280)

for meta := range merge(r1, r2, r3) {
	saveMeta(meta)
}
```

---

# 🧠 Задание

Реализуй Fan-In/Fan-Out для:

```
thumbnail generator
```

требования:

```
fan-out: resize workers
fan-in: merge processed image metadata
```

---

# ⚠️ Типичные ошибки

* **Deadlock** — `merge` не закрывает итоговый канал после завершения всех input-каналов.
* **Goroutine leak** — forward-горутины висят, если consumer перестал читать merged-поток.
* **Race condition** — параллельные воркеры обновляют общий map/slice без синхронизации.

Как избегать:

* закрывать merged-канал после `WaitGroup.Wait()`
* делать отмену через `context.Context`, если downstream завершился раньше
* для shared-данных использовать `sync.Mutex`, `sync.Map` или отдельный aggregator

---

# ✅ Мини-чеклист перед продом

* Количество fan-out воркеров ограничено и конфигурируется.
* `merge` корректно закрывает итоговый канал после всех источников.
* Обработан сценарий раннего завершения consumer (cancel/backpressure).
* Если важен порядок — добавлен механизм reordering по ID.
* Видны метрики: per-worker throughput и общий merge lag.

