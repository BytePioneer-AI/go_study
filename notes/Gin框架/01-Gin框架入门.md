# 01 - Gin 框架入门

## 本节目标

这一节先掌握 2 个核心问题：

1. Gin 到底是什么，它和标准库 `net/http` 是什么关系
2. 怎么写一个最小可运行的 Gin 服务

如果你之前写过 Spring Boot、Express、FastAPI、Flask，这一节最重要的心智转换是：

**Gin 不是一个“超大一体化框架”，它更像是构建在 `net/http` 之上的轻量 Web 框架。**

也就是说：
- Go 标准库本身就能写 HTTP 服务
- Gin 是在标准库之上提供了更顺手的路由、上下文、JSON 处理等能力
- 你学 Gin，最好一直知道它底下还是 HTTP Handler 这套模型

---

## 1. Gin 是什么

Gin 是 Go 里非常常见的 Web 框架，常用来开发：

- REST API
- 后端服务
- Web 应用接口层

它受欢迎的原因主要有三个：

1. 上手快
2. 路由和请求处理写法简洁
3. 和 Go 标准库生态兼容

### 和其他语言框架对比

#### 在 Java / Spring Boot 里

你可能习惯的是：
- 注解定义路由
- 容器帮你管理大量组件
- 框架能力很多，很完整

#### 在 Node.js / Express 里

你可能更熟悉：
- `app.get(...)`
- `req`, `res`
- 中间件链

#### 在 Gin 里

你会看到很像 Express 的地方：

```go
r := gin.Default()
r.GET("/ping", func(c *gin.Context) {
	c.JSON(200, gin.H{"message": "pong"})
})
r.Run(":8080")
```

但它背后的风格仍然是 Go：
- 显式处理请求
- 类型清晰
- 组合优于魔法配置

所以你可以先把 Gin 理解成：

**一个让你更方便写 HTTP 服务的 Go 框架，而不是替代 Go 本身。**

---

## 2. Gin 和 `net/http` 的关系

这点很重要。

很多初学者会误以为：
- 学了 Gin 就不用懂标准库 HTTP 了
- Gin 是一套完全独立的系统

其实不是。

Gin 底层还是建立在 Go 标准库 `net/http` 之上。

### 标准库写法

先看标准库最小服务：

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	http.HandleFunc("/ping", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintln(w, "pong")
	})

	http.ListenAndServe(":8080", nil)
}
```

### Gin 写法

```go
package main

import "github.com/gin-gonic/gin"

func main() {
	r := gin.Default()

	r.GET("/ping", func(c *gin.Context) {
		c.JSON(200, gin.H{"message": "pong"})
	})

	r.Run(":8080")
}
```

### 你要抓住的差异

和标准库相比，Gin 主要帮你做了这些事：

1. 更方便的路由注册
2. 把请求和响应封装进 `Context`
3. 更方便返回 JSON
4. 内置常用中间件（日志、异常恢复）

所以学习路径最好是：

**把 Gin 看成“更好用的 HTTP 开发工具”，不是“黑盒框架”。**

---

## 3. 最小可运行 Gin 服务

下面是你这一节最重要的示例。

```go
package main

import "github.com/gin-gonic/gin"

func main() {
	r := gin.Default()

	r.GET("/ping", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "pong",
		})
	})

	if err := r.Run(":8080"); err != nil {
		panic(err)
	}
}
```

---

## 4. 逐行理解这个例子

### `r := gin.Default()`

这一步相当于创建一个 Gin 路由引擎。

它不是简单 new 一个对象而已，还顺手帮你挂了两个常用中间件：

- Logger：打印请求日志
- Recovery：程序 panic 时恢复，避免服务直接崩掉

如果你之前写过 Express，可以把它类比成：
- 创建一个 app
- 顺便默认装了一些常用中间件

但 Go 里更强调显式对象和方法调用，不靠注解扫描。

### `r.GET("/ping", ...)`

这表示注册一个 GET 路由。

含义是：
- 当客户端用 GET 请求访问 `/ping`
- 就执行这个处理函数

这里你会先接触到 `c *gin.Context`。

你现在先把它理解成：

**Gin 把“这次请求相关的信息”和“怎么返回响应”的能力，都放到了 `Context` 里。**

后面学参数、JSON 请求体、中间件时，你会一直用到它。

### `c.JSON(200, gin.H{"message": "pong"})`

这一行表示返回 JSON 响应。

- `200` 是 HTTP 状态码
- `gin.H` 本质上是 `map[string]any`
- Gin 会帮你把它序列化成 JSON

返回给客户端的内容大致是：

```json
{"message":"pong"}
```

### `r.Run(":8080")`

表示启动 HTTP 服务并监听 8080 端口。

注意这个写法：

- `":8080"` 不是“访问地址”
- 是“监听本机 8080 端口”

也就是说，服务启动后，你通常会用浏览器或 curl 访问：

```text
http://localhost:8080/ping
```

### 为什么这里要处理 error

`r.Run(...)` 会返回 `error`。

这又回到了你前面学过的 Go 风格：

**会失败的操作，Go 通常返回 error，而不是抛异常。**

比如：
- 端口被占用
- 权限不足
- 启动失败

所以 Go 惯用法是检查它：

```go
if err := r.Run(":8080"); err != nil {
	panic(err)
}
```

---

## 5. 怎么安装 Gin

在 Go 里，依赖管理仍然是你前面学过的 `go.mod` 那套。

如果你要在当前模块里使用 Gin，常见做法是先写 import，然后整理依赖。

例如代码里有：

```go
import "github.com/gin-gonic/gin"
```

然后执行：

```bash
go mod tidy
```

Go 会自动把 Gin 加到 `go.mod` / `go.sum`。

### 和其他语言对比

#### Node.js

你可能先执行：

```bash
npm install express
```

#### Python

你可能先执行：

```bash
pip install fastapi
```

#### Go

Go 更常见的是：
- 先写 `import`
- 再用工具链整理依赖

这再次体现了 Go 的风格：

**依赖管理更多交给工具，而不是手动维护。**

---

## 6. Gin 里几个你先记住的名字

这一节不展开，只先建立最小认知：

### `gin.Engine`

也就是你这里的 `r`。

它负责：
- 注册路由
- 挂中间件
- 启动服务

### `gin.Context`

也就是处理函数里的 `c`。

它负责：
- 读取请求参数
- 读取请求体
- 设置响应状态码
- 返回 JSON / 字符串 / 文件等

### `gin.H`

只是一个简写：

```go
type H map[string]any
```

它的作用就是让你临时返回 JSON 时更方便。

但正式项目里，更常见的方式通常是返回结构体，这个后面会学。

---

## 7. 这一节你真正要建立的心智模型

先不要把 Gin 学成“背 API”。

你现在最应该抓住的是这 3 句话：

1. **Gin 是构建在 `net/http` 之上的轻量框架**
2. **Gin 的核心入口通常是 `Engine` 和 `Context`**
3. **写 Gin 服务，本质上还是在写 HTTP 请求到响应的处理过程**

如果这 3 句你建立起来了，后面学路由、参数、中间件就会顺很多。

---

## 练习题

### 练习 1：最小 Gin 服务

请你自己创建一个新文件，比如 `gin_hello.go`，写一个最小 Gin 服务：

要求：
- 监听 `:8080`
- 注册一个 `GET /hello` 路由
- 返回 JSON：

```json
{"message":"hello gin"}
```

### 参考答案

```go
package main

import "github.com/gin-gonic/gin"

func main() {
	r := gin.Default()

	r.GET("/hello", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "hello gin",
		})
	})

	if err := r.Run(":8080"); err != nil {
		panic(err)
	}
}
```

### 预期运行结果

启动后访问：

```text
http://localhost:8080/hello
```

返回：

```json
{"message":"hello gin"}
```

---

### 练习 2：再加一个路由

在练习 1 基础上，再增加一个路由：

- `GET /ping`
- 返回：

```json
{"message":"pong"}
```

### 参考答案

```go
package main

import "github.com/gin-gonic/gin"

func main() {
	r := gin.Default()

	r.GET("/hello", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "hello gin",
		})
	})

	r.GET("/ping", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "pong",
		})
	})

	if err := r.Run(":8080"); err != nil {
		panic(err)
	}
}
```

### 预期运行结果

访问：

```text
http://localhost:8080/hello
```

返回：

```json
{"message":"hello gin"}
```

访问：

```text
http://localhost:8080/ping
```

返回：

```json
{"message":"pong"}
```

---

## 建议你现在自己做的事

你先不要让我替你运行。

按你的学习规则，建议你自己在当前目录做这几步：

```bash
go get github.com/gin-gonic/gin@latest
go mod tidy
go run gin_hello.go
```

然后浏览器访问：

```text
http://localhost:8080/hello
http://localhost:8080/ping
```

如果你愿意更贴近 Go 的依赖管理习惯，也可以先写 `import`，再直接：

```bash
go mod tidy
go run gin_hello.go
```

---

## 本节小结

你现在学到的是：
- Gin 是什么
- 它和 `net/http` 的关系
- 怎么写一个最小可运行 Gin 服务

下一步我们会进入：

**Gin 路由与请求处理** —— 路径参数、Query 参数、接收请求数据。
