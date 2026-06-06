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
