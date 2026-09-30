# Clash 和 Surge 远程规则集

本仓库根据 `data/emby-endpoints.json` 自动生成媒体服务域名规则：

- `直连`：29 个域名，生成到 `emby-direct`
- `非直连`：21 个域名，生成到 `emby-proxy`

规则使用精确域名匹配。源数据中的端口仅用于记录服务入口，Clash 和 Surge 的这些远程规则文件本身不匹配端口。

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
