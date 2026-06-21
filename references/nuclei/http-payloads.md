# Nuclei HTTP Payloads 与 Fuzzing

> 爆破、模糊测试、通用漏洞检测。

## Payloads（暴力破解）

```yaml
payloads:
  password:
    - admin
    - guest
  paths: helpers/wordlists/paths.txt   # 外部文件需 -lfa
```

### 三种 Attack 模式

| 模式 | 行为 |
|------|------|
| `batteringram`（默认） | 同一值替换所有位置 |
| `pitchfork` | 各位置独立 payload 集，按索引配对 |
| `clusterbomb` | 笛卡尔积 |

```yaml
attack: clusterbomb
payloads:
  username: users.txt
  password: passwords.txt
```

---

## Fuzzing（v3.2.4+）

Payloads 的超集——通用漏洞检测，无需事先知道漏洞细节。

### 核心字段

| 字段 | 说明 |
|------|------|
| `part` | `query`(默认), `path`, `header`, `cookie`, `body`, `request` |
| `type` | `replace`(默认), `prefix`, `postfix`, `infix`, `replace-regex` |
| `mode` | `multiple`(一次全替), `single`(逐个) |
| `fuzz` | payload 来源，支持变量/DSL/`{{interactsh-url}}` |
| `keys` / `keys-regex` | 精确/正则匹配参数名 |
| `values` | 按参数值正则匹配 |
| `stop-at-first-match` | 命中即停 |
| `pre-condition` | 前置条件（全部 matcher 类型） |

### Pre-condition（v3.2.4+）

```yaml
pre-condition:
  - type: dsl
    dsl:
      - method == POST
      - len(body) > 0
    condition: and
```

### Key-Value 抽象

一条规则自动适用 JSON/XML/Form/Multipart 多种格式。Nuclei 自动将各部位解析为键值对。

### Analyzer: time_delay

基于线性回归验证时间盲注：
```yaml
matchers:
  - type: word
    part: analyzer
    words: ["true"]
```
占位符 `[SLEEPTIME]`, `[INFERENCE]`

### 反射 XSS 检测

```yaml
http:
  - pre-condition:
      - type: dsl
        dsl: ['method == "GET"']
    payloads:
      reflection: ["6842'\"><9967"]
    stop-at-first-match: true
    fuzzing:
      - part: query
        type: postfix
        mode: single
        fuzz: ["{{reflection}}"]
    matchers-condition: and
    matchers:
      - type: word
        part: body
        words: ["{{reflection}}"]
      - type: word
        part: header
        words: ["text/html"]
```

### 定向命令注入 Fuzzing

```yaml
fuzzing:
  - part: query
    type: postfix
    mode: single
    keys-regex: ['^(cmd|exec|command|ping|ip|host|url|path|file)$']
    fuzz: ["{{interactsh-url}}"]
    matchers:
      - type: word
        part: interactsh_protocol
        words: ["dns"]
```
