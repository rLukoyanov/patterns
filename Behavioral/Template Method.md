# 1️⃣ Template Method

---

# 🧩 Проблема

Есть общий алгоритм из нескольких шагов,
но часть шагов различается для разных реализаций.

Если копировать алгоритм в каждый класс, появляется дублирование.

---

# 💡 Идея

Зафиксировать каркас алгоритма в базовой структуре,
а изменяемые шаги вынести в переопределяемые методы.

---

# 🏗 Структура

```
Template method (fixed flow)
  ├─ Step1()
  ├─ Step2() (override)
  └─ Step3() (override)
```

---

# 💻 Реализация на Go

```go
package main

import "fmt"

type Processor interface {
	Read()
	Transform()
	Save()
}

type Pipeline struct {
	p Processor
}

func (pl Pipeline) Run() {
	pl.p.Read()
	pl.p.Transform()
	pl.p.Save()
}

type CSVProcessor struct{}

func (CSVProcessor) Read()      { fmt.Println("Read CSV") }
func (CSVProcessor) Transform() { fmt.Println("Transform CSV") }
func (CSVProcessor) Save()      { fmt.Println("Save CSV result") }

type JSONProcessor struct{}

func (JSONProcessor) Read()      { fmt.Println("Read JSON") }
func (JSONProcessor) Transform() { fmt.Println("Transform JSON") }
func (JSONProcessor) Save()      { fmt.Println("Save JSON result") }

func main() {
	Pipeline{p: CSVProcessor{}}.Run()
	Pipeline{p: JSONProcessor{}}.Run()
}
```

---

# 📌 Когда использовать

Когда:

* есть общий алгоритм с вариативными шагами
* важно централизовать порядок выполнения шагов
* нужно уменьшить дублирование

---

# 🚫 Когда не использовать

Не нужен если:

* шаги почти всегда уникальны
* общий каркас отсутствует

---

# 🏭 Реальный пример

ETL-пайплайн:

```go
Read -> Transform -> Save
```

для CSV, JSON, XML с одинаковым потоком.

---

# 🧠 Задание

Реализуй Template Method для:

```
DeploymentPipeline
```

шаги:

```
Build
Test
Deploy
```

с вариантами:

```
StagingPipeline
ProductionPipeline
```
