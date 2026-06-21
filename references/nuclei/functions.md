# Nuclei Helper Functions（70 个）

> 编写 DSL 表达式时查阅。全部函数通过 `{{ }}` 包裹。

## 编码/解码（13 个）

```yaml
{{base64("hello")}}         # → aGVsbG8=
{{base64_decode("aGVsbG8=")}} # → hello
{{base64_py("hello")}}       # Python 风格 Base64（带 \n）
{{hex_encode("aa")}}         # → 6161
{{hex_decode("6161")}}       # → aa
{{url_encode("https://")}}   # URL 编码
{{url_decode("https%3A")}}   # URL 解码
{{html_escape("<body>")}}    # 转义 → &lt;body&gt;
{{html_unescape("&lt;")}}   # 反转义 → <
{{gzip("hello")}}            # GZip 压缩
{{gzip_decode(...)}}         # GZip 解压
{{zlib("hello")}}            # Zlib 压缩
{{zlib_decode(...)}}         # Zlib 解压
```

## 哈希/加密（6 个）

```yaml
{{md5("hello")}}                         # → 5d41402abc4b2a76b9719d911017c592
{{sha1("hello")}}                        # SHA1
{{sha256("hello")}}                      # SHA256
{{mmh3("hello")}}                        # MurmurHash3 → 整数
{{hmac("sha1", "test", "secret")}}       # HMAC
{{aes_gcm("AES256Key...", "plaintext")}} # AES-GCM（返回字节，常配合 hex_encode）
```

## 进制转换（4 个）

```yaml
{{bin_to_dec("0b1010")}}  # → 10
{{oct_to_dec("0o123")}}   # → 83
{{hex_to_dec("ff")}}      # → 255
{{dec_to_hex(7001)}}      # → 1b59
```

## 字符串操作（16 个）

```yaml
{{concat("Hello", 123, "world")}}         # → Hello123world
{{join("_", 123, "hello")}}               # → 123_hello
{{to_lower("HELLO")}}                     # → hello
{{to_upper("hello")}}                     # → HELLO
{{reverse("abc")}}                        # → cba
{{repeat("../", 5)}}                      # → ../../../../../
{{replace("Hello", "He", "Ha")}}          # → Hallo
{{replace_regex("He123llo", "(\\d+)", "")}}  # → Hello
{{trim("aaaHelloddd", "ad")}}             # → Hello
{{trim_left("aaaHello", "a")}}            # → Hello
{{trim_right("Helloaaa", "a")}}           # → Hello
{{trim_prefix("aaHello", "aa")}}          # → Hello
{{trim_suffix("Helloaa", "aa")}}          # → Hello
{{trim_space("  Hello  ")}}               # → "Hello"
{{remove_bad_chars("abcd", "bc")}}        # → ad
{{len("Hello")}}                          # → 5
```

## 字符串判断（12 个）

```yaml
{{contains("Hello", "lo")}}                     # → true
{{contains_all("Hello everyone", "lo", "every")}} # → true
{{contains_any("Hello", "abc", "llo")}}         # → true
{{starts_with("Hello", "He")}}                  # → true
{{ends_with("Hello", "lo")}}                    # → true
{{line_starts_with("Hi\nHello", "He")}}         # → true
{{line_ends_with("Hello\nHi", "lo")}}           # → true
{{equals_any(status_code, 200, 201)}}           # → true/false
{{regex("H([a-z]+)o", "Hello")}}               # → true
{{regex_any("pattern", "str1", "str2")}}       # → true if any match
{{regex_all("pattern", "str1", "str2")}}       # → true if all match
{{compare_versions('v1.0.0', '>v0.0.1', '<v1.0.1')}} # → true
```

## 随机生成（7 个）

```yaml
{{rand_base(5, "abc")}}           # → 从 "abc" 取 5 字符
{{rand_char("abc")}}              # → 随机单字符
{{rand_int(1, 10)}}               # → 随机整数 1~10
{{rand_text_alpha(10)}}           # → 随机 10 字母（可排除 badchars）
{{rand_text_alphanumeric(10)}}    # → 随机 10 字母数字
{{rand_text_numeric(10)}}         # → 随机 10 数字
{{rand_ip("192.168.0.0/24")}}    # → 192.168.0.171
```

## 日期时间（4 个）

```yaml
{{date_time("%Y-%m-%d %H:%M")}}           # → 2024-01-01 14:30
{{unix_time(10)}}                          # Unix 时间戳 + 10 秒
{{to_unix_time("2022-01-13T16:30:10Z")}}  # 字符串 → Unix 时间戳
{{wait_for(10)}}                           # 暂停 10 秒
```

## JSON/JWT（3 个）

```yaml
{{generate_jwt('{"user":"admin"}', "HS256", "secret")}}  # 生成 JWT
{{json_minify('{"a": 1, "b": 2}')}}                      # → {"a":1,"b":2}
{{json_prettify('{"a":1}')}}                              # 美化输出
```

JWT 算法：HS256, HS384, HS512, RS256, RS384, RS512, PS256, PS384, PS512, ES256, ES384, ES512, EdDSA, NONE

## DNS（2 个）

```yaml
{{resolve("localhost", 4)}}     # → 127.0.0.1
{{ip_format("127.0.0.1", 3)}}   # → 0177.0.0.01 (index 1~11)
```
格式：`4`/`a`/`6`/`aaaa`/`cname`/`ns`/`txt`/`srv`/`ptr`/`mx`/`soa`/`caa`

## 反序列化 Payload（2 个）

### Java（ysoserial）
```yaml
{{generate_java_gadget("commons-collections3.1", "wget http://{{interactsh-url}}", "base64")}}
```
Gadget：`dns`(URLDNS), `commons-collections3.1`, `commons-collections4.0`, `jdk7u21`, `jdk8u20`, `groovy1`
编码：`base64`(默认), `gzip-base64`, `gzip`, `hex`, `raw`

### .NET
```yaml
{{generate_dotnet_gadget("windows-identity", "calc", "binary", "base64-raw")}}
```
Gadget：`windows-identity`, `claims-principal`, `dataset`, `object-data-provider`, `text-formatting-runproperties`, `type-confuse-delegate`, `object-ref`, `veeam-crypto-keyinfo`, `axhost-state-dll`, `dll-reflection`, `viewstate`

格式化器：`binary`/`binaryformatter`, `soap`/`soapformatter`, `soap-exceptions`, `los`/`losformatter`
编码：`raw`, `hex`, `gzip`, `gzip-base64`, `base64-raw`

## 调试（1 个）

```yaml
{{print_debug(1+2, "Hello")}}   # → 输出 "3 Hello" 到 stdout
```
