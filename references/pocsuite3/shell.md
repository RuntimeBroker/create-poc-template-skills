# Shell Payload 与 Shellcode

> `_shell()` 方法中需要反弹/绑定 Shell 或生成 Shellcode 时查阅。

## 反向 Shell

```python
from pocsuite3.api import REVERSE_PAYLOAD
# 反向连接 Payload 列表（GTFOBins 风格）
```

Shell 模式下监听地址的占位符机制：参数值中的 `{0}` → `connect_back_host`，`{1}` → `connect_back_port`。

## 绑定 Shell

```python
from pocsuite3.api import BIND_PAYLOAD, bind_shell, bind_tcp_shell, bind_telnet_shell

BIND_PAYLOAD       # 绑定 Shell Payload 列表
bind_shell          # 通用绑定 Shell
bind_tcp_shell      # TCP 绑定
bind_telnet_shell   # Telnet 绑定
```

## Shellcode 生成

```python
from pocsuite3.api import generate_shellcode_list

generate_shellcode_list(
    listener_ip=get_listener_ip(),
    listener_port=get_listener_port(),
    os_target=OS.LINUX,           # OS.LINUX / OS.WINDOWS
    os_target_arch=OS_ARCH.X86   # OS_ARCH.X86 / OS_ARCH.X64
)
```

## Shellcode / WebShell 字典

```python
from pocsuite3.api import OSShellcodes, WebShell

OSShellcodes   # 各类 OS Shellcode 集合（Linux/Windows x86/x64）
WebShell       # WebShell 工具
```

## 编码工具

```python
from pocsuite3.api import encoder_bash_payload, encoder_powershell_payload

encoder_bash_payload(cmd)        # Bash 命令编码
encoder_powershell_payload(cmd)  # PowerShell 命令编码
```

## 监听地址

```python
from pocsuite3.api import get_listener_ip, get_listener_port, DEFAULT_LISTENER_PORT

get_listener_ip()       # = conf.connect_back_host
get_listener_port()     # = conf.connect_back_port
DEFAULT_LISTENER_PORT   # 6666
```
