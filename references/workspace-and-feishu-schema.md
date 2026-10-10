# 工作区、watchlist 与飞书 CLI 字段

## 客户首次使用

客户默认没有作者的工作区、MCP 配置、飞书 Base、飞书 token 或 ASIN 清单。首次配置必须在客户自己的电脑和账号中完成；不要复制作者的 `config/mcporter.json`、Base token、飞书授权、自动化文件或 `output/`。首次业务输入只收集要监控的 ASIN 清单和站点；MCP key 的保存位置由客户自行选择，Skill 不要求客户报告位置。

## 当前工作区入口

对当前项目使用客户明确给出的本地工作区或当前项目根目录。分发给其他客户时，不要把作者的本机绝对路径、项目文件、ASIN、Base 或历史输出复制过去。

优先检查：

- `config/amazon_competitor_watchlist.json`：角色、优先级、目标/竞品池和 daily light 范围；
- `config/competitor_keyword_watchlist.json`：当前固定 SellerSprite + 飞书 CLI runner 的实际 ASIN、关键词和飞书目标；
- `config/mcporter.json`：仅做存在性/可读性检查，绝不打印内容；
- `scripts/run_amazon_us_competitor_watch.py`：项目内固定每日入口；
- `output/competitor-intelligence/YYYY-MM-DD/`：每日 JSON/CSV、ASIN 状态和日期级状态；
- `amazon-traffic-dashboard/data/`：仅在用户另外要求本地经营数据时读取，不把它与 SellerSprite 市场观察混成同一来源。

如果另一份配置与固定 runner 不一致，先报告冲突并使用用户明确指定的那一份；不要静默合并、替换或新增 ASIN。

## 角色归一化

建议将每个对象归一化为：

| 字段 | 值 | 含义 |
| --- | --- | --- |
| `monitor_scope` | `own_product` | 自有/目标产品，允许与本地店铺数据按 ASIN 对照，但 SellerSprite 仍是市场观察 |
| `monitor_scope` | `competitor` | 外部竞品，只输出市场观察，不输出店铺真实销售结论 |
| `monitor_scope` | `unclassified` | 客户未声明归属，先照常监控，不额外索取品牌或店铺资料 |
| `role` | `target` / `own_fba` | watchlist 明确标注的自有产品 |
| `role` | `core_p1` / `core_p2` / `competitor` | 已确认竞品 |
| `role` | `unclassified` | 未声明归属的监控对象 |
| `coverage` | `ASIN池` / `非全店覆盖` / `全店覆盖` | 说明监控覆盖范围 |

不允许从品牌名称、历史数据或 ASIN 猜测所有权。没有角色信息时写 `unclassified` 并继续监控，不把补充归属变成首次配置的额外问题。

## 飞书 Base 建议字段

日快照表建议一行对应 `日期 + ASIN + 监控类型`：

| 字段 | 建议类型 | 说明 |
| --- | --- | --- |
| `记录标题` | text | `YYYY-MM-DD ASIN 监控类型`，用于人工定位 |
| `采集时间` | datetime | 采集时间，不代替业务日期 |
| `日期` | date/text | 北京时间业务日期 |
| `站点` | text | 默认 `US` |
| `ASIN` | text | 10 位大写 ASIN |
| `监控类型` | select/text | `自有产品`、`竞品` 或 `未分类` |
| `品牌` | text | SellerSprite 返回值 |
| `产品标题` | text | SellerSprite 返回值 |
| `价格` | number/currency | 缺失留空，不写 0 |
| `Coupon` | text | 保留原文本或摘要 |
| `评分` | number | 缺失留空 |
| `评论数` | number | 缺失留空 |
| `BSR/类目排名` | text/number | 保留 MCP 语义 |
| `自然关键词数` | number | 来自关键词工具的观察值 |
| `广告关键词数` | number | 来自关键词工具的观察值 |
| `主流量词` | text | 数组用 `; ` 拼接 |
| `疑似广告词` | text | 只能写“疑似”，不能写确定投放 |
| `数据源` | text | MCP 工具名 |
| `覆盖说明` | text | ASIN 池/非全店覆盖等 |
| `变化摘要` | text | 与上一有效快照的差异 |
| `状态` | select/text | `completed`/`partial`/`failed` |
| `备注` | text | 缺失字段、错误和证据路径 |

变化表建议字段：`日期`、`ASIN`、`监控类型`、`变化类型`、`变化前`、`变化后`、`变化幅度`、`严重级别`、`建议动作`、`数据源`、`备注`。

## 兼容当前已有表

当前固定 runner 已使用过的字段名包括：

`记录标题`、`采集时间`、`站点`、`ASIN`、`产品标题`、`品牌`、`价格`、`Coupon`、`评分`、`评论数`、`BSR/类目排名`、`类目路径`、`主流量词`、`疑似广告词`、`数据源`、`覆盖说明`、`备注`、`变化类型`、`严重级别`、`建议动作`。

写入前仍必须用 `lark-cli base +field-list` 读取真实字段和类型。不要因为建议 schema 与线上表略有不同，就直接创建同名字段或覆盖现有表。

关键词明细表每行建议使用 `日期 + ASIN + 关键词 + 流量类型` 作为定位键；关键词矩阵表按关键词为行、日期为列时，先读取现有日期列和记录 ID，再更新对应单元格。
