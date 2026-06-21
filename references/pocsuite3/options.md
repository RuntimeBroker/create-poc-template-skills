# Pocsuite3 参数系统

> `_options()` 自定义参数 + 框架自动注入选项 + get/set 方法。

## 自定义参数：`_options()`

返回 `OrderedDict`，定义攻击模式下的交互参数：

```python
from collections import OrderedDict
from pocsuite3.api import OptString, OptBool, OptInteger, OptPort, OptIP, OptDict, OptFloat, OptItems

def _options(self):
    o = OrderedDict()
    o["username"] = OptString('', description='登录用户名', require=True)
    o["cmd"] = OptString('id', description='执行的命令', require=False)
    o["timeout"] = OptInteger(10, description='超时秒数')
    return o
```

## 8 种 Opt* 类型

| 类型 | Python 类型 | CLI | 说明 |
|------|-----------|-----|------|
| `OptString(def, desc, req)` | str | ✅ | 字符串 |
| `OptIP(def, desc, req)` | str (IP) | ✅ | IP 地址 |
| `OptPort(def, desc, req)` | int | ✅ | 端口号 |
| `OptInteger(def, desc, req)` | int | ✅ | 整数 |
| `OptFloat(def, desc, req)` | float | ✅ | 浮点数 |
| `OptBool(def, desc, req)` | bool | ❌ CLI | 布尔值 |
| `OptDict(dict, desc, req)` | dict | ❌ CLI | 选 key → value |
| `OptItems(list, desc, req, selected)` | str | ❌ CLI | 列表选项 |

## 两类自动注入选项

### HTTP 协议 PoC（默认）
```python
self.global_options["target"]  = OptString("目标", require=True)
self.global_options["referer"] = OptString("Referer")
self.global_options["agent"]   = OptString("User-Agent")
self.global_options["proxy"]   = OptString("代理 protocol://host:port")
self.global_options["timeout"] = OptInteger(10, "超时")
```

### 非 HTTP PoC（`protocol != HTTP`）
```python
self.global_options["rhost"] = OptString("主机", require=True)
self.global_options["rport"] = OptPort("端口", require=True)
```

### 实现了 `_shell()` 时
```python
self.payload_options["lhost"] = OptString(self.host_ip, "监听地址")
self.payload_options["lport"] = OptPort(6666, "监听端口")
```

## 获取参数

| 方法 | 来源 dict | 用途 |
|------|----------|------|
| `self.get_option("key")` | `self.options` | 自定义参数（`_options()` 定义） |
| `self.getg_option("key")` | `self.global_options` | 全局参数（timeout/proxy...） |
| `self.getp_option("key")` | `self.payload_options` | Payload 参数（lhost/lport） |

> `get_option()` 中若值含 `{0}`/`{1}`，自动替换为 `connect_back_host`/`connect_back_port`。

## 设置参数

```python
self.set_option("key", value)     # 自定义参数
self.setg_option("key", value)    # 全局参数
self.setp_option("key", value)    # payload 参数
```

## 获取全部 + 验证

```python
self.get_options()                 # 合并三个 OrderedDict
self.check_requirement(*args)      # 验证 require=True 的必填参数
```
