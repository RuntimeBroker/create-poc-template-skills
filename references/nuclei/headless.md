# Nuclei Headless 协议

> 浏览器自动化。需 `-headless` 标志。

## 常用 Action

| Action | 功能 | 关键参数 |
|--------|------|----------|
| `navigate` | 跳转 | `url` |
| `script` | 执行 JS | `code`（必须 `() => ...`） |
| `click` | 点击 | 选择器 |
| `text` | 输入文本 | `value` + 选择器 |
| `screenshot` | 截图 | `to` |
| `select` | 下拉选择 | `selected`/`value` |
| `files` | 文件上传 | `value` |
| `waitload`/`waitidle`/`waitstable` | 等待 | `duration` |
| `waitdialog` | 等弹窗 | `max-duration`，**必须设 name** |
| `extract` | 提取属性 | `target: attribute` |
| `setheader`/`addheader`/`deleteheader` | 头操作 | `part`+`key`+`value` |
| `keyboard` | 按键 | `keys`（`\r`=回车） |
| `sleep` | 休眠 | `duration` |

## 选择器

| 值 | 说明 |
|----|------|
| `r` / `regex` | CSS + 文本正则 |
| `x` / `xpath` | XPath |
| `js` | JS 函数返回元素 |
| `search` | 搜索文本/XPATH/CSS |
| 默认 | CSS 选择器 |

## 完整示例：自动登录

```yaml
headless:
  - steps:
      - args: {url: "{{BaseURL}}/login.php"}
        action: navigate
      - action: waitload
      - args: {by: xpath, xpath: //input[@name='username']}
        action: click
      - args: {by: xpath, value: "admin", xpath: //input[@name='username']}
        action: text
      - args: {by: xpath, value: "password", xpath: //input[@name='password']}
        action: text
      - args: {by: xpath, xpath: //button[@type='submit']}
        action: click
      - action: waitload
    matchers:
      - type: word
        part: body
        words: ["Welcome"]
```

## waitdialog 输出变量

设 `name: mydialog`，自动生成：
- `mydialog` (bool), `mydialog_type` (str), `mydialog_message` (str)
