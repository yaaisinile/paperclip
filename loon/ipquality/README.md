# 节点 IP 质量检测

> ✅ 维护中 · 仅支持 Loon

在 Loon 的节点或策略组页面查看所选节点的出口 IP、IP 类型与风险信息，并检测常见流媒体和 AI 服务可用性。结果按 VPS `IP.Check.Place` / `xykt/IPQuality` 的多源展示口径呈现，不生成自定义综合评分。

## 文件

- `ipquality.js` — Loon `generic` 交互脚本
- `ipquality.lpx` — 一键导入插件

## 使用步骤

1. 在 Loon 中导入下方插件地址。
2. 打开节点或策略组页面，选中目标后运行「节点 IP 质量检测」。
3. 等待报告弹窗；首次开启流媒体检测时耗时会稍长。

推荐直接对具体节点运行。对策略组运行时，结果代表本次实际被策略组选中的出口。

## Loon

```ini
[Script]
generic script-path=https://raw.githubusercontent.com/yaaisinile/paperclip/refs/heads/main/loon/ipquality/ipquality.js, tag=节点 IP 质量检测, timeout=50, img-url=shield.lefthalf.filled.system, enable=true
```

也可直接导入插件：

`https://raw.githubusercontent.com/yaaisinile/paperclip/refs/heads/main/loon/ipquality/ipquality.lpx`

## 插件参数

| 参数 | 默认值 | 说明 |
|---|---:|---|
| `MaskIP` | 关闭 | 隐藏报告中 IP 的后半段，适合截图分享 |
| `MediaTest` | 开启 | 检测 TikTok、Disney+、Netflix、YouTube、Prime Video、Reddit、ChatGPT、Gemini |
| `MapNotification` | 关闭 | 完成后发送通知；点按通知可在 Apple 地图查看 IP 数据库估算的位置 |

## 报告内容

- **基础信息**：出口 IP、ASN、组织、国家/地区、城市、时区、坐标及 IP 类型。
- **多源风险**：分别展示各数据库返回的类型、风险评分和代理/VPN/Tor/机房等标记；数据冲突时保留原始结果。
- **服务可用性**：展示各服务本次请求的可用状态与地区。
- **数据状态**：显示本次成功来源、未返回来源和出口是否存在差异。

## 数据说明

- 脚本只展示本次实际返回的数据：没有返回不等于低风险、家宽或已解锁。
- 若不同检测服务看到的公网 IP 不同，会标注“分流出口”。这通常由节点服务商的不同上游、负载均衡或按域名分流造成；该结果只作为补充，不会用于评价主出口 IP。
- 地理位置来自 IP 数据库，代表运营商登记或城市级估算位置，不是设备 GPS。
- 检测会将节点出口 IP 提交给报告内列出的第三方查询服务；介意时请勿使用。

## 维护记录

| 日期 | 变更 |
|---|---|
| 2026-10-12 | r9：新增 Gemini 可用性检测，用于筛选出口所在国家/地区不支持的节点 |
| 2026-10-12 | r10：重写 Gemini 判定——以落地页内嵌应用数据令牌（`45631641,null,true`）为核心信号（对齐 RegionRestrictionCheck 口径），可提取地区码；移除「标题含 Gemini 即可用」的误判源；落地页无令牌判为不可用（覆盖 Google 风控标记机房 IP 的情形） |
| 2026-10-12 | r11：修正令牌匹配——实测当前落地页令牌为 JSON 键 `"45631641":true`（非旧格式 `45631641,null,true`），修复可用节点被误判不支持；新增 `":false` 直接判不可用 |
| 2026-10-12 | r12：修复可用节点仍误判不可用——根因是页面里引号被转义为 `\"45631641\":true`、且支持页页脚含 `support.google.com/gemini?p=avatar` 帮助链接被旧规则误判；改为兼容转义引号、仅在 3xx 重定向 Location 指向说明页或带 `not_available` 参数时判不可用，并用桌面 Chrome UA 与无斜杠 URL；无令牌且无负面信号时判未确认而非不可用 |
| 2026-10-12 | r13：修正反向误判（不可用节点显示可用）——`\"45631641\":true` 是**所有节点都有的全局功能标志**（含被风控/地区不支持节点），不能作可用信号；改以 WIZ 应用数据令牌 `45631641,null,true` 为唯一可用信号（Google 仅在该出口真正可用时内嵌），只有全局标志而无 WIZ 令牌的页面判为「地区不支持 / IP 被风控」 |
| 2026-10-12 | r14：修正"全可用/全不可用"二值误判——实测确认 `45631641`（无论 `,null,true` 还是 `":true`）是 Google 在**每个** Gemini 页面都嵌入的全局功能标志，与出口国家无关，作为信号会导致所有节点同值；改为**基于页面内容**判定：含 "not available in your country" 文案或 `not_available` 重定向 → 地区不支持；落地为 Gemini 应用页（标题含 Gemini）→ 可访问；其余 → 未确认（绝不误判）。配套 web 测试版 SOURCE_URL 同步、缓存戳 poc19 |
| 2026-10-12 | r15：按用户反馈加**版本日期时间**（便于确认是否更新成功）+ **Gemini 诊断指纹**；实测确认「不支持地区」匿名请求也返回同款 Gemini 应用壳（静态 HTML 无 not-available 文案），故 r14 的"应用壳→可用"会把不支持节点误判为可用，且 `,2,1,200,"XXX"` 地区码非出口地理（US 节点取到 TWN、JP 节点取到 USA）。诊断指纹把 ST/TT/NAV/NAVR/G/W/C/ADV/CMP/SIGN/RC 等信号随结果输出，待用户回传「支持/不支持」两节点的指纹以定位真实区分信号。版本号形如 `2026-10-12 17:24 (r15)`，web 测试版 poc20 |
| 2026-10-12 | r16：用户回传双节点指纹**完全一致**（ST=200 TT=1 G=1 CMP=1 SIGN=1，仅 RC 互换），证实静态落地页无判别力。改以 **Gemini API 端点地区裁决**为主判据：`generativelanguage.googleapis.com/v1beta/models` 由 Google 按出口 IP 服务端判定（同一套私有地理风控库）——支持地区返回 `400 API key not valid`（位置检查先通过才轮到 key 校验），不支持地区返回 `403 User location is not supported for the API use.`；429 判限流；API 不可用时回退落地页信号，应用壳单独出现不再判可用（归 unknown）。web 测试版 poc22 |
| 2026-10-12 | r18：**r17 单看机房标记被实测证伪**——用户 4 节点样本显示可用节点(203.10.99.26 JP)同样全源命中机房+VPN（与不可用节点同 /24 同 ASN），机房标记无判别力。反推出有效判据（与详情页「风险因素」同一份多源数据）：**任一源命中 滥用(abuse)** → ❌（3 个不可用节点均命中，可用节点零命中）；**≥2 独立源命中 代理(proxy)** → ❌（不可用 TW 家宽节点 3 源命中，可用节点仅 IP2Location 1 源）；国家黑名单照旧；无多源代理数据时回退 ip-api/proxycheck；其余（含纯机房/VPN 标记）→ ✅。单测 8 例全过，与 4 个实测节点判定完全一致。主脚本 `205d812`，web 测试版 poc24 |
| 2026-10-12 | r17：**r16 API 判法被实测证伪**——用户实测香港(明确不支持)与美国(访问不了)节点均返回 `API=400 key-gate`，证实 `generativelanguage` 在 key 校验路径**不做地区拦截**，400/403 不能作为地区信号。改回**本脚本已抓取的出口 IP geo 数据**判定，主判据为 **机房/数据中心/代理 IP**（`ip-api` 的 `hosting`/`proxy` + `proxycheck` 的 `proxy`/`vpn`）→ Google 对 Gemini 按 IP 信誉严拦截，与出口国家无关；辅判据为**国家黑名单**（RU/BY/IR/KP/SY/CU/AF）。关键点：香港/澳门/台湾均在 Gemini 网页版官方支持列表(230+ 地区)内，这些地区节点访问不了是 IP 信誉(机房/VPN)问题，由 datacenter 判定处理。单元验证 7 例全过（US机房/HK-VPN→datacenter❌，JP干净→available✅，RU→region-blocked❌）。主脚本 `0a52ad7`，web 测试版 poc23 |
| 2026-07-19 | r8：补充 ASN/组织、IPPure 分流出口识别、Apple 地图通知与多源稳定性修复 |
| 2026-07-18 | r1：初版进入 testing |

## 已知限制

- 第三方数据库更新频率、口径、限流和风控各不相同，部分来源未返回属于正常情况。
- 流媒体结果只反映检测当时的网络响应，不代替登录账号后的实际播放。
- 当前优先检测 IPv4，IPv6 尚未单独出报告。
- Loon `generic` 没有原始 TCP 与节点 DNS 查询能力，因此无法等价提供 VPS 脚本中的 SMTP 25、DNSBL、DNS 泄漏、traceroute、MTU 或 NAT 类型检测。

## 致谢

- [Roddy-D 的 Loon 节点质量查询插件](https://github.com/Roddy-D/Loon_plugins)
- [xykt 的 IPQuality](https://github.com/xykt/IPQuality)
