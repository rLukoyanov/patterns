# 1️⃣ Pipeline

---

# 🧩 Проблема

Данные проходят через несколько последовательных этапов обработки.

Если писать всё в одной функции, код сложно масштабировать и тестировать.

---

# 💡 Идея

Разделить обработку на этапы (stages),
где каждый этап — отдельная горутина с входным и выходным каналом.

---

# 🏗 Структура

```
Source -> Stage 1 -> Stage 2 -> Stage 3 -> Sink
```

---

# 💻 Реализация на Go

```go
package main

import "fmt"

func generate(nums ...int) <-chan int {
	out := make(chan int)
	go func() {
		defer close(out)
		for _, n := range nums {
			out <- n
		}
	}()
	return out
}

func square(in <-chan int) <-chan int {
	out := make(chan int)
	go func() {
		defer close(out)
		for n := range in {
			out <- n * n
		}
	}()
	return out
}

func filterEven(in <-chan int) <-chan int {
	out := make(chan int)
	go func() {
		defer close(out)
		for n := range in {
			if n%2 == 0 {
				out <- n
			}
		}
	}()
	return out
}

func main() {
	in := generate(1, 2, 3, 4, 5)
	squared := square(in)
	even := filterEven(squared)

	for v := range even {
		fmt.Println(v)
	}
}
```

---

# 📌 Когда использовать

Когда:

* обработка естественно разбивается на этапы
* этапы можно выполнять конкурентно
* нужен потоковый режим обработки

---

# 🚫 Когда не использовать

Не нужен если:

* логика очень короткая и линейная
* стоимость межэтапной коммуникации выше пользы

---

# 🏭 Реальный пример

ETL: чтение данных → нормализация → валидация → запись в БД.

Из реального проекта:

- Go Blog — canonical пример pipelines и cancellation: https://github.com/golang/website/blob/master/_content/blog/pipelines.md

Ещё примеры из реальных проектов:

- Apache Beam (Go SDK) — распределённые data pipelines: https://github.com/apache/beam/blob/master/sdks/go/pkg/beam/pardo.go
- Benthos — потоковые конвейеры обработки данных: https://github.com/redpanda-data/benthos/blob/main/internal/pipeline/processor.go
- `segmentio/kafka-go` — pipeline чтение → обработка → запись сообщений: https://github.com/segmentio/kafka-go/blob/main/reader.go
- Milvus DataNode/DataCoord (Go) — пайплайны обработки вектора данных: https://github.com/milvus-io/milvus/blob/master/internal/datanode/data_sync_service.go
- OpenTelemetry Collector — процессоры/экспортеры как pipeline стадий: https://github.com/open-telemetry/opentelemetry-collector/blob/main/service/pipelines/config.go
- `go-kit` endpoint middleware chain — композиция шагов обработки запросов: https://github.com/go-kit/kit/blob/master/endpoint/endpoint.go

Короткий пример (stage + fan-in):

```go
in := gen(1, 2, 3, 4)
c1 := sq(in)
c2 := sq(in)

for n := range merge(c1, c2) {
	fmt.Println(n)
}
```

---

# 🧠 Задание

Реализуй Pipeline для:

```
log processing
```

этапы:

```
Parse -> Enrich -> Aggregate
```

---

# ⚠️ Типичные ошибки

* **Deadlock** — один из stage перестал читать/писать, цепочка каналов зависла.
* **Goroutine leak** — downstream завершился, а upstream продолжает отправку в канал.
* **Race condition** — несколько stage пишут в общий accumulator без синхронизации.

Как избегать:

* в каждом stage гарантировать `close(out)` и корректный `range in`
* прокидывать `context.Context` через pipeline для ранней отмены
* не делить mutable state между stage без `mutex/atomic`

---

# ✅ Мини-чеклист перед продом

* У каждого stage есть четкий контракт входа/выхода и правила закрытия каналов.
* Поддерживается ранняя отмена через `context.Context`.
* Нет блокирующих отправок без потребителя downstream.
* Добавлены метрики по каждому stage (latency, throughput, error rate).
* Тестами покрыты сценарии частичной ошибки и раннего завершения.

