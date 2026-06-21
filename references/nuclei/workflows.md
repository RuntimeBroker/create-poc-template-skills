# Nuclei Workflows（工作流）

> 链式扫描——按条件编排多个模板的执行顺序。

## 通用工作流

```yaml
workflows:
  - template: http/exposures/configs/git-config.yaml
  - template: http/exposures/configs/exposed-svn.yaml
  - tags: xss,ssrf,cve,lfi
  - template: http/cves/
```

## 条件工作流——基于模板匹配

```yaml
workflows:
  - template: http/technologies/jira-detect.yaml
    subtemplates:
      - tags: jira
      - template: exploits/jira/
```

## 条件工作流——基于匹配器名称

```yaml
workflows:
  - template: http/technologies/tech-detect.yaml
    matchers:
      - name: vbulletin
        subtemplates:
          - template: exploits/vbulletin-exp1.yaml
          - template: exploits/vbulletin-exp2.yaml
      - name: jboss
        subtemplates:
          - template: exploits/jboss-exp1.yaml
```

## 多层嵌套条件链

```yaml
workflows:
  - template: http/technologies/tech-detect.yaml
    matchers:
      - name: apache
        subtemplates:
          - template: http/technologies/apache-version.yaml
            subtemplates:
              - template: http/cves/2020/xxxx.yaml
```

## 共享执行上下文

工作流中所有模板共享 cookie jar 和命名提取器结果。

提取方模板：
```yaml
id: extract-href
http:
  - path: ["{{BaseURL}}/path1"]
    extractors:
      - type: regex
        part: body
        name: links
        regex: ['href="(.*)"']
        group: 1
```

消费方模板直接引用 `{{links}}`：
```yaml
http:
  - raw:
      - |
        GET /path2 HTTP/1.1
        Host: {{Hostname}}
        {{links}}
```
