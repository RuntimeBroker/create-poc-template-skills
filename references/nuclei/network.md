# Nuclei Network (TCP) 协议

> 发送/接收原始字节，类似可自动化的 Netcat。

```yaml
tcp:
  - inputs:
      - data: "TEST\r\n"
      - data: "50494e47"
        type: hex
      - data: 'hex_decode("50494e47")\r\n'
    host:
      - "{{Hostname}}"
      - "tls://{{Hostname}}"   # TLS
    port: 27017
    read-size: 2048
    matchers:
      - type: word
        words: ["logicalSessionTimeout"]
```

## 字段说明

| 字段 | 说明 |
|------|------|
| `inputs.data` | 发送数据（支持纯文本/hex/DSL表达式） |
| `inputs.type: hex` | hex 解码为原始字节 |
| `inputs.read-size` | 读取 N 字节 |
| `inputs.name` | 命名读取数据，供 matcher `part: name` |
| `host` | 目标，`tls://` 前缀启用 TLS |
| `port` | 端口（v3.1.0+ 支持逗号分隔多端口） |

## 多端口 + 保留端口

```yaml
port: 5432,5433
exclude-ports: 80,443
```

默认保留：80, 443, 8080, 8443, 8081, 53
