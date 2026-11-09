# AgentRoute · 分流规则库

<a id="about"></a>

**把 Claude 的网络配置交给 Agent。**

面向 Windows 和 Mac 上的 Claude Code 与 Claude 桌面端。Agent 先检查你现有的代理，说明要改哪里，再按你的授权用 AgentRoute 替换分流并核对结果。

[![自动构建](https://github.com/ryuu-restart/agent-route/actions/workflows/run.yml/badge.svg)](https://github.com/ryuu-restart/agent-route/actions/workflows/run.yml) [![规则库发布时间](https://img.shields.io/github/last-commit/ryuu-restart/agent-route/release?label=%E8%A7%84%E5%88%99%E5%BA%93%E5%8F%91%E5%B8%83%E6%97%B6%E9%97%B4&color=2d8e76)](https://github.com/ryuu-restart/agent-route/commits/release)

**[让 Agent 接入](#agent-prompt)** · [验证 Claude](#claude-clients) · [规则订阅](#rule-subscriptions) · [浏览器检测](#browser-checks)

![AgentRoute 分流图。国内与内网流量直连，海外与其余流量走现有代理，命中拦截名单的流量被拦截。](assets/routing-map.svg)

<a id="agent-prompt"></a>

## 让我的 Agent 接入

把下面这段交给能操作这台电脑的 Agent。

```text
请读取下方索引及其接入说明。
https://raw.githubusercontent.com/ryuu-restart/agent-route/release/index.json
先检查当前代理和节点，告诉我需要改哪里。
按我的授权备份配置，用 AgentRoute 替换分流。
沿用我的节点订阅和代理组，核对两类更新。
告诉我实际测到了什么，最后提醒我更新规则。
```

**禁止使用 Claude（包括 Claude Code 及其他基于 Claude 模型的 Agent）执行本项目的接入、复查或维护任务。**

<a id="claude-clients"></a>

## 验证 Claude Code 与桌面端

用连接记录核对 Claude 请求实际命中的规则和代理组，再在你使用的客户端里验证登录和一次常用操作。想检查浏览器的出口 IP、DNS 和 WebRTC 时，可以打开[浏览器检测](#browser-checks)。[Claude Code 文档](https://code.claude.com/docs/en/setup) · [Claude 桌面端文档](https://support.claude.com/en/articles/10065433-install-claude-desktop)

Claude 的使用资格以 Anthropic 的地区和账号政策为准。当前支持地区未列中国大陆。[官方支持地区](https://www.anthropic.com/legal/supported-regions)

<a id="rule-subscriptions"></a>

## 按客户端拿规则

把对应的入口交给 Agent。它会识别客户端的覆盖方式，并接入对应格式的原生远程规则。

[Mihomo](https://raw.githubusercontent.com/ryuu-restart/agent-route/master/subscriptions/mihomo.json) · [Surge](https://raw.githubusercontent.com/ryuu-restart/agent-route/master/subscriptions/surge.json) · [Shadowrocket](https://raw.githubusercontent.com/ryuu-restart/agent-route/master/subscriptions/shadowrocket.json) · [sing-box](https://raw.githubusercontent.com/ryuu-restart/agent-route/master/subscriptions/sing-box.json)

上方是供 Agent 读取的订阅入口。客户端刷新的是下表的规则集。手动接入也可直接使用这些地址。

| 规则 | Mihomo | Surge / Shadowrocket | sing-box |
| --- | --- | --- | --- |
| 内网 | [YAML](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/mihomo/private.yaml) | [RULE-SET](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/surge/private.list) | [JSON](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/sing-box/private.json) |
| 拦截 | [YAML](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/mihomo/reject.yaml) | [RULE-SET](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/surge/reject.list) | [JSON](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/sing-box/reject.json) |
| 海外代理 | [YAML](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/mihomo/proxy.yaml) | [RULE-SET](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/surge/proxy.list) | [JSON](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/sing-box/proxy.json) |
| 国内直连 | [YAML](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/mihomo/direct.yaml) | [RULE-SET](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/surge/direct.list) | [JSON](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/sing-box/direct.json) |
| 中国 IP 兜底 | [YAML](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/mihomo/cn-ip.yaml) | [RULE-SET](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/surge/cn-ip.list) | [JSON](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/sing-box/cn-ip.json) |

按实际核心选一列，使用持久覆盖将业务分流替换为这些规则。内网和国内流量按规则直连，海外和其余流量使用现有代理组，命中拦截名单的流量被拦截。覆盖条件和所需的技术规则见[接入说明](AGENTS.md)与[客户端适配](docs/CLIENTS.md#订阅适配)。

<a id="记得经常更新"></a>

规则持续发布到 `release`。请开启客户端规则集自动刷新，或定期手动刷新。

<a id="browser-checks"></a>

## 浏览器检测

用平时上网的浏览器打开 [IP 检测](https://browserleaks.com/ip)、[DNS 检测](https://browserleaks.com/dns)、[WebRTC 检测](https://browserleaks.com/webrtc)。结果只覆盖当前浏览器的本次探测。

<a id="反馈问题"></a>

## 参考资料

[规则索引](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/index.json) · [可选模板与历史验证](examples/README.md) · [高级 DNS / WebRTC 验收](docs/DNS-WEBRTC.md) · [同步与回退](docs/SYNC.md) · [维护说明](docs/MAINTAINING.md) · [反馈问题](AGENTS.md#issue-feedback)

规则来源、处理方式和许可见 [NOTICE](NOTICE.md)、[sources.json](sources.json) 与 [licenses](licenses/)。[本仓库许可](LICENSE)与上游声明分别适用。
