---
name: amazon-sellersprite-feishu-cli-monitor
description: "面向从未使用过 SellerSprite MCP 或飞书 CLI 的客户，先按官方飞书 CLI 流程完成本地配置与授权，指导客户自行配置 SellerSprite MCP，再只收集要监控的 ASIN 清单和站点，由 Codex 编排每日 Amazon 市场监控：SellerSprite MCP 取数、本地保存 JSON/CSV、飞书 CLI 写入并核验。适用于客户指定站点的 Amazon ASIN；不登录 Amazon、不操作紫鸟、不自动修改广告、Listing、价格或库存。"
---

# Amazon SellerSprite + 飞书 CLI 监控

## 目标与分工

把“每天看自己的产品和竞品”固定成：

`watchlist -> SellerSprite MCP -> 本地快照/差异 -> 飞书 CLI -> 写入验收 -> 中文摘要`

- Codex：读取范围、编排调用、归一化结果、判断状态和汇报异常。
- SellerSprite MCP：提供市场侧商品、价格、Coupon、排名、评论、关键词和流量观察值。
- 本地文件：保存原始 MCP 返回、当日快照、差异和状态，作为可追溯证据。
- 飞书 CLI：把结果写入用户自己的飞书多维表格，保留历史并支持筛选。

SellerSprite 的销量、流量、广告词和预测字段都是市场观察或估算；即使 ASIN 是自有产品，也不能当作店铺后台真实销售、广告花费或利润。客户没有主动说明归属时，先按 `unclassified` 监控，不要为了分组额外索取品牌、店铺或后台资料。

本 Skill 面向首次使用者。默认假设客户没有安装飞书 CLI、没有完成飞书授权、没有注册 SellerSprite MCP，也没有配置 MCP key；首次运行必须先走配置和隐私确认，不得直接开始采集。

## 适用范围

当用户要求以下任一事项时使用：

- 帮客户从零配置 SellerSprite MCP、MCP key、飞书 CLI 和飞书授权；
- 每天监控客户确认的 ASIN 清单；清单可以包含自有产品和竞品；
- 记录价格、Coupon、评分、评论数、BSR/类目排名、A+、视频、主图、自然/广告关键词数量；
- 对比今日与上一有效快照，输出价格、促销、排名、内容和关键词异常；
- 把 SellerSprite 结果通过飞书 CLI 写入 Base，核对当天记录是否真的新增或更新；
- 创建或维护每日监控自动化。

不用于登录 Amazon、打开紫鸟、下载 Amazon Business Report/广告报表，或自动改广告、Listing、价格、库存和采购。自有产品的店铺真实数据如需分析，只能读取用户已放入本地工作区的文件，并与市场观察单独标注。

## 首次使用：先配置，再采集

第一次对话必须先读取 [references/first-run-setup.md](references/first-run-setup.md)，并按其中顺序完成：

1. 检查客户本机是否有 `node`、`npm`、`lark-cli`、`mcporter` 和可用的 `sellersprite-mcp` 注册；没有就先停在配置阶段。
2. 告知客户需要准备 SellerSprite MCP key，但由客户自行决定把 key 放在终端环境、密钥管理器、MCP 配置或其他安全位置；Skill 不规定位置、不替客户搬运、不读取或管理 key。绝不要求客户把 key 粘贴到聊天，也不把 key 写入 Skill、自动化提示词、脚本、日志或飞书。
3. 引导客户通过飞书 CLI 完成自己的应用配置和用户授权；飞书 App Secret、access token、COI 等只在本机安全配置，不能发到聊天。
4. 只向客户收集两项业务输入：要监控的 ASIN 清单，以及站点/市场（未指定时默认 `US`）。ASIN 不需要拆成“自有”和“竞品”两份；客户主动提供归属时再标记，否则保持 `unclassified`。不要追问关键词、Base URL、表名、运行频率、运行时间、品牌或店铺资料。
5. 在第一次 SellerSprite 调用前，明确展示数据流和最小外发范围，并要求客户确认：只向 SellerSprite 发送查询所需的 `marketplace + ASIN`；不发送客户产品名称、品牌资料、销售/广告/财务数据、本地文件、飞书原始数据、凭据或 Cookie。
6. 配置、ASIN 和站点确认后才执行首次单 ASIN 测试；先确认 SellerSprite 返回，再按官方飞书 CLI/当前 `$lark-base` 流程选择或建立写入目标，确认飞书 CLI 写入和回读，最后才建立每日自动化。

如果客户要求“任何数据都不能离开本机”，必须说明 SellerSprite MCP 查询本身需要把最小 ASIN 查询参数发送到 SellerSprite；在客户不接受这一点时停止 MCP 采集，不能用模糊表述绕过隐私边界。

详细安装、授权、密钥保存、脱敏和首次测试流程见 [references/first-run-setup.md](references/first-run-setup.md)。

## 输入路由

1. 优先读取客户当前工作区的配置，不从记忆重建 ASIN、关键词或飞书目标。Skill 本身不得携带作者的 ASIN、品牌、Base、绝对路径或历史输出。
2. 客户给出要监控的 ASIN 清单；如果客户给的是商品链接，只做本地只读提取，不打开 Amazon 页面。没有客户明确确认的 ASIN 不得加入每日 watchlist。
3. 已完成配置后，角色优先从客户已有配置读取；客户没有提供归属时统一标记 `unclassified`，照常监控，不为了角色分类额外索取资料。
4. 如果是当前工作区的固定 SellerSprite + 飞书 CLI 入口，才使用 `config/competitor_keyword_watchlist.json` 和 `scripts/run_amazon_us_competitor_watch.py`；直接复用已验证 runner，不现场重拼 MCP 和飞书写入逻辑。
5. 若客户只说“我的所有产品”，只要求客户提供或确认 ASIN 清单；本地库存/产品文件只能作为候选，不能自动外发或自动加入监控。
6. 店铺工具不可用时降级为客户提供的重点 ASIN 池，摘要写明“非全店覆盖”；不得声称 SellerSprite 已覆盖整店。

配置字段、表字段和本地证据目录见 [references/workspace-and-feishu-schema.md](references/workspace-and-feishu-schema.md)。

## 每日运行

### 1. 有界预检

在任何 MCP 调用或飞书写入前，做有明确超时的只读检查：

- watchlist、`scripts/run_amazon_us_competitor_watch.py`、`config/mcporter.json` 可读；
- `mcporter`、`lark-cli` 存在，Python 入口可编译；
- 已配置的 SellerSprite MCP 服务器和飞书用户授权可验证，但不打印配置、密钥、token 或完整认证响应。

预检超时或配置不可读时立即报告“未执行 + 具体文件/阶段”，状态为 `blocked`；不要无限等待或拿历史结果代替当天数据。

### 2. 采集与本地留证

优先运行工作区已经验证的固定入口，例如：

```text
python3 scripts/run_amazon_us_competitor_watch.py --date YYYY-MM-DD
```

它负责读取 watchlist/MCP 配置、调用当前可用的 SellerSprite 工具、生成 JSON/CSV，并尝试通过飞书 CLI 写入现有 Base。不要为了“看起来完整”在 Skill 运行时重新拼接另一套调用。

每日轻监控默认关注：

- 商品详情：标题、品牌、价格、Coupon、评分、评论数、BSR/类目排名、类目路径、A+、视频、主图；
- 关键词/流量：默认不调用；只有客户在首次配置完成后另行明确要求，才增加相应工具和字段；
- 差异：与同一 ASIN 的上一有效快照比较，记录今日值、上一值、变化量、数据源和建议复核动作。

关键词默认为空。只有用户或当前批准配置明确要求关键词时，才调用关键词工具；未命中不是排名为 0，字段缺失不是数值 0。

本地至少检查：每个 ASIN 的 `daily_snapshot.json`、`daily_changes.json`、`run_summary.json`、`run_status.json`，以及日期目录的 `amazon-us-run_status.json`。聚合文件缺失时，即使部分 ASIN 和 MCP 成功，也只能报 `partial`。

### 3. 飞书 CLI 写入与验收

默认使用 `lark-cli base +...`，显式 `--as user` 操作用户自己的 Base；不要把本地 Base token、App Secret、access token 或 COI 写入 Skill、自动化提示词、日志或摘要。

飞书 CLI 未安装、未完成用户授权或 Base 目标未确认时，不得写入。客户没有 Base 时，先展示将创建的 Base/表/字段范围，得到明确同意后再按 `$lark-base` 当前 reference 建立；不要复制作者的 Base 或凭据。

写入前：

1. 用户给 URL 时用 `lark-cli base +url-resolve --url "<url>" --as user`；用户给名称时用 `+title-resolve`。不要猜 `base_token`、`table_id` 或 `record_id`。
2. 用 `+table-list`/`+field-list` 读取真实表和字段类型；只写存储字段，不向 formula、lookup、系统字段或附件字段硬塞普通值。
3. 每个 `日期 + ASIN + 监控类型` 作为幂等键，优先 `+record-upsert`；重跑当天应更新，不应重复追加。
4. 写入后用 `+record-search` 或有明确筛选条件的 `+record-list` 回读当天记录，核对日期、ASIN、角色和关键字段。仅命令返回成功但未核验，不写成“飞书已验收”。
5. 批量写入每批不超过 200 条，同一张表串行写；遇到 `1254291` 按短暂冲突处理，不能并发轰炸。

如果 CLI 返回 `confirmation_required`，向用户说明目标和风险，等待明确同意后才在原命令末尾加 `--yes`；不能静默绕过。`91403` 或权限错误不要循环换身份，分开报告“本地完成、飞书 partial”。

表字段建议和现有固定入口的兼容字段见 [references/workspace-and-feishu-schema.md](references/workspace-and-feishu-schema.md)。

### 4. 结果摘要

中文、结论先行，至少包含：

- 监控范围：站点、已确认 ASIN 数、归属未分类数量、是否全店覆盖；
- SellerSprite：调用尝试/成功、失败工具和具体 ASIN；
- 主要变化：价格/Coupon、评分/评论、BSR、内容、关键词/流量；
- 本地证据：日期目录和状态文件的绝对路径；
- 飞书 CLI：写入尝试数、回读验收数、权限/字段错误；
- 总体状态：`completed`、`partial`、`blocked` 或 `waiting`。

不要因为某个竞品失败而丢弃其他 ASIN 的真实结果，也不要把“没有异常”写成“没有变化数据”。

## 每日自动化

用户要求“每天运行/每天监控”时，使用 Codex automation 工具创建或更新任务；不要在 Skill 中手写 raw RRULE。优先复用当前同一项目、同一 watchlist 的已有自动化，避免重复创建。

只有首次单 ASIN 测试已通过、飞书写入和回读已通过、客户已经确认每日范围后，才创建自动化。首次配置未完成时，自动化只能报告 `waiting/blocked`，不能尝试用默认 key、默认 Base 或历史 ASIN 运行。

自动化提示词只写 Skill 名称、非敏感的本地路径、批准的监控范围、运行时间和验收要求；不得嵌入 API key、飞书 token、Base token 或认证配置原文。

每日自动化仍必须执行有界预检、保留本地快照、分开汇报 MCP 与飞书 CLI 状态。它只监控和记录，不修改 Amazon 账号或紫鸟，也不自动调整任何经营设置。

## 状态判定

详细故障分流见 [references/verification-and-failure.md](references/verification-and-failure.md)。核心口径：

- `completed`：批准范围内的 ASIN 本地快照完整，预期飞书记录写入并回读验收；
- `partial`：部分 ASIN/MCP/飞书写入或聚合产物缺失，但仍有可追溯结果；
- `blocked`：预检、配置、认证或权限在采集前阻断，不能声称已采集；
- `waiting`：没有批准的监控对象，或正在等待客户完成飞书授权/确认写入目标。

失败时保留已经生成的原始返回和错误产物；不使用旧快照填空，不手工伪造飞书记录。
