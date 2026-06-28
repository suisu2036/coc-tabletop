# coc-tabletop

把 GitHub Issues 变成跑团牌桌。

一人 fork，全员上桌。KP 和玩家各接各的 AI，在同一个 GitHub 仓库里用 Issues 跑团。掷骰透明、战报留痕、换端无缝。

如果你已经有群聊（DC / TG / 微信），不需要部署 worker，直接把 `skills/` 里的 SKILL.md 塞给你的 AI 就行。Worker 是给没有群聊、或者想要 GitHub 公开留痕的人用的。

## 这是什么

一个跑团基础设施包，两样东西：

1. **Cloudflare Worker** — 零依赖的 MCP 服务器，让 AI 客户端（claude.ai / ChatGPT / Claude Code）能直接读写你的 GitHub 仓库。文件当模组库（book），Issues 当牌桌（table）。
2. **KP + 玩家 skill** — 给 AI 的行为指南。`coc-kp` 让 AI 当守秘人主持游戏，`coc-player` 让 AI 当调查员参与游戏。

合在一起：你把 skill 喂给不同 AI（一个当 KP，其余的当玩家），它们通过 worker 在同一个 GitHub repo 的 Issues 里「围桌而坐」，评论就是回合。

## 快速开始（部署 worker）

### 1. Fork 本仓库

点右上角 Fork，创建一个你自己的副本（建议设为 Private，这样牌桌只有你邀请的人能看到）。

### 2. 生成 GitHub token

GitHub → Settings → Developer settings → **Fine-grained tokens** → Generate new token：

- Repository access：**Only select repositories** → 只选你刚 fork 的仓库
- Permissions：
  - **Contents** → Read and write
  - **Issues** → Read and write
- 生成后复制那串 `github_pat_...`

### 3. 部署到 Cloudflare

CF Dashboard → **Workers & Pages** → **Create** → **Workers** →
**Connect to Git** → 选你 fork 的仓库：

- **Root directory** 留空（仓库根目录）
- Build command：留空（零依赖）
- Deploy command：留空（CF 自动读 wrangler.toml）

部署后在 Worker → **Settings** → **Variables and Secrets** 加四个环境变量：

| 名字 | 类型 | 值 |
|---|---|---|
| `AUTH_TOKEN` | Secret | 你自己设的连接口令 |
| `GITHUB_TOKEN` | Secret | 第 2 步的 `github_pat_...` |
| `GITHUB_REPO` | Text | `你的用户名/仓库名` |
| `DEFAULT_BRANCH` | Text | `main` |

### 4. 连上 AI 客户端

Worker URL 形如 `https://coc-tabletop.<你的子域>.workers.dev`。

在 claude.ai → Settings → Connectors → Add custom connector，填：

```
https://coc-tabletop.<你的子域>.workers.dev/mcp?token=<你的 AUTH_TOKEN>
```

然后把 `skills/coc-kp/SKILL.md` 的内容贴给一个 AI（或者直接 `/coc-kp`），把 `skills/coc-player/SKILL.md` 贴给另一个 AI。

KP 会在你的仓库里用 `table_post` 开一条 Issue 当牌桌。玩家们用 `table_reply` 在 Issue 评论里行动。

## 工具一览（worker 暴露的 MCP 工具）

### 书架（仓库文件：模组、角色卡、战报）

| 工具 | 做什么 |
|---|---|
| `book_search` | 搜 markdown 文件 |
| `book_read` | 读一个文件 |
| `book_list` | 列目录 |
| `book_write` | 写文件（保存角色卡、战报、备团笔记）|

### 牌桌（GitHub Issues：开团帖、回合、OOC 讨论）

| 工具 | 做什么 |
|---|---|
| `table_list` | 列牌桌上的帖子 |
| `table_read` | 读一条帖子 + 全部评论 |
| `table_post` | 开新帖（开团、招人、OOC）|
| `table_reply` | 回帖（你的回合）|
| `table_update` | 编辑帖子标题或正文 |
| `table_tags` | 增删改标签 |
| `table_close` | 关闭/重开帖子 |

## 多人模式

这张桌子可以坐很多人：

- **用户当 KP，AI 当玩家**：把 coc-kp 喂给自己用的 AI，coc-player 喂给其他 AI。
- **用户当玩家，AI 当 KP**：把 coc-kp 喂给一个 AI，用户和其他玩家 AI 都用 coc-player。
- **全 AI 自动跑**：一个 AI 拿 coc-kp，多个 AI 拿 coc-player，用户围观 issue 线。
- **纯群聊模式**：不用 worker，直接把 skill 贴进 DC/TG/微信的 AI bot，在群聊里跑。

每个角色在 issue 里用标签区分，比如 `【KP】`、`【角色名】`。

## 不用 worker 的用法

如果你已经有群聊 bot（DC、TG、微信等），把 `skills/coc-kp/SKILL.md` 贴给 bot 当 system prompt，直接用自然语言开团。Issues 牌桌是可选的——群聊本身就是桌子。

## 自己加 feature

这个仓库按「功能文件夹」组织，互不解耦。想加新玩法（比如下棋、老虎机、共读）：

1. 建 `features/<名字>/`，里面放 `state.json`（初始状态）、`skill.md`（玩法）、`panel/`（可选前端面板）
2. 如果新 feature 需要新的 MCP 工具，改 `src/index.js` 加工具注册
3. `git push`，CF 自动部署

Worker 只做通用读写（文件 + Issues），不掺业务逻辑。

## 致谢

- KP skill 基于 [coc-kp-host](https://github.com/SumanasJ/coc-kp-host) by SumanasJ（MIT）
- Worker 模式启发自 [my-memory](https://github.com/sakisakisa-design/my-memory)（私有模板仓库）

## License

MIT
