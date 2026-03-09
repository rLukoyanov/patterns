# 1️⃣ Proxy

---

# 🧩 Проблема

Нужно контролировать доступ к объекту
(ленивая инициализация, кэш, авторизация, логирование).

---

# 💡 Идея

Создать прокси с тем же интерфейсом,
который перехватывает вызовы и добавляет контроль.

---

# 🏗 Структура

```
Client
  ↓
Subject interface
  ├─ RealSubject
  └─ Proxy
```

---

# 💻 Реализация на Go

```go
package main

import "fmt"

type Image interface {
	Display()
}

type RealImage struct {
	filename string
}

func NewRealImage(filename string) *RealImage {
	fmt.Println("Loading image from disk:", filename)
	return &RealImage{filename: filename}
}

func (i RealImage) Display() {
	fmt.Println("Display image:", i.filename)
}

type ImageProxy struct {
	filename string
	real     *RealImage
}

func (p *ImageProxy) Display() {
	if p.real == nil {
		p.real = NewRealImage(p.filename)
	}
	p.real.Display()
}

func main() {
	img := &ImageProxy{filename: "big_photo.png"}
	img.Display()
	img.Display()
}
```

---

# 📌 Когда использовать

Когда:

* нужна ленивая загрузка
* нужно добавить доступ/кэш/логирование
* важно сохранить тот же интерфейс

---

# 🚫 Когда не использовать

Не нужен если:

* дополнительный контроль не требуется
* накладные расходы прокси критичны

---

# 🏭 Реальный пример

API proxy с проверкой токена и кэшированием ответов.

---

# 🧠 Задание

Реализуй Proxy для:

```
ReportService
```

требования:

```
AuthProxy
CacheProxy
```

общий метод:

```
GetReport(id string)
```
