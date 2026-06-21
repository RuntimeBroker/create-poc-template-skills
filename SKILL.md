---
name: create-poc-template
description: >
  漏洞 PoC/Exp 开发技能，涵盖 Pocsuite3 和 Nuclei 两大框架的完整开发文档。
  当用户需要编写漏洞验证脚本、开发 PoC、编写 Exploit、使用 Pocsuite3 或 Nuclei、
  进行漏洞扫描模板开发、将漏洞分析转化为可执行验证代码时，使用此技能。
  即使用户没有明确说"Pocsuite3"或"Nuclei"，只要涉及漏洞验证脚本开发就应该触发。
---

# 漏洞 PoC/Exp 开发指南

本技能覆盖 **Pocsuite3**（Python SDK）和 **Nuclei**（YAML 模板）两个框架的完整开发知识，帮助 agent 快速编写高质量的漏洞验证和利用脚本。

---

## 框架选择指南

| 场景 | 推荐框架 | 原因 |
|------|---------|------|
| 需要复杂逻辑、加密、自定义协议 | Pocsuite3 | Python 生态，灵活 |
| HTTP 服务的快速漏洞扫描 | Nuclei | YAML 简洁，内置匹配器 |
| 需要浏览器自动化 | Nuclei (Headless) | 内置 Chrome 控制 |
| 需要与 Seebug/ZoomEye 联动 | Pocsuite3 | 内置 API 集成 |
| 需要批量大规模扫描 | Nuclei | 扫描引擎性能优秀 |
| 需要 DNSLog 盲打验证 | 两者都支持 | pocsuite3 用 Ceye，nuclei 用 interactsh |
| 需要反序列化 Payload 生成 | Nuclei | 内置 Java/.NET gadget 函数 |
| 需要写 Exploit 而非仅验证 | Pocsuite3 (attack 模式) | 支持交互 shell |

**选择优先级**：如果是简单 HTTP 漏洞且主要为检测 → 优先 Nuclei。如果需要复杂利用、Python 库依赖或有复杂逻辑 → 用 Pocsuite3。两者可从 v2.0.0 起互通（Pocsuite3 兼容 Nuclei 模板）。

---

## 快速参考

### Pocsuite3 最小 PoC 模板

```python
from pocsuite3.api import Output, POCBase, register_poc, requests, logger

class DemoPOC(POCBase):
    vulID = '0'                    # 漏洞 ID
    version = '1.0'
    author = ['anonymous']
    vulDate = '2024-01-01'
    createDate = '2024-01-01'
    updateDate = '2024-01-01'
    references = ['https://example.com']
    name = '漏洞名称'
    appPowerLink = 'https://example.com'
    appName = '目标应用名称'
    appVersion = '1.0'
    vulType = 'RCE'               # 漏洞类型
    desc = '''漏洞描述'''
    samples = []
    install_requires = []         # 额外依赖

    def _verify(self):
        """验证模式：确认漏洞存在，对目标无害"""
        result = {}
        try:
            # 编写验证逻辑
            r = requests.get(self.url)
            if "vulnerable" in r.text:
                result['VerifyInfo'] = {}
                result['VerifyInfo']['URL'] = self.url
                result['VerifyInfo']['Payload'] = 'your_payload'
        except Exception as e:
            logger.error(str(e))
        return self.parse_output(result)

    def _attack(self):
        """攻击模式（可选）：实现利用逻辑"""
        result = {}
        # 编写利用逻辑
        return self.parse_output(result)

    def _shell(self):
        """Shell 模式（可选）：反向连接"""
        # 编写 reverse shell 逻辑
        pass

    def parse_output(self, result):
        output = Output(self)
        if result:
            output.success(result)
        else:
            output.fail('target is not vulnerable')
        return output

register_poc(DemoPOC)
```

### Nuclei 最小模板

```yaml
id: vulnerability-name

info:
  name: 漏洞名称
  author: anonymous
  severity: medium
  description: 漏洞描述
  reference:
    - https://example.com
  tags: cve,rce

http:
  - method: GET
    path:
      - "{{BaseURL}}/vulnerable-path"
    matchers:
      - type: word
        words:
          - "vulnerable_string"
```

---

## 详细文档

> 按需查阅——写 PoC 时只读需要的部分，不浪费 token。

### Pocsuite3（对照源码 v2.1.0）

| 文件 | 内容 | 何时查阅 |
|------|------|---------|
| **[POCBase](references/pocsuite3/pocbase.md)** | 全部属性(22) + 核心方法 + 生命周期 | 写 PoC 必读 |
| **[Output](references/pocsuite3/output.md)** | Output 类属性 + 6 个方法 | 处理返回结果 |
| **[参数系统](references/pocsuite3/options.md)** | _options() + 8 种 Opt* + 自动注入 + get/set | 需要加参数 |
| **[VUL_TYPE](references/pocsuite3/enum-vul.md)** | 59 种漏洞类型 | 设置 vulType |
| **[POC_CATEGORY](references/pocsuite3/enum-category.md)** | EXPLOITS/TOOLS/PROTOCOL + PLUGIN_TYPE | 设置 category |
| **[Shell](references/pocsuite3/shell.md)** | REVERSE/BIND + Shellcode + WebShell + 编码 | 写 _shell() |
| **[搜索引擎](references/pocsuite3/search.md)** | ZoomEye/Seebug/Shodan/Fofa/Quake/Hunter/Censys | 批量获取目标 |
| **[DNSLog](references/pocsuite3/dnslog.md)** | CEye + Interactsh | 盲打验证 |
| **[编程接口](references/pocsuite3/api.md)** | 编程调用 + 远程文件 + 插件 + 工具函数 | 高级用法 |
| **[完整示例](references/pocsuite3/examples.md)** | 3 个 PoC + CLI + 最佳实践 | 参考代码 |

### Nuclei（对照官方 docs）

| 文件 | 内容 | 何时查阅 |
|------|------|---------|
| **[模板结构](references/nuclei/structure.md)** | id/info/variables | 新写模板必读 |
| **[HTTP Basic](references/nuclei/http-basic.md)** | GET/POST/Raw/Unsafe/路径变量/Cookie/多请求条件 | 写 HTTP 模板 |
| **[HTTP Payloads](references/nuclei/http-payloads.md)** | Payloads(3 attack) + Fuzzing(pre-condition/part/type/mode/analyzer) | 爆破/模糊测试 |
| **[HTTP Advanced](references/nuclei/http-advanced.md)** | Race + Connection Tampering(Pipelining/Pooling) | 条件竞争/高速请求 |
| **[DNS](references/nuclei/dns.md)** | DNS 协议 | DNS 查询 |
| **[Code](references/nuclei/code.md)** | Code 协议（需 `-code`） | 外部代码执行 |
| **[Headless](references/nuclei/headless.md)** | 20+ Action + 选择器（需 `-headless`） | 浏览器自动化 |
| **[Network](references/nuclei/network.md)** | TCP/Network 协议 | 原始字节收发 |
| **[File](references/nuclei/file.md)** | File 协议（需 `-file`） | 本地文件扫描 |
| **[Flow](references/nuclei/flow.md)** | 条件短路 + JS 编排 + internal matchers | 请求编排 |
| **[Multi-Protocol](references/nuclei/multi-protocol.md)** | 跨协议变量共享 | 多协议联动 |
| **[JavaScript](references/nuclei/javascript.md)** | JS 运行时 + 20 模块 + 3 示例 | 复杂协议/本地检测 |
| **[Matchers](references/nuclei/matchers.md)** | 7 种类型 + AND/OR/Negative/Internal/Global | 编写检测逻辑 |
| **[Extractors](references/nuclei/extractors.md)** | 5 种类型 + 动态提取 | 提取数据/跨请求传递 |
| **[OOB](references/nuclei/oob.md)** | Preprocessors(randstr) + OOB(interactsh) | 预处理/盲打 |
| **[Functions](references/nuclei/functions.md)** | 70 个 DSL 函数 | 写 DSL 表达式 |
| **[Workflows](references/nuclei/workflows.md)** | 条件/匹配器分支/嵌套/上下文共享 | 链式扫描 |

---

## 开发最佳实践

### 通用原则

1. **先验证后利用**：verify 模式要尽可能对目标无害，不修改文件、不植入后门
2. **精确匹配减少误报**：用多个 matcher 组合（condition: and）而非单一条件
3. **变量化可配置项**：将可变参数（端口、路径、命令）抽取为变量，不要硬编码
4. **超时与错误处理**：网络请求必须设置 timeout，处理连接异常
5. **遵守规范**：Pocsuite3 遵循 [PoC 编写规范](https://pocsuite.org/guide/poc-introduction.html)，Nuclei 遵循 [模板结构](https://docs.projectdiscovery.io/templates/structure)

### Pocsuite3 专项

- 继承 POCBase，实现 `_verify()`（必须），`_attack()` 和 `_shell()`（可选）
- 使用 `self.parse_output({})` 统一返回结果
- 利用 `self._check()` 自动做端口/协议/蜜罐检测
- 通过 `self.get_option('key')` 获取用户自定义参数

### Nuclei 专项

- id 不能包含空格
- 用 tags 做好分类（cve, rce, sqli, xss 等）
- 善用动态提取器传递 CSRF token 等运行时值
- 条件竞争用 `race: true` + `race_count`（单请求）或 `threads`（多请求）
- Workflows 实现多步骤链式漏洞检测
