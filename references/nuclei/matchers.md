# Nuclei Matchers（匹配器）

> 判断请求结果是否符合漏洞条件。

## 7 种类型

| 类型 | 匹配对象 | 示例 |
|------|---------|------|
| `status` | HTTP 状态码 | `status: [200, 302]` |
| `size` | 响应大小 | `size: [1024]` |
| `word` | 关键词 | `words: ["[core]"]` |
| `regex` | 正则 | `regex: ['secret[0-9]+']` |
| `binary` | 十六进制 | `binary: ["504B0304"]` |
| `dsl` | DSL 表达式 | `dsl: ["len(body)<1024"]` |
| `xpath` | XML/HTML | `xpath: ["/html/head/title[...]"]` |

## 条件控制

```yaml
# 组内 AND（默认 OR）
matchers:
  - type: word
    words: ["[core]", "[config]"]
    condition: and

# 多组全局 AND
matchers-condition: and
matchers:
  - type: word
    words: ["X-Powered-By: PHP"]
    part: header
  - type: word
    words: ["PHP"]
    part: body

# 负向匹配
matchers:
  - type: word
    words: ["PHPSESSID"]
    negative: true

# 内部匹配（仅 flow 判断，不输出结果）
matchers:
  - type: dsl
    dsl: ['status_code == 200']
    internal: true

# 全局匹配器（被动匹配所有 HTTP 响应，需 -egm）
http:
  - global-matchers: true
    matchers:
      - type: regex
        regex: ['-----BEGIN ((EC|PGP|DSA|RSA|OPENSSH) )?PRIVATE KEY']
        part: body
```

## DSL 可用 Response 变量

| 变量 | 说明 |
|------|------|
| `status_code` | HTTP 状态码 |
| `content_length` | Content-Length |
| `body` | 响应体 |
| `all_headers` | 所有头部字符串 |
| `header_name` | 特定头（连字符→下划线） |
| `raw` | 原始响应 |
