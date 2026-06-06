# SatanDS DuShengCDN Commercial Releases

DuShengCDN 商业版二进制发布仓库。本仓库只发布可交付安装包、安装脚本、SHA-256 校验文件和 Release 元数据，不发布商业源码。

> 商业源码位于私有仓库；客户服务器、Agent 和 DNS Worker 不需要 GitHub token，也不应该持有授权签发私钥。

## 一键安装 Server

### 最新正式版

在新的 Linux 服务器上以 `root` 或具备 `sudo` 权限的用户执行：

```bash
curl -fsSL https://github.com/SatanDS/SatanDS-DuShengCDN-releases/releases/latest/download/install-commercial.sh | bash
```

> GitHub 的 `latest` 只会指向最新正式 Release。若当前商业版仍是 Pre-release，请使用下面的“指定版本安装”命令。

安装脚本会自动完成：

- 下载当前正式版 `dushengcdn-server-linux-amd64` 或 `linux-arm64`。
- 下载同名 `.sha256` 并校验二进制完整性。
- 安装到 `/opt/dushengcdn`。
- 创建 systemd 服务 `dushengcdn`。
- 默认监听 `3010` 端口。
- 默认使用 SQLite 数据库 `/opt/dushengcdn/data/dushengcdn.db`。
- 写入在线授权地址 `https://www.satandu.com`。
- 开启商业授权在线激活与 72 小时租约，并在到期前 6 小时自动续约。

安装完成后访问：

```text
http://服务器IP:3010
```

默认管理员账号：

```text
用户名：root
密码：安装结束时输出的 Initial root password
```

也可以随时在服务器上查看初始密码：

```bash
grep '^DUSHENGCDN_INITIAL_ROOT_PASSWORD=' /opt/dushengcdn/dushengcdn.env
```

## 带授权安装

如果已经拿到商业授权 token，可以安装时直接写入：

```bash
curl -fsSL https://github.com/SatanDS/SatanDS-DuShengCDN-releases/releases/latest/download/install-commercial.sh | bash -s -- \
  --license-token '你的商业授权token'
```

安装后也可以进入面板：

```text
系统治理 -> 商业授权
```

粘贴 `dscdn_license_v1...` 授权 token，面板会向 `https://www.satandu.com` 发起在线激活，成功后获得短期签名租约。租约默认有效 72 小时，到期前 6 小时自动续约。

## 常用安装参数

自定义端口：

```bash
curl -fsSL https://github.com/SatanDS/SatanDS-DuShengCDN-releases/releases/latest/download/install-commercial.sh | bash -s -- \
  --http-port 3010
```

自定义安装目录和服务名：

```bash
curl -fsSL https://github.com/SatanDS/SatanDS-DuShengCDN-releases/releases/latest/download/install-commercial.sh | bash -s -- \
  --install-dir /opt/dushengcdn \
  --service-name dushengcdn
```

### 指定版本安装

```bash
VERSION='v1.0.27-private.27-g73acbcf9072c'
curl -fsSL "https://github.com/SatanDS/SatanDS-DuShengCDN-releases/releases/download/${VERSION}/install-commercial.sh" | bash -s -- \
  --version "${VERSION}"
```

把 `VERSION` 替换为 Release 页面中要安装的版本号。Pre-release 版本不会出现在 `/releases/latest/download/...` 里，必须显式指定版本。

## 服务管理

查看运行状态：

```bash
systemctl status dushengcdn --no-pager
```

查看实时日志：

```bash
journalctl -u dushengcdn -f
```

重启面板：

```bash
systemctl restart dushengcdn
```

查看配置文件：

```bash
cat /opt/dushengcdn/dushengcdn.env
```

## 安装 Agent

在面板创建节点后，复制面板给出的发现 token 或节点 token，在边缘节点服务器执行：

```bash
curl -fsSL https://github.com/SatanDS/SatanDS-DuShengCDN-releases/releases/latest/download/install-agent.sh | bash -s -- \
  --server-url 'http://面板服务器IP:3010' \
  --discovery-token '面板中的发现token'
```

也可以使用节点专属 token：

```bash
curl -fsSL https://github.com/SatanDS/SatanDS-DuShengCDN-releases/releases/latest/download/install-agent.sh | bash -s -- \
  --server-url 'http://面板服务器IP:3010' \
  --agent-token '节点专属token'
```

Agent 会安装 OpenResty 运行依赖，创建 systemd 服务，拉取面板配置，应用代理服务配置，并持续上报心跳、版本、资源和请求观测数据。

## 安装 DNS Worker

在面板创建 DNS 响应端后，复制 DNS Worker token，在需要提供权威 DNS 响应的服务器执行：

```bash
curl -fsSL https://github.com/SatanDS/SatanDS-DuShengCDN-releases/releases/latest/download/install-dns-worker.sh | bash -s -- \
  --server-url 'http://面板服务器IP:3010' \
  --token 'DNS响应端token'
```

DNS Worker 默认监听 UDP/TCP `53`，会拉取只读调度快照，按来源 IP、国家/地区、节点池权重和健康状态返回合适的边缘 IP，并上报 DNS 查询 rollup 供面板观测。

## 商业版功能

商业版包含完整源码构建出的二进制能力：

- 商业授权：客户侧支持安装商业授权 token、查看授权状态、展示节点/站点额度、在线激活。
- 商业更新：面板右上角版本入口检查本仓库 Release。
- 全球态势板：节点在线率、运行健康、窗口请求、资源压力、带宽峰值和缓存命中率总览。
- 网站配置：反向代理站点、多域名、源站池、负载均衡、缓存策略、CC 防护、地区限制、认证配置和发布流程。
- 配置发布：支持草稿变更、发布摘要、配置版本、激活、历史记录和回滚。
- Agent/OpenResty：节点自动注册、配置拉取、OpenResty reload、失败回滚、健康上报、缓存清理和预热。
- 边缘资源：节点池、公网 IP 池、池内权重、排空状态、调度开关、节点健康和最近应用状态。
- 智能解析：Cloudflare 自动解析、本地自建权威 DNS、多节点智能解析、GSLB 调度、权重分流和冷却防抖。
- DNS 观测：查询量、成功率、动态解析、错误查询、SERVFAIL/NXDOMAIN 趋势、来源作用域、响应目标和 Worker 健康。
- DNS Worker：权威 DNS UDP/TCP 响应、快照一致性、GeoIP/CIDR 匹配、查询限速、UDP 截断保护和 NS/Glue 辅助检查。
- TLS 证书：证书托管、ACME 申请、域名资产管理和站点证书绑定。
- 访问观测：访问日志、请求趋势、TOP URL/IP/地区、节点可用率、资源快照和健康事件。
- 系统治理：个人设置、运行参数、商业授权、认证与安全、数据保留和品牌公告集中管理。
- 生产安全：Release 二进制发布、SHA-256 校验、构建裁剪、混淆、水印和不下发 GitHub token 的更新链路。
- 
- 源站填写说明：

* `源站` 页面维护的是可复用地址目录，只填写 IP、域名或主机名，例如 `10.0.0.10`、`origin.internal`，不要填写协议和端口。
* `规则配置` 里的 `源站地址` 需要填写完整 URL，协议和端口都在这里配置，例如 `https://origin.internal:443`。
* 多源站负载均衡时，每行一个完整 URL，并保持相同协议；多源站模式不要填写 path 或 query。
* 界面中已统一使用 `源站地址` 命名；旧文档或旧习惯里的“上游地址”在这里都对应 `源站地址`。

GeoIP 地区限制说明：

* GeoIP 地区限制基于节点侧 OpenResty 实时执行，用于按国家或地区代码放行或拦截访问；功能不依赖 Cloudflare 橙云，适合自建 CDN 节点直接使用。
* 进入 `网站配置` -> 选择站点 `配置` -> `地区限制` 分区，可以按国家或地区代码限制访问。
* `拦截列表内地区`：列表内地区返回 `403`，无法识别地区的请求继续放行。
* `只允许列表内地区`：只有列表内地区可以访问，无法识别地区的请求也会返回 `403`。
* 国家或地区代码使用 ISO 3166-1 两位代码，例如 `CN`、`US`、`HK`，可一行一个，也可以用逗号分隔。
* 地区识别默认依赖 Agent 节点本地 `GeoLite2-Country.mmdb` 数据库，不再要求 Cloudflare 橙云或 `CF-IPCountry` 请求头；OpenResty 会按真实客户端 IP 查询国家码。
* 真实 IP 优先读取 `CF-Connecting-IP`、`X-Real-IP`、`X-Forwarded-For`，最后使用连接 IP；前置 HTTPS 反代需要正确透传这些请求头。
* OpenResty 会缓存 `IP -> 国家码`，默认缓存有效识别结果 24 小时；本地库和在线 API 都查不到时会短暂缓存未知结果，避免每个请求重复查库。
* Agent 会自动下载并更新 Country 数据库，默认来源为 `https://raw.githubusercontent.com/Loyalsoldier/geoip/release/GeoLite2-Country.mmdb`，默认路径为 Agent 数据目录下的 `var/lib/dushengcdn/geoip/GeoLite2-Country.mmdb`，默认每 24 小时检查更新一次。
* 如你后续搭建自己的在线 IP 精确查询服务，可在 Agent 配置里设置 `geoip_lookup_api_url` 和 `geoip_lookup_api_token`，或安装时追加 `--geoip-api-url`、`--geoip-api-token`；查询顺序是先本地 GeoIP，未识别到国家码时再请求 API。API 返回 JSON 中的 `country_code`、`countryCode`、`iso_code`、`isoCode` 或 `country` 字段均可识别。
* 修改地区限制后需要发布并激活新配置，Agent 应用 OpenResty 配置后才会在节点侧生效。

站点 CC 防护说明：

* CC 防护是站点级入口，默认关闭；里面包含访问频率防护、计算验证和恶意请求规则。
* 进入 `网站配置` -> 选择站点 `配置` -> `CC 防护`，可以为单个站点启用短时间高频访问识别、命中后拦截或转入计算验证，并按需打开本地恶意请求规则。
* 节点访问处理顺序为：真实 IP 识别 -> GeoIP 地区限制 -> 恶意请求规则 -> CC 频率判断 -> 计算验证 -> 反向代理源站。
* CC 防护、计算验证和恶意请求规则里的 IP/CIDR 白名单、黑名单、排除规则均支持 IPv4 与 IPv6，例如 `203.0.113.0/24`、`2001:db8::/32`。
* CC 频率命中可选择 `观察模式`、`拦截模式` 或 `转入计算验证`；转入计算验证时会使用同一入口下方的计算验证算法、难度、白名单和黑名单配置。
* 恶意请求规则支持观察/拦截模式，内置规则包括 SQL 注入、XSS、路径穿越、敏感路径扫描和常见恶意工具 User-Agent。
* 白名单、排除名单和自定义拦截规则按需添加；路径支持精确匹配，也支持以 `*` 结尾的前缀匹配，例如 `/api/public/*`。
* 当前恶意请求规则是轻量本地规则引擎，不是完整 ModSecurity / OWASP CRS；优点是部署简单、资源占用低，复杂规则集可后续再扩展。
* 修改 CC 防护后需要发布并激活新配置，Agent 应用 OpenResty 配置后才会在节点侧生效。命中规则发布到节点并产生新访问后，可在「观测计量」->「访问明细」的状态码旁查看 `!` 了解原因。

界面交互说明：

* 操作结果提示已统一为右上角浮层，提示内容会展示更具体的错误信息，按内容自动适配宽度，并在默认 8 秒后自动消失。
* 删除、回滚、禁用等需要确认的高风险操作会使用页面居中的主题化确认弹窗，避免浅色主题白底白字或深色主题黑底黑字。
* 节点详情页会静默刷新运行状态，不再因为自动刷新反复显示“刷新中”；只有手动点击更新、同步等按钮时才显示操作中的状态。
* 节点 IP 会优先结合 `X-Forwarded-For`、`X-Real-IP`、`CF-Connecting-IP` 等反代头识别真实公网 IP，所以 HTTPS 反代必须保留上面的请求头配置。

## 升级

面板右上角版本入口会检查本仓库最新正式 Release。生产环境建议优先使用正式版；Pre-release 可以手动指定版本安装或升级。

手动重新安装到同一路径时，安装脚本会保留已有 `/opt/dushengcdn/dushengcdn.env` 和数据目录：

```bash
curl -fsSL https://github.com/SatanDS/SatanDS-DuShengCDN-releases/releases/latest/download/install-commercial.sh | bash
```

升级前建议至少保留一份数据备份：

```bash
systemctl stop dushengcdn
cp -a /opt/dushengcdn/data /opt/dushengcdn/data.backup.$(date +%Y%m%d%H%M%S)
systemctl start dushengcdn
```

## 界面预览

新版界面预览图将使用雾面玻璃主界面、网站配置、本地自建解析、边缘资源、授权验证和智能解析配置页截图。

为避免公开泄露客户信息，预览图上传前请先打码：

- 右上角用户名。
- 真实域名、IP、源站地址。
- 授权 token、客户名称、机器指纹和租约 ID。
- 节点 token、DNS Worker token、发现 token。

建议图片文件放在：

```text
docs/assets/readme/dashboard-overview.png
docs/assets/readme/site-config.png
docs/assets/readme/authoritative-dns.png
docs/assets/readme/edge-resources.png
docs/assets/readme/license-status.png
docs/assets/readme/smart-dns-weights.png
```

## 安全说明

- 本仓库是公开二进制分发仓库，不包含商业源码。
- 不要把商业源码仓库访问权限、GitHub PAT、Release 写入 token 或授权签发私钥交给客户服务器。
- 客户服务器只需要授权 token；授权私钥只应存在开发者签发环境或私有 CI Secret。
- 离线授权可以提高门槛，在线激活和短期租约用于限制泄露授权、欠费停用和异常机器数。

## 支持

商业授权、版本升级、部署问题和授权续约请联系 SatanDu / SatanDS。
