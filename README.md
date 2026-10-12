# Clash 和 Surge 远程规则集

本仓库根据 `data/emby-endpoints.json` 自动生成媒体服务域名规则：

- `直连`：29 个域名，生成到 `emby-direct`
- `非直连`：21 个域名，生成到 `emby-proxy`

规则使用域名后缀匹配：例如 `emby.example.com` 会匹配该域名本身及其子域名。源数据中的端口仅用于记录服务入口，Clash 和 Surge 的这些远程规则文件本身不匹配端口。

## 文件

```text
data/emby-endpoints.json       # 唯一数据源，后续只修改这里
rules/clash/emby-direct.yaml   # Clash 直连规则
rules/clash/emby-proxy.yaml    # Clash 代理规则
rules/surge/emby-direct.list   # Surge 直连规则
rules/surge/emby-proxy.list    # Surge 代理规则
scripts/build_rules.py         # 生成器和校验器
```

## Clash 使用方式

把下面的 `OWNER/REPO` 替换成你的 GitHub 用户名和仓库名：

```yaml
rule-providers:
  emby-direct:
    type: http
    behavior: classical
    format: yaml
    url: https://raw.githubusercontent.com/OWNER/REPO/main/rules/clash/emby-direct.yaml
    path: ./ruleset/emby-direct.yaml
    interval: 86400
  emby-proxy:
    type: http
    behavior: classical
    format: yaml
    url: https://raw.githubusercontent.com/OWNER/REPO/main/rules/clash/emby-proxy.yaml
    path: ./ruleset/emby-proxy.yaml
    interval: 86400

rules:
  - RULE-SET,emby-direct,DIRECT
  - RULE-SET,emby-proxy,PROXY
```

## Surge 使用方式

在 `[Rule]` 中加入：

```ini
RULE-SET,https://raw.githubusercontent.com/OWNER/REPO/main/rules/surge/emby-direct.list,DIRECT
RULE-SET,https://raw.githubusercontent.com/OWNER/REPO/main/rules/surge/emby-proxy.list,PROXY
```

其中 `PROXY` 可以替换为你的 Surge 策略组名称。

## 更新规则

只修改 `data/emby-endpoints.json`，然后运行：

```bash
python scripts/build_rules.py
python scripts/build_rules.py --check
```

GitHub Actions 会在提交时自动检查生成文件是否与数据源一致。

## 手动规则分支

`manual-rules` 分支另含你手动整理的规则，原有 `main` 分支规则不受影响。源数据在 `data/manual-rules.json`，生成文件按策略拆分，并保留原始 `DOMAIN` / `DOMAIN-SUFFIX` 匹配类型。使用 `python scripts/build_manual_rules.py` 生成，或加 `--check` 校验。

Clash 引用方式（把规则放在原 Emby 规则集之前，以便手动规则优先命中）：

```yaml
rule-providers:
  manual-proxy:
    type: http
    behavior: classical
    format: yaml
    url: https://raw.githubusercontent.com/youko-nobody/rules/manual-rules/rules/clash/manual-proxy.yaml
    path: ./ruleset/manual-proxy.yaml
    interval: 86400
  manual-direct:
    type: http
    behavior: classical
    format: yaml
    url: https://raw.githubusercontent.com/youko-nobody/rules/manual-rules/rules/clash/manual-direct.yaml
    path: ./ruleset/manual-direct.yaml
    interval: 86400

rules:
  - RULE-SET,manual-proxy,Emby代理
  - RULE-SET,manual-direct,DIRECT
```

Surge 在 `[Rule]` 中添加：

```ini
RULE-SET,https://raw.githubusercontent.com/youko-nobody/rules/manual-rules/rules/surge/manual-proxy.list,Emby代理
RULE-SET,https://raw.githubusercontent.com/youko-nobody/rules/manual-rules/rules/surge/manual-direct.list,DIRECT
```

代理规则放在直连规则之前，可确保 `myg.0008866.xyz` 先命中精确代理规则，再由 `0008866.xyz` 后缀规则处理其余子域名。

## 发布到 GitHub

```bash
git init
git add .
git commit -m "Add Clash and Surge remote rule sets"
git branch -M main
git remote add origin https://github.com/OWNER/REPO.git
git push -u origin main
```

仓库公开后，规则文件即可通过 `raw.githubusercontent.com` 地址远程订阅。
