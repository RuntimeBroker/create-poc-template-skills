# 空间搜索引擎

> 需要批量获取目标或查询漏洞信息时查阅。

```python
from pocsuite3.api import ZoomEye, Seebug, Shodan, Fofa, Quake, Hunter, Censys
```

| API | 用途 | 说明 |
|-----|------|------|
| `ZoomEye` | 批量获取目标 | 支持 `--dork` 命令行搜索 |
| `Seebug` | 读取 PoC | `pocsuite -u target --ssvid 12345` |
| `Shodan` | 网络空间搜索 | — |
| `Fofa` | 网络空间搜索 | — |
| `Quake` | 网络空间搜索 | 360 |
| `Hunter` | 网络空间搜索 | 奇安信 |
| `Censys` | 网络空间搜索 | — |

> 详细 API 用法见各平台官方文档。Pocsuite3 封装了认证和查询接口。
