# SatanDS DuShengCDN Commercial Releases

DuShengCDN 商业版二进制发布仓库。本仓库只发布可交付安装包、安装脚本、SHA-256 校验文件和 Release 元数据。


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
- DNS Worker：权威 DNS UDP/TCP 响应、快照一致性、GeoIP/CIDR/CN运营商 匹配分流、查询限速、UDP 截断保护和 NS/Glue 辅助检查。
- TLS 证书：证书托管、ACME 申请、域名资产管理和站点证书绑定。
- 访问观测：访问日志、请求趋势、TOP URL/IP/地区、节点可用率、资源快照和健康事件。
- 系统治理：个人设置、运行参数、商业授权、认证与安全、数据保留和品牌公告集中管理。
- 生产安全：Release 二进制发布、SHA-256 校验、构建裁剪、混淆、水印和不下发 GitHub token 的更新链路。



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
#### HTTPS 反代到管理端

生产环境建议在面板服务器上用 Nginx、OpenResty 或宝塔反向代理对外提供 HTTPS，再转发到 DuShengCDN 管理端端口。反代配置必须保留真实客户端 IP 头，否则节点注册和心跳经过反代时可能只能识别到内网 IP。

如果 Docker Compose 使用默认端口映射 `3000:3000`，反代目标是 `http://127.0.0.1:3000`；如果你改成 `3010:3000`，反代目标则改为 `http://127.0.0.1:3010`。

Nginx / OpenResty 示例：

```nginx
server {
    listen 443 ssl http2;
    server_name cdn.example.com;

    ssl_certificate /path/to/fullchain.pem;
    ssl_certificate_key /path/to/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

Nginx Proxy Manager 可在 `Proxy Hosts` -> `Add Proxy Host` 或 `Edit Proxy Host` 中配置：

* `Forward Hostname / IP` 填写面板容器所在机器，例如 `127.0.0.1`
* `Forward Port` 填写宿主机映射端口，例如你使用 `3010:3000` 时这里填写 `3010`
* 建议开启 `Websockets Support`，否则 Agent 的 WebSocket 通知可能退回到普通心跳轮询
* 在齿轮图标或 `Advanced` -> `Custom Nginx Configuration` 中填入下面的请求头配置

```nginx
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
```

宝塔面板可在网站的“反向代理 -> 配置文件”里把 `proxy_set_header` 相关配置加入对应的 `location /` 块，保存后重载 Nginx。

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

粘贴 `dscdn_license_v1...` 授权 token，面板会向授权服务器发起在线激活，成功后获得签名租约。

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

本地安装脚本会自动检测 Linux / macOS 环境，缺少 OpenResty 时会尝试通过系统包管理器安装；Docker 方式则使用内置 OpenResty 的 Agent 镜像。
你可以在控制面板的节点管理->详情->节点信息->节点标识与部署复制安装命令，或直接使用下面的脚本：

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

### 卸载 Agent

如需彻底卸载 Agent 并清空本地数据，可执行：

```bash
curl -fsSL https://raw.githubusercontent.com/SatanDS/DuShengCDN/main/scripts/uninstall-agent.sh | bash
```

卸载脚本会先停止并移除 `dushengcdn-agent.service`、删除整个 `/opt/dushengcdn-agent` 目录，不会删除本机 OpenResty。

在管理端删除在线节点时，Server 会通过 Agent 连接下发卸载指令；Agent 收到后会执行本机卸载流程并退出。节点离线时，面板只会删除节点记录，需要你后续在节点服务器上手动执行卸载脚本。

## 安装 DNS Worker

如果要让域名按每次 DNS 查询来源实时调度到不同边缘节点，需要在管理端左侧「流量调度-智能解析」创建 DNS Zone（创建托管域名） 和 DNS Worker（创建 DNS 响应端），然后打开「迁移向导」检查 Cloudflare 模式网站的 Zone、Worker、公网探测和 GSLB 准备状态；满足条件时可点击「一键切换」，也可以在网站详情「负载均衡」里手动切换到本地自建解析。之后把域名 NS 委派到 DNS Worker。完成注册商 NS 配置后，可以在 Zone 详情点击「检查委派」确认公网 NS 是否匹配；如果使用 `ns1.example.com` 这类 Zone 内 NS，还需要在注册商配置 Glue/主机记录。面板本机可以同时部署 DNS Worker；使用 `scripts/install-server.sh` 部署面板时，脚本可默认自动创建名为 `DNS服务响应端` 的 Worker、探测公网 IP 并安装同机 Worker。手动或多机部署时，仍可复制 Token 后单独运行 DNS Worker 安装脚本、Docker命令。
在面板创建 DNS 响应端后，复制 DNS Worker token，在需要提供权威 DNS 响应的服务器执行：

```bash
curl -fsSL https://github.com/SatanDS/SatanDS-DuShengCDN-releases/releases/latest/download/install-dns-worker.sh | bash -s -- \
  --server-url 'http://面板服务器IP:3010' \
  --token 'DNS响应端token'
```

DNS Worker 默认监听 UDP/TCP `53`，会拉取只读调度快照，按来源 IP、国家/地区、节点池权重和健康状态返回合适的边缘 IP，并上报 DNS 查询 rollup 供面板观测。

脚本默认写入 `/opt/dushengcdn-dns-worker`，创建 `dushengcdn-dns-worker.service`，监听 UDP/TCP `53`，并把快照缓存保存在安装目录的 `data/dns-worker-snapshot.json`。启动服务前会检查默认监听端口是否已被其它进程占用；如果本机已有 `systemd-resolved`、`named`、`dnsmasq` 等本地 DNS 服务，请先停用/改端口，或用 `--listen PUBLIC_IP:53` 只绑定 Worker 公网地址。脚本会优先下载 GitHub Release 中的 DNS Worker 二进制；如果当前仓库还没有 Release，会自动安装 Go 并从源码构建，源码构建会把当前 Git 版本写入 Worker，避免版本显示为 `dev`。源码构建同样会复用本机已有 Go，并在自动下载 Go 时多源重试。脚本还会默认下载 Country MMDB 到 `data/geoip/GeoLite2-Country.mmdb`，用于国家代码节点池匹配；可用 `--geoip-database` 指向已有文件、用 `--geoip-database-url` 指定自建下载源，或用 `--no-geoip-download` 关闭自动下载。

如果 Worker 和面板在同一台机器，`--server-url` 可以使用面板本机可访问地址，`--listen` 建议显式绑定公网地址，例如 `--listen 203.0.113.10:53`。安装后用 `systemctl status dushengcdn-dns-worker`、`ss -lntup | grep ':53'`、`ss -lnuap | grep ':53'` 和 `dig @PUBLIC_IP example.com SOA` 验证。

也可以在 Worker 主机运行只读诊断脚本，一次性检查服务、监听、快照、日志和 SOA/NS 查询：

```bash
cd /opt/dushengcdn
bash scripts/diagnose-dns-worker.sh --public-ip PUBLIC_IP --zone example.com
```

面板和 DNS Worker 同机部署时，正式切换 NS 前可运行闭环验收脚本：

```bash
cd /opt/dushengcdn
bash scripts/verify-authoritative-dns.sh --public-ip PUBLIC_IP --zone example.com
```

Docker 运行示例也可继续使用：

```bash
DUSHENGCDN_VERSION=v1.0.0
docker run -d --name dushengcdn-dns-worker --restart unless-stopped \
  -p 53:53/udp -p 53:53/tcp \
  -v dushengcdn-dns-worker-data:/data \
  -e DUSHENGCDN_DNS_WORKER_SERVER_URL=https://cdn.example.com \
  -e DUSHENGCDN_DNS_WORKER_TOKEN=YOUR_DNS_WORKER_TOKEN \
  -e DUSHENGCDN_DNS_WORKER_QUERY_RATE_LIMIT=200 \
  -e DUSHENGCDN_DNS_WORKER_UDP_RESPONSE_SIZE=1232 \
  ghcr.io/satands/dushengcdn-dns-worker:${DUSHENGCDN_VERSION:?set DUSHENGCDN_VERSION}
```

需要按国家代码匹配节点池时，再额外挂载本地 Country MMDB 并设置路径：

```bash
  -v /path/to/GeoLite2-Country.mmdb:/geoip/GeoLite2-Country.mmdb:ro \
  -e DUSHENGCDN_DNS_WORKER_GEOIP_DATABASE_PATH=/geoip/GeoLite2-Country.mmdb \
```

只使用来源 CIDR 或全局调度时可以省略 GeoIP。

触发DNS运营商分流库更新：
```bash
systemctl start dushengcdn-dns-worker-source-database-update.service
systemctl status dushengcdn-dns-worker-source-database-update.service
```
查看 7 天DNS运营商自动更新定时器：
```bash
systemctl list-timers '*source-database*'
```

生产环境建议至少部署两个 DNS Worker，并同时放行 UDP/TCP `53`。如果安装脚本或 Docker 启动提示 `address already in use` / 端口占用，先用 `ss -lntu '( sport = :53 )'` 或 `lsof -nP -i :53` 找到占用者；常见占用来自 `systemd-resolved`、`named` 或 `dnsmasq`。Worker 本地快照缓存会写入 SHA-256 checksum 元数据，启动加载时会校验完整性，并从快照中的 GSLB 防抖状态恢复最近可用选择；运行中产生的新防抖状态会随 heartbeat 批量回传 Server，同时兼容旧版本生成的裸快照 JSON。Worker 默认按来源 IP 每秒限制 `200` 次查询，并把 UDP 响应上限限制为 `1232` 字节；超大响应会设置 TC 位让递归解析器回退 TCP。安装脚本会默认准备本地 Country MMDB；如果使用 Docker 或源码方式部署且要按国家代码匹配 GSLB 节点池，需要自行配置本地 MaxMind Country MMDB。如果在节点池里配置来源 CIDR，则会优先按来源 IP/ECS 命中 `cidr:...` 作用域；启用 `weighted` 或 `load_aware` 时会追加 `|bucket:xx` 分流桶，未命中且无法识别国家时会回退到 `global` 作用域。

DNS 响应端上报心跳后，左侧「本地自建解析」会展示最近 24 小时的查询量、查询趋势、SERVFAIL/NXDOMAIN 趋势、响应端快照一致性、响应端查询延迟、可用率、错误率、最近公网探测健康状态、GeoIP 国家库加载状态、来源作用域、响应端/托管域名/站点维度、返回目标分布和当前调度状态，便于确认实时多节点智能解析是否按预期分流；「GSLB 调度状态」会列出每个站点、A/AAAA、`global`、`country:HK`、`cidr:203.0.113.0/24` 或 `global|bucket:42` 等来源作用域的当前实际目标、期望目标和防抖状态；「GSLB 调度模拟」可在真实流量到达前按站点、记录类型、来源 IP 和来源国家代码预演当前快照会返回的边缘 IP，并展示节点池匹配、候选节点、跳过节点、负载指标时间和节点多地探测摘要。这里的响应端延迟是 DNS 响应端本地处理真实 DNS 查询的耗时；节点多地探测 RTT 表示各边缘节点到响应端 NS 的主动探测耗时，默认只用于观测与排障；开启「按响应端探测结果筛选节点」后，才会影响本地自建解析选点。DNS 响应端列表的「探测」会由面板对响应端公网地址发起 UDP/TCP 53 SOA 查询，适合检查端口映射、防火墙和公网可达性；最近一次探测结果会保存在响应端列表和可用性面板中，并会作为迁移向导的切换准备条件。托管域名详情的委派检查用于确认注册商 NS 和 Glue 配置是否到位。

卸载 DNS Worker：

```bash
curl -fsSL https://raw.githubusercontent.com/SatanDS/DuShengCDN/main/scripts/uninstall-dns-worker.sh | bash
```

源站填写说明：

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

配置 Cloudflare 自动解析与防护:

*如果域名已经接入 Cloudflare，可以让 DuShengCDN 自动创建或更新 DNS 记录，并在节点离线或遇到攻击流量时自动调整解析策略。

*准备 Cloudflare API Token：

* 权限需要包含 `Zone Read` 和 `DNS Edit`
* Token 范围建议只授权给需要托管的 Zone
* 不建议使用 Global API Key
* 可以直接填写原始 Token，也兼容 `Bearer ...` 或包含 `api_token` / `apiToken` / `token` 的 JSON

在管理端操作：

1. 在左侧「Cloudflare 账号」准备 Cloudflare 账号。
2. 在节点详情中维护节点池、公网 IP 池、池内权重、调度开关和排空状态。
3. 新建网站配置时选择默认节点池，并开启 `创建时启用负载均衡`；已有站点可在详情页的 `负载均衡` 分区维护。
4. 选择 Cloudflare 账号，记录类型通常选择 `A`；IPv6 节点选择 `AAAA`。
5. `记录内容` 留空时，系统会自动选择该节点池中的在线公网 IP，并把 `自动选择在线节点 IP` 打开。
6. 如需跨 HK、EU 等多个节点池分流，在「负载均衡」分区启用 `多节点智能解析`，点击 `+` 逐行添加真实节点池、池权重、可选国家代码和来源网段，例如池名 `hk`、池权重 `80`、国家代码 `HK,TW`、来源网段 `203.0.113.0/24`。来源网段会优先于国家代码匹配。
7. 如需正常状态也隐藏源站，可开启 `常态开启 Cloudflare 代理`；如需攻击期自动切换，将 `攻击防护模式` 设为 `自动`，再选择 `Cloudflare` 或 `自定义清洗池`。

自动解析行为：

* 创建网站时会立即向 Cloudflare 创建或更新 DNS 记录。
* 后台每 1 分钟巡检一次已开启自动解析的规则。
* 开启 `自动选择在线节点 IP` 后，节点离线、代理服务不健康、节点被排空、关闭调度或节点公网 IP 池没有对应 A/AAAA 地址时会跳过该节点。
* 反向代理里的默认节点池是默认承载池：未启用多节点智能解析时，自动解析从这里选公网 IP，缓存清理/预热也下发到这里。启用多节点智能解析后，A/AAAA 返回 IP 改由「负载均衡」里的节点池权重决定，默认节点池仍作为缓存、攻击防护回退和运行时兜底。
* 自动解析可以按“健康优先（冷却防抖）”、节点池内权重或负载感知评分选择，并支持同步多个 A/AAAA 目标。健康优先只判断在线、代理服务健康、调度开关、排空状态和最近心跳；处理器、内存、连接数只属于负载感知。
* 多节点智能解析模式可绑定多个节点池，按来源网段、国家代码、池权重、节点池内权重、代理服务连接数、处理器压力、内存压力和负载阈值选择解析目标；网站配置里可维护最大连接数、最大处理器压力和最大内存压力。
* 本地自建解析模式下，选择按权重优先或按压力优先时，会按来源 IP/ECS 生成稳定分流桶，所以 HK 池权重 80、EU 池权重 20 这类配置会在不同来源桶之间形成接近 8:2 的解析答案分布；Cloudflare 模式只同步一组静态记录，不具备逐查询来源分流。
* 多节点智能解析会记录最近一次实际目标和期望目标，旧目标仍健康且冷却时间未到时不会反复切换。
* Cloudflare 同步模式不是逐请求实时调度，而是后台巡检重算并同步记录；实际流量还会受到解析缓存时间和运营商 DNS 缓存影响。
* 本地自建解析模式需要把域名 NS 委派到 DuShengCDN DNS 响应端（DNS Worker）；响应端会在查询时实时执行多节点智能解析。左侧「本地自建解析」的「迁移向导」可检查 Cloudflare 模式网站是否已有匹配托管域名、在线响应端、公网 UDP/TCP 53 探测和多节点策略，满足条件时可一键切换到本地自建解析；「GSLB 调度模拟」可按来源 IP 和国家代码预演当前快照返回目标，并解释节点候选/跳过原因、负载指标时间和节点多地探测摘要。节点多地探测默认仅用于观测；在设置页「本地解析运行参数」启用按响应端探测结果筛选节点后，无新鲜成功探测的边缘节点不会进入本地自建解析候选。托管域名详情可检查公网 NS 是否匹配，并在域名内 NS 需要 Glue/主机记录时提示。未配置本地 GeoIP 库时，国家代码匹配会回退到 `global` 作用域，详见 `docs/design/authoritative-dns-gslb.md`。
* ACME 申请证书可选择 Cloudflare 账号验证，也可选择本地自建解析托管域名验证；本地方式会临时写入 `_acme-challenge` TXT 记录并在验证后清理。
* 手动填写的 DNS 记录内容不会被后台覆盖；A/AAAA 可用逗号、空格或换行填写多个目标。
* 多域名规则默认会同步规则里的所有域名；单域名规则可在详情页手动指定记录名称。
* 删除规则时，如果该规则曾由 DuShengCDN 创建 DNS 记录，会尝试同步删除对应 Cloudflare DNS 记录。

攻击自动防护：

* `攻击防护模式` 设为 `自动` 后，系统会按最近 5 分钟请求聚合判断是否进入攻击期。
* 默认请求量阈值为 `20000`，默认错误率阈值为 `30%`。
* 防护提供方选 `Cloudflare` 时，攻击期会暂停多节点智能解析，多 A/AAAA 目标临时回到网站默认节点池，并强制同步 Cloudflare 橙云；指标恢复正常后，下一轮巡检回到原来的固定记录、默认节点池或多节点智能解析策略，并恢复 `常态开启 Cloudflare 代理` 的设置。
* 防护提供方选 `自定义清洗池` 时，攻击期会暂停 GSLB，只把 DNS 解析到指定节点/IP 池里的在线公网 IP，并关闭 Cloudflare 橙云代理，适合切到自有抗 D 清洗入口；指标恢复正常后自动回到正常调度。
* 阈值可在管理端设置项中调整：`CloudflareDDoSRequestThreshold` 和 `CloudflareDDoSErrorRateThreshold`。

注意：Cloudflare 自动解析只负责 DNS 记录与橙云状态，不会替代发布流程。反向代理配置修改后仍需发布并激活版本，Agent 才会拉取并应用 OpenResty 配置。

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
```text
docs/assets/readme/dashboard-overview.png
docs/assets/readme/site-config.png
docs/assets/readme/authoritative-dns.png
docs/assets/readme/edge-resources.png
docs/assets/readme/license-status.png
docs/assets/readme/smart-dns-weights.png
```

## 安全说明

- 本仓库是公开二进制分发仓库。

## 支持

商业授权、版本升级、部署问题和授权续约请联系 邮箱1371059663@qq.com。
