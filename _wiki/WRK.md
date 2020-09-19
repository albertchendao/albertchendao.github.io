---
layout: article
title: WRK
tags: []
key: 83c1b7eb-5b67-4656-9bb2-eaca386cc3e1
---

WRK 用了压测软件性能.

<!--more-->

### 安装

Linux 上: 

```bash
git clone https://github.com/wg/wrk
cd wrk
# 会在 wrk 目录下生成 wrk 可执行文件
make
```

MAC 上:

```bash
brew install wrk
```

### 参数

```bash
-c:总的连接数（每个线程处理的连接数=总连接数/线程数）
-d:测试的持续时间,如2s(2second),2m(2minute),2h(hour)
-t:需要执行的线程总数
-s:执行Lua脚本,这里写lua脚本的路径和名称,后面会给出案例
-H:需要添加的头信息,注意header的语法,举例,-H “token: abcdef”,说明一下,token,冒号,空格,abcdefg（不要忘记空格,否则会报错的）.
—timeout:超时的时间
—latency:显示延迟统计信息
```

### 返回结果

```bash
Latency:响应时间
Req/Sec:每个线程每秒钟的执行的连接数
Avg:平均
Max:最大
Stdev:标准差
+/- Stdev: 正负一个标准差占比
Requests/sec:每秒请求数（也就是QPS）,这是一项压力测试的性能指标,通过这个参数可以看出吞吐量
Latency Distribution,如果命名中添加了—latency就会出现相关信息
```

### LUA 脚本

wrk 提供的几个 lua 的 hook 函数: 

#### setup

这个函数在目标 IP 地址已经解析完, 并且所有 thread 已经生成, 但是还没有开始时被调用. 每个线程执行一次这个函数. 
可以通过thread:get(name),  thread:set(name, value)设置线程级别的变量. 

#### init

每次请求发送之前被调用. 
可以接受 wrk 命令行的额外参数. 通过 -- 指定. 

#### delay

这个函数返回一个数值, 在这次请求执行完以后延迟多长时间执行下一个请求. 可以对应 thinking time 的场景. 

#### request

通过这个函数可以每次请求之前修改本次请求的属性. 返回一个字符串. 这个函数要慎用, 会影响测试端性能. 

#### response

每次请求返回以后被调用. 可以根据响应内容做特殊处理, 比如遇到特殊响应停止执行测试, 或输出到控制台等等

示例:

```lua
function response(status, headers, body)  
if status ~= 200 then  
      print(body)  
      wrk.thread:stop()  
   end  
end
```

#### done

在所有请求执行完以后调用, 一般用于自定义统计结果.

```lua
done = function(summary, latency, requests)  
   io.write("------------------------------\n")  
for _, p in pairs({ 50, 90, 99, 99.999 }) do  
      n = latency:percentile(p)  
      io.write(string.format("%g%%,%d\n", p, n))  
   end  
end
```

#### 完整示例

```lua
local counter = 1  
local threads = {}  

function setup(thread)  
   thread:set("id", counter)  
   table.insert(threads, thread)  
   counter = counter + 1  
end  

function init(args)  
   requests  = 0  
   responses = 0  
   local msg = "thread %d created"  
   print(msg:format(id))  
end  

function request()  
   requests = requests + 1  
return wrk.request()  
end  
function response(status, headers, body)  
   responses = responses + 1  
end  

function done(summary, latency, requests)  
for index, thread in ipairs(threads) do  
      local id        = thread:get("id")  
      local requests  = thread:get("requests")  
      local responses = thread:get("responses")  
      local msg = "thread %d made %d requests and got %d responses"  
      print(msg:format(id, requests, responses))  
   end  
end
```

自定义 URL:

```lua
counter = 1  
math.randomseed(os.time())  
math.random(); math.random(); math.random()  
function file_exists(file)  
  local f = io.open(file, "rb")  
if f then f:close() end  
return f ~= nil  
end  

function shuffle(paths)  
  local j, k  
  local n = #paths  
for i = 1, n do  
    j, k = math.random(n), math.random(n)  
    paths[j], paths[k] = paths[k], paths[j]  
  end  
return paths  
end  

function non_empty_lines_from(file)  
if not file_exists(file) then return {} end  
  lines = {}  
for line in io.lines(file) do  
if not (line == '') then  
      lines[#lines + 1] = line  
    end  
  end  
return shuffle(lines)  
end  
paths = non_empty_lines_from("paths.txt")  
if #paths <= 0 then  
  print("multiplepaths: No paths found. You have to create a file paths.txt with one path per line")  
  os.exit()  
end  
print("multiplepaths: Found " .. #paths .. " paths")  
request = function()  
    path = paths[counter]  
    counter = counter + 1  
if counter > #paths then  
      counter = 1  
    end  
return wrk.format(nil, path)  
end
```

使用 Cookie:

```lua
function getCookie(cookies, name)  
  local start = string.find(cookies, name .. "=")  
if start == nil then  
return nil  
  end  
return string.sub(cookies, start + #name + 1, string.find(cookies, ";", start) - 1)  
end  
response = function(status, headers, body)  
  local token = getCookie(headers["Set-Cookie"], "token")  
if token ~= nil then  
    wrk.headers["Cookie"] = "token=" .. token  
  end  
end
```

### 实际使用

```bash
# 使用8个线程200个连接,对bing首页进行了30秒的压测,并要求在压测结果中输出响应延迟信息
wrk -t8 -c200 -d30s --latency  "http://www.bing.com"
```

结果:

```bash
Running 30s test @ http://www.bing.com （压测时间30s）
  8 threads and 200 connections （共8个测试线程,200个连接）
  Thread Stats   Avg      Stdev     Max   +/- Stdev
              （平均值） （标准差）（最大值）（正负一个标准差所占比例）
    Latency    46.67ms  215.38ms   1.67s    95.59%
    （延迟）
    Req/Sec     7.91k     1.15k   10.26k    70.77%
    （处理中的请求数）
  Latency Distribution （延迟分布）
     50%    2.93ms
     75%    3.78ms
     90%    4.73ms
     99%    1.35s （99分位的延迟）
  1790465 requests in 30.01s, 684.08MB read （30.01秒内共处理完成了1790465个请求,读取了684.08MB数据）
Requests/sec:  59658.29 （平均每秒处理完成59658.29个请求）
Transfer/sec:     22.79MB （平均每秒读取数据22.79MB）
```
