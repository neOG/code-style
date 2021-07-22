## 代码风格

&nbsp;

### 缩进

使用 4 个空格作为缩进的标记，虽然 Lua 并没有这样的语法要求（保持和 APISIX 风格一致）。

```lua
-- No
if a then
ngx.say("hello")
end

-- Yes
if a then
    ngx.say("hello")
end
```

建议在使用的编辑器中，把 tab 改为 4 个空格，来简化操作。

&nbsp;

### 空格

在操作符的两边，都需要用一个空格来做分隔。

```lua
-- No
local i=1
local s    =    "apisix"

-- Yes
local i = 1
local s = "apisix"
```

&nbsp;

### 空行

不要在行尾增加一个分号。

```lua
-- No
if a then
    ngx.say("hello");
end;

-- Yes
if a then
    ngx.say("hello")
end
```

&nbsp;

不要为了节省代码的行数，为了显得“简洁”，而把多行代码变为一行。这样会在定位错误的时候不知道到底哪一段代码出了问题。

```lua
-- No
if a then ngx.say("hello") end

-- Yes
if a then
    ngx.say("hello")
end
```

&nbsp;

函数之间需要用两个空行来做分隔。

```lua
-- No
local function foo()
end
local function bar()
end

-- Yes
local function foo()
end


local function bar()
end
```

&nbsp;

如果有多个 `if elseif` 的分支，它们之间需要一个空行来做分隔。

```lua
-- No
if a == 1 then
    foo()
elseif a== 2 then
    bar()
elseif a == 3 then
    run()
else
    error()
end

-- Yes
if a == 1 then
    foo()

elseif a== 2 then
    bar()

elseif a == 3 then
    run()

else
    error()
end
```

&nbsp;

### 注释

单行注释，注意**空格**，使用英文注释时，开头字母请大写，结尾处使用正确标点符号，单行注释长度不宜过长，如果注释内容很长，请分行注释。如果注释仅仅只有单个词汇请不要添加标点符号。

```lua
-- 这是单行注释。

-- This is a single line comment.
```

&nbsp;

### 每行最大长度

每行不能超过 80 个字符，超过的话，需要换行并对齐。

```lua
-- No
return limit_conn_new("plugin-limit-conn", conf.conn, conf.burst, conf.default_conn_delay)

-- Yes
return limit_conn_new("plugin-limit-conn", conf.conn, conf.burst,
                      conf.default_conn_delay)
```

&nbsp;

在换行对齐的时候，要体现出上下两行的对应关系。就上面的示例而言，第二行函数的参数，要在第一行左括号的右边。

如果是字符串拼接的对齐，需要把 .. 放到下一行中。

```lua
-- No
return limit_conn_new("plugin-limit-conn" ..  "plugin-limit-conn" ..
                      "plugin-limit-conn")
-- Yes
return limit_conn_new("plugin-limit-conn" .. "plugin-limit-conn"
                      .. "plugin-limit-conn")
```

&nbsp;

## 代码规范

&nbsp;

### 变量

应该永远使用局部变量，不要使用全局变量。

```lua
-- No
i = 1
s = "apisix"

-- Yes
local i = 1
local s = "apisix"
```

&nbsp;

变量命名使用 `snake_case` 风格。

```lua
-- No
local IndexArr = 1
local str_Name = "apisix"

-- Yes
local index_arr = 1
local str_name = "apisix"
```

&nbsp;

对于常量要使用全部大写。

```lua
-- No
local max_int = 65535
local server_name = "huapi"

-- Yes
local MAX_INT = 65535
local SERVER_NAME = "huapi"
```

&nbsp;

### 字符串

不要在热代码路径上拼接字符串。

```lua
-- No
local s = ""
for i = 1, 100000 do
	s = s .. "a"
end

-- Yes
local t = {}
for i = 1, 100000 do
	t[i] = "a"
end
local s =  table.concat(t, "")
```

&nbsp;

### 数组

使用 `table.new` 来预先分配数组。

```lua
-- No
local t = {}
for i = 1, 100 do
    t[i] = i
end

-- Yes
local new_tab = require "table.new"
local t = new_tab(100, 0)
for i = 1, 100 do
    t[i] = i
end
```

&nbsp;

不要在数组中使用 `nil`。

```lua
-- No
local t = {1, 2, nil, 3}
```

&nbsp;

如果一定要使用空值，请用 `ngx.null` 来表示。

```lua
-- No
local t = {1, 2, ngx.null, 3}
```

&nbsp;

### 函数

函数的命名也同样遵循 `snake_case`。

```lua
-- No
local function testNginx()
end

-- Yes
local function test_nginx()
end
```

&nbsp;

函数应该尽可能早的返回。

```lua
-- No
local function check(age, name)
    local ret = true
    if age < 20 then
        ret = false
    end

    if name == "a" then
        ret = false
    end
    -- do something else
    return ret
end

-- Yes
local function check(age, name)
    if age < 20 then
        return false
    end

    if name == "a" then
        return false
    end
    -- do something else
    return true
end
```

&nbsp;

### 模块

所有 `require` 的库都要 `local` 化。

```lua
-- No
local function foo()
    local ok, err = ngx.timer.at(delay, handler)
end

-- Yes
local timer_at = ngx.timer.at

local function foo()
    local ok, err = timer_at(delay, handler)
end
```

&nbsp;

为了风格的统一，`require` 和 `ngx` 也需要 `local` 化。

```lua
-- No
local core = require("apisix.core")
local timer_at = ngx.timer.at

local function foo()
    local ok, err = timer_at(delay, handler)
end

-- Yes
local ngx = ngx
local require = require
local core = require("apisix.core")
local timer_at = ngx.timer.at

local function foo()
    local ok, err = timer_at(delay, handler)
end
```

&nbsp;

### 错误处理

对于有错误信息返回的函数，必须对错误信息进行判断和处理。

```lua
-- No
local sock = ngx.socket.tcp()
local ok = sock:connect("www.google.com", 80)
ngx.say("successfully connected to google!")

-- Yes
local sock = ngx.socket.tcp()
local ok, err = sock:connect("www.google.com", 80)
if not ok then
    ngx.say("failed to connect to google: ", err)
    return
end
ngx.say("successfully connected to google!")
```

&nbsp;

自己编写的函数，错误信息要作为第二个参数，用字符串的格式返回。

```lua
-- No
local function foo()
    local ok, err = func()
    if not ok then
        return false
    end
    return true
end

-- No
local function foo()
    local ok, err = func()
    if not ok then
        return false, {msg = err}
    end
    return true
end

-- Yes
local function foo()
    local ok, err = func()
    if not ok then
        return false, "failed to call func(): " .. err
    end
    return true
end
```

&nbsp;

## 代码建议

- **代码尽可能简单**，考虑性能时先做好推断，综合考虑提升的性能和增加的复杂度程度，然后再决定如何做。
- **高级特性尽可能不用**
- **数据校验：** 加载的 `xml`、`json` `schema` 时，需做好数据校验，若校验失败，要立即处理，使服务器无法启动或配置失败。不要等出错时，回过头来检查是数据格式问题还是逻辑问题。
- **日志：** 重视每个产生 `err` 的函数， 必须对错误信息进行判断和处理，并记录好错误日志。

