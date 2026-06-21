# Nuclei HTTP 高级特性

> 条件竞争、连接篡改（Pipelining / Connection Pooling）。

## Race Condition（条件竞争）

### 单请求重复
```yaml
race: true
race_count: 10
```
Gate 机制——所有请求在最后一个字节前暂停，同步发出。

### 多请求并发
```yaml
threads: 5
race: true
```

```yaml
http:
  - raw:
      - |
        POST /coupons HTTP/1.1
        Host: {{Hostname}}
        promo_code=20OFF
    race: true
    race_count: 10
    matchers:
      - type: status
        status: [200]
```

---

## HTTP Pipelining（管线化）

同一连接发送多个请求，需目标支持 Pipeline：

```yaml
unsafe: true
pipeline: true
pipeline-concurrent-connections: 40
pipeline-requests-per-connection: 25000
```

```yaml
http:
  - raw:
      - |
        GET /{{path}} HTTP/1.1
        Host: {{Hostname}}
    attack: batteringram
    payloads:
      path: path_wordlist.txt
    unsafe: true
    pipeline: true
    pipeline-concurrent-connections: 40
    pipeline-requests-per-connection: 25000
```

---

## Connection Pooling（连接池）

```yaml
threads: 40   # ⚠️ 不能用 Connection: Close
```

```yaml
http:
  - raw:
      - |
        GET /protected HTTP/1.1
        Host: {{Hostname}}
        Authorization: Basic {{base64('admin:{{password}}')}}
    attack: batteringram
    payloads:
      password: password.txt
    threads: 40
```

| 特性 | Pipelining | Pooling |
|------|-----------|---------|
| 适用 | 批量路径探测 | 爆破/Fuzzing |
| 核心属性 | `pipeline: true` | `threads: N` |
| 限制 | 需目标支持 Pipeline | 不能 Connection: Close |
