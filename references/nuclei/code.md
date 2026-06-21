# Nuclei Code 协议

> 外部代码执行。需 `-code` 标志。

```yaml
code:
  - engine:
      - py
      - python3
    source: |
      import sys, base64
      oast = sys.stdin.read().strip()
      print(base64.b64encode(oast.encode()).decode())
    extractors:
      - type: dsl
        dsl: [response]
```

## 字段说明

| 字段 | 说明 |
|------|------|
| `engine` | 解释器列表：`bash`, `sh`, `py`, `python3`, `go`, `ps`, `pwsh`, `powershell` |
| `source` | 代码片段（`\|` 管道符）或外部文件路径 |
| `args` | 可选，传递额外参数（如 `-ExecutionPolicy Bypass`） |
| `pattern` | 可选，临时文件扩展名（如 `"*.ps1"`） |

## 输出引用

```yaml
http:
  - method: GET
    path:
      - "{{BaseURL}}/?x={{code_1_response}}"
```

## Matcher / Extractor 可用 part

`response`（输出）, `stderr`（标准错误）
