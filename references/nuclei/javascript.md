# Nuclei JavaScript 协议（v3+）

> JavaScript 协议是 Nuclei v3 引入的图灵完备检测引擎，基于 Goja (ECMAScript 5.1)。用于 YAML DSL 无法覆盖的复杂场景。**⚠️ 不取代 matchers/extractors，是补充。**

## 模板结构

```yaml
javascript:
  - code: |                    # 必需：JS 代码，最后一条表达式值作为输出
      var m = require("nuclei/ssh");
      var c = m.SSHClient();
      c.Connect(Host, Port, Username, Password);

    pre-condition: |            # 可选：前置条件，返回 true 才执行 code
      ...

    init: |                     # 可选：模板编译后执行一次（预加载数据）
      ...

    args:                       # 可选：传递变量，支持 DSL
      Host: "{{Host}}"
      Port: "22"

    attack: clusterbomb         # 可选：攻击模式
    payloads:                   # 可选：payload 字典
      ...
    threads: 10                 # 可选：并发数
    stop-at-first-match: true   # 可选：命中即停
```

## 三个代码节点

| 节点 | 执行时机 | 用途 |
|------|---------|------|
| `init` | 模板编译后、处理目标前，**只执行一次** | 从磁盘预加载密钥/字典等 |
| `pre-condition` | 每个目标，在 `code` 之前 | 判断是否满足条件再走主逻辑 |
| `code` | 每个目标，pre-condition 通过后 | 核心检测逻辑 |

### init 内置函数

| 函数 | 说明 |
|------|------|
| `updatePayload(key, value)` | 更新指定 key 的 payload 列表 |
| `set(key, value)` | 设置变量键值对 |

## 内置模块（15+）

所有模块通过 `require("nuclei/<name>")` 加载：

| 模块 | 用途 | 关键方法 |
|------|------|---------|
| `nuclei/ssh` | SSH 连接/指纹/爆破 | `SSHClient()`, `ConnectSSHInfoMode()`, `Connect()` |
| `nuclei/ftp` | FTP 协议操作 | — |
| `nuclei/rdp` | RDP 协议操作 | — |
| `nuclei/kerberos` | Kerberos 认证 | — |
| `nuclei/redis` | Redis 协议操作 | — |
| `nuclei/fs` | 文件系统操作 | `ReadFilesFromDir(path)` |
| `nuclei/net` | 网络连接 | — |
| `nuclei/ldap` | LDAP 协议 | — |
| `nuclei/mssql` | MS SQL 协议 | — |
| `nuclei/mysql` | MySQL 协议 | — |
| `nuclei/postgres` | PostgreSQL 协议 | — |
| `nuclei/smb` | SMB 协议 | — |
| `nuclei/smtp` | SMTP 协议 | — |
| `nuclei/oracle` | Oracle DB 协议 | — |
| `nuclei/pop3` | POP3 协议 | — |
| `nuclei/telnet` | Telnet 协议 | — |
| `nuclei/vnc` | VNC 协议 | — |
| `nuclei/rsync` | Rsync 协议 | — |
| `nuclei/ikev2` | IKEv2 (VPN) 协议 | — |
| `nuclei/structs` | 二进制结构体编解码 | — |

## 完整示例

### 示例 1：SSH 指纹识别

```yaml
id: ssh-server-fingerprint
info:
  name: Fingerprint SSH Server Software
  author: pdteam
  severity: info

javascript:
  - code: |
      var m = require("nuclei/ssh");
      var c = m.SSHClient();
      var response = c.ConnectSSHInfoMode(Host, Port);
      to_json(response);
    args:
      Host: "{{Host}}"
      Port: "22"
    extractors:
      - type: json
        json:
          - '.ServerID.Raw'
```

- `to_json(response)` — 将对象序列化为 JSON，供 json extractor 提取
- 最后一条表达式的值作为模板输出
- 出错时暴露 `error` 变量供 matcher/extractor 使用

### 示例 2：SSH 密码爆破 + Pre-condition + Clusterbomb

```yaml
id: ssh-brute
info:
  name: SSH Credential Stuffing
  author: tarunKoyalwar
  severity: critical

javascript:
  - pre-condition: |
      var m = require("nuclei/ssh");
      var c = m.SSHClient();
      var response = c.ConnectSSHInfoMode(Host, Port);
      response["UserAuth"].includes("password")

    code: |
      var m = require("nuclei/ssh");
      var c = m.SSHClient();
      c.Connect(Host, Port, Username, Password);

    args:
      Host: "{{Host}}"
      Port: "22"
      Username: "{{usernames}}"
      Password: "{{passwords}}"

    threads: 10
    attack: clusterbomb
    payloads:
      usernames: helpers/wordlists/wp-users.txt
      passwords: helpers/wordlists/wp-passwords.txt
    stop-at-first-match: true

    matchers:
      - type: dsl
        dsl:
          - "response == true"
          - "success == true"
        condition: and
```

**工作原理**：
1. `pre-condition` 先连接 SSH 检查是否支持密码认证，不支持的直接跳过
2. `code` 用 clusterbomb 模式遍历用户名/密码组合
3. `stop-at-first-match: true` 命中即停
4. `threads: 10` 控制并发

### 示例 3：init 预加载数据

```yaml
variables:
  keysDir: "helpers/"

javascript:
  - init: |
      let m = require('nuclei/fs');
      let privatekeys = m.ReadFilesFromDir(keysDir);
      updatePayload('keys', privatekeys);
    payloads:
      keys:
        - key1
        - key2
```

`init` 只执行一次，从磁盘目录读取所有私钥文件到 `keys` payload 中，后续每个目标直接使用预加载的数据。

## 适用场景总结

| 场景 | 为什么不用 YAML DSL |
|------|-------------------|
| SSH/RDP/Kerberos 等复杂协议 | 需要多步握手、状态管理 |
| 本地权限提升检测 | 根本不需要发网络请求 |
| 内核漏洞检测 | 需要检查 `/proc`、系统调用 |
| Redis/SMB 缓冲区溢出 | 需要构造非标准数据包 |
| LDAP 多步认证 | 认证流程有状态依赖 |
| 系统配置错误 | 读取本地文件并判断 |

## 与 Flow 协议的区别

| | JavaScript Protocol | Flow Protocol |
|------|---------|------|
| 用途 | 编写检测逻辑本身 | 编排已有请求的执行顺序 |
| 执行内容 | 自定义 JS 代码 | 调用 `http()` / `dns()` / `ssl()` |
| 输出 | 最后表达式值 | 协议请求的响应 |
| 场景 | 协议漏洞/本地检测 | 条件短路/循环/vhost 枚举 |

## 注意事项

- 不是通用 JS 运行时，不支持 `import` 或外部 npm 包
- 仅支持 ECMAScript 5.1 语法
- 不取代 matchers/extractors——两者仍可照常使用
- 出错时 `error` 变量自动暴露，可用于 matcher 判断
