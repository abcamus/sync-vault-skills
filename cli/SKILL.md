---
name: "sync-vault-cli"
description: "Use Sync Vault CLI commands (obsidian sync-vault:list/search/read/info/doctor/config/device). Invoke when user wants to inspect/search/read cloud files or diagnose Sync Vault from the CLI."
---

# Sync Vault CLI

本技能用于让大模型在需要“通过 Sync Vault CLI 查询/检索/读取云端文件、获取账号信息、诊断同步状态、查看配置或设备列表”时，能稳定、正确地调用本项目提供的 CLI 命令。

## 适用范围与前提

- 适用：Obsidian Sync Vault 插件已加载且已注册 CLI handler（Obsidian 版本需要支持 `registerCliHandler`）。
- 命令形式：`obsidian sync-vault:<subcommand> key=value key2=value2`（所有子命令都支持 `help=true`）。
- 输出形式：
  - `list/search/info/doctor`：输出为 **JSON 字符串**（可直接解析）。
  - `read`：输出为文本（可能被套餐截断；PDF 在 Free 下会拒绝）。
- 套餐限制（重要）：
  - `search`：Free 通常仅返回前 5 条；Basic/Pro 支持分页。
  - `list`：Free 通常最多返回 20 条；Basic/Pro 支持 `limit/offset` 分页。
  - `read`：Free 文本读取通常最多 10KB 且会追加截断提示；PDF 读取是 Pro 功能。

## 快速自检工作流（推荐顺序）

1. `obsidian sync-vault`：列出可用命令（确认 CLI 已可用）
2. `obsidian sync-vault:info cloud=<cloud>`：确认账号可用/容量健康
3. 文件发现优先：
   - 有关键词：`obsidian sync-vault:search ...`
   - Provider 不支持 search 或想要稳定遍历：`obsidian sync-vault:list ...`
4. 内容读取：`obsidian sync-vault:read path=<filePath> cloud=<cloud>`

如遇异常或用户反馈“同步失败/打不开/很慢”：
- `obsidian sync-vault:doctor cloud=<cloud>` 获取诊断报告（JSON）

## 云类型（cloud 参数）

- 常用：`baidu | aliyun | quark | onedrive | onelife`
- 其他（视实际接入与登录状态）：`cos | infinicloud | nutstore`
- 建议：cloud 统一使用小写字符串；若用户未指定 cloud，部分命令会使用当前选择的云盘（例如 `info/doctor` 的内部逻辑）。

## 命令参考

### 1) 帮助

- 主帮助：`obsidian sync-vault`
- 子命令帮助：`obsidian sync-vault:list help=true`（任意子命令都支持）

### 2) list：列出目录文件（支持过滤/递归/分页）

`obsidian sync-vault:list path=<path> cloud=<cloud> limit=<n> offset=<n> type=<type> minSize=<bytes> modifiedAfter=<iso|ms> recursive=true|false`

- `path`：默认 `/`
- `type`：`markdown|md|pdf|image|video|audio|folder|text...`
- `recursive=true`：会触发递归列举（可能较慢）

输出 JSON 结构（核心字段）：
- `files[]`: `{ id, path, name, type, isFolder, size, modified }`
- `pagination`: `{ total, returned, limit, offset, hasMore, nextOffset }`

分页策略：
- 如果 `pagination.hasMore=true`，用 `offset=pagination.nextOffset` 继续取下一页。

### 3) search：关键词检索（文件级，不读内容）

`obsidian sync-vault:search query=<keyword> cloud=<cloud> limit=<n> offset=<n> path=<prefix> type=<type>`

- 约束：至少提供 `query/type/path` 之一，否则会返回结构化错误。
- Provider 不支持 search 时：通常会返回 `isError=true`，并给出建议改用 `list`。

输出 JSON 结构同样包含：
- `files[]` 与 `pagination{ hasMore, nextOffset }`

### 4) read：读取文件内容（文本/部分二进制会按文本解码；PDF 特殊）

`obsidian sync-vault:read path=<filePath> cloud=<cloud>`

- `path` 必填。
- PDF：Free 会拒绝；Pro 会尝试流式解析并抽取前若干页文本（最多 20 页）。

### 5) info：账号与容量信息（执行任何批量任务前建议先跑）

`obsidian sync-vault:info cloud=<cloud>`

- 输出为 JSON（包含 token 状态、容量、告警等）。适合在自动化流程里作为“预检查”。

### 6) doctor：诊断 Sync Vault 状态（网络/登录/容量/同步/Relay）

`obsidian sync-vault:doctor cloud=<cloud>`

- 输出为 JSON，包含：
  - `checks[]`（每项包含 category/status/severity/message/fixHint）
  - `health{ ok, score, summary, riskLevel }`
  - `recentErrors[]`
- 适合把结果直接回显给用户，或根据 `fixHint` 给出下一步建议。

### 7) config：查看/设置插件配置（谨慎使用）

- 列出（会过滤敏感字段）：`obsidian sync-vault:config action=ls`
- 设置：`obsidian sync-vault:config action=set key=<key> value=<value>`

安全约束（必须遵守）：
- 不要在未获得用户明确同意时执行 `action=set`。
- 不要尝试设置/输出敏感字段（密码、cookie、token、激活码等）。

### 8) device：实时协作设备管理（Live Sync）

- 列出已发现设备：`obsidian sync-vault:device action=ls`
- 发送消息（可能打扰他人）：`obsidian sync-vault:device action=send peerId=<id|all> message=<text>`

安全约束：
- `send` 仅在用户明确要求时使用；默认只做 `ls` 观察。

## 典型任务模板

### A. “帮我找云端某类文件并列出最近修改的 50 个”
1. `obsidian sync-vault:info cloud=<cloud>`
2. `obsidian sync-vault:list path=<path> cloud=<cloud> recursive=true type=<type> limit=50 offset=0`

### B. “搜索关键词，必要时翻页直到找到目标”
1. `obsidian sync-vault:search query=<q> cloud=<cloud> limit=20 offset=0`
2. 若 `hasMore=true` 且未找到，继续 `offset=<nextOffset>`

### C. “读取并摘要某个文档”
1. `obsidian sync-vault:read path=<filePath> cloud=<cloud>`
2. 若输出提示被截断或 PDF 被拒绝：提示用户升级/改用其他来源。

## 错误处理与回退策略

- `search` 返回 “not supported/unsupported”：改用 `list` + `type/path/recursive` 组合过滤。
- `read` 报 “File not found”：先 `list` 父目录确认路径是否正确（注意大小写与编码）。
- `info/doctor` 显示未登录：提示用户在插件设置中完成登录/授权后重试。

## 禁止事项（必须遵守）

- 不要输出或回写任何密钥、cookie、token、激活码。
- 未经用户明确同意，不要执行 `config set`、`device send` 等可能产生副作用的命令。