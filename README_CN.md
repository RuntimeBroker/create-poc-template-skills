# create-poc-template

[English](README.md)

一个给 AI 编码 agent 用的 PoC/Exp 开发技能包，整理了 Pocsuite3 和 Nuclei 两个框架的完整开发文档，让 agent 写漏洞验证脚本时不用再瞎编 API。

## 为什么会写这个

用各种 AI 编码工具写了大量漏洞检测脚本后发现一个规律：agent 擅长理解漏洞原理、构造 payload、组织代码逻辑，但一到具体框架的 API 调用就开始编。`self.get_option()` 写成 `self.options.get()`，`Matchers` 里拼错 `condition` 字段，输出的 POCBase 属性名对不上源码。

本质问题不是模型能力不够，是它手边没有准确的参考文档。搜索引擎返回的博客大多过时、简略或者干脆是错的。于是就动了念头直接把框架源码和官方文档拆成 agent 能精准检索的碎片。

## 里面有什么

### Pocsuite3（对照源码 v2.1.0，10 个文件）

从 `knownsec/pocsuite3` 仓库的 `api/__init__.py`、`lib/core/poc.py`、`lib/core/enums.py`、`lib/utils/__init__.py` 逐一核对后整理：

- **POCBase 基类**：22 个可定义属性、13 个运行时属性、全部方法签名和参数
- **Output 结果类**：6 个方法、属性列表
- **参数系统**：`_options()` 写法、8 种 Opt* 类型（CLI 模式的限制也标了）、自动注入的 3 类选项、get/set 全部方法
- **59 种 VUL_TYPE**：从 CODE_EXECUTION 到 OTHER，按代码执行/注入/文件/认证/信息泄露/溢出/其他分组
- **24 个 POC_CATEGORY**：EXPLOITS(4) + TOOLS(1) + PROTOCOL(19，含端口映射表)
- **Shell/Shellcode**：REVERSE_PAYLOAD、BIND_PAYLOAD、bind_shell 系列、generate_shellcode_list、OSShellcodes、WebShell、Bash/PowerShell 编码
- **搜索引擎**：ZoomEye、Seebug、Shodan、Fofa、Quake、Hunter、Censys
- **DNSLog**：CEye、Interactsh 用法
- **编程接口与工具**：init_pocsuite/start_pocsuite、PHTTPServer、Nuclei 模板加载、远程文件列表、插件系统
- **3 个完整示例**：Webmin RCE (CVE-2019-15107)、Flask SSTI RCE、Redis 未授权访问，外加 CLI 用法和最佳实践

### Nuclei（对照官方文档，17 个文件）

从 `docs.projectdiscovery.io` 的 templates 目录逐页抓取后整理：

- **模板结构**：id、info 块、变量
- **HTTP 三层拆分**：
  - Basic：路径变量表、请求头/体、Cookie、Raw、Unsafe、多请求条件判断
  - Payloads：batteringram/pitchfork/clusterbomb 三种攻击模式、Fuzzing(pre-condition/part/type/mode/keys/analyzer)
  - Advanced：Race Condition、HTTP Pipelining、Connection Pooling
- **DNS**：全部记录类型、可用 part 值
- **Code**：引擎列表、args/pattern、输出引用
- **Headless**：20+ Action 速查表、选择器语法、waitdialog、完整登录检测示例
- **Network (TCP)**：hex 输入、read-size、TLS、多端口
- **File**：扩展名过滤、denylist、默认排除列表
- **Flow**：条件短路、JS 编排（iterate/set/template/log）、vhost 枚举示例、Internal Matchers、Dedupe
- **Multi-Protocol**：协议前缀变量表、DNS+HTTP+SSL 三协议示例、与 Workflow 对比
- **JavaScript 协议**：code/pre-condition/init 三节点、20 个 nuclei/* 内置模块、SSH 指纹/SSH 爆破/init 预加载 3 个示例
- **Matchers**：7 种类型 + AND/OR/Negative/Internal/Global + DSL 变量表
- **Extractors**：5 种类型 + 动态提取器（跨请求值共享）
- **OOB**：Preprocessors(randstr) + OOB Testing(interactsh 三种 part)
- **Helper Functions**：70 个 DSL 函数（编码/哈希/字符串/随机/时间/JWT/DNS/反序列化）
- **Workflows**：通用/条件/匹配器分支/嵌套链/上下文共享

## 文件怎么拆分

总共 28 个文件，每个尽量控制在 100 行以内。拆这么细是因为 skill 系统有个机制：`references/` 目录下的文件不会自动加载到上下文，agent 按需 read，只占它真正读了的那份。写一个 HTTP 模板大概只需要 3 个文件、200 行——而不是把 1200 行 nuclei 全吞进去。

```
poc-dev/
├── SKILL.md                   入口，框架选择指南 + 文档索引
└── references/
    ├── pocsuite3/             10 files
    │   ├── pocbase.md         POCBase 全部属性方法
    │   ├── output.md          Output 类
    │   ├── options.md         参数系统
    │   ├── enum-vul.md        59 种漏洞类型
    │   ├── enum-category.md   POC_CATEGORY + PLUGIN_TYPE
    │   ├── shell.md           Shell/Shellcode/WebShell
    │   ├── search.md          7 个搜索引擎
    │   ├── dnslog.md          CEye + Interactsh
    │   ├── api.md             编程接口/远程文件/工具
    │   └── examples.md        3 个 PoC + CLI + 最佳实践
    └── nuclei/                17 files
        ├── structure.md       模板骨架
        ├── http-basic.md      GET/POST/Raw/Unsafe/Cookie
        ├── http-payloads.md   Payloads(3 attack) + Fuzzing
        ├── http-advanced.md   Race + Pipelining + Pooling
        ├── dns.md             DNS 协议
        ├── code.md            Code 协议
        ├── headless.md        浏览器自动化
        ├── network.md         TCP 原始字节
        ├── file.md            本地文件扫描
        ├── flow.md            条件短路 + JS 编排
        ├── multi-protocol.md  跨协议变量共享
        ├── javascript.md      JS 运行时 + 20 个模块
        ├── matchers.md        7 种匹配器
        ├── extractors.md      5 种提取器
        ├── oob.md             Preprocessors + OOB
        ├── functions.md       70 个 DSL 函数
        └── workflows.md       工作流编排
```

## 怎么用

这个 skill 适用于任何支持 skill 格式的 AI 编码工具（Claude Code、Codex 等）。

会先判断场景：HTTP 漏洞简单验证 → Nuclei YAML；需要复杂 Python 逻辑、攻击利用 → Pocsuite3；浏览器相关 → Nuclei Headless；非标准协议 → Nuclei JavaScript。

## 可能的问题

- **API 跟实际行为不一致**：Pocsuite3 部分对照的是 GitHub 上 v2.1.0 的源码，如果你装的是开发版或者更老的版本，API 可能对不上。Nuclei 部分同理，文档站更新比 release 快
- **非 HTTP 协议的 PoC 示例偏少**：Redis 未授权是一个，但 SMB/SSH/FTP 等协议没有覆盖示例，只有 API 参考
- **Nuclei JavaScript 协议的 20 个模块只有 SSH 有完整示例**，其他模块只有名称列表——官方文档这部分本身就不全

## 许可

MIT。见 [LICENSE](LICENSE)。
