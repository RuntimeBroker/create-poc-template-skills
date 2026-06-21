# Nuclei File 协议

> 扫描本地文件系统。需 `-file` 标志。

```yaml
file:
  - extensions:
      - all               # 所有非默认排除扩展
      - txt
    denylist: [go, py]
    max-size: 5242880      # 默认 5MB
    no-recursive: false
    extractors:
      - type: regex
        name: api-key
        regex: ["AIza[0-9A-Za-z\\-_]{35}"]
```

## 运行

```bash
nuclei -t file.yaml -file -target /path/to/scan
nuclei -t file.yaml -file -target single.txt
```

## 参数

| 参数 | 说明 | 默认 |
|------|------|------|
| `extensions` | 匹配扩展名，`all`=全部 | — |
| `denylist` | 排除扩展 | — |
| `max-size` | 单文件最大字节 | 5242880 (5MB) |
| `no-recursive` | 禁用递归 | false |

默认排除（共 65 种，不对这些扩展名发起扫描）：`3g2, 3gp, 7z, apk, arj, avi, axd, bmp, css, csv, deb, dll, doc, drv, eot, exe, flv, gif, gifv, gz, h264, ico, iso, jar, jpeg, jpg, lock, m4a, m4v, map, mkv, mov, mp3, mp4, mpeg, mpg, msi, ogg, ogm, ogv, otf, pdf, pkg, png, ppt, psd, rar, rm, rpm, svg, swf, sys, tar, tar.gz, tif, tiff, ttf, txt, vob, wav, webm, wmv, woff, woff2, xcf, xls, xlsx, zip`

> `txt` 也在默认排除里——文本文件太普遍，通常不会敏感。如需扫描 `.txt`，在 `extensions` 里显式加上。
