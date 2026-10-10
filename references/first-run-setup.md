# 首次配置、输入收集与隐私边界

这份流程默认面向没有使用过 SellerSprite MCP 和飞书 CLI 的客户。配置必须在客户自己的电脑、MCP 配置和飞书账号中完成；作者的 key、Base、授权和 ASIN 清单一律不复用。首次业务输入只收集“要监控的 ASIN 清单”和“站点/市场”。

## 1. 先告诉客户这套链路做什么

用一句话说明：

`Codex 负责安排动作，SellerSprite MCP 负责查市场数据，飞书 CLI 负责把已整理结果写入客户自己的 Base，本地文件负责保留证据。`

先讲清楚四个结果：

- 配好后可以监控客户确认的 ASIN 清单，清单可以包含自有产品和竞品；
- 每个 ASIN 每天保留一条快照，并记录变化；
- 飞书只接收归一化后的必要字段，不接收原始 MCP 大 payload；
- 广告、Listing、价格、库存和采购不会被自动修改。

## 2. 首次输入清单

只向客户收集以下两项内容。ASIN 可以在聊天中提供；密钥和私密配置不可以在聊天中提供。

```text
需要监控的 ASIN 或商品链接：
站点/市场：默认 US
```

必须让客户确认：

- ASIN 清单确实是本次监控范围；
- 站点/市场；未指定时按 `US`；
- 是否接受将 `marketplace + ASIN` 作为最小查询参数发送给 SellerSprite。

ASIN 不要求分成“自有”和“竞品”两组。客户主动说明归属时可以写入 `own_product`/`competitor`，否则使用 `unclassified`，不追问品牌名、店铺名、关键词、Base、表名、频率或时间，也不从历史文件或“看起来像自有”的 ASIN 猜测所有权。

## 3. 配置飞书 CLI

先做只读检查：

```text
node --version
npm --version
command -v lark-cli
lark-cli --help
```

客户没有 CLI 时，按飞书 CLI 官方仓库的推荐方式，由客户在自己的终端安装：

```text
npx @larksuite/cli@latest install
```

官方入口：<https://github.com/larksuite/cli>。如果客户机器上已经有 CLI，先查看版本和帮助；需要更新时使用官方支持的 `lark-cli update`。

如果客户没有自己的飞书应用配置，按官方引导执行 `lark-cli config init --new`；如果已有应用，使用客户自己的 App ID，并通过安全 stdin/配置向导输入 App Secret。App Secret 不放在命令行、Skill、日志或聊天中。

然后检查用户授权：

```text
lark-cli auth status --json --verify
```

未授权时，按官方 AI Agent 流程发起；先使用官方推荐的常用权限集合：

```text
lark-cli auth login --recommend --no-wait --json
```

如果后续 Base 操作返回明确的 `missing_scopes`，只针对返回的 scope 增量授权：

```text
lark-cli auth login --scope "<missing_scope>" --no-wait --json
```

把 CLI 返回的授权链接原样交给客户；如果由 Agent 发起授权，还必须按 `$lark-shared` 生成并展示二维码，然后结束当前轮次等待客户完成授权。不要在同一轮继续轮询 device code，不要缓存授权链接或 device code。客户回来确认后，再完成 device-code 流程并验证：

```text
lark-cli auth status --json --verify
```

如果只是 Base 操作，不要为了方便把 `--as user` 改成 `--as bot`；权限错误按 `$lark-shared` 和 `$lark-base` 分流，不循环切身份。

## 4. 配置 SellerSprite MCP

先只读检查客户本机是否已经有 `mcporter` 和 `sellersprite-mcp`：

```text
command -v mcporter
mcporter config doctor
mcporter config list
mcporter list sellersprite-mcp --schema --config <客户自己的 mcporter 配置>
```

如果没有 `sellersprite-mcp`，让客户按 SellerSprite 提供的 MCP 配置方式在自己的 `mcporter`/secret store 中注册 endpoint 和 API key。不要猜 endpoint、参数格式或工具名，也不要复制作者的 `config/mcporter.json`。

客户需要准备 SellerSprite MCP key，但保存方式由客户自己选择。Skill 不要求客户选择某一种方式，也不需要知道 key 放在哪里；只做“不泄露”和“能否使用”的验证：

- 客户自行在其选定的终端环境、密钥管理器、MCP 配置或其他安全位置完成配置；
- Codex 只验证服务器存在、schema 可读和调用是否成功，不读取或管理保存位置；
- 不接受客户把 key 粘贴到聊天；不读取、回显、保存或转发 key；
- 不把 key 放进自动化 prompt、Skill、源码、JSON/CSV 输出、异常日志或飞书记录。

完成后只做最小连通性验证。不要在连通性测试中附带客户品牌、产品标题、销售数据、广告数据、本地文件或飞书内容。

## 5. 脱敏和最小外发原则

### 发给 SellerSprite MCP 的内容

默认只发送完成查询必需的：

- `marketplace`；
- 客户确认过的 `asin`；
- 当前工具要求的最小分页/日期参数（仅在客户明确开启关键词或趋势查询时）。

不发送：客户姓名、公司信息、品牌策略、供应商、采购成本、广告预算、销售报表、财务数据、库存文件、飞书 Base 原始记录、Cookie、API key、App Secret、access token、机器路径和整个 watchlist 文件。

ASIN 本身是查询所需的业务标识，因此不能承诺“完全不外发 ASIN”。如果客户不接受 ASIN 发给 SellerSprite，就必须停止 SellerSprite 采集并说明原因。

### 写入客户自己的飞书

飞书 CLI 只写归一化快照、变化和状态字段；不写原始 MCP payload、密钥、本地绝对路径中的私密目录、客户上传的原始销售/财务文件或无关个人信息。记录中可以保留 ASIN，因为它是客户选择的业务主键。

### 留在本地

原始 MCP 返回、错误上下文、配置检查结果和 JSON/CSV 快照留在客户本机。日志只写脱敏错误；发现 `secret`、`token`、`key`、Cookie 等模式时先遮蔽再输出。自动化 prompt 只写非敏感路径和范围，不写任何凭据。

首次调用前要把上述数据流告诉客户，并取得明确确认；新增外部目的地、扩大字段范围或更换监控池时重新确认。

## 6. 单 ASIN 首次测试

配置和输入都确认后，按这个顺序：

1. 只选一个客户确认的 ASIN，优先列表中的第一项；
2. 调用 SellerSprite 最小工具，确认返回有可处理字段；
3. 在本地生成一份带日期的快照和 `run_status.json`；
4. 按飞书 CLI 官方 Base 能力和当前 `$lark-base` 指引选择或建立写入目标；不把 Base URL、表名或其他资源信息列为首次业务输入。若需要新建，使用默认的监控用途命名和建议字段，并在 CLI 的写入确认门前向客户展示目标与风险；
5. 读取真实 table/field 结构，不猜字段名和类型；
6. 通过飞书 CLI 写入一条测试记录；
7. 回读当天 ASIN，核对日期、ASIN、关键字段和是否真的有记录；归属未知时核对 `unclassified`，不强行要求自有/竞品分类；
8. 只有以上都通过，才扩展到全量 ASIN 和每日自动化。

测试失败时保留错误产物，报告卡在 SellerSprite、权限、字段、写入还是回读，不使用旧数据补齐。

## 7. 客户确认后的每日运行

首次测试通过后，才按主 Skill 执行 daily light：

- 只跑客户确认的 ASIN；关键词默认关闭，不在首次配置时追问；
- 客户提供归属时分别标记，未提供时标记 `unclassified`；
- 每个 ASIN 一条当天快照，变化另行记录；
- 飞书写入使用 `--as user` 和幂等 upsert；
- 写后回读验证；
- 任何失败分开报告本地、SellerSprite 和飞书状态。

## 8. 不接受的做法

- 让客户把 MCP key、App Secret、access token 或 Base token 发到聊天；
- 规定客户必须把 MCP key 放在某一种指定位置，或要求客户告诉 Skill key 的保存位置；
- 复制作者的 MCP/飞书配置给客户；
- 没有客户确认就把“所有产品”或新竞品加入监控；
- 把 ASIN 之外的客户资料打包发给 SellerSprite；
- 把原始返回、凭据或客户文件写入飞书；
- 以“已经脱敏”为理由掩盖 ASIN 实际会发给 SellerSprite 这一事实；
- 配置未完成就运行自动化，或用历史数据冒充首次测试结果。
