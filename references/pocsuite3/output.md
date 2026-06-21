# Output 类（返回结果）

> 所有 `parse_output()` 方法创建并返回 Output 对象。

## 属性

```python
status       # OUTPUT_STATUS.SUCCESS / FAILED
result       # dict — 存放 VerifyInfo / AttackInfo
error_msg    # (ERROR_TYPE_ID, message)
url          # 从 PoC 继承
mode         # 从 PoC 继承（verify/attack/shell）
vul_id       # 从 PoC 继承（= poc.vulID）
name         # 从 PoC 继承
app_name     # 从 PoC 继承
app_version  # 从 PoC 继承
poc_attrs    # dict — 框架通过 inspect 自动收集的 PoC 非私有属性
params       # 请求参数
```

## 方法

| 方法 | 签名 | 说明 |
|------|------|------|
| `success` | `(result: dict)` | 标记成功，`status = SUCCESS`，存储 result |
| `fail` | `(error: str)` | 标记失败，`status = FAILED`，`error_msg = (0, error)` |
| `error` | `(error: str)` | 同 fail + 设置 `self.expt` |
| `is_success` | `() → bool` | 返回 `bool(self.status)` |
| `show_result` | `()` | 打印键值对，URL/IP 类键自动 `mosaic()` 脱敏 |
| `to_dict` | `() → dict` | 返回 `self.__dict__` |

## 使用

```python
def parse_output(self, result):
    output = Output(self)
    if result:
        output.success(result)       # 传入 {"VerifyInfo": {...}} 或 {"AttackInfo": {...}}
    else:
        output.fail('target is not vulnerable')
    return output
```
