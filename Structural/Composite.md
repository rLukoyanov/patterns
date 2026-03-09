# 1️⃣ Composite

---

# 🧩 Проблема

Нужно работать с древовидной структурой
(файлы/папки, меню, оргструктура)
и одинаково обрабатывать отдельные элементы и группы.

---

# 💡 Идея

Определить общий интерфейс компонента,
а листья и контейнеры реализуют его одинаково для клиента.

---

# 🏗 Структура

```
Client
  ↓
Component interface
  ├─ Leaf
  └─ Composite (children: []Component)
```

---

# 💻 Реализация на Go

```go
package main

import "fmt"

type Node interface {
	Print(prefix string)
}

type File struct {
	name string
}

func (f File) Print(prefix string) {
	fmt.Println(prefix + "- " + f.name)
}

type Folder struct {
	name     string
	children []Node
}

func (d *Folder) Add(node Node) {
	d.children = append(d.children, node)
}

func (d Folder) Print(prefix string) {
	fmt.Println(prefix + "+ " + d.name)
	for _, child := range d.children {
		child.Print(prefix + "  ")
	}
}

func main() {
	root := &Folder{name: "root"}
	root.Add(File{name: "readme.md"})

	src := &Folder{name: "src"}
	src.Add(File{name: "main.go"})
	root.Add(src)

	root.Print("")
}
```

---

# 📌 Когда использовать

Когда:

* данные имеют иерархию дерево
* нужно единообразно работать с элементами и группами
* рекурсивная обработка естественна

---

# 🚫 Когда не использовать

Не нужен если:

* структура плоская
* деревья не используются в доменной модели

---

# 🏭 Реальный пример

Файловая система: файл (leaf) и директория (composite).

---

# 🧠 Задание

Реализуй Composite для:

```
Organization
```

элементы:

```
Employee
Department
```

общий метод:

```
GetSalary() int
```
