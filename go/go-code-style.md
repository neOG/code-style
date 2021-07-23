## 代码风格

&nbsp;

### 语句分组

&nbsp;

#### `import` 导入

```go
// No
import "a"
import "b"

// Yes
import (
    "a"
    "b"
)
```

&nbsp;

#### 常量、变量等声明

```go
// No
const a = 1
const b = 2

var a = 1
var b = 2

type Area float64
type Volume float64

// Yes
const (
    a = 1
    b = 2
)

var (
    a = 1
    b = 2
)

type (
    Area   float64
    Volume float64
)
```

&nbsp;

#### `import` 顺序分组

```go
// Yes but no enough.
import (
    "fmt"
    "os"
    "go.uber.org/atomic"
    "golang.org/x/sync/errgroup"
)

// Good
import (
    "fmt"
    "os"

    "go.uber.org/atomic"
    "golang.org/x/sync/errgroup"
)
```

&nbsp;

&nbsp;

### 命名

- **做有意义的区分：** Product 和 ProductInfo 和 ProductData 没有区别，NameString和Name没有区别，要区分名称，就要以读者能鉴别不同之处的方式来区分 。
- **函数命名规则：** 驼峰式命名，名字可以长但是得把功能，必要的参数描述清楚，函数名应当是动词或动词短语，如 postPayment、deletePage、save 等。
- **结构体命名规则：** 结构体名应该是名词或名词短语，如 Custome、WikiPage、Account、AddressParser，结构体名不应当是动词。
- **包名命名规则：** 包名应该为小写单词，不要混合大小写，尽量不要使用下划线。
- **接口命名规则：** 单个函数的接口名以 "er" 作为后缀，如 Reader、Writer。接口的实现则去掉 “er”。

&nbsp;

#### `import` 别名

当 package 的名字和 import 的 path 的最后一个元素不同的时候，必须要起别名。

```go
// Yes
import (
    "net/http"

    client "example.com/client-go"
    trace "example.com/trace/v2"
)
```

&nbsp;

import 别名要尽量避免，只有在不得不起别名的时候再这么做，比如避免冲突。

```go
// No
import (
    "fmt"
    "os"

    nettrace "golang.net/x/trace"
)

// Yes
import (
    "fmt"
    "os"
    "runtime/trace"

    nettrace "golang.net/x/trace"
)
```

&nbsp;

&nbsp;

### 错误处理

- error 作为函数的值返回，必须对 error 进行处理。
- 错误描述如果是英文必须为小写，不需要标点结尾。

&nbsp;

```go
// No
if err != nil {
// error handling
} else {
// normal code
}


// Yes
if err != nil {
// error handling
    return // or continue, etc.
}
// normal code
```

&nbsp;

#### 异常优先处理

优先处理异常情况，快速返回，避免代码块过多嵌套。

```go
// No
for _, v := range data {
    if v.F1 == 1 {
        v = process(v)
        if err := v.Call(); err == nil {
            v.Send()
        } else {
            return err
        }
    } else {
        log.Printf("Invalid v: %v", v)
    }
}


// Yes
for _, v := range data {
    if v.F1 != 1 {
        log.Printf("Invalid v: %v", v)
        continue
    }

    v = process(v)
    if err := v.Call(); err != nil {
        return err
    }
    v.Send()
}

```

&nbsp;

&nbsp;

### 避免多余 `if-else`

很多情况下，if - else 语句都能通过一个 if 语句表达。

```go
// No
var a int
if b {
    a = 100
} else {
    a = 10
}


// Yes
a := 10
if b {
    a = 100
}
```

&nbsp;

&nbsp;

### 两级 (two-level) 变量声明

所有两级变量声明就是一个声明的右值来自另一个表达式，这个时候第一级变量声明就不需要指明类型，除非这两个地方的数据类型不同。

```go
// No
var _s string = F()

func F() string { return "A" }


// Yes
var _s = F()
// Since F already states that it returns a string, we don't need to specify
// the type again.

func F() string { return "A" }
```

&nbsp;

上面说的第二种两边数据类型不同的情况。

```go
type myError struct{}

func (myError) Error() string { return "error" }

func F() myError { return myError{} }

var _e error = F()
// F returns an object of type myError but we want error.
```

&nbsp;

&nbsp;

### struct 嵌套

struct 中的嵌套类型在 field 列表排在最前面，并且用空行分隔开。

```go
package main

// No
type Client struct {
    version int
    http.Client
}

// Yes
type Client struct {
    http.Client

    version int
}
```

&nbsp;

&nbsp;

### struct 初始化

struct 初始化的时候带上 field。

```go
// No
k := User{"John", "Doe", true}


// Yes
k := User{
    FirstName: "John",
    LastName: "Doe",
    Admin: true,
}
```

&nbsp;

&nbsp;

### 使用原生字符串，避免转义

Go 支持使用反引号，也就是 “`” 来表示原生字符串，在需要转义的场景下，我们应该尽量使用这种方案来替换。

```go
// No
wantError := "unknown name:\"test\""


// Yes
wantError := `unknown error:"test"`
```

&nbsp;

&nbsp;

### struct 引用初始化

使用 `&T{}` 而不是 `new(T)` 来声明对 T 类型的引用，使用 `&T{}` 的方式我们可以和 struct 声明方式 `T{}` 保持统一。

```go
// No
sval := T{Name: "foo"}

// inconsistent
sptr := new(T)
sptr.Name = "bar"


// Yes
sval := T{Name: "foo"}

sptr := &T{Name: "bar"}
```

&nbsp;

&nbsp;

### 字符串 string format

如果我们要在 Printf 外面声明 format 字符串的话，使用 const，而不是变量，这样 go vet 可以对 format 字符串做静态分析。

```go
// No
msg := "unexpected values %v, %v\n"
fmt.Printf(msg, 1, 2)


// Yes
const msg = "unexpected values %v, %v\n"
fmt.Printf(msg, 1, 2)
```

&nbsp;

&nbsp;

### 避免参数语义不明确

Naked Parameter 指的应该是意义不明确的参数，这种情况会破坏代码的可读性，可以使用 C 分格的注释（`/*...*/`）进行注释。

```go
// No
// func printInfo(name string, isLocal, done bool)

printInfo("foo", true, true)


// Yes
// func printInfo(name string, isLocal, done bool)

printInfo("foo", true /* isLocal */, true /* done */)


// Better
type Region int

const (
    UnknownRegion Region = iota
    Local
)

type Status int

const (
    StatusReady = iota + 1
    StatusDone
  // Maybe we will have a StatusInProgress in the future.
)

func printInfo(name string, region Region, status Status)
```

&nbsp;

&nbsp;

### 避免 scope

```go
// No
err := ioutil.WriteFile(name, data, 0644)
if err != nil {
    return err
}

// Yes
if err := ioutil.WriteFile(name, data, 0644); err != nil {
    return err
}
```

在某些情况下，scope 是不可避免的，比如：

```go
// No
if data, err := ioutil.ReadFile(name); err == nil {
err = cfg.Decode(data)
if err != nil {
    return err
}

fmt.Println(cfg)
    return nil
} else {
    return err
}


// Yes
data, err := ioutil.ReadFile(name)
if err != nil {
    return err
}

if err := cfg.Decode(data); err != nil {
    return err
}

fmt.Println(cfg)
return nil
```

&nbsp;

&nbsp;



