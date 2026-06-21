# Nuclei Multi-Protocol（v3+）

> 同一模板串联多协议，按定义顺序串行执行。字段自动导出。

## 协议前缀变量

所有协议的响应字段自动以 `协议名_字段名` 导出：

| 协议 | 常见变量 |
|------|---------|
| ssl | `ssl_subject_cn`, `ssl_subject_an`, `ssl_domains` |
| dns | `dns_cname`, `dns_a`, `dns_mx` |
| http | `http_body`, `http_header`, `http_status_code` |
| code | `code_response` |

## DNS + HTTP 子域名接管

```yaml
dns:
  - name: "{{FQDN}}"
    type: cname
http:
  - method: GET
    path: ["{{BaseURL}}"]
    matchers:
      - type: dsl
        dsl:
          - contains(http_body,'Domain not found')
          - contains(dns_cname, 'github.io')
        condition: and
```

## DNS + SSL + HTTP 三协议

```yaml
dns:
  - name: "{{FQDN}}"
    type: cname
ssl:
  - address: "{{Hostname}}"
http:
  - method: GET
    path: ["{{BaseURL}}"]
    matchers:
      - type: dsl
        dsl:
          - contains(http_body,'ProjectDiscovery.io')
          - trim_suffix(dns_cname,'.ghost.io.') == 'projectdiscovery'
          - ssl_subject_cn == 'blog.projectdiscovery.io'
        condition: and
```

## 动态提取器跨协议导出

```yaml
dns:
  - name: "{{FQDN}}"
    type: cname
    extractors:
      - type: dsl
        name: exported_cname
        dsl: [cname]
        internal: true
# http 中直接用 {{exported_cname}}
```

## 与 Workflow 对比

| | Multi-Protocol | Workflow |
|------|---------|------|
| 核心 | 单模板内多协议 | 编排多独立模板 |
| 变量 | 原生 `协议名_字段名` | 动态提取器传递 |
| 条件 | 不支持（串行） | 支持分支 |

> 限制：不支持重复协议（`dns → http → dns`），仅 v3+。
