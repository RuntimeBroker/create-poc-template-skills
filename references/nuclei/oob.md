# Nuclei Preprocessors 与 OOB Testing

## Preprocessors（预处理器）

模板加载时执行。当前支持 `randstr`：

```yaml
http:
  - method: POST
    path: ["{{BaseURL}}/echo"]
    headers:
      cmd: echo '{{randstr}}'
    matchers:
      - type: word
        words: ['{{randstr}}']
```

- 同模板内值一致
- `{{randstr_1}}`, `{{randstr_2}}` 生成多独立值

---

## OOB Testing（带外测试）

使用 `{{interactsh-url}}` 盲打验证：

```yaml
http:
  - raw:
      - |
        GET /api/redirect?url=https://{{interactsh-url}} HTTP/1.1
        Host: {{Hostname}}
```

### Matcher part

| part | 说明 |
|------|------|
| `interactsh_protocol` | 协议：`dns`, `http`, `smtp` |
| `interactsh_request` | interactsh 收到的请求内容 |
| `interactsh_response` | interactsh 返回的响应 |

### DNS 交互（最常用，非侵入）

```yaml
matchers:
  - type: word
    part: interactsh_protocol
    words: ["dns"]
```

### HTTP 交互 + 内容校验

```yaml
matchers-condition: and
matchers:
  - type: word
    part: interactsh_protocol
    words: ["http"]
  - type: regex
    part: interactsh_request
    regex: ['root:.*:0:0:']
```
