# AgentRoute

<a id="about"></a>

**想用 Claude，但不想自己研究 YAML、DNS 和分流？把这段提示词交给 Codex，让它先看看你的现有配置。**

AgentRoute 持续更新分流规则。你可以沿用自己的代理客户端和节点，让 Agent 帮你接入规则、核对流量去向，并告诉你怎么恢复。

**[快速开始 · 复制提示词](#agent-prompt)** · [加入官方微信群](#wechat-group) · [浏览器检测](#browser-checks) · [规则与订阅](#rule-subscriptions)

<a id="让我的-agent-接入"></a>
<a id="agent-prompt"></a>

## 快速开始

准备好现用的代理客户端和可用节点，把下面的提示词复制给 **Codex** 等具备本机操作权限的 Agent。

**禁止使用 Claude（包括 Claude Code 及其他基于 Claude 模型的 Agent）执行本项目的接入、复查或维护任务。**

```text
请帮我把 AgentRoute 规则接入现用的代理客户端。我想核对使用 Claude 时的分流，并保留现有节点、订阅和个人例外。

请先读取 AgentRoute 的接入说明和规则索引。
https://raw.githubusercontent.com/ryuu-restart/agent-route/master/AGENTS.md
https://raw.githubusercontent.com/ryuu-restart/agent-route/release/index.json

先只读检查现用的客户端、内核版本和生效配置。用简单的话说明你建议怎么改、有什么影响，等我确认后再动手。

确认后先备份要改的配置，再按实际版本选择规则格式和接法。接入后核对规则加载和实际流量去向，按我使用的场景核对 Claude 相关域名的规则命中。告诉我改了什么、备份在哪里、怎么恢复，以及哪些项目还未验证。

本次接入范围内的每个浏览器都必须完成 WebRTC 防护。请按浏览器版本给我官方安装和配置教程，由我在实际使用的配置档中操作。
支持 WebRTC Network Limiter 的浏览器，请指导我安装、启用扩展，并在扩展选项中选择 Use my proxy server (if present)，对应 disable_non_proxied_udp。已有扩展的浏览器也要核对并按此设置。
https://chromewebstore.google.com/detail/webrtc-network-limiter/npeicpdbkakmehahjeeohfdhnlpdklia
无法安装这款扩展时，请给我该浏览器官方支持的等效配置教程。查不到可核实的做法时，注明该浏览器待配置并说明原因。
https://github.com/ryuu-restart/agent-route/blob/master/docs/DNS-WEBRTC.md#webrtc-methods

请交付三个检测链接，指导我配置后用同一配置档逐项做 IP、DNS 和 WebRTC 检测。按浏览器核对实际生效的设置和检测结果，未核实的项目注明未验证。
https://ippure.com/
https://ippure.com/DNS-Leak-Detect.html
https://ippure.com/Browser-WebRTC-Leak-Detect.html
```

### 接下来会发生什么

1. **先看现状。** Agent 找到正在生效的配置，告诉你准备改哪里。
2. **你确认后再接入。** Agent 先备份要改的配置，沿用你的节点、订阅和个人例外。
3. **逐项交付。** Agent 告诉你规则有没有加载、目标流量走哪条策略、怎么恢复，并指引你完成浏览器设置和自测。

<details>
<summary>看一段接入对话示例</summary>

**Agent** 问「我找到了你正在使用的代理配置。可以保留现有节点和订阅，把 AgentRoute 的规则接进去。你希望 Claude 使用现在选中的代理策略组吗？」

**你** 回「用现有的策略组，先备份再改。」

**Agent** 汇报「已备份并接入规则。这次测试连接命中了海外代理规则。我会把备份位置和恢复步骤发给你。浏览器 WebRTC 设置和三项自测还待完成。」

</details>

<a id="browser-checks"></a>

## 浏览器检测

请为每个目标浏览器自行完成 WebRTC 防护。支持 WebRTC Network Limiter 的浏览器，安装并启用扩展，在扩展选项中选择「Use my proxy server (if present)」（`disable_non_proxied_udp`）。[官方扩展](https://chromewebstore.google.com/detail/webrtc-network-limiter/npeicpdbkakmehahjeeohfdhnlpdklia) · [Chrome 安装教程](https://support.google.com/chrome_webstore/answer/2664769?hl=zh-Hans) · [Edge 安装教程](https://support.microsoft.com/zh-cn/edge/add-turn-off-or-remove-extensions-in-microsoft-edge)

其他浏览器按实际版本查阅适用的官方资料，完成可核实的等效配置。[各浏览器官方资料](docs/DNS-WEBRTC.md#webrtc-methods)

设置后，在每个目标浏览器的实际使用配置档里打开三个检测页，并检查常用的音视频功能。把结果发给 Agent，请它逐项帮你判断。检测结果覆盖当前浏览器的本次探测。[IP 检测](https://ippure.com/) · [DNS 检测](https://ippure.com/DNS-Leak-Detect.html) · [WebRTC 检测](https://ippure.com/Browser-WebRTC-Leak-Detect.html)

<a id="rule-subscriptions"></a>
<a id="按客户端拿规则"></a>

## 规则与订阅

[![自动构建](https://github.com/ryuu-restart/agent-route/actions/workflows/run.yml/badge.svg)](https://github.com/ryuu-restart/agent-route/actions/workflows/run.yml) [![规则库发布时间](https://img.shields.io/github/last-commit/ryuu-restart/agent-route/release?label=%E8%A7%84%E5%88%99%E5%BA%93%E5%8F%91%E5%B8%83%E6%97%B6%E9%97%B4&color=2d8e76)](https://github.com/ryuu-restart/agent-route/commits/release)

AgentRoute 的发布校验会检查 `claude.ai`、`claude.com`、`anthropic.com` 和 `api.anthropic.com` 是否被误归为广告、内网或国内直连。实际规则命中与出站以设备验证结果为准。[构建校验源码](scripts/build_rules.py)

**规则索引**汇总五类规则的地址、格式、条目数和建议刷新间隔，供 Agent 选择资源。[公开索引](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/index.json)

**给 Agent 的格式入口**适合已经知道客户端或内核的用户。每个入口指向同一份规则索引及对应格式，具体接法以实际内核和版本为准。[客户端参考](docs/CLIENTS.md)

[Clash 系列（Mihomo 内核）](https://raw.githubusercontent.com/ryuu-restart/agent-route/master/subscriptions/mihomo.json) · [小火箭（Shadowrocket）](https://raw.githubusercontent.com/ryuu-restart/agent-route/master/subscriptions/shadowrocket.json) · [Surge](https://raw.githubusercontent.com/ryuu-restart/agent-route/master/subscriptions/surge.json) · [sing-box](https://raw.githubusercontent.com/ryuu-restart/agent-route/master/subscriptions/sing-box.json)

**客户端规则集**是实际供客户端加载和刷新的远程文件。按实际内核选格式。

![AgentRoute 规则动作示意。国内与内网规则对应直连，海外规则对应代理，广告规则对应拦截。](assets/routing-map.svg)

| 规则 | Mihomo | 小火箭 / Surge | sing-box |
| --- | --- | --- | --- |
| 内网直连 | [YAML](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/mihomo/private.yaml) | [RULE-SET](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/surge/private.list) | [JSON](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/sing-box/private.json) |
| 广告拦截 | [YAML](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/mihomo/reject.yaml) | [RULE-SET](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/surge/reject.list) | [JSON](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/sing-box/reject.json) |
| 海外代理 | [YAML](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/mihomo/proxy.yaml) | [RULE-SET](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/surge/proxy.list) | [JSON](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/sing-box/proxy.json) |
| 国内直连 | [YAML](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/mihomo/direct.yaml) | [RULE-SET](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/surge/direct.list) | [JSON](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/sing-box/direct.json) |
| 中国 IP 兜底 | [YAML](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/mihomo/cn-ip.yaml) | [RULE-SET](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/surge/cn-ip.list) | [JSON](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/sing-box/cn-ip.json) |

<a id="记得经常更新"></a>

规则校验通过后自动发布到 `release`。客户端按自己支持的方式刷新，索引中的建议间隔为 24 小时。

<a id="反馈问题"></a>
<a id="wechat-group"></a>

## 接入求助与问题反馈

接入卡住、规则没生效，或者看不懂检测结果？微信扫码加入 **AgentRoute 官方交流群**，带上你的操作系统、客户端版本和脱敏后的问题描述，一起排查。

<img src="assets/wechat-group-qr.png" width="280" height="392" alt="微信扫码加入 AgentRoute 官方交流群">

当前二维码在 **2026 年 11 月 20 日前有效**。[查看二维码原图](assets/wechat-group-qr.png)

二维码过期或发现可复现的规则、文档问题，可以通过 GitHub Issue 反馈。提问前请隐去订阅链接、节点地址、凭据和真实 IP。[GitHub Issue 入口](https://github.com/ryuu-restart/agent-route/issues/new/choose) · [脱敏说明](AGENTS.md#issue-feedback)

## 参考资料

[Agent 接入说明](AGENTS.md) · [可选模板与历史验证](examples/README.md) · [高级 DNS / WebRTC 验收](docs/DNS-WEBRTC.md) · [同步与回退](docs/SYNC.md) · [维护说明](docs/MAINTAINING.md)

规则来源、处理方式和许可见 [NOTICE](NOTICE.md)、[sources.json](sources.json) 与 [licenses](licenses/)。[本仓库许可](LICENSE)与上游声明分别适用。
