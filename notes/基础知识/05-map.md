# 05 - map

## 本节目标

这一节先掌握两个最核心的问题：

1. Go 的 `map` 是什么
2. Go 里读取、写入、判断 key 是否存在，分别怎么写

如果你之前写过 Python、JavaScript、Java，这一节最重要的心智转换是：

**Go 的 map 很常用，但它不是“什么都能随便放”的动态对象，而是有明确 key 类型和 value 类型的哈希表。**

---

## 1. map 是什么

先看最基本的写法：

```go
scores := map[string]int{
	"Alice": 95,
	"Bob":   88,
}
```

这里的：

```go
map[string]int
```

意思是：

- key 类型是 `string`
- value 类型是 `int`

所以这个 map 可以理解为：

- 用字符串做键
- 用整数做值

### 和其他语言的对比

#### Python

```python
scores = {
    "Alice": 95,
    "Bob": 88,
}
```

#### JavaScript

```javascript
const scores = {
  Alice: 95,
  Bob: 88,
}
```

#### Java

你可能会写：

```java
Map<String, Integer> scores = new HashMap<>();
```

### Go 的特点

Go 的 `map` 和 Java 的泛型 map 更像，而不是 JavaScript 那种“动态对象”。

也就是说，类型一开始就很明确：

- `map[string]int`
- `map[string]string`
- `map[int]bool`

它不是“随时塞不同类型值”的容器。

---

## 2. 读取 map

先看例子：

```go
package main

import "fmt"

func main() {
	scores := map[string]int{
		"Alice": 95,
		"Bob":   88,
	}

	fmt.Println(scores["Alice"])
}
```

读取方式和很多语言类似：

```go
scores["Alice"]
```

这会拿到 key 为 `"Alice"` 的值。

---

## 3. 写入和修改 map

写入和修改也都用下标语法：

```go
package main

import "fmt"

func main() {
	scores := map[string]int{}
	scores["Alice"] = 95
	scores["Bob"] = 88
	scores["Bob"] = 90

	fmt.Println(scores)
}
```

这里：

- 如果 key 原来不存在，就是新增
- 如果 key 已存在，就是覆盖

这点和大多数语言很像。

---

## 4. 删除 key：`delete`

Go 删除 map 元素不用方法调用，而是内置函数：

```go
delete(scores, "Bob")
```

完整例子：

```go
package main

import "fmt"

func main() {
	scores := map[string]int{
		"Alice": 95,
		"Bob":   88,
	}

	delete(scores, "Bob")
	fmt.Println(scores)
}
```

### 和其他语言的对比

- Python：`del scores["Bob"]`
- JavaScript：`delete scores.Bob`
- Java：`map.remove("Bob")`

Go 的风格是：

- 不搞对象方法式 API
- 直接用内置函数 `delete`

这很符合 Go 的整体风格：简单、直接。

---

## 5. 一个关键点：读取不存在的 key，不会报错

这是 Go map 初学者非常容易踩坑的地方。

看例子：

```go
package main

import "fmt"

func main() {
	scores := map[string]int{
		"Alice": 95,
	}

	fmt.Println(scores["Bob"])
}
```

这里不会抛异常，也不会报错。

输出是：

```go
0
```

为什么？

因为当 key 不存在时，Go 会返回 **value 类型的零值**。

比如：

- `map[string]int` → 不存在时返回 `0`
- `map[string]string` → 不存在时返回 `""`
- `map[string]bool` → 不存在时返回 `false`

### 这意味着什么

这意味着你不能只靠返回值判断 key 是否存在。

例如：

```go
v := scores["Bob"]
```

如果 `v == 0`，有两种可能：

1. `Bob` 不存在
2. `Bob` 存在，但值刚好就是 `0`

所以仅看 `v` 不够。

---

## 6. 判断 key 是否存在：`value, ok := map[key]`

这是 Go map 最重要的惯用法之一。

```go
package main

import "fmt"

func main() {
	scores := map[string]int{
		"Alice": 95,
	}

	value, ok := scores["Bob"]
	fmt.Println(value)
	fmt.Println(ok)
}
```

这里：

- `value` 是取到的值
- `ok` 是一个 `bool`
- 如果 key 存在，`ok == true`
- 如果 key 不存在，`ok == false`

这和你前面学过的 Go 风格是一致的：

**用额外返回值显式告诉你“有没有成功拿到”。**

### 推荐写法

```go
if value, ok := scores["Bob"]; ok {
	fmt.Println("找到：", value)
} else {
	fmt.Println("没找到")
}
```

这个写法以后你会经常见到。

---

## 7. 用 `make` 创建 map

除了字面量，你也会经常看到：

```go
scores := make(map[string]int)
```

这表示：

- 创建一个可用的 map
- key 是 `string`
- value 是 `int`

然后你就可以继续写：

```go
scores["Alice"] = 95
```

### 和切片的 `make` 类似吗？

形式上类似，但你当前先记住使用场景：

- 切片常用 `make([]T, len, cap)`
- map 常用 `make(map[K]V)`

---

## 8. 零值 map：声明了但没初始化，不能直接写入

这是另一个非常常见的坑。

看例子：

```go
package main

func main() {
	var scores map[string]int
	scores["Alice"] = 95
}
```

这会在运行时报错。

为什么？

因为：

```go
var scores map[string]int
```

只是声明了一个 map 变量，但还没有真正初始化底层结构。

你可以把它理解成：

- 变量有了
- 但 map 还没准备好

所以 map 通常要么：

### 方式 1：字面量初始化

```go
scores := map[string]int{}
```

### 方式 2：用 `make`

```go
scores := make(map[string]int)
```

这两种都可以。

---

## 9. 遍历 map：`for range`

看例子：

```go
package main

import "fmt"

func main() {
	scores := map[string]int{
		"Alice": 95,
		"Bob":   88,
	}

	for name, score := range scores {
		fmt.Println(name, score)
	}
}
```

这表示：

- 每次取出一组 key/value
- 用 `range` 遍历整个 map

### 一个重要提醒

**map 的遍历顺序不保证固定。**

也就是说，同一段程序多次运行时，输出顺序可能不同。

如果你之前习惯了某些语言里“看起来总是按插入顺序”，在 Go 里不要依赖这个行为。

如果你需要固定顺序，通常做法是：

1. 先把 key 取出来放进切片
2. 排序
3. 再按排序后的 key 访问 map

当前阶段你先记住：

**Go map 遍历顺序不要当成稳定的。**

---

## 10. key 类型必须是可比较的

你现在先只记最实用的一层：

这些常见类型可以做 key：

- `string`
- `int`
- `bool`

而切片不能直接做 key。

例如：

```go
map[[]int]string
```

这是不合法的。

因为切片不能用 `==` 做整体比较，而 map key 必须可比较。

当前阶段不用展开太多规则，先记住：

**string 和整数常用作 key，切片不能直接作 key。**

---

## 11. 可运行示例

你可以在当前目录新建一个文件，比如 `map_demo.go`：

```go
package main

import "fmt"

func main() {
	scores := make(map[string]int)
	scores["Alice"] = 95
	scores["Bob"] = 88

	if value, ok := scores["Alice"]; ok {
		fmt.Println("Alice:", value)
	}

	delete(scores, "Bob")
	fmt.Println(scores)
}
```

你自己运行：

```bash
go run map_demo.go
```

建议你重点观察：

1. `make(map[string]int)` 的写法
2. `value, ok := scores[key]` 的存在性判断
3. `delete` 的用法

---

## 练习题

目标：

练习 map 的创建、读取、存在性判断、删除和遍历。

要求：

1. 创建一个 `map[string]int`
2. 写入两个学生成绩
3. 用 `value, ok := m[key]` 判断某个 key 是否存在
4. 删除一个 key
5. 打印 map

### 参考答案代码

```go
package main

import "fmt"

func main() {
	scores := make(map[string]int)
	scores["Alice"] = 95
	scores["Bob"] = 88

	if value, ok := scores["Alice"]; ok {
		fmt.Println("Alice:", value)
	}

	delete(scores, "Bob")

	for name, score := range scores {
		fmt.Println(name, score)
	}
}
```

### 预期运行结果

```text
Alice: 95
Alice 95
```

说明：遍历 map 的顺序不保证固定；这个例子删除了 `Bob`，因此最终只会剩下 `Alice`。

---

## 本节小结

这一节你先记住 5 句话：

1. **`map[K]V` 是 Go 的键值对容器**
2. **读取不存在的 key，会返回 value 类型的零值**
3. **判断 key 是否存在，要用 `value, ok := m[key]`**
4. **写 map 前要先初始化，常用字面量或 `make`**
5. **map 遍历顺序不保证固定**

你现在学到的位置是：**已经掌握了 Go 三个常见集合类型中的前两个半：数组、切片、map**。

下一步最自然的内容是：**切片和 map 的组合使用，以及什么时候该用数组 / 切片 / map**。
