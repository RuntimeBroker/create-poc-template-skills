# Pocsuite3 编程接口与工具

> 程序化调用框架、加载 PoC、HTTP 服务、远程文件等。

## 编程接口

```python
from pocsuite3.api import init_pocsuite, start_pocsuite, get_results

init_pocsuite({
    'url': 'http://target.com', 'poc': 'path/to/poc.py',
    'mode': 'verify', 'verbose': 1
})
start_pocsuite()
for r in get_results():
    print(r.to_dict())
```

## PoC 动态加载

```python
from pocsuite3.api import load_file_to_module, load_string_to_module

load_file_to_module('path/to/poc.py')   # 从文件加载
load_string_to_module(code_str)         # 从字符串加载
```

## Nuclei 模板兼容

```python
from pocsuite3.api import Nuclei
# 直接加载 Nuclei YAML 模板作为 PoC 运行
```

## 本地 HTTP 服务

```python
from pocsuite3.api import PHTTPServer
server = PHTTPServer(bind_ip='0.0.0.0', bind_port=666, use_https=False)
# SSRF 回连验证、文件托管等
```

## 插件系统

```python
from pocsuite3.api import PluginBase, register_plugin, PLUGIN_TYPE

@register_plugin
class MyPlugin(PluginBase):
    type = PLUGIN_TYPE.TARGETS
    def init(self): pass
    def start(self): pass
```

## 远程文件列表

| URL | 用途 |
|-----|------|
| `https://pocsuite.org/include_files/v.jsp` | 验证模式 JSP webshell |
| `https://pocsuite.org/include_files/a.jsp` | 攻击模式 JSP webshell |
| `https://pocsuite.org/include_files/php_attack.txt` | PHP 攻击 one-liner |
| `https://pocsuite.org/include_files/php_verify.txt` | PHP 验证（输出 MD5） |
| `https://pocsuite.org/include_files/xxe_verify.xml` | XXE 验证 |

## 其他工具函数

```python
from pocsuite3.api import (
    crawl,              # 简单爬虫
    random_str,         # random_str(length=10)
    get_middle_text,    # get_middle_text(text, prefix, suffix)
    check_port,         # check_port(host, port)
    get_host_ip,        # 本机 WAN IP
    get_host_ipv6,      # 本机 IPv6
    mosaic,             # 日志脱敏
    single_time_warn_message,  # 一次性警告
    urlparse,           # URL 解析（支持 IPv6）
    OrderedSet,         # 有序集合
)
```
