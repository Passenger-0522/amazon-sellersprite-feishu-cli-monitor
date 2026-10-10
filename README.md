# Amazon SellerSprite MCP + Feishu CLI Monitor

一个面向 Amazon 运营的 Codex Skill：通过 SellerSprite MCP 采集用户确认的 ASIN 市场观察数据，保留本地快照和变化，再通过飞书 CLI 写入多维表格并回读验收。

> 社区项目，与 Amazon、SellerSprite（卖家精灵）和飞书官方无隶属或背书关系。

## 它解决什么问题

把每天重复执行的竞品巡检固定为一条可复用工作流：

```text
ASIN watchlist
  -> SellerSprite MCP
  -> 本地 JSON/CSV 快照与差异
  -> 飞书 CLI
  -> 写入回读验收
  -> 中文结果摘要
```

适合以下场景：

- 监控自有产品和重点竞品的价格、Coupon、评分、评论数和类目排名；
- 按需观察自然关键词、广告关键词和流量词数量；
- 比较当天与上一份有效快照的变化；
- 将归一化结果写入飞书多维表格；
- 在首次单 ASIN 测试通过后建立每日定时任务；
- 将失败区分为本地、SellerSprite MCP 和飞书写入三个环节。

## 安装前准备

你需要在自己的电脑和账号中准备：

- Codex，且支持本地 Skills；
- Node.js 与 npm；
- `mcporter`；
- [飞书 CLI](https://github.com/larksuite/cli)；
- 可用的 SellerSprite MCP 权限和 API Key；
- 一个由你自己控制的飞书应用和飞书账号。

SellerSprite 网页端会员与 MCP 权限请分别向 SellerSprite 确认。不要把 API Key、App Secret、access token 或 Base token 发到聊天、Issue、日志或源码中。

## 安装

直接克隆到 Codex Skills 目录：

```bash
git clone https://github.com/Passenger-0522/amazon-sellersprite-feishu-cli-monitor.git \
  ~/.codex/skills/amazon-sellersprite-feishu-cli-monitor
```

重新启动或刷新 Codex 后，使用：

```text
Use $amazon-sellersprite-feishu-cli-monitor to monitor these ASINs daily in US and write verified snapshots to my Feishu Base.
```

首次使用时，Skill 只会向你收集两项业务输入：

```text
需要监控的 ASIN 或商品链接：
站点/市场：默认 US
```

关键词监控默认关闭。完成基础配置和单 ASIN 验证后，再按需要扩展关键词、趋势或自动化。

## 安全与数据边界

- 不登录或操作 Amazon Seller Central、Amazon Ads 或买家账号；
- 不打开或控制紫鸟浏览器；
- 不自动修改广告、Listing、价格、库存、货件或采购；
- 默认只向 SellerSprite 发送完成查询所需的 `marketplace + ASIN`；
- 原始 MCP 返回和错误上下文留在客户本机；
- 飞书只接收归一化后的必要字段；
- SellerSprite 的销量、流量、广告词和预测属于市场观察或估算，不等同于店铺后台事实；
- 缺失值保留为空，不伪造为 `0`；
- MCP 成功不等于飞书成功，飞书写入后必须回读确认。

如果你不能接受将 ASIN 作为查询参数发送给 SellerSprite，请不要运行 SellerSprite 采集。

## 状态口径

| 状态 | 含义 |
| --- | --- |
| `completed` | 批准范围内的本地快照完整，飞书记录已写入并回读确认 |
| `partial` | 部分 ASIN、MCP 工具、飞书写入或聚合产物缺失 |
| `blocked` | 配置、预检、认证或权限在采集前阻断 |
| `waiting` | 正在等待 ASIN 范围、隐私确认、飞书授权或写入目标 |

## 目录结构

```text
.
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    ├── first-run-setup.md
    ├── verification-and-failure.md
    └── workspace-and-feishu-schema.md
```

- `SKILL.md`：主流程、授权边界和运行状态；
- `first-run-setup.md`：首次配置、最小外发范围与单 ASIN 测试；
- `workspace-and-feishu-schema.md`：watchlist、本地证据和飞书字段建议；
- `verification-and-failure.md`：验收门与故障分流。

## 许可证

[MIT License](LICENSE)

