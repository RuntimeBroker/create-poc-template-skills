# Nuclei Extractors（提取器）

> 从响应中提取数据，供后续请求或输出使用。

## 5 种类型

| 类型 | 适用场景 | 示例 |
|------|---------|------|
| `regex` | 正则匹配 | `regex: ["(A3T[A-Z0-9]|AKIA)[A-Z0-9]{16}"]` |
| `kval` | HTTP 头/Cookie | `kval: ["content_type"]`（连字符→下划线） |
| `json` | JSON 响应 | `json: ['.[] \| .id']`（JQ 语法） |
| `xpath` | HTML 结构 | `xpath: ['//a']` + `attribute: href` |
| `dsl` | DSL 表达式 | `dsl: ["len(body)"]` |

## 动态提取器——跨请求值共享

```yaml
extractors:
  - type: regex
    name: csrf_token       # 变量名
    part: body
    group: 1               # 捕获组
    regex:
      - '<input\sname="csrf_token"\svalue="([[:alnum:]]{16})"\s/>'
    internal: true          # 不输出到终端
```

后续请求直接引用：
```yaml
http:
  - raw:
      - |
        POST /submit HTTP/1.1
        Host: {{Hostname}}
        csrf={{csrf_token}}&data=test
```

## v3.1.4+ 同请求复用

```yaml
extractors:
  - type: regex
    name: title
    group: 1
    regex: ['<title>(.*)<\/title>']
    internal: true
  - type: dsl
    dsl: ['"Page Title: " + title']   # 直接用上一个 extractor 的 title
```
