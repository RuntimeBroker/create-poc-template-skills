# Nuclei HTTP 基本请求

> 写 HTTP 模板必读。GET/POST、路径变量、请求头体、Cookie、Raw/Unsafe、多请求条件。

## 基本请求

```yaml
http:
  - method: GET
    path:
      - "{{BaseURL}}/vuln.php"
    redirects: true        # 默认不跟随
    max-redirects: 3
```

## 路径变量

| 变量 | 示例值 |
|------|--------|
| `{{BaseURL}}` | `https://example.com:443/foo/bar.php` |
| `{{RootURL}}` | `https://example.com:443` |
| `{{Hostname}}` | `example.com:443` |
| `{{Host}}` | `example.com` |
| `{{Port}}` | `443` |
| `{{Path}}` | `/foo` |
| `{{File}}` | `bar.php` |
| `{{Scheme}}` | `https` |

## 请求头与请求体

```yaml
headers:
  Content-Type: application/json
  User-Agent: Custom-UA

body: '{"key": "value"}'
# 或 body: "user=admin&pass=test"
```

## Session / Cookie

默认复用 cookie 维持会话：
```yaml
disable-cookie: true   # 禁用
```

## Raw Request（原始请求）

```yaml
http:
  - raw:
      - |
        POST /api/login HTTP/1.1
        Host: {{Hostname}}
        Content-Type: application/json

        {"user": "admin", "pass": "test"}
```

## Unsafe HTTP（畸形请求）

```yaml
http:
  - unsafe: true
    raw:
      - |
        GET / HTTP/1.1
        Host: {{Hostname}}
        Transfer-Encoding: chunked
```

## 多请求条件判断

`_n` 后缀引用前 N 个请求的响应：

```yaml
matchers:
  - type: dsl
    dsl:
      - "status_code_1 == 404 && status_code_2 == 200 && contains(body_2, 'secret')"
```

可用：`status_code_N`, `body_N`, `all_headers_N`, `content_length_N`
