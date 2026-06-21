# create-poc-template

[中文](README_CN.md)

A skill for AI coding agents, containing the full development reference for Pocsuite3 and Nuclei — so agents stop hallucinating API calls when writing PoC and exploit scripts.

## Why

After watching various AI coding tools write dozens of PoCs, a pattern emerged: the agent understands vulnerability logic, constructs payloads, and structures code just fine. Then it calls `self.options.get()` instead of `self.get_option()`, misspells `condition` in a Nuclei matcher, or invents POCBase attributes that don't exist.

This isn't a capability problem. It's a reference problem. The agent has no accurate docs at hand, and whatever blog posts search engines return are usually outdated or wrong.

The fix: tear apart the framework source and official docs into small, precisely searchable pieces the agent can load on demand.

## Contents

### Pocsuite3 (verified against v2.1.0 source, 10 files)

Cross-referenced against `api/__init__.py`, `lib/core/poc.py`, `lib/core/enums.py`, and `lib/utils/__init__.py` from the `knownsec/pocsuite3` repo:

- **POCBase class**: all 22 declarable attributes, 13 runtime-injected attributes, every method with signatures
- **Output class**: 6 methods, full attribute list
- **Parameter system**: `_options()` pattern, all 8 Opt* types (including CLI limitations), 3 categories of auto-injected options, get/set methods
- **59 VUL_TYPE values**: from CODE_EXECUTION to OTHER, grouped by category
- **24 POC_CATEGORY entries**: EXPLOITS(4) + TOOLS(1) + PROTOCOL(19, with port mappings)
- **Shell & shellcode**: REVERSE_PAYLOAD, BIND_PAYLOAD, bind_shell variants, generate_shellcode_list, OSShellcodes, WebShell, bash/powershell encoders
- **Search engines**: ZoomEye, Seebug, Shodan, Fofa, Quake, Hunter, Censys
- **DNSLog**: CEye and Interactsh
- **Programmatic API & tools**: init_pocsuite/start_pocsuite, PHTTPServer, Nuclei template loader, remote files, plugin system
- **3 complete examples**: Webmin RCE (CVE-2019-15107), Flask SSTI RCE, Redis unauth — plus CLI usage and best practices

### Nuclei (verified against official docs, 17 files)

Pulled from every relevant page under `docs.projectdiscovery.io/templates/`:

- **Template structure**: id, info block, variables
- **HTTP split three ways**:
  - Basic: path variable table, headers/body, cookies, raw requests, unsafe, multi-request conditions
  - Payloads: batteringram/pitchfork/clusterbomb, Fuzzing (pre-condition/part/type/mode/keys/analyzer/time_delay)
  - Advanced: Race conditions, HTTP pipelining, connection pooling
- **DNS**: all record types, available part values
- **Code**: engine list, args/pattern, output references
- **Headless**: 20+ action quick-reference, selector syntax, waitdialog, login detection example
- **Network (TCP)**: hex input, read-size, TLS, multi-port
- **File**: extension filtering, denylist, default exclusions
- **Flow**: conditional short-circuit, JS orchestration (iterate/set/template/log), vhost enum example, internal matchers, Dedupe
- **Multi-Protocol**: protocol prefix variable table, DNS+HTTP+SSL example, comparison with Workflows
- **JavaScript protocol**: code/pre-condition/init nodes, 20 nuclei/* modules, SSH fingerprint/SSH bruteforce/init preload examples
- **Matchers**: 7 types + AND/OR/Negative/Internal/Global conditions + DSL response variables
- **Extractors**: 5 types + dynamic extractors (cross-request value sharing)
- **OOB**: Preprocessors (randstr) + OOB testing (interactsh, 3 match parts)
- **Helper Functions**: 70 DSL functions (encoding, hashing, string ops, random, datetime, JWT, DNS, deserialization)
- **Workflows**: generic, conditional, matcher-branched, nested chaining, shared context

## Why 28 files

The skill system's `references/` directory doesn't auto-load into context. The agent reads files only when it needs them. Writing an HTTP template means loading roughly 3 files and 200 lines, not swallowing 1200 lines of nuclei docs whole.

Most files are under 100 lines. The longest is the examples file at 277.

```
poc-dev/
├── SKILL.md                   Entry point: framework selection + doc index
└── references/
    ├── pocsuite3/             10 files
    │   ├── pocbase.md         POCBase class reference
    │   ├── output.md          Output class
    │   ├── options.md         Parameter system
    │   ├── enum-vul.md        59 vulnerability types
    │   ├── enum-category.md   POC_CATEGORY + PLUGIN_TYPE
    │   ├── shell.md           Shell/shellcode/webshell
    │   ├── search.md          7 search engines
    │   ├── dnslog.md          CEye + Interactsh
    │   ├── api.md             Programmatic API, remote files, tools
    │   └── examples.md        3 PoCs + CLI + best practices
    └── nuclei/                17 files
        ├── structure.md       Template skeleton
        ├── http-basic.md      GET/POST, raw, unsafe, cookies, multi-request
        ├── http-payloads.md   Payloads (3 attack modes) + Fuzzing
        ├── http-advanced.md   Race conditions, pipelining, pooling
        ├── dns.md             DNS protocol
        ├── code.md            Code protocol
        ├── headless.md        Browser automation
        ├── network.md         TCP raw bytes
        ├── file.md            Local file scanning
        ├── flow.md            Conditional flow + JS orchestration
        ├── multi-protocol.md  Cross-protocol variable sharing
        ├── javascript.md      JS runtime + 20 modules
        ├── matchers.md        7 matcher types
        ├── extractors.md      5 extractor types
        ├── oob.md             Preprocessors + OOB testing
        ├── functions.md       70 DSL functions
        └── workflows.md       Workflow orchestration
```

## Usage

This skill works with any AI coding tool that supports the skill format (Claude Code, Codex, and others).

It picks the framework based on context: simple HTTP detection → Nuclei YAML; complex Python logic or exploitation → Pocsuite3; browser automation → Nuclei Headless; non-standard protocols → Nuclei JavaScript.

## Known gaps

- **Non-HTTP protocol PoC examples are sparse**: Redis unauth is covered, but SMB/SSH/FTP have only API references, no full examples
- **Only SSH has examples for JavaScript protocol modules**: the other 19 modules are listed by name only — the official docs themselves are thin here
- **Version drift**: Pocsuite3 docs target v2.1.0 from GitHub source. Nuclei docs were pulled from the docs site and may be ahead of or behind your installed version

## License

MIT. See [LICENSE](LICENSE).
