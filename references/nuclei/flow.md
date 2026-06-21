# Nuclei Flow 协议（v3）

> JavaScript 流程引擎（ECMAScript 5.1 via Goja）。条件执行 + 请求编排。

## 条件短路

```yaml
flow: http(1) && http(2)    # 请求 1 匹配成功才执行请求 2
```

## JS 编排（5 个核心函数）

| 函数 | 作用 |
|------|------|
| `protocol(index)` | 执行协议请求：`http(1)`, `dns("extract-vps")` |
| `iterate(val)` | 安全遍历数组/映射/字符串/数字 |
| `set("key", val)` | 注入变量到模板上下文 |
| `template["key"]` | 读取模板上下文值 |
| `log(val)` | 调试输出 |

## Vhost 枚举示例

```yaml
flow: |
  ssl();
  for (let vhost of iterate(template["ssl_domains"])) {
    set("vhost", vhost);
    http();
  }

ssl:
  - address: "{{Host}}:{{Port}}"

http:
  - raw:
      - |
        GET / HTTP/1.1
        Host: {{vhost}}
    matchers:
      - type: dsl
        dsl:
          - status_code != 400 && status_code != 502
```

## Internal Matchers（v3.1.4+）

前置请求标记 `internal: true` 跳过输出，仅条件判断：
```yaml
matchers:
  - type: dsl
    dsl: ['status_code == 200 && contains(body, "Backup")']
    condition: and
    internal: true
```

## 去重

```javascript
let uniq = new Dedupe();
uniq.Add(template["ptrValue"]);
log(uniq.Values())
```
