# AgentRoute · 分流规则库

[![自动构建](https://github.com/ryuu-restart/agent-route/actions/workflows/run.yml/badge.svg)](https://github.com/ryuu-restart/agent-route/actions/workflows/run.yml) [![规则库发布时间](https://img.shields.io/github/last-commit/ryuu-restart/agent-route/release?label=%E8%A7%84%E5%88%99%E5%BA%93%E5%8F%91%E5%B8%83%E6%97%B6%E9%97%B4)](https://github.com/ryuu-restart/agent-route/commits/release)

<a id="about"></a>

面向中国大陆的 Windows 和 Mac 用户。已有可用节点时，把自动更新的规则加到支持规则集的客户端，用于国内直连、海外代理和广告拦截。

<a id="agent-prompt"></a>

## 让我的 Agent 接入

复制这句话给能操作目标设备的 Agent。**禁止使用 Claude（包括 Claude Code 及其他基于 Claude 模型的 Agent）执行本项目的接入、复查或维护任务。**

```text
请按 https://github.com/ryuu-restart/agent-route 的 AGENTS.md 把规则合并到我当前使用的代理配置，保留节点、订阅和用户例外，验证分流后提醒我更新。
```

## 按客户端拿规则

| 规则 | Mihomo | Surge / Shadowrocket | sing-box |
| --- | --- | --- | --- |
| 内网 | [YAML](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/mihomo/private.yaml) | [RULE-SET](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/surge/private.list) | [JSON](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/sing-box/private.json) |
| 拦截 | [YAML](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/mihomo/reject.yaml) | [RULE-SET](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/surge/reject.list) | [JSON](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/sing-box/reject.json) |
| 海外代理 | [YAML](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/mihomo/proxy.yaml) | [RULE-SET](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/surge/proxy.list) | [JSON](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/sing-box/proxy.json) |
| 国内直连 | [YAML](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/mihomo/direct.yaml) | [RULE-SET](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/surge/direct.list) | [JSON](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/sing-box/direct.json) |
| 中国 IP 兜底 | [YAML](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/mihomo/cn-ip.yaml) | [RULE-SET](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/surge/cn-ip.list) | [JSON](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/sing-box/cn-ip.json) |

按实际核心选一列，在客户端添加远程规则集。保留原有规则的优先级，将适用项按表中顺序加在最终兜底之前。各项策略和中国 IP 兜底的条件见[接入说明](AGENTS.md)与[客户端参考](docs/CLIENTS.md#资源与规则语义)。

<a id="记得经常更新"></a>

规则持续更新，请开启客户端自动刷新，或定期手动刷新。

## 浏览器检测

用平时上网的浏览器打开 [IP 检测](https://browserleaks.com/ip)、[DNS 检测](https://browserleaks.com/dns)、[WebRTC 检测](https://browserleaks.com/webrtc)。结果只覆盖当前浏览器的本次探测。

<a id="反馈问题"></a>

## 参考资料

[规则索引](https://raw.githubusercontent.com/ryuu-restart/agent-route/release/index.json) · [可选模板与历史验证](examples/README.md) · [高级 DNS / WebRTC 验收](docs/DNS-WEBRTC.md) · [同步与回退](docs/SYNC.md) · [维护说明](docs/MAINTAINING.md) · [反馈问题](AGENTS.md#issue-feedback)

规则来源、处理方式和许可见 [NOTICE](NOTICE.md)、[sources.json](sources.json) 与 [licenses](licenses/)。[本仓库许可](LICENSE)与上游声明分别适用。
