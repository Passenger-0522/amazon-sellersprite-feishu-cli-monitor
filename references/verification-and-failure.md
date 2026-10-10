# 验收与故障分流

## 三个独立验收门

1. **本地采集门**：批准范围、今日原始 MCP 返回、ASIN 快照/差异和状态文件齐全。
2. **飞书 CLI 门**：命令成功、写入数量可解释，并回读确认日期、ASIN、角色和关键字段。
3. **结论门**：每个异常都能追溯到今日值、上一值、MCP 工具和本地文件；缺失值保持 `null`/空/`—`。

三道门不能互相替代。MCP 成功不等于飞书已写入；飞书写入成功也不等于数据完整。

## 常见情况

| 现象 | 判定 | 处理 |
| --- | --- | --- |
| `mcporter` 配置或入口读取超时 | `blocked`，未执行 | 报具体文件，停止，不调用业务工具 |
| SellerSprite `ENOTFOUND`、`fetch failed` 或连接失败 | 通常 `partial`/`blocked` | 保留错误产物，MCP 成功数如实为 0；不以历史快照代替今日结果 |
| 某个 ASIN 失败，其他 ASIN 成功 | `partial` | 继续已批准范围内其他 ASIN，列出失败 ASIN/工具 |
| 本地快照完整，飞书 CLI 91403/权限/Keychain 失败 | 本地完成、飞书 `partial` | 保留本地产物，不循环换身份或无限重试 |
| CLI 返回 `confirmation_required` | 等待用户确认 | 展示目标和风险；用户未明确同意前不加 `--yes` |
| ASIN 状态文件完成，但日期级聚合文件缺失 | `partial` | 不能报整体 completed，注明缺失聚合产物 |
| SellerSprite 某字段空或 `data=[]` | 数据缺失 | 保留空值并标注来源，不解释成无需求、0 销量或0排名 |
| 关键词未命中 | 正常观察值 | 写“未命中/未返回”，不是把排名写成 0 |
| 店铺维度工具不存在 | `非全店覆盖` | 改用已确认重点 ASIN 池，并在摘要声明覆盖边界 |

## 不自动做的事

- 不登录或进入任何 Amazon 账号，不打开、唤醒或控制紫鸟；
- 不替用户下载 Business Report 或广告报表；
- 不将 SellerSprite 估算销量、流量、关键词或预测包装成后台事实；
- 不自动调价、改广告、改 Listing、改库存、下采购单或写回 Amazon；
- 不手工补写一条“成功”的飞书记录来掩盖写入失败。

## 摘要证据

至少保留以下绝对路径：

- 日期级 `amazon-us-run_status.json`；
- 每个 ASIN 的 `run_summary.json` 和 `run_status.json`；
- 当日 `daily_snapshot.json`、`daily_changes.json`；
- MCP 原始返回目录；
- 飞书 CLI 最终返回或回读结果（不得包含密钥）。
