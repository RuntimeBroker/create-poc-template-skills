# Nuclei DNS 协议

> DNS 查询模板。A/CNAME/MX/TXT/AAAA 等记录类型。

```yaml
dns:
  - name: "{{FQDN}}"
    type: A              # A/NS/CNAME/SOA/PTR/MX/TXT/AAAA
    class: inet
    recursion: true
    retries: 3
    matchers:
      - type: word
        words:
          - "IN\tCNAME"
          - "IN\tA"
        condition: and
```

## Matcher / Extractor 可用 part

| part | 含义 |
|------|------|
| `request` | DNS 请求 |
| `rcode` | DNS 响应码 |
| `question` | DNS 问题消息 |
| `extra` | DNS 消息额外字段 |
| `answer` | DNS 消息答案区 |
| `ns` | DNS 消息授权区 |
| `raw` / `all` / `body` | 原始 DNS 消息 |
