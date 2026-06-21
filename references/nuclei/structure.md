# Nuclei 模板结构

> 新手入门。模板的骨架、info 块字段、变量定义。

## 模板文件结构

```
template.yaml
├── id                   # 唯一标识（必填，不能含空格）
├── info                 # 信息块（必填）
│   ├── name             # 模板名称
│   ├── author           # 作者
│   ├── severity         # critical/high/medium/low/info
│   ├── description      # 描述
│   ├── reference        # 参考链接
│   ├── tags             # 标签（cve,rce,sqli...）
│   └── metadata         # 可选，集成外部引擎
├── variables            # 可选，模板级变量
├── flow                 # v3 可选，JS 流程编排
├── <protocol>            # http/dns/code/headless/tcp/file
├── matchers             # 匹配条件
└── extractors           # 提取器（可选）
```

## info 块

```yaml
info:
  name: Git Config File Detection Template
  author: Ice3man
  severity: medium
  description: Searches for /.git/config pattern.
  reference:
    - https://www.acunetix.com/vulnerabilities/web/git-repository-found/
  tags: git,config
  metadata:
    shodan-query: 'vuln:CVE-2021-26855'
```

## Variables（变量）

```yaml
variables:
  target_path: "/admin/config"
  auth: "{{base64('admin:password')}}"
  oast: "{{interactsh-url}}"
  rand_str: "{{rand_text_alphanumeric(8)}}"
```

- 值计算后不再改变，整个模板内可用
- 支持 `dns`, `http`, `headless`, `network` 协议
- 通过 `{{变量名}}` 引用
