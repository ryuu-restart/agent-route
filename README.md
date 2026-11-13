<div align="center">

# AgentRoute

<a id="about"></a>

**想用 Claude？把代理规则和浏览器防护交给 Agent。**

沿用你的客户端和节点。复制一段提示词，让 Agent 备份配置、替换旧规则、配好浏览器，再交付检测结果。

[![自动构建](https://github.com/ryuu-restart/agent-route/actions/workflows/run.yml/badge.svg)](https://github.com/ryuu-restart/agent-route/actions/workflows/run.yml) [![规则库发布时间](https://img.shields.io/github/last-commit/ryuu-restart/agent-route/release?label=%E8%A7%84%E5%88%99%E5%BA%93%E5%8F%91%E5%B8%83%E6%97%B6%E9%97%B4&color=2d8e76)](https://github.com/ryuu-restart/agent-route/commits/release)

**[快速开始](#agent-prompt)** · [接入前后对比](#test-results) · [规则直链](#rule-subscriptions) · [官方微信群](#wechat-group)

</div>

<img src="assets/routing-map.svg" width="1080" height="310" alt="业务分流示意：国内与内网直连，海外走代理，广告被拦截。节点沿用用户现有配置。">

- **Agent 直接接入**：先备份，再替换旧业务分流与默认兜底；保留节点订阅，交付恢复方法。
- **三种原生格式**：提供 Mihomo、Surge / Shadowrocket、sing-box 规则，Agent 按实际版本适配。
- **防护与检测一起做**：接入常见 WebRTC 探测拦截，为支持的浏览器补装并配置防护，逐项检查 IP、DNS 和 WebRTC。
- **每日校验发布**：规则校验通过后自动更新，建议客户端每 24 小时刷新。

<a id="让我的-agent-接入"></a>
<a id="agent-prompt"></a>

## 快速开始

准备好**可用的代理客户端和节点**，复制下面的提示词，交给能操作本机文件和浏览器的 **Codex 等 Agent**。遇到系统或浏览器要求本人确认时，按 Agent 提示完成确认即可。

```text
请读取以下规范与索引，把 AgentRoute 接入我现用的代理客户端：
https://raw.githubusercontent.com/ryuu-restart/agent-route/master/AGENTS.md
https://raw.githubusercontent.com/ryuu-restart/agent-route/release/index.json
我授权你先备份，再替换旧业务分流和默认兜底，保留节点订阅与必要技术规则。
同时安装 webrtc-guard，执行客户端重载和一次现有节点订阅刷新。
检查本机已安装的所有浏览器，在实际使用的配置档自动安装并配置 WebRTC 防护。
我接受防护可能影响通话、会议和 P2P。核对环境后按规范直接执行。
完成实际分流及 IP、DNS、WebRTC 检测，交付结果、备份与恢复方法；未核实项注明未验证。
```

> [!CAUTION]
> **禁止使用 Claude（包括 Claude Code 及其他基于 Claude 模型的 Agent）** 执行本项目的接入、复查或维护任务。

<a id="test-results"></a>

### 一次接入，前后对照

**同一个 Chrome 配置档，同一个节点。** 这次 FlClash + Chrome 试跑中，WebRTC 从 9 项中 3 项返回直连 IP，变为 9 项均未报告泄露。

<picture>
  <source media="(max-width: 767px)" srcset="assets/test-results-2026-11-12-mobile.svg">
  <img src="assets/test-results-2026-11-12.svg" width="1200" alt="2026-11-12 接入前后对照，Chrome 本次 WebRTC 探测有改善。完整数据可展开下方文字版对照。">
</picture>

<details>
<summary>展开文字版对照</summary>

2026-11-12 用户试跑，来自截图与 Agent 回报：

- **Chrome WebRTC**：接入前 9 项中 3 项返回直连 IP；接入后 9 项均未报告泄露。
- **DNS 截图可见结果**：接入前为国内电信及香港解析器；接入后为香港 Google / Cloudflare，未出现前图的国内电信。
- **规则入口**：从 2,822 条、0 个外部资源，变为 27 条入口及 5 个远程规则集。
- **节点与策略组**：前后均为 15 个节点、26 个策略组。

</details>

[查看完整测试记录与未验证项](docs/TEST-RESULTS.md)

<a id="rule-subscriptions"></a>
<a id="按客户端拿规则"></a>

## 规则直链

**Agent 格式入口：** [Clash / Mihomo](https://raw.githubusercontent.com/ryuu-restart/agent-route/master/subscriptions/mihomo.json) · [Surge](https://raw.githubusercontent.com/ryuu-restart/agent-route/master/subscriptions/surge.json) · [Shadowrocket](https://raw.githubusercontent.com/ryuu-restart/agent-route/master/subscriptions/shadowrocket.json) · [sing-box](https://raw.githubusercontent.com/ryuu-restart/agent-route/master/subscriptions/sing-box.json) · [完整索引](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/index.json)

| 规则 | 默认动作 | Mihomo | Surge / 小火箭 | sing-box |
| --- | --- | --- | --- | --- |
| 内网直连 | `DIRECT` | [YAML](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/mihomo/private.yaml) | [LIST](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/surge/private.list) | [JSON](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/sing-box/private.json) |
| 广告拦截 | `REJECT` | [YAML](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/mihomo/reject.yaml) | [LIST](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/surge/reject.list) | [JSON](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/sing-box/reject.json) |
| 海外代理 | `PROXY` | [YAML](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/mihomo/proxy.yaml) | [LIST](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/surge/proxy.list) | [JSON](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/sing-box/proxy.json) |
| 国内直连 | `DIRECT` | [YAML](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/mihomo/direct.yaml) | [LIST](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/surge/direct.list) | [JSON](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/sing-box/direct.json) |
| 中国 IP 兜底 | `DIRECT` | [YAML](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/mihomo/cn-ip.yaml) | [LIST](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/surge/cn-ip.list) | [JSON](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/sing-box/cn-ip.json) |
| WebRTC 探测拦截 | `REJECT` | [YAML¹](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/mihomo/webrtc-guard.yaml) | [LIST](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/surge/webrtc-guard.list) | [JSON](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/sing-box/webrtc-guard.json) |

¹ WebRTC 防护使用 Mihomo `classical` 格式，优先于业务规则拒绝已接管的常见探测，可能影响通话、会议及 P2P。[接入顺序与覆盖范围](docs/WEBRTC-GUARD.md)

<a id="记得经常更新"></a>

未匹配的流量默认走代理。规则发布到 `release` 后，由客户端按设置刷新，建议间隔 **24 小时**。[手动接入参考](docs/CLIENTS.md)

<a id="browser-checks"></a>

## 接入后，看这两项

1. **看 Agent 的交付结果。** 旧规则是否已替换、Claude 请求实际走哪条出站、各浏览器是否配置完成，以及备份如何恢复。
2. **用同一配置档复测。** 打开下方三个页面，核对网页出口、DNS 解析器和 WebRTC 探测结果。

**[IP 检测](https://ippure.com/) · [DNS 检测](https://ippure.com/DNS-Leak-Detect.html) · [WebRTC 检测](https://ippure.com/Browser-WebRTC-Leak-Detect.html)**

[浏览器配置与官方依据](docs/DNS-WEBRTC.md#webrtc-methods) · [完整验收标准](docs/DNS-WEBRTC.md#acceptance)

<a id="反馈问题"></a>
<a id="wechat-group"></a>

## 有问题，一起排查

微信扫码加入 **AgentRoute 官方交流群**。接入卡住、规则没生效，或看不懂检测结果，都可以来交流。

<img src="assets/wechat-group-qr.png" width="280" height="392" alt="微信扫码加入 AgentRoute 官方交流群">

二维码在 **2026 年 11 月 20 日前有效**。[查看原图](assets/wechat-group-qr.png) · [提交 GitHub Issue](https://github.com/ryuu-restart/agent-route/issues/new/choose)

反馈时附上系统与客户端版本；请隐去订阅链接、节点地址、凭据和真实 IP。[脱敏说明](AGENTS.md#issue-feedback)

## 文档与许可

[Agent 接入说明](AGENTS.md) · [可选模板与历史验证](examples/README.md) · [DNS / WebRTC 验收](docs/DNS-WEBRTC.md) · [同步与回退](docs/SYNC.md) · [维护说明](docs/MAINTAINING.md)

规则来源与处理方式见 [NOTICE](NOTICE.md)、[sources.json](sources.json)；[本仓库许可](LICENSE)与[上游许可](licenses/)分别适用。
