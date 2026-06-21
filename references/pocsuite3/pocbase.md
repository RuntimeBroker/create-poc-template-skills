# Pocsuite3 POCBase 基类（对照源码 v2.1.0）

> 写 PoC 必读。类的全部属性、核心方法、生命周期。

## 可定义属性（类体中赋值）

| 属性 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `vulID` | str | ✓ | 漏洞编号（CVE/SSVID） |
| `name` | str | ✓ | 漏洞名称 |
| `appName` | str | ✓ | 目标应用 |
| `appVersion` | str | — | 影响版本 |
| `author` | list | — | 作者 |
| `vulDate` | str | — | 漏洞公开日期 |
| `createDate` | str | — | PoC 创建日期 |
| `updateDate` | str | — | PoC 更新日期 |
| `references` | list | — | 参考链接 |
| `appPowerLink` | str | — | 厂商主页 |
| `vulType` | VUL_TYPE | — | 漏洞类型 |
| `category` | POC_CATEGORY | — | PoC 分类 |
| `desc` | str | — | 漏洞描述 |
| `pocDesc` | str | — | PoC 使用说明 |
| `dork` | str/dict | — | ZoomEye 搜索词 + `_check()` 关键词 |
| `samples` | list | — | 测试样本 URL |
| `install_requires` | list | — | 额外 pip 依赖 |
| `protocol` | PROTOCOL | — | 目标协议（默认 HTTP） |
| `protocol_default_port` | int | — | 协议端口（0=内置默认） |
| `suricata_request` | str | — | Suricata 请求匹配 |
| `suricata_response` | str | — | Suricata 响应匹配 |

## 运行时属性（框架注入）

| 属性 | 来源 | 说明 |
|------|------|------|
| `self.url` | `build_url()` | 完整 URL |
| `self.scheme` | URL 解析 | 协议 |
| `self.rhost` | URL 解析 | 主机名 |
| `self.rport` | URL 解析 | 端口 |
| `self.netloc` | URL 解析 | `host:port` |
| `self.host_ip` | `get_host_ip()` | 本机 WAN IP |
| `self.target` | `execute()` | 原始目标 |
| `self.headers` | `execute()` | 请求头 |
| `self.params` | `execute()` | 请求参数 |
| `self.mode` | `execute()` | `verify`/`attack`/`shell` |
| `self.current_protocol` | 类属性 | 协议类型 |
| `self.expt` | 异常捕获 | `(ERROR_TYPE_ID, exception)` |

## 核心方法

### `_verify()` — 必须重写
验证漏洞存在。只读，无害。

### `_attack()` — 可选重写
利用漏洞：执行命令、提取数据、写 webshell。

### `_shell()` — 可选重写
返回回调函数，框架反复调用执行命令。

### `execute(target, headers, params, mode, verbose)` — 框架入口
流程：`build_url()` → `_execute()` → 按 mode 分发 → 自动处理 ConnectTimeout(重试)/HTTPError/ConnectionError

### `build_url(target)` — URL 构建
根据 `current_protocol` 构建 URL，内置 19 协议默认端口：
FTP:21, SSH:22, TELNET:23, REDIS:6379, SMTP:25, DNS:53, SNMP:161, SMB:445, MQTT:1883, MYSQL:3306, RDP:3389, UPNP:1900, AJP:8009, XMPP:5222, WINBOX:8291, MEMCACHED:11211, BACNET:47808, T3:7001

### `_check(dork, allow_redirects, return_obj, is_http, honeypot_check)` — 前置检查
1. 端口 → 2. 非 HTTP 跳过 → 3. HTTP→HTTPS 自动纠正 → 4. dork 匹配 → 5. 蜜罐检测。`conf.get('no_check', False)` 跳过。

### `parse_output(result={})` — 统一输出
```python
def parse_output(self, result={}):
    output = Output(self)
    if result: output.success(result)
    else: output.fail('Internet nothing returned')
    return output
```

### 辅助方法
- `get_infos()` — PoC 元信息字典
- `get_category()` — 返回分类（未定义则 `'Unknown'`）
- `_run()` — GUI 桩方法
