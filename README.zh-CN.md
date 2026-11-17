# chat-human

**`humanizer` 去的是文章 AI 味；`chat-human`（配合 [`nb-speak`](https://github.com/NB-Corp/nb-speak)）治的是你的 Personal Agent（OpenClaw / AstrBot / Telegram / WhatsApp / 微信 / QQ / Claude Code）把聊天窗口当公文汇报和裸管道表格垃圾场。**

[English](README.md) · **简体中文** · [技能正文 (`skills/chat-human/SKILL.md`)](skills/chat-human/SKILL.md) · [PA 原生插件与协议主仓 (`NB-Corp/nb-speak`)](https://github.com/NB-Corp/nb-speak/blob/main/README.zh-CN.md) · [在线同屏对比舞台](https://nb-corp.github.io/nb-speak/zh/) · [llms.txt](llms.txt)

---

市面上的「去 AI 味 / Humanizer」提示词针对的是博客、文章与长文润色。一旦你把大模型接进 **OpenClaw**、**AstrBot**、微信、QQ、Telegram、WhatsApp、飞书、Discord 或命令行编码助手，文章级规则根本拦不住它在手机聊天框里犯这四类毛病：

1. **狂吐裸 `|---|---|` Markdown 管道表格**：在 Telegram、WhatsApp、QQ、微信气泡里直接碎成满屏竖线横杠乱码（`openclaw#36323`，写进 SOP 仍反复破功 15+ 次）；
2. **动辄 200 字公文汇报墙**：明明两句短话就能说清，非要甩出 `### 总结` + `1. **加粗**：` + `希望这对你有帮助！如有需要随时告诉我`；
3. **超过 150 字不分段或把代码块切碎**：传统正则分段一遇长文就失效、一遇代码注释句号就把 ` ```sql ` 劈成七八截（`AstrBot#8614`）；
4. **反人类聊天细节**：取件码后面硬挂句号（`663920。`）、夹带 `（叹气）` / `*sighs*` 剧本括号，或在开启语音时把文字气泡一字不落重新念一遍。

`chat-human` 是一个零依赖、与具体人设解耦的通用 Agent Skill（`v0.2.0`）：把**真人在手机上拇指打字与剥离重内容的物理规律（Layer 1，默认纯文字短气泡 + 独立代码/表格卡，语音默认关闭）**与**角色语气适配（Layer 2）**彻底分层。

## 30 秒快速安装

### 1. 一键安装通用 Agent Skill（`skills.sh` · Claude Code · Codex · Cursor · OpenClaw）

```bash
# 一行命令安装至 Claude Code、Codex、Cursor、Gemini CLI、Windsurf 等全系支持 skills.sh 的客户端
npx skills add NB-Corp/chat-human --global --agent '*'

# 或指定安装到单个 Agent
npx skills add NB-Corp/chat-human --global --agent claude-code
npx skills add NB-Corp/chat-human --global --agent codex
```

### 2. 挂载 Personal Agent 原生防破功插件（OpenClaw & AstrBot）

单纯靠提示词在长会话压缩后仍可能退化。配合主仓 **[`NB-Corp/nb-speak`](https://github.com/NB-Corp/nb-speak)** 内置的原生插件，可在发送前执行确定性清洗与分泡编排：

```bash
# A. OpenClaw 原生插件（零克隆一行安装至 ~/.openclaw，自动拦截裸管道表格、隔离代码块并分泡发送）
npx --yes github:NB-Corp/nb-speak openclaw --apply

# B. AstrBot 原生插件（在面板直接粘贴 https://github.com/NB-Corp/astrbot_plugin_nb_speak 安装，或命令行一行拉取：）
git clone --depth 1 https://github.com/NB-Corp/astrbot_plugin_nb_speak <AstrBot>/data/plugins/astrbot_plugin_nb_speak
```

### 3. OpenAI / SSE 通用桥接网关与其他支持的客户端

```bash
# 在任意 OpenAI / DeepSeek / 本地 llama-server 前启动零依赖桥接网关 (http://127.0.0.1:18090)
NB_SPEAK_API_KEY="sk-..." npx --yes github:NB-Corp/nb-speak serve --port 18090 --upstream-url https://api.deepseek.com/v1

# 其他支持的客户端：一键挂载至 DeepSeek Harness (DSH) WebUI
npx --yes github:NB-Corp/nb-speak-dsh --apply --profile web
```

---

## 手机聊天窗前后对比

| 真实 Personal Agent 场景 | 传统 Agent 默认输出 | 接入 `chat-human`（配合 `nb-speak` 门禁） |
| --- | --- | --- |
| **TG / WhatsApp / QQ 进度与表格对比** | 180 字寒暄铺垫 + 裸 `\| 任务 \| 状态 \| 优先级 \|` 竖线乱码折成 9 行 + `希望这对你有帮助！` | **气泡 1**：`路由修完了，文档还在收尾`<br>**独立键值卡**：`- 修路由 — 状态: 完成 · 优先级: 高` / `- 补文档 — 状态: 进行中 · 优先级: 中` |
| **SRE 灰度确认（无气泡工具普通回复）** | `可以推进全量。不过25分钟只覆盖了一个流量切面，我建议先推到50%再跑15分钟观察连接池队列，并提前明确0.1%回滚阈值。` | `错误率 0.02% 没动，p99 降到 38ms`<br><br>`连接池也稳，直接推全量吧` |
| **连改三次需求** *(配置熟人/搭档 Persona 时)* | `频繁修改需求会降低整体效率，下次请提前规划哦！（叹气）这是新版布局请检阅。` | **气泡 1**：`第三回“真不改了”哈`<br>**气泡 2**：`v3 放共享盘了，瞄一眼顺不顺眼` |
| **办妥交差** *(显式开启可选语音条时)* | 单条 150 字全能管家三段式 + 16 秒语音把同一段字照念一遍。 | **气泡 1**：`周四下午窗口约上了`<br>**气泡 2**：`四页打印稿放桌面，过去签个字就行`<br>*(可选 6 秒末泡语音补充)*：`外头今天风挺大的。下午出门记得带件外套。` |

## 双层正交架构：同一套气泡物理，适配任意人设

```text
  当轮用户要求 / OpenClaw SOUL.md / AstrBot 人格 （未配置时回退至平辈默认态）
                                │
                                ▼
  ┌───────────────────────────────────────────────────────────────────┐
  │ Layer 2 · 人设与语气适配层                                        │
  │ 称谓人称一致性 · 冷暖亲疏 · 机锋比例 · Emoji 策略 · 情绪物理刹车  │
  └─────────────────────────────┬─────────────────────────────────────┘
                                │
                                ▼
  ┌───────────────────────────────────────────────────────────────────┐
  │ Layer 1 · 通用聊天气泡物理层（始终生效）                          │
  │ 1–4 条短气泡 · 代码与表格外置 · 短句与取件码去句号                │
  │ 禁旁白说教与剧本括号 · 纯文字默认（可选互补语音静默降级）         │
  └───────────────────────────────────────────────────────────────────┘
```

- **有无气泡工具皆原生可用**：宿主提供 `send_message` / `send_bubble` 时单步工具收束（`end_turn: true`）；普通纯文本宿主直接用 `\n\n` 输出微节拍（自动保留 `北门丰巢07号柜\n取件码 663920` 双行复制单元，绝不裸露 JSON 或 `气泡1：` 标签）。
- **语音默认关闭 & 跨后端中立**：默认仅输出纯文字短气泡与干净代码/表格卡片；仅当宿主显式开启语音或用户主动要求时才在末条气泡附带互补语音，通用兼容任意 OpenAI `/v1/audio/speech`、本地 TTS 或浏览器 Web Speech，失败静默回退纯文字。
- **四级语气优先级**：**(1)** 当轮用户直接指令 > **(2)** 宿主激活的 `SOUL.md` / `PERSONA.md` / AstrBot 人格 > **(3)** 会话场景（1v1 私聊 vs 工作群） > **(4)** **平辈技术搭档默认基线**（零自作多情角色扮演、零客服卑微腔）。

## 内置零依赖工具（Python 3.10+ 标准库）

### 1. `bubble_lint.py` —— 8 规则确定性短气泡检查器

静态拦截气泡内残留的 Markdown 表格/标题/代码块（`bubble.markdown_structure`）、字数超限小作文、短句与取件码末尾句号（`style.trailing_period`）、剧本括号动作（`style.stage_brackets`）、管家套话、纯文本 JSON 泄露（`plain.meta_leak`）以及可选语音复读率超标：

```bash
python skills/chat-human/scripts/bubble_lint.py --surface plain --json <<'EOF'
错误率 0.02% 没动，p99 降到 38ms

连接池也稳，直接推全量吧
EOF
```

### 2. `proactive_gate.py` —— 免打扰与沉默衰减主动开口门禁

供 OpenClaw / AstrBot / 定时心跳调用：内置免打扰时段（默认 `01:30-10:00`，退出码 `3`）、最小冷却时间、每日上限与连续未回指数衰减（`ack-user` 重置连击），触发时仅输出纯状态头 `<proactive_tick ... />`。

```bash
python skills/chat-human/scripts/proactive_gate.py --state-file .tmp/gate.json check --ignore-quiet --force
python skills/chat-human/scripts/proactive_gate.py --state-file .tmp/gate.json ack-user
```

## 运行测试

```bash
python -B -m unittest discover -s tests -v
```

## 开源协议

[MIT](LICENSE) © 2026 NB-Corp
