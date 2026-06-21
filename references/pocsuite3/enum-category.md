# POC_CATEGORY + PLUGIN_TYPE

> 设置 `category` / `protocol` / 插件 `type` 时查阅。

## POC_CATEGORY

### EXPLOITS（漏洞利用）
```python
POC_CATEGORY.EXPLOITS.REMOTE      # Remote — 远程利用
POC_CATEGORY.EXPLOITS.LOCAL       # Local  — 本地利用
POC_CATEGORY.EXPLOITS.WEBAPP      # WebApp — Web 应用
POC_CATEGORY.EXPLOITS.DOS         # DoS    — 拒绝服务
```

### TOOLS（工具）
```python
POC_CATEGORY.TOOLS.CRACK          # Crack  — 凭证破解
```

### PROTOCOL（协议，设置 `protocol` 属性用）
```python
POC_CATEGORY.PROTOCOL.HTTP        # Http       FTP       # Ftp
POC_CATEGORY.PROTOCOL.SSH         # Ssh        TELNET    # Telnet
POC_CATEGORY.PROTOCOL.REDIS       # Redis      SMTP      # SMTP
POC_CATEGORY.PROTOCOL.DNS         # DNS        SNMP      # SNMP
POC_CATEGORY.PROTOCOL.SMB         # SMB        MQTT      # MQTT
POC_CATEGORY.PROTOCOL.MYSQL       # MySQL      RDP       # RDP
POC_CATEGORY.PROTOCOL.UPNP        # UPnP       AJP       # AJP
POC_CATEGORY.PROTOCOL.XMPP        # XMPP       WINBOX    # Winbox
POC_CATEGORY.PROTOCOL.MEMCACHED   # Memcached  BACNET    # BACnet
POC_CATEGORY.PROTOCOL.T3          # T3 (WebLogic)
```

非 HTTP PoC 示例：
```python
class RedisPOC(POCBase):
    protocol = POC_CATEGORY.PROTOCOL.REDIS
    protocol_default_port = 6379
```

## PLUGIN_TYPE

```python
PLUGIN_TYPE.TARGETS   # 'targets' — 自定义目标来源
PLUGIN_TYPE.POCS      # 'pocs'    — 自定义 PoC 加载
PLUGIN_TYPE.RESULTS   # 'results' — 自定义结果处理
```
