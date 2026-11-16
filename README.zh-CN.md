# chat-human

**面向 AI Agent 的手机短气泡对话机制、纯文本微节拍排版、互补语音条与可插拔 Persona 语气适配技能。**

[English](README.md) · **简体中文** · [技能正文 (`skills/chat-human/SKILL.md`)](skills/chat-human/SKILL.md) · [DSH 原生气泡插件 (`NB-Corp/nb-speak-dsh`)](https://github.com/NB-Corp/nb-speak-dsh) · [双工具协议与桥接 (`NB-Corp/nb-speak`)](https://github.com/NB-Corp/nb-speak) · [在线同屏工作台](https://nb-corp.github.io/nb-speak/zh/) · [llms.txt](llms.txt)

---

常规的「去 AI 味 / Humanizer」提示词针对的是文章、长文和小说。当你把 Agent 接进微信、飞书、Telegram、QQ、Discord、Slack、支持多条消息的 `SendMessage` 工具，或是普通的纯文本流式聊天窗口时，文章级规则拦不住它在手机屏幕上发 180 字编号备忘录、顺手给同事上三段边界风险课、每轮走一遍「`记好啦！+ 顺手替你办妥落盘 + 快去吃热乎的乖乖睡觉吧！`」的全能管家三段式，或者把文字气泡换几个词在语音条里重新播音念一遍。

`chat-human` 是一个彻底通用化的 Agent Skill：把**真人在手机上用拇指打字、按住说话的物理交互规律（Layer 1）**与**角色人设及语气偏好（Layer 2）**解耦开来，同时原生适配**多气泡工具模式（`SendMessage` / `messages[]`）**与**无气泡工具的普通回复模式**。

```text
  当轮用户要求 / 外部 Persona / SOUL.md （未配置时回退至平辈默认态）
                                │
                                ▼
  ┌───────────────────────────────────────────────────────────────────┐
  │ Layer 2 · 人设与语气适配层                                        │
  │ 称谓人称锁定 · 冷暖亲疏 · 机锋比例 · Emoji策略 · TTS 音色选择     │
  └─────────────────────────────┬─────────────────────────────────────┘
                                │
                                ▼
  ┌───────────────────────────────────────────────────────────────────┐
  │ Layer 1 · 通用聊天气泡物理层（始终生效）                          │
  │ SendMessage 多泡或纯文本 `\n\n` 微节拍 · 省主语 · 短句去句号      │
  │ 禁旁白说教 · 重内容外置 · 单条互补语音静默降级 · 情绪物理与衰减   │
  └───────────────────────────────────────────────────────────────────┘
```

## 效果对比

| 场景 | 典型聊天机器人 | 接入 `chat-human` 后 |
| --- | --- | --- |
| **办妥交差 + 语音条** | **单条长段**：`单子我已经帮你约好啦！1. 锁定了周四下午时段；2. 四页打印稿已整理放到桌面。现在别看屏幕啦，快去吃顿热乎的乖乖补觉吧！✨` <br>**语音条**：把上面这段话一字不落念足 16 秒。 | **气泡 1**：`周四下午窗口约上了` <br>**气泡 2**：`四页打印稿放桌面，过去签个字就行` <br>**语音条（6 秒，仅挂末条气泡）**：`外头今天风挺大的。下午出门记得带件外套呀。` |
| **SRE 灰度确认（无气泡工具普通回复）** | `可以推进全量。不过25分钟只覆盖了一个流量切面，我建议先推到50%再跑15分钟观察连接池等待队列，并提前明确0.1%回滚阈值。` | `错误率0.02%没动，p99降到38ms`<br><br>`连接池也稳，直接全量吧` |
| **连改三次需求** *(配置熟人/伴侣 Persona 时)* | `频繁修改需求会降低整体效率，下次请提前规划哦！（叹气）哼，这是新版布局请检阅。` | **气泡 1**：`第三回“真不改了”哈` <br>**气泡 2**：`v3放共享盘了，自己瞄一眼顺不顺眼` |

## 双通道原生适配：有气泡工具与普通纯文本回复皆宜

- **多气泡工具宿主（`SendMessage` / `messages[]`）**：每一轮（含两三字的随口应答）均通过工具发送并置 `end_turn: true`，避免漏调工具或重复刷出纯文本尾巴；可选 `tts` 严格只挂末条气泡；若宿主未配置语音合成服务则静默降级为纯文字泡，绝不把「语音插件未配置」之类系统报错捅进聊天框。
- **普通回复宿主（CLI、Web 对话框、单次回复 Webhook）**：在没有气泡工具时直接按相同手机字数预算、省主语与去句号规律输出，用空行（`\n\n`）切分 1–3 个短节拍，紧耦合复制单元（如快递柜号 + 取件码）用单换行 `\n` 保持一体，绝不裸吐 `{"messages": ...}` JSON 或 `气泡1：` 标签。

## 通用化设计：同一套气泡物理，适配任意 Persona

`chat-human` 不硬编码任何单一性格。语气按严格优先级解析：

1. **当前轮用户的直接要求**（如「帮我拟一句发给合作方李总的微信，带个握手 emoji」）
2. **宿主激活的 Persona / `PERSONA.md` / `SOUL.md` / 系统提示词**（伴侣、毒舌开发搭档、游戏损友、冷静 SRE 同事等）
3. **当前会话场景**（1v1 私聊 vs. 工作群聊）
4. **平辈自然默认态（Neutral Peer Baseline）**（未指定任何人设时：干净、平等、利落的熟人同事口吻，不越界暧昧，也不说下属套话）

同一条部署进展在 4 种不同 Persona 下的写法对比、人称一致性（熟人伴侣间锁死 `你` 不滑向客服腔 `您`）以及**情绪物理倍率**（深夜三点 1 条超短消息 + 零语音条收摊、遇到线上故障或低落情绪立即触发安全刹车），详见 [`references/persona-and-mood-physics.md`](skills/chat-human/references/persona-and-mood-physics.md)。

## 安装

通过 Agent Skills CLI（`skills.sh`）安装：

```bash
npx skills add NB-Corp/chat-human
# 或从统一集成双工具协议与 OpenAI/SSE 桥接网关的主仓安装：
npx skills add NB-Corp/nb-speak
```

正在使用 **DeepSeek Harness (`DSH`)**？可直接一行命令零克隆挂载独立原生真气泡与贴麦语音插件（[`NB-Corp/nb-speak-dsh`](https://github.com/NB-Corp/nb-speak-dsh) · [DSH 在线展台](https://nb-corp.github.io/nb-speak-dsh/zh/)）：

```bash
npx --yes github:NB-Corp/nb-speak-dsh --apply --profile web
```

或将 [`skills/chat-human`](skills/chat-human) 直接复制 / 软链到对应 Agent 运行时的技能目录：

```bash
# 通用 / OpenAI Codex / Kiki
cp -r skills/chat-human ~/.agents/skills/chat-human

# Claude Code
cp -r skills/chat-human ~/.claude/skills/chat-human
```

## 内置零依赖校验与门禁脚本（Python 3.10+ 标准库）

### 1. `bubble_lint.py` —— 多气泡、纯文本节拍与语音互补确定性检查器

自动拦截常见工程与文风翻车：多气泡顶层误挂 `voice_mode` 导致连发语音、纯文本模式裸露 JSON 或 `气泡1：` 标签（`--surface plain`）、字数超限小作文、气泡内混入 Markdown 表格/标题/代码块/编号墙、短句与取件码末尾冗余句号、RPG 剧本括号动作（`*sighs*`、`（叹气）`）、管家下属套话、系统 TTS 报错泄露，以及文字泡与 `tts.text` 同义复读或漏转口语数字：

```bash
python skills/chat-human/scripts/bubble_lint.py --surface json --json <<'EOF'
{
  "end_turn": true,
  "messages": [
    { "text": "周四下午窗口约上了" },
    {
      "text": "四页打印稿放桌面，过去签个字就行",
      "voice_mode": "append",
      "tts": {
        "text": "外头今天风挺大的。下午出门记得带件外套呀。"
      }
    }
  ]
}
EOF
```

校验无气泡工具的普通回复可传 `--surface plain`；当轮或当前 Persona 明确允许使用 Unicode Emoji 时可传 `--persona-emoji allow`。

### 2. `proactive_gate.py` —— 主动开口概率与沉默衰减门禁

供定时心跳或调度器调用，避免定时脚本机械刷屏或塞预制台词：

- 免打扰时段拦截（默认 `--quiet-hours 01:30-10:00`，退出码 `3`）、最小冷却时间与每日上限控制（退出码 `4`）；
- 连续未回**沉默衰减**（主动发消息对方未回时指数下调概率，`ack-user` 在用户回复后重置连击）；
- 命中触发（退出码 `0`）仅输出纯状态头 `<proactive_tick ... />`，不含任何预制文案。

```bash
python skills/chat-human/scripts/proactive_gate.py --state-file .tmp/gate.json check --ignore-quiet --force
python skills/chat-human/scripts/proactive_gate.py --state-file .tmp/gate.json ack-user
```

## 仓库结构

```text
chat-human/
├── skills/
│   └── chat-human/
│       ├── SKILL.md                              # 双层核心工作流与发送前自检
│       ├── agents/openai.yaml                    # 界面元数据
│       ├── references/
│       │   ├── bubble-and-voice-patterns.md      # 10 大机器病灶对照、双通道格式与语音口语规范
│       │   └── persona-and-mood-physics.md       # Persona 优先级、4 类人设同题对比与情绪物理倍率
│       └── scripts/
│           ├── bubble_lint.py                    # 多气泡、纯文本节拍与 TTS 互补校验脚本
│           └── proactive_gate.py                 # 免打扰与沉默衰减主动唤醒门禁
├── tests/
│   └── test_scripts.py                           # 标准库单元测试
├── README.md
├── README.zh-CN.md
└── LICENSE
```

## 运行测试

```bash
python -B -m unittest discover -s tests -v
```

## 开源协议

[MIT](LICENSE)
