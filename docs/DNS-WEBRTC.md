# DNS 与 WebRTC 防护：Agent 排查、修复与验收

本指南与共享分流规则配套。**DNS 查询路径、普通连接路径、WebRTC 候选地址必须分别检查。** 规则列表不包含浏览器设置，安装扩展也不会替你配置 DNS。这里只提供公开方法，不代表维护者或阅读者的设备已经通过验收。参考资料核对日期：2026-11-01。

## 1. 先记录目标和现场

按 [AGENTS 第 1、2 步](../AGENTS.md) 记录现状并保存配置前像，聊天或公开回执只写脱敏结果。本仓库默认是国内直连、海外与未知代理，不是所有请求隐藏本地出口。

先通过 [TUN 前置检查](CLIENTS.md#tun-required)。只有系统代理时，引导使用者开启 TUN 并等待确认，不能执行后续接入或把非 TUN 的命令行结果作为最终验收。

不要为了取“关闭代理时的基线”中断用户现用连接。已有的运营商出口资料、用户授权的独立测试环境可辅助对照；拿不到可靠基线就标注，不能凭 IP 归属地推断无泄露。

## 2. DNS 参考策略

下表是这套规则的通用设计方向，需要按设备能力适配。国内直连属于有意的分流，不承诺对国内站点隐藏本机公网出口。

| 查询类别 | 解析路径 | 后续连接 |
| --- | --- | --- |
| localhost、LAN、private 域名 | 本机系统 / 局域网 DNS；适用时使用系统 mDNS | 本地或直连 |
| 用户明确指定的例外 | 用户认可的独立解析策略 | 对应例外策略 |
| 国内域名 | 国内 DoH，如阿里 `https://223.5.5.5/dns-query`、`https://223.6.6.6/dns-query` | DIRECT |
| 海外域名 | 海外 DoH，如 Cloudflare `https://1.1.1.1/dns-query`、`https://1.0.0.1/dns-query`，连接本身走用户自己的代理组 | PROXY |
| 未知公网域名 | 默认用经代理的海外解析器；失败不悄悄退回运营商明文 DNS | 优先代理；中国 IP 兜底取决于实际解析流程 |
| 节点服务器域名 | 单独的、用户认可的引导解析 | 必须在代理尚未建立时也能连接，避免递归依赖 |

DoH 加密查询与 DNS 经代理是两件事：直连 DoH 仍可让解析服务看到本地出口。只把地址换成 1.1.1.1 也不能证明走了代理。多个解析器如果并发查询，**每个**都必须符合对应的直连 / 代理策略，不能只检查第一个。

本仓库的 `cn-ip` 不是“所有 DNS 查询直连”的开关。应先匹配域名规则，再处理确实得到真实地址的 IP 规则。不要把 FakeIP 当真实中国 IP；IPv4 与 IPv6 混合答案也不应因为其中一个地址在中国就整体放行。

<a id="private-dns-preflight"></a>

### 内网解析前置检查

模板的 `system`、`server:system`、`force-syslib` 和 `dns-local` 都依赖目标设备实际的系统解析配置，名称不等于“仅在内网解析”。应用前检查系统解析器和分域解析：macOS 可只读查看 `scutil --dns`，Windows 查看 `Get-DnsClientServerAddress`，Linux 按环境用 `resolvectl status`。只记必要的解析器和路径，不公开整份输出。

- 已确认是可信 LAN / 分域解析器，且不会把这些查询继续发给公网时，可使用对应的本地解析路径；回环 stub 也要核对它的上游和本代理是否形成循环。
- 系统只有公网 DNS，或者不能查清上游时，**首次应用前暂停**。告诉用户“当前系统 DNS 不能保证内网域名不外发”，请其提供认可的 LAN / 分域解析方式；不能凭空猜一个路由器地址，也不能把公共 DoH 当作内网解析器。
- 用户明确不需要相关内网域名、并同意阻止这些域名的解析时，才可按实际客户端能力在本地拒绝相应 DNS 查询。这是会影响可用性的目标选择，必须列入方案和报告，不能静默开启。Mihomo 的 `rcode://refused`、sing-box DNS 的 `reject` 可作为能力依据；须保留用户例外并重新验收。Surge / Shadowrocket 按实际版本确认，不能照搬字段。
- 没有可接受的处理方案就保留原配置，状态为等待用户；不能因为没有实际路由器服务就把 private 查询不外发这一策略标为不适用。NXDOMAIN 只表示没有解析结果，不证明查询留在内网；没有观察物理线包时也不要宣称抓到了实际外发。

依据：[Mihomo DNS 类型](https://wiki.metacubex.one/config/dns/type/)、[sing-box DNS 动作](https://sing-box.sagernet.org/configuration/dns/rule_action/)。TUN 开启也不改变上述上游选择要求。

## 3. 各客户端如何修复 DNS

`examples/full/` 里的模板已经写好了下面的做法，优先从模板合并，再按本节核对。每个字段的作用和踩过的坑如下，以便用户配置与模板不同时也能判断。

### Mihomo / Clash Meta

模板：[full/mihomo.yaml](../examples/full/mihomo.yaml)。先看现有的 `dns` 段，不要新建第二段。

| 字段 | 作用 | 不这样做的后果 |
| --- | --- | --- |
| `respect-rules: true` | DNS 连接也走路由规则 | `1.1.1.1` 的查询可能直接从本地出口发出 |
| `proxy-server-nameserver` | 只用来解析节点域名，这里走直连 | 官方要求与 `respect-rules` 配对；缺了会陷入“解析节点要先连上节点”的循环 |
| `direct-nameserver` + `direct-nameserver-follow-policy: true` | 直连连接二次解析时仍按域名策略 | 内网域名可能被二次发到公网 DoH |
| `nameserver-policy` 的 `rule-set:sr-private: [system]` | 内网域名走系统 DNS | 内网名查询外泄，或解析失败 |
| `fake-ip-filter` 的 `rule-set:sr-private` | 内网域名拿真实地址 | 内网设备拿到 FakeIP，连不上 |
| `sniffer` + `parse-pure-ip: true` | 从 TLS / HTTP / QUIC 里取回域名 | 只有 IP 的流量匹配不到域名规则，落到 cn-ip 或默认代理 |
| 规则中的 DoH 地址（如 `IP-CIDR,1.1.1.1/32,PROXY,no-resolve`） | 明确每个解析器的路径 | 只写了首选、漏了备用时，备用解析器可能走错路 |

使用 TUN 时另查 DNS 劫持和 IPv6 路由是否覆盖目标应用。只开系统 HTTP 代理通常不能接管所有应用的 UDP DNS。GUI 客户端的“DNS 覆写”“追加系统 DNS”会改变上面的策略，按模板配置时要关闭。规则资源的 `proxy:` 只管下载文件，不能代替 DNS 出站策略。

字段定义以 [Mihomo DNS 文档](https://wiki.metacubex.one/config/dns/) 为准。旧 Clash 分支不要照搬新 Mihomo 的全部字段。

### Surge

模板：[full/surge.conf](../examples/full/surge.conf)。

- `[Host]` 用 `RULE-SET:地址 = server:解析器` 按名单分流查询（Mac 5.10.0+）。只有名单中的域名条目生效，自上而下首条命中。逗号分隔多个解析器需要 iOS 5.21.0+ / Mac 6.8.0+，所以模板只写首选。
- **`encrypted-dns-follow-outbound-mode = true` 必须开。** 默认情况下加密 DNS 的连接始终走 DIRECT，不经过规则。
- **开启后还有一个坑：** 如果匹配到的代理节点地址是域名，且没有可用的 `[Host]` IP 映射，这条 DNS 连接可能回退为 DIRECT。官方支持为节点域名增加 `[Host]` IP 映射，或使用 IP 地址节点；前者需要核实当前有效 IP，并在 IP 变化时维护。按 [AGENTS 第 2 步](../AGENTS.md) 说明代价并取得用户选择，不要替用户切换。应用后用 `surge-cli dns trace` 和连接记录核实，不能仅凭写了映射就报告通过。
- `always-real-ip` 让内网和节点域名拿真实地址。`hijack-dns = *:53` 让发往任意服务器的明文 DNS 也进入 Surge。
- `.local` 用 `server:force-syslib` 才能走 mDNS；`server:system` 在增强模式下走的是单播 DNS。这个选项只给 `.local` 用，官方提示用于普通域名可能造成递归。
- 不要给 cn-ip 加 `no-resolve`，否则尚未解析的域名会跳过它。

依据：[本地 DNS 映射](https://manual.nssurge.com/dns/local-dns-mapping.html)、[加密 DNS](https://manual.nssurge.com/dns/encrypted-dns.html)、[高级 DNS](https://manual.nssurge.com/dns/advanced.html)、[规则选项](https://manual.nssurge.com/rules/overview.html)。

### Shadowrocket

模板：[full/shadowrocket.conf](../examples/full/shadowrocket.conf)，**只做过文本结构检查，未在实体设备上验证**，要如实报告。字段含义取自社区维护的 [Shadowrocket 手册](https://github.com/LOWERTOP/Shadowrocket)，它不是官方文档，以实际 App 版本为准。

| 字段 | 作用 | 模板的写法 |
| --- | --- | --- |
| `dns-server` | DNS 覆写。普通 DoH 地址不会经过节点，要加 `#proxy` 才由默认节点转发 | Cloudflare DoH 加 `#proxy` |
| `fallback-dns-server` | 覆写失败或超过 2 秒时使用。留空等于系统 DNS | 同上，避免退回运营商解析 |
| `direct-dns-server` | 解析命中直连域名规则的域名 | 阿里 DoH |
| `proxy-dns-server` | **只解析节点域名**，不是“代理域名的 DNS” | 阿里 DoH，不依赖节点本身 |
| `[Host]` 的 `server:system` | 把指定域名交给系统 DNS | `*.lan`、`*.local`、`*.home.arpa` |

命中代理规则的域名默认由远端节点解析，不经过上面这些解析器。

**已知限制：** 手册没有写 `[Host]` 能引用规则集，所以 `private` 名单不能整体走本地解析，其中的域名命中直连规则后会发给 `direct-dns-server`。用户确实使用的内网域名，要逐条加入 `[Host]`。`dns-direct-system = true` 能让所有直连域名走系统 DNS，但国内查询也会一起改走系统解析器，模板没有采用。

手册里 `hijack-dns` 的例子是具体地址，模板的 `*:53` 没有例子佐证，要看实际日志确认。不要照搬 Surge 的嵌套 `extended-matching`：Shadowrocket 会把这个后缀当成地址的一部分。

### sing-box

模板：[full/sing-box.json](../examples/full/sing-box.json)，需要 1.14.0+。JSON 不能写注释，所以说明都放在这里。公共 `sing-box/*.json` 是规则集，不是 DNS 配置。

| 部分 | 作用 |
| --- | --- |
| `dns-local` 处理 private 和本地后缀 | 内网名不发到公网 DoH，也不拿 FakeIP |
| 名单内域名的 A / AAAA 返回 `dns-fake` | 连接时还能还原出域名，由路由决定去向 |
| `evaluate` → `match_response` + `sr-non-cn-ip` → `respond` | 未知域名先经代理查询；只要有一个答案不在中国就保持 FakeIP；全部在中国才返回真实地址 |
| `{"port": 53, "action": "hijack-dns"}` | 把 DNS 流量交给 DNS 模块，需要 TUN 才能接管系统查询 |
| 域名规则之后的两条 `sniff` | 没有域名的连接先嗅探，然后再过一遍域名规则 |
| cn-ip 放在嗅探之后 | 裸 IP 只有在所有域名判断都做完后才按 IP 判断 |
| `domain_resolver: dns-direct` | 直连出站和域名节点用它引导解析。每个域名节点都要加上，这是唯一允许改节点的字段。不加时 `sing-box check` 也能通过，但节点连不上（1.14.4 隔离实例实测） |
| `store_fakeip: true` | 重启后保留映射，否则旧 FakeIP 连接会失败 |

模板的 `dns-proxy` 只有一个上游，因为 1.14 每个服务器标签只绑定一个地址。字段以 [DNS 规则](https://sing-box.sagernet.org/configuration/dns/rule/)、[DNS 动作](https://sing-box.sagernet.org/configuration/dns/rule_action/) 和 [迁移文档](https://sing-box.sagernet.org/migration/) 为准，不要混用已弃用的写法。

## 4. 浏览器安全 DNS 与 IPv6

浏览器可能启用独立的“安全 DNS”或由企业策略指定解析器。先确定它是否绕过客户端的分域名策略：

- 如果由本地代理客户端统一负责 DNS，可让浏览器使用系统 / 客户端解析路径，再由客户端提供 DoH。关闭浏览器的独立 DoH 不等于关闭客户端的加密 DNS。
- 如果保留浏览器独立 DoH，必须检查该连接经过哪个出口、是否破坏 LAN / private 域名解析，以及是否与客户端策略冲突。
- IPv6 必须单独验证；IPv4 代理成功不能证明 IPv6 被接管。优先修正 IPv6 路由与 DNS，无法验证时明确说明范围，不把全局关闭 IPv6 当默认修复。
- DNS over TLS / HTTPS 不使用明文 53 端口；因此“没看到 UDP 53”不足以证明所有查询都按预期出站。

浏览器与系统 DNS 选项以实际版本和用户策略为准，不直接编辑正在使用的浏览器内部数据库或以脚本覆盖企业管理设置。

<a id="webrtc"></a>

## 5. WebRTC：使用 WebRTC Network Limiter

WebRTC 会建立 ICE 候选并尝试音视频 / 数据连接。仅配置 HTTP(S) 代理时，某些 WebRTC UDP 路径可能不受它控制；完整 VPN / TUN 则要看 UDP、IPv6 与路由覆盖，不能笼统说 VPN 只代理网页流量。候选可能是本地 host 地址、STUN 获得的 srflx 地址或 TURN 的 relay 地址；这些类型本身不等于泄露，必须与预期出口比较。参见 [RFC 8828](https://www.rfc-editor.org/rfc/rfc8828.html)。

Chrome / 支持该扩展的 Chromium 桌面浏览器使用 **WebRTC Network Limiter**：

1. 打开 [Chrome Web Store 官方条目](https://chromewebstore.google.com/detail/webrtc-network-limiter/npeicpdbkakmehahjeeohfdhnlpdklia)，核对名称、Google 开发者标识和扩展 ID `npeicpdbkakmehahjeeohfdhnlpdklia`，在目标浏览器配置档安装。
2. 打开扩展的 **Options / 选项**，选择对应策略值 `disable_non_proxied_udp` 的选项。**0.2.1.4 的界面名称是 Use my proxy server (if present)**（第四项），并没有 Disable non-proxied UDP 这串字；其他版本可能使用后一个名称。该版本的选项标识与已安装官方扩展代码已核对，不能因为名称不同误选其他项。版本或界面再变化时，先核对实际策略值，不凭中文翻译或单选项位置猜测。
3. 保存并重新打开测试页面；重启浏览器后再检查选项仍然生效。按实际使用需要核对无痕模式中的扩展权限；另一个浏览器配置档不自动继承安装。
4. 如果选项不能保存或被覆盖，检查扩展冲突、启用状态与企业策略。受管理策略限制时报告具体原因，不绕过组织控制。

Agent 操作不了浏览器时，一步步带用户做：安装前让用户核对名称和开发者；在选项页选中后截图发回；重启浏览器后再看一次选项。

**接管方式不同，效果也不同，不要从选项名字推断结果。** Chrome 的 [API 文档](https://developer.chrome.com/docs/extensions/reference/api/privacy) 只列出 `disable_non_proxied_udp` 这个取值，没有说明没有配置代理时的行为。

| 接管方式 | 要注意什么 |
| --- | --- |
| 系统或浏览器代理 | 浏览器能看到代理，这是该选项的主要场景。重点看 UDP 是否还有直连路径 |
| TUN / 增强模式 | 浏览器看不到代理，流量由隧道接管。重点看 UDP 和 IPv6 是否真的进了隧道，以及通话是否还能用 |
| 全隧道 VPN | 同 TUN。另查 VPN 是否覆盖 IPv6 |

无论哪种方式，都以下面第 6 节的 ICE 候选和真实通话结果为准。

官方条目说明，禁止非代理 UDP 并非安装后的默认配置，因此必须核对选项。它可能使实时通话改走较慢的路径或失败，需要做业务回归。[Chrome 隐私 API](https://developer.chrome.com/docs/extensions/reference/api/privacy) 定义了对应 IP handling policy。

该扩展限制 WebRTC 网络路径，并不等于关闭所有 WebRTC 功能，也不保护其他浏览器、原生 App、DNS、定位权限、账号或 Cookie。手机 Chrome、Safari、Firefox 等不能直接按桌面 Chrome 扩展方案验收；不支持时标为待适配，不自动安装其他插件或禁用整套音视频功能。

<a id="acceptance"></a>

## 6. 怎么验证，才算有证据

### 验收表

全程保持用户的代理开启。每项都要有实际输出作证据；没做的写“未验证”，确实不适用的写明理由。按 [AGENTS 的状态表](../AGENTS.md#执行范围与最终状态) 汇总；隔离实例的结果只覆盖该实例，不代表用户的 TUN、浏览器或手机已经通过。

未确认 TUN 时停在前置检查，不能继续本表并汇总成接入通过。非 TUN 隔离测试只可另列为诊断证据。

| 项目 | 怎么测 | 通过标准 |
| --- | --- | --- |
| 规则加载 | 见下方命令 | 完整接入要求五类规则加载；Surge 托管受限接入按 [CLIENTS](CLIENTS.md#limited-integration) 核对四类模块资源及主配置代理行为。能读到条目数的客户端，条目数大于 0，与 [index.json](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/index.json) 中 `datasets[].resources.<格式>.entries` 同一量级；读不到的每类至少命中一个样本，注明没读到条目数 |
| 分流命中 | 用下面的样本逐个查 | 每个样本命中预期规则。只看命中的规则，不看 HTTP 结果 |
| 网页可用性 | 按下方“真实请求”复查国内、海外 HTTPS 和适用的内网服务，对比改动前结果 | 原本可用的服务仍可用，无新增 DNS、连接或 TLS 错误；规则命中正确不能代替此项 |
| TUN、出口与 IPv6 | 核对运行状态和路由，再按下方“出口与 IPv6”验证实际目标应用 | TUN 确实接管目标流量，出口符合当前节点；IPv6 须有实际地址和路径证据，不能仅以 `curl -6` 成功判定 |
| DNS 路径 | 见“DNS 验证” | 海外和未知查询到达经代理的解析器；private 不进公网 DNS |
| WebRTC | 见“WebRTC 验证” | 按判断表记录结果，常用通话由用户确认 |
| 持久化 | 刷新一次订阅；征得用户同意后重启客户端 | 规则和 DNS 都还在。没有订阅的写“不适用”，没重启的写“重启后未验证” |

分流样本：

| 域名或地址 | 预期 |
| --- | --- |
| `www.baidu.com` | `direct` 名单，DIRECT |
| `www.google.com` | `proxy` 名单，走用户的代理组 |
| `doubleclick.net` | `reject` 名单，拒绝 |
| `router.asus.com` | `private` 名单，DIRECT |
| `114.114.114.114` | `cn-ip`，DIRECT |
| `192.168.1.1` | LAN，DIRECT |
| 用户自己的节点地址 | 节点例外，DIRECT |
| `example.com` | 2026-11-01 核对不在名单；本次先查实际规则资源，仍未知时才预期默认代理 |

`doubleclick.net` 会被拒绝，`router.asus.com`、`192.168.1.1`、`114.114.114.114` 可能连不上（没有这台设备，或地址不提供网页），这些都不算失败。

### 真实请求

与分流命中分开验收：代理保持开启，在改动前后用相同的接管方式访问国内、海外 HTTPS 各一个，以及用户实际使用的内网服务。可用 `https://www.baidu.com/`、`https://www.google.com/`，遇到站点自身限制时选用户认可的同类站点；不为 LAN 样本不存在而判失败。每个公网站点做 3 次有界请求，记录成功数、HTTP 状态和 DNS / TLS / 超时错误，单次不超过 15 秒，不无限重试到成功。

TUN 已确认后，按实际目标应用的路径测试，不额外加 HTTP 代理。必要时仅在本次 curl 进程排除环境 HTTP 代理（`--noproxy '*'`），系统 TUN 仍保持开启。例：

```sh
curl --noproxy '*' --connect-timeout 5 --max-time 15 -sS -o /dev/null -w '%{http_code}\n' https://www.baidu.com/
curl --noproxy '*' --connect-timeout 5 --max-time 15 -sS -o /dev/null -w '%{http_code}\n' https://www.google.com/
```

Windows PowerShell 使用 `curl.exe`，空输出设备改为 `NUL`，按实际 shell 处理引号。`curl` 不会自动读取 GUI 的系统 HTTP 代理，显式 `-x` 只可用于另列的核心诊断，不能绕过 TUN 要求，也不能替代目标浏览器的真实页面访问。改动后新增的间歇失败不能被“规则命中正确”抵消，先定位，无法修复则恢复受影响的配置；改动前已经失败的服务写明基线，不能宣称本次修复了它。

### 出口与 IPv6

1. 用目标浏览器打开 `https://www.cloudflare.com/cdn-cgi/trace`，记录脱敏的 `ip=` 和 `loc=`；命令行测试按上一节选实际接管方式。它只证明该次连接的出口，不能凭国家判断它属于所选节点。
2. **经过 HTTP 或由代理解析目标域名的 SOCKS 代理时，`curl -6` 不足以约束代理对目标的地址族。** HTTP CONNECT 把目标交给代理连接，不能把 curl 到回环代理的连接当作 IPv6 目标连接。使用经确认只有 AAAA 的测试端点，或能够明确返回 IPv6 观测地址的端点，并结合客户端目的地址 / 出站记录核对。例如 [ipify 官方 IPv6 端点](https://www.ipify.org/) `https://api6.ipify.org/`；仍要检查返回的确实是 IPv6，不能只看 HTTP 成功。没有合适端点或节点不支持则写原因和未验证项。
3. **设备接管另测。** 在目标浏览器或应用访问 IPv6 测试端点，结合 TUN / VPN 的 IPv6 路由和连接记录，确认目标流量被接管。显式 HTTP 代理的 IPv6 出口成功不证明系统 IPv6、UDP 或其他应用没有旁路。纯系统代理模式也不能宣称接管了整机 IPv6。
4. 目标网络没有 IPv6 时，设备 IPv6 连通性可记“不适用：当前网络无 IPv6”，同时写“有 IPv6 网络时的行为未验证”；有 IPv6 但测不到路径则是未验证。不要为通过验收关闭 IPv6 或切换节点。

依据：[curl 的 HTTP 代理机制](https://everything.curl.dev/usingcurl/proxies/http.html)、[curl `--ipv6`](https://curl.se/docs/manpage.html#-6)。

### 各客户端的查看方法

```sh
# Mihomo：地址和密钥取自用户的 external-controller / secret，不要写进回复。
curl -s --noproxy '*' -H "Authorization: Bearer $SECRET" http://127.0.0.1:9090/providers/rules   # ruleCount、updatedAt
# 先开着日志流再发测试请求，看 match RuleSet(sr-proxy) using 这类行；连接失败时看 dial DIRECT (match RuleSet/sr-private)。
curl -sN --max-time 30 --noproxy '*' -H "Authorization: Bearer $SECRET" 'http://127.0.0.1:9090/logs?level=info'
# /connections 只显示进行中的连接：短请求结束后就查不到，被拒绝的连接也查不到。

# Surge（Mac）：路径见 CLIENTS 的配置检查。不产生真实连接就能看规则判断。
surge-cli external-resource list
surge-cli rule match www.google.com
surge-cli rule explain www.google.com
surge-cli dns lookup www.google.com
surge-cli dns trace www.google.com

# sing-box：备份后把 log.level 临时改为 debug，测完恢复，并写进报告。
#   规则集：下载或更新时有 updated rule-set sr-proxy 行；1.14.4 的 Clash API /providers/rules 返回空列表，读不到条目数。
#   命中：match[N] rule_set=sr-proxy => route(PROXY)。落到默认出站时没有 match 行，看 outbound 连接行。
```

地址、端口和命令路径都替换为现场值。日志流在一个有界会话中观察，同时从另一会话发请求；30 秒期限后的 curl 超时是结束观察，不是路由失败。测试完关闭观察会话。DNS 缓存命中可能不产生新的解析器连接，先确认是否缓存命中；缺少连接记录时不能从空结果推断 DNS 已走代理，不擅自清空全机缓存。

控制接口没有开启时不要擅自打开：请用户在设置里临时打开，或让用户看客户端的连接页，都做不到就写“未验证”。Shadowrocket 没有上述工具，用 App 内的连接记录人工核对，并如实说明。

### DNS 验证

1. 保持用户现有代理开启，使用已知国内、海外、private / LAN 和未知域名四类测试；每类检查 A 与 AAAA。未知域名可用 `example.com`，不用不存在的随机字符串代替成功解析测试。
2. 在客户端的 DNS / 连接记录中核对命中规则、解析器、出站和地址族；必要时在用户授权下短时观察物理网卡与隧道。避免把原始业务日志或完整域名历史公开。
3. 海外与未知查询应按既定策略到达经代理的解析器；private 不应被发送到公网 DNS；国内直连解析器出现本地出口属于本套策略的预期。
4. DNS 检测网站只展示其观测到的递归解析器，不能单凭其国家 / 名称判断真实客户端出口或证明所有应用安全。结合客户端记录、实际网络路径和 IPv6 结果判断。
5. 用户自己切换过网络或节点，或同意重启应用后再复查；Agent 不主动切换节点。只有本次确实执行的组合才写“通过”。

**先触发 DNS 模块，再看日志。** TUN 已接管明文 DNS 后，在目标系统执行 `dig example.com A`、`dig example.com AAAA`（Windows 可用 `nslookup -type=A example.com`、`nslookup -type=AAAA example.com`），按四类域名分别替换并关联本次请求的客户端 DNS 日志。sing-box 应看到 `hijack-dns` 和 DNS 模块处理记录；若只有缓存，先区分缓存结果，不能当作新的上游路径证据。普通 `curl` 或 SOCKS 的远端域名请求可能不经过本地 DNS 模块，不能用网页成功替代此项。没有可观察的 DNS 入口或看不到对应日志时写未验证，不为测试新增监听端口或切回非 TUN。

查看解析器走了哪条路：

```sh
# Mihomo：先查询，再看解析器连接。destinationIP 为 1.1.1.1 / 1.0.0.1 的 chains 应包含主代理组，223.5.5.5 / 223.6.6.6 的应为 DIRECT。
curl -s --noproxy '*' -H "Authorization: Bearer $SECRET" 'http://127.0.0.1:9090/dns/query?name=example.com&type=A'
curl -s --noproxy '*' -H "Authorization: Bearer $SECRET" http://127.0.0.1:9090/connections

# Surge（Mac）
surge-cli dns trace example.com

# sing-box：看 debug 日志里的 dns: 行和对应出站，看不出走了哪个出站就写“未验证”。
```

### WebRTC 验证

1. 在同一浏览器配置档确认普通 HTTPS 请求的预期代理出口；这是本次比较基线，不需要关闭代理。
2. 打开 WebRTC 官方 [Trickle ICE 示例](https://webrtc.github.io/samples/src/content/peerconnection/trickle-ice/)，查看 STUN / TURN 配置并收集候选。测试会联系所选服务器，使用用户认可的测试端点，不添加来历不明的 TURN 账号。Agent 操作不了浏览器时，让用户点 Gather candidates，把结果表截图发给你；里面有公网地址，不要贴到公开地方。
3. 对比 host / srflx / relay 候选与预期路径，特别检查是否出现未授权直连的本地运营商公网 IPv4 / IPv6。relay 是中继服务器地址，不要求与 HTTP 出口完全相同；代理的多个出口也需结合路径核实。
4. host 候选显示 mDNS 名称不等于已证明公网地址隐藏。网络失败、权限差异或单次页面空白都不能单独记作保护通过。按下表判断：

   | 看到什么 | 结论 |
   | --- | --- |
   | 出现未授权的本地运营商公网 IPv4 / IPv6 | 未通过 |
   | 有公网候选，且都是代理出口或已知中继 | 通过 |
   | 没有公网候选 | 还不能下结论。征得用户同意后开一个没装扩展的临时配置档对照：对照档能拿到 srflx，说明 STUN 可达、选项起了作用，记为通过；对照档也没有，记“未验证”。对照会让 STUN 服务器看到对照档的出口，要先告诉用户 |

5. 征得用户同意后重新打开浏览器并复测（未保存的页面可能丢失）。音视频 / WebRTC 业务的功能回归只能由用户确认，没做就写“未验证”。出现通话问题时记录影响，按用户选择回退该选项或研究受支持的代理 / TURN 路径，不静默放宽防护。

<a id="report"></a>

## 7. Agent 交付格式

```text
检查时间与环境：系统、浏览器配置档、核心版本
最终状态：完整通过 / 部分完成 / 等待用户 / 失败已回退（恢复未确认时如实写明）
接入方式：完整 / Surge 托管受限，用户接受的限制；本次实际操作的设备或隔离实例
文档与模板：代码版本或本地候选稿身份，是否已发布
规则来源：查询到的 release commit SHA、manifest 时间、所用格式、哈希结果；各客户端资源的实际更新时间
规则加载：五类规则的条目数或命中证据、自动刷新周期
受限接入：四类模块资源、主配置代理规则证据；未接入的 proxy 层不得写为已加载
配置位置：改动放在哪里、刷新订阅后是否还在、重启后是否还在
TUN 前置：客户端中的实际名称、运行状态、目标流量与 DNS 接管证据；未启用则等待用户
代理范围：TUN / 对应系统隧道及 rule / global / direct 模式；IPv6 代理出口与设备接管分别记录
分流：每个样本命中的规则
网页可用性：国内 / 海外各 3 次及适用内网服务，改动前后成功数与新增错误
DNS：国内 / 海外 / private / 未知查询的解析器与出站（脱敏）；系统 DNS 上游及内网解析的前置判断
WebRTC：Network Limiter 的实际选项，重启后是否保持
测试：普通网页出口、ICE 候选分类、音视频回归结果
变更：修改了哪些字段，模板补充了什么，自己的节点和例外是否保留
节点例外：写入了多少条，节点变化后要做什么
回退：对应配置前像与扩展原选项的恢复位置，发给用户的恢复方法
未验证：移动端、无痕档、IPv6、其他 App、切网等未实际测试项
不适用：项目及理由；等待用户时给出一个具体下一步
```

原生 URL 约每 24 小时各自刷新，不保证所有资源同代；`git ls-remote` 得到的是查询时的发布代，不自动等于客户端已加载的代。只能观察到资源更新时间时就这样报告，不能把最新 SHA 冒充已加载 SHA；严格同代需求见 [SYNC](SYNC.md)。每个结果都带“通过 / 失败 / 未验证 / 不适用”和简短证据。

只修改用户已授权的目标，优先保存配置前像并逐项验证。发现 DNS 绕路就修 DNS 路径，发现 WebRTC 绕路就查浏览器策略与实际 UDP / IPv6 路由；不要用重装服务器、换节点或覆盖所有配置替代定位。
