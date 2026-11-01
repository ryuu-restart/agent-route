# 客户端接入

本文回答三个问题：用哪个模板、改动放哪里才不会丢、每个客户端有哪些坑。整体流程见 [AGENTS](../AGENTS.md)。

<a id="tun-required"></a>

## TUN 前置检查

**本项目要求 TUN。** 识别为只有系统代理、仅浏览器代理或 TUN 未启用时，先让使用者开启 TUN / 增强模式，再接入规则。系统代理能打开网页不代表满足条件，受限接入也不能以此替代。

1. 确认当前客户端和版本，记录模式、节点选择、TUN 设置、DNS 与路由前像；先发给用户不依赖 Agent 的恢复方法。若开启可能断开 Agent 且用户无法恢复，停在等待用户。
2. 给出该版本界面中的实际入口，让使用者打开 TUN / 增强模式；系统要求的管理员、虚拟网卡或系统扩展弹窗由使用者确认。不要自行安装驱动或更换客户端。命令行核心由使用者准备并启用对应 TUN 入站和系统权限，不能把 `mixed` 入站当成 TUN。
3. Surge 的增强模式、移动端的系统 VPN 隧道等名称须按当前客户端核对其实际隧道接管能力。读取可用的运行状态和系统路由，并从目标应用验证流量、DNS 确实进入该客户端；只看到 TUN 配置或虚拟网卡存在还不够。IPv6 覆盖另行验收。
4. 证据不足、客户端不支持、启用失败或用户不愿启用，都保持原配置并写“等待用户开启 / 确认 TUN”，给出一个具体下一步。此时不能应用模板或报告部分接入成功。TUN 已确认后，才开始 AGENTS 第 2 步。

TUN 不自动解决 global / direct 模式、DNS 上游选择或 IPv6 旁路；规则模式和后续验收仍须分别确认。本次新增 TUN 切换的恢复应保留用户原有代理，不能用关闭全部代理恢复直连作为常规验收手段。

### 命令行核心的具体准备

检测到命令行核心只有 `mixed` / SOCKS / HTTP 入站时，Agent 先查当前系统路由和已安装的核心版本，为使用者准备一份保留原配置的 TUN 候选文件，给出检查、启动和恢复命令。不要只让非技术用户“自行配置 TUN”。下面是字段起点，不是完整配置或已经验收过的运行方案：

- **Mihomo 1.19.32**：将下列字段合并进已有 `tun`，保留用户原有的其他 TUN 字段。DNS 模块先沿用用户已确认的原解析方案；缺少 DNS 方案时先补齐，不能猜上游。

  ```yaml
  tun:
    enable: true
    stack: mixed
    auto-route: true
    auto-detect-interface: true
    dns-hijack:
      - any:53
      - tcp://any:53
  ```

- **sing-box 1.14.4**：在 `inbounds` 中新增下面一项，不覆盖已有入站；把示例地址替换为根据现场路由确认未占用的地址段。保留 `route.auto_detect_interface: true`，DNS 接管用 full 模板已有的 `port: 53` / `hijack-dns` 规则。尚未合并 full 模板时，要先为候选配置补齐这条 DNS 接管规则和用户原有的 DNS 方案，不能把它当作已完成 AgentRoute 接入。

  ```json
  {"type":"tun","tag":"agentroute-tun","address":["172.19.0.1/30","fdfe:dcba:9876::1/126"],"auto_route":true,"stack":"mixed"}
  ```

先运行本文的原生配置检查；成功后，把已核实的可执行文件、候选文件和工作目录的**实际绝对路径**填入启动命令发给使用者，例如 `sudo "/实际路径/mihomo" -d "/实际工作目录" -f "/实际候选.yaml"` 或 `sudo "/实际路径/sing-box" run -D "/实际工作目录" -c "/实际候选.json"`，路径含空格时也须正确引用。`sudo` 只适用于需要它的 macOS / Linux 环境，由使用者本人确认密码和系统权限；Windows 由使用者在管理员终端运行实际命令。已有客户端服务时优先用其正式 TUN 入口，不另起第二个核心或第二条 TUN。

使用者前台启动的进程可用 Ctrl+C 停止；恢复方法还须写明如何恢复候选前像、恢复原有客户端运行方式，并核对路由和联网已恢复。后台服务使用其原有管理方式，不能把 Ctrl+C 当作所有客户端的恢复方法。配置检查通过仍不证明系统权限、路由或 DNS 接管生效；用户启动后再按上方第 3 项确认。Agent 为验收启动的临时进程必须有有限期限和已验证的清理，不以这段启动示例绕过当前环境的进程管理规则。

字段依据：[Mihomo TUN](https://wiki.metacubex.one/config/inbound/tun/)、[sing-box TUN](https://sing-box.sagernet.org/configuration/inbound/tun/)。GUI 不照抄上述启动方式，由 Agent 按现场界面给出开关位置和系统授权步骤。

## 选模板

| 客户端 | 模板 | 规则格式 |
| --- | --- | --- |
| Mihomo / Clash Meta / FlClash / Clash Verge Rev | [full/mihomo.yaml](../examples/full/mihomo.yaml) | YAML `payload`，域名 `behavior: domain`，IP `ipcidr` |
| Surge | [full/surge.conf](../examples/full/surge.conf) | 无策略列的 `RULE-SET` 文本 |
| Shadowrocket | [full/shadowrocket.conf](../examples/full/shadowrocket.conf) | 同 Surge；刷新周期在 App 内设置 |
| sing-box | [full/sing-box.json](../examples/full/sing-box.json) | JSON source rule-set version 3 |

模板包含 DNS、嗅探、DNS 服务器路由和完整规则顺序。用户的 DNS 已按 [DNS-WEBRTC](DNS-WEBRTC.md) 配好时，可以只合并路由部分，但仍要补齐所有引用的规则集定义；Mihomo 包括 `rule-providers`。sing-box 的 JSON 没有标记，合并 `route.rule_set`、`route.rules`，并核对 `route.final`、域名解析器和直连出站，不能漏掉默认代理或重复定义出站。

<a id="baseline"></a>

### 验证基线

| 客户端 | 版本 | 验证到什么程度 |
| --- | --- | --- |
| Mihomo | 1.19.32 | 配置检查；隔离实例中规则加载、DNS 与分流命中 |
| sing-box | 1.14.4（模板需要 1.14.0+） | 配置检查；隔离实例中 DNS 答案筛选与分流命中 |
| Surge | Mac 5.10.0+（`[Host]` 引用规则集需要） | 只做配置检查 |
| Shadowrocket | 未限定 | **只做文本结构检查，实体 iOS 未验证** |

以上都未覆盖真实 TUN、实体手机、持续负载和真实泄露测试。基线是验证过的版本，不是最低要求；明确的下限只有表里写的 sing-box 1.14.0 和 Surge Mac 5.10.0。低于基线时先跑配置检查，不通过就按实际版本的文档适配，适配不了再问用户是否升级。旧版 Clash 分支不支持全部字段。

<a id="config-check"></a>

### 配置检查命令

| 客户端 | 命令 | 说明 |
| --- | --- | --- |
| Mihomo 核心 | `mihomo -t -d <配置目录> -f <配置文件>` | |
| sing-box | `sing-box check -c <配置文件>` | |
| Surge Mac | `/Applications/Surge.app/Contents/Applications/surge-cli --check <配置文件>` | `surge-cli` 不一定在系统路径里 |
| FlClash、Clash Verge Rev 等图形客户端 | 按当前版本确认 | 自带核心不一定提供 `mihomo -t`，例如 FlClash 0.9.2 只带 `FlClashCore`。用户同意安装后，可以用同版本官方核心检查；否则写“未做原生检查”，靠导入时的报错和第 6 步验收 |
| Shadowrocket | 无 | 只能做文本结构检查 |

<a id="where-changes-survive"></a>

## 改动放哪里

**这一节决定接入能否长期有效。** 配置由订阅生成时，直接改文件，下次刷新就被冲掉。先判断配置来源，再选位置。

| 客户端 | 配置来源 | 改动放哪里 | 依据 |
| --- | --- | --- | --- |
| Mihomo 核心 | 本地文件 | 直接改文件。节点来自订阅的，用 `proxy-providers`，规则不受订阅影响 | 官方文档 |
| Clash Verge Rev | 订阅 | 规则用“编辑规则”的前置规则；`rule-providers`、DNS 和嗅探用“扩展脚本”，订阅里没有这些字段时也可用“扩展配置” | [规则](https://www.clashverge.dev/guide/rules.html)、[扩展](https://www.clashverge.dev/guide/extend.html) |
| FlClash | 订阅 | 完整订阅负责 DNS，要关闭“DNS 覆写”和“追加系统 DNS”。规则和规则集优先放进 Override（覆写）入口，刷新订阅后确认还在；找不到或不生效就停下问用户，不要改订阅原文 | 2026-10 实测 DNS 开关；0.9.2 程序内能看到 Override、附加规则和 Rule providers，未实测 |
| Surge | 自己的配置 | 直接改，或用 Detached Profile | [手册](https://manual.nssurge.com/) |
| Surge | 托管配置 | 用模块，但有限制，见下方 | [模块](https://manual.nssurge.com/profile/module.html) |
| Shadowrocket | 任意 | 按当前 App 版本确认本地配置和模块的行为，确认不了就停下问用户 | 未核实 |
| sing-box GUI | 远程配置 | 远程配置更新会覆盖。改用本地配置（先问用户），或让生成方合并模板 | 未核实，按当前版本确认 |
| 其他 GUI | 订阅 | 找“覆写 / 扩展 / Override”功能。找不到就停下问用户，不要改订阅原文 | — |

### Clash Verge Rev 的合并陷阱

扩展配置写到的列表和 `dns` 字段会**整段替换**，包括 `nameserver-policy`，没写的字段保留。所以：

- 要保留订阅原有的条目，用扩展脚本（`main(config)`）追加，不要用扩展配置覆盖。
- `RULE-SET` 规则依赖 `rule-providers`，官方规则页没有讲规则集放哪里。用扩展脚本把 `sr-*` 合并进 `config["rule-providers"]`，再写前置规则。
- `mixed-port`、`mode`、`ipv6`、`allow-lan`、`tun.enable` 等由应用设置页控制，写在扩展里会被丢弃，要到设置页改。
- 官方文档明确“编辑规则”不会随订阅更新失效；扩展是否每次刷新后重新应用，文档没有逐字说明，接入后刷新一次订阅实测。

GUI 保存覆写或规则时可能立即应用。先在候选副本合并规则、providers 和 DNS 并检查；能用一份扩展脚本同时提交的，优先一次提交。必须分步保存时，先加入 providers，再加入依赖它们的规则和 DNS；每次保存后的有效配置都必须完整。检查应用实际生成的配置，不能只检查编辑器里的片段。不要把通用分流放在用户例外之前，也不要保留抢先命中的旧最终规则。

### Surge 托管配置的模块限制

托管配置不能直接编辑，模块是官方给的办法，但：

- 模块里的规则**只能用内置策略**（DIRECT、REJECT、REJECT-TINYGIF），`RULE-SET,...proxy.list,PROXY` 放不进去。
- 模块不能改 `[Proxy]` 和 `[Proxy Group]`，模块的启用状态也不跨设备同步。
- 模块规则插在主配置规则之前。原来主配置里的强制代理例外可能被模块的 direct / cn-ip 规则遮住，不能只在模块内排序就声称保留了原优先级。

因此默认推荐经用户同意改用自己可编辑的配置，再按完整模板合并。保留托管配置时，必须先说明下方受限方案并取得用户选择，不能静默降级。

<a id="limited-integration"></a>

### Surge 托管配置：受限接入

只适用于用户选择保留托管配置、接受缺少 AgentRoute `proxy` 层的情况。它是部分完成，不是完整接入通过。

先通过上方 TUN 检查；没有 TUN 的托管配置不能进入本分支。

1. 先核对主配置的现有规则、默认策略和全部用户例外。private、reject、direct、cn-ip 可放入模块；`proxy` 层及默认代理依赖主配置，不能把代理策略名写进模块规则。
2. 对每个用户强制代理域名和 IP 检查是否会被模块提前直连。冲突无法在当前能力下消除时停止，推荐可编辑配置，不以删除用户例外解决。节点与 LAN 例外也必须保留。
3. 单独核对海外 DoH 的连接路径。模块不能写指向自定义代理组的规则，不能假定主配置会把 `1.1.1.1` 送入代理；若 DNS 仍直连，不得报告 DNS 通过。
4. 按 DNS-WEBRTC 验收四类模块资源，并为海外、未知域名提供主配置实际命中的规则和出口证据；AgentRoute `proxy` 资源写“未接入，托管模块限制”，不要求伪造第五类资源。节点例外、用户例外、DNS、网页可用性等检查不因此豁免。
5. 交付写“受限接入，部分完成”，列出缺失的 `proxy` 优先层和已验证的主配置行为。主配置更新可能改变该行为，刷新后必须重新核对。

以上限制来自 [Surge 官方模块手册](https://manual.nssurge.com/profile/module.html)。Shadowrocket、其他 GUI 或未知版本的能力不足，不能自动套用此分支或扩大它的豁免。

### 订阅刷新后复查

配置放好后，手动刷新一次订阅，再确认规则和 DNS 还在。这一步没做，就不能报告“已持久化”。

## 规则顺序及原因

**用户例外 → 节点地址 → 本地与 LAN → DNS 服务器地址 → private → reject → proxy → direct → cn-ip → 默认代理。**

| 顺序 | 为什么 |
| --- | --- |
| 用户例外最先 | 用户明确的选择优先于任何名单 |
| 节点地址早于一切代理规则 | 否则连接节点的流量可能又被送进代理，形成循环 |
| FakeIP 段早于 LAN | `fc00::/7` 这条 LAN 规则包含 FakeIP 的 IPv6 段，顺序反了 FakeIP 流量会被当成内网直连 |
| DNS 服务器地址早于名单 | 决定 DoH 连接走直连还是代理，每个备用解析器都要有规则 |
| private 早于 reject 和 proxy | 精确的 `router.asus.com` 不应被 `.asus.com` 海外后缀覆盖 |
| proxy 早于 direct | 防止海外域名被国内名单误判直连 |
| cn-ip 在所有域名规则之后 | 它是兜底，且需要解析出真实 IP 才能判断 |

几个细节：

- 这里的 `private` 指通用内网域名规则，不是维护者的私密数据。
- 不要把精确域名扩成后缀。Mihomo 的 `+.example.com` 同时覆盖根域名和子域名；sing-box 的 `domain` 和 `domain_suffix` 分开保存。
- `direct` **只含域名**，`cn-ip` **只含公网中国 IP**。cn-ip 已剔除 LAN 和合成地址。
- 个人强制代理域名、业务直连 IP、App 专用例外、广告白名单和节点不在共享数据里，放在用户自己的配置中。

### cn-ip 能兜住未知域名吗

取决于客户端的解析流程。模板有意不给 cn-ip 加 `no-resolve`。

- **Mihomo / Surge**：未知域名会解析后再匹配 IP。解析失败走默认代理。
- **sing-box**：路由规则不会替域名做解析。模板在 DNS 层判断：答案**全部**是中国 IP 才返回真实地址，连接随后命中 cn-ip；混合、海外或解析失败都继续走 FakeIP 和代理。这段逻辑需要 `sing-box/non-cn-ip.json`，它只用于 DNS 判断，不是路由规则集。
- **直接以 SOCKS / HTTP 传入 sing-box 的未知域名**不经过这个 DNS 判断，会走默认代理。这是偏安全的结果，不算故障。

## 稳定资源地址

基础地址为 `https://raw.githubusercontent.com/ryuu-restart/agent-route/release/`，全部公开，无需凭据。

| 规则 | 动作 | Surge / Shadowrocket | Mihomo | sing-box |
| --- | --- | --- | --- | --- |
| 私有 / 内网域名 | DIRECT | [private.list](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/surge/private.list) | [private.yaml](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/mihomo/private.yaml) | [private.json](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/sing-box/private.json) |
| 广告域名 | REJECT | [reject.list](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/surge/reject.list) | [reject.yaml](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/mihomo/reject.yaml) | [reject.json](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/sing-box/reject.json) |
| 海外域名 | 自己的 PROXY | [proxy.list](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/surge/proxy.list) | [proxy.yaml](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/mihomo/proxy.yaml) | [proxy.json](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/sing-box/proxy.json) |
| 国内域名 | DIRECT | [direct.list](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/surge/direct.list) | [direct.yaml](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/mihomo/direct.yaml) | [direct.json](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/sing-box/direct.json) |
| 中国 IPv4 / IPv6 | DIRECT 兜底 | [cn-ip.list](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/surge/cn-ip.list) | [cn-ip.yaml](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/mihomo/cn-ip.yaml) | [cn-ip.json](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/sing-box/cn-ip.json) |

机器入口：[index.json](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/index.json)；同批时间、来源及 SHA-256：[manifest.json](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/manifest.json)。`index.json` 是本项目版本化的索引，不是客户端配置或通用标准协议。

规则下载默认经 `PROXY`：`raw.githubusercontent.com` 在部分网络无法直连。代理不可用时，客户端继续使用已缓存的上一份规则。

## 更新与失败保留

规则每天 **UTC+8 06:30** 计划构建，调度可能延迟，以 manifest 时间为准。每次先跑测试，再生成并通过官方 Mihomo / sing-box 原生检查，最后一次性切换 `release` 分支。失败时不发布，旧规则仍然可用。

客户端按远程地址约每 24 小时刷新，各设备刷新时刻不同。

## 兼容旧消费者

根目录原有的 `direct.list` / `direct.json` / `direct.domains` / `direct.ipcidr`、`private.domains`、`proxy-source.txt`、`reject.domains` / `reject.list` / `reject.json` 和 manifest 字段继续保留。旧 `direct.list` / `direct.json` 混合了域名、LAN 和 IP，不能用来替换新目录中的纯域名资源。消费者不要假设 manifest 的 `files` 永远只有固定数量。

## 依据与局限

格式依据：[Mihomo rule-providers](https://wiki.metacubex.one/config/rule-providers/) · [Surge RULE-SET](https://manual.nssurge.com/rules/ruleset.html) · [sing-box rule-set](https://sing-box.sagernet.org/configuration/rule-set/)。DNS 和域名过滤不能可靠消除与正常内容共用域名的视频广告；原生检查不等于每台设备的真实网络验收。
