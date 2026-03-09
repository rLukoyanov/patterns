# 1️⃣ Producer / Consumer

---

# 🧩 Проблема

Один компонент генерирует данные,
другой потребляет их с другой скоростью.

Нужен безопасный буфер и развязка скоростей.

---

# 💡 Идея

Producer пишет в канал,
Consumer читает из канала.

Буфер канала выравнивает пики нагрузки.

---

# 🏗 Структура

```
Producer(s) -> Buffered channel -> Consumer(s)
```

---

# 💻 Реализация на Go

```go
package main

import (
	"fmt"
	"time"
)

func producer(queue chan<- int) {
	defer close(queue)
	for i := 1; i <= 5; i++ {
		queue <- i
		fmt.Println("produced:", i)
	}
}

func consumer(queue <-chan int, done chan<- struct{}) {
	for item := range queue {
		fmt.Println("consumed:", item)
		time.Sleep(200 * time.Millisecond)
	}
	done <- struct{}{}
}

func main() {
	queue := make(chan int, 2)
	done := make(chan struct{})

	go producer(queue)
	go consumer(queue, done)

	<-done
}
```

---

# 📌 Когда использовать

Когда:

* производство и потребление идут с разной скоростью
* нужна асинхронная очередь между компонентами
* требуется разгрузить источник данных

---

# 🚫 Когда не использовать

Не нужен если:

* обмен строго синхронный
* задержка через очередь недопустима

---

# 🏭 Реальный пример

Сервис логов: producer пишет события, consumer отправляет их в хранилище.

Из реального проекта:

- Segment `kafka-go` — producer/consumer-модель для Kafka: https://github.com/segmentio/kafka-go/blob/main/reader.go

Ещё примеры из реальных проектов:

- Sarama — high-level consumer groups для Kafka: https://github.com/IBM/sarama/blob/main/consumer_group.go
- NATS Server/Client — pub/sub и queue subscribers: https://github.com/nats-io/nats-server/blob/main/server/client.go
- RabbitMQ tutorials (Go) — классический producer/consumer с очередями: https://github.com/rabbitmq/rabbitmq-tutorials/blob/main/go/receive.go
- Watermill — event-driven producer/consumer abstractions: https://github.com/ThreeDotsLabs/watermill/blob/master/message/router.go
- NSQ — распределённая messaging-платформа для producer/consumer: https://github.com/nsqio/nsq/blob/master/nsqd/channel.go
- `streadway/amqp` — AMQP producer/consumer для RabbitMQ: https://github.com/streadway/amqp/blob/master/channel.go

Короткий пример consumer-цикла:

```go
reader := kafka.NewReader(kafka.ReaderConfig{Brokers: brokers, Topic: "events"})
defer reader.Close()

for {
	m, err := reader.ReadMessage(ctx)
	if err != nil { break }
	process(m.Value)
}
```

---

# 🧠 Задание

Реализуй Producer/Consumer для:

```
email queue
```

требования:

```
producer кладет письма в буфер
consumer отправляет письма батчами
```

---

# ⚠️ Типичные ошибки

* **Deadlock** — producer/consumer ждут друг друга из-за неверного размера буфера или закрытия канала.
* **Goroutine leak** — consumer не получает сигнал завершения и остается в `range` навсегда.
* **Race condition** — несколько consumer одновременно меняют общий счетчик/батч без блокировок.

Как избегать:

* четко определять, кто и когда закрывает очередь
* отправлять сигнал остановки (`done`/`context`) для всех consumer
* защищать общий state через `mutex/atomic` или выделенный single-writer поток

---

# ✅ Мини-чеклист перед продом

* Размер буфера очереди выбран по нагрузочному профилю.
* Consumer-ы поддерживают корректное завершение по stop/cancel сигналу.
* Есть backpressure-стратегия при переполнении (drop/retry/block).
* Обработаны poison-message и повторная доставка (если применимо).
* Добавлены метрики очереди: depth, processing time, failed items.

