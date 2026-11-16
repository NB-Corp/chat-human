# chat-human

**面向 AI Agent 的手机短气泡对话机制、互补语音条与可插拔 Persona 语气适配技能。**

[English](README.md) · **简体中文** · [技能正文 (`skills/chat-human/SKILL.md`)](skills/chat-human/SKILL.md)

---

常规的「去 AI 味 / Humanizer」提示词针对的是文章、长文和小说。当你把 Agent 接进微信、飞书、Telegram、QQ、Discord、Slack 或支持多条消息的 `SendMessage` 工具时，文章级规则拦不住它在手机聊天框里发 200 字编号总结、每轮雷打不动走一遍「`记好啦！+ 顺手替你办妥落盘X + 快去吃热乎的乖乖睡觉吧！`」的全能管家三段式，或者把文字气泡换几个近义词在语音条里重新播音朗读一遍。

`chat-human` 是一个彻底通用化的 Agent Skill：把**真人在手机上用拇指打字、按住说话的物理交互规律（Layer 1）**与**角色人设及语气偏好（Layer 2）**解耦开来。

```text
  当轮用户要求 / 外部 Persona / SOUL.md （未配置时回退至平辈默认态）
                                │
                                ▼
  ┌───────────────────────────────────────────────────────────────────┐
  │ Layer 2 · 人设与语气适配层                                        │
  │ 称谓人称 · 冷暖亲疏 · 机锋比例 · Emoji/表情包策略 · TTS 音色选择  │
  └─────────────────────────────┬─────────────────────────────────────┘
                                │
                                ▼
  ┌───────────────────────────────────────────────────────────────────┐
  │ Layer 1 · 通用聊天气泡物理层（始终生效）                          │
  │ 1–3 条手机短气泡 · 共享语境省主语 · 戒断全能管家三段式            │
  │ 重内容外置 · 单条互补语音禁复读 · 情绪物理倍率与沉默衰减          │
  └───────────────────────────────────────────────────────────────────┘
```

## 效果对比

| 场景 | 典型聊天机器人 | 接入 `chat-human` 后 |
| --- | --- | --- |
| **办妥交差 + 语音条** | **单条长段**：`单子我已经帮你约好啦！1. 锁定了周四下午时段；2. 四页打印稿已整理放到桌面。现在别看屏幕啦，快去吃顿热乎的乖乖补觉吧！✨` <br>**语音条**：把上面这段话一字不落念足 16 秒。 | **气泡 1**：`周四下午窗口约上了` <br>**气泡 2**：`四页打印稿放桌面，过去签个字就行` <br>**语音条（6 秒，仅挂末条气泡）**：`外头今天风挺大的。下午出门记得顺手带件外套呀。` |
| **连改三次需求** *(配置熟人/伴侣 Persona 时)* | `频繁修改需求会降低整体效率，下次请提前规划哦！（叹气）哼，这是新版布局请检阅。` | **气泡 1**：`又改版是吧` <br>**气泡 2**：`第三版推上去了，自己瞄一眼顺不顺眼` |

## 通用化设计：同一套气泡物理，适配任意 Persona

`chat-human` 不硬编码任何单一性格（不会在你想要干练同事时强行撒娇，也不会在你配置亲密人设时退化成舞台剧小作文）。语气按严格优先级解析：

1. **当前轮用户的直接要求**（如「这句回导师正式点」「这里加点 emoji」）
2. **宿主激活的 Persona / `PERSONA.md` / `SOUL.md` / 系统提示词**（伴侣、毒舌开发搭档、游戏损友、高管秘书等）
3. **当前会话场景**（1v1 私聊 vs. 工作群聊）
4. **平辈自然默认态（Neutral Peer Baseline）**（未指定任何人设时：干净、平等、利落的熟人同事口吻，不越界暧昧，也不说下属套话）

同一条部署进展在 4 种不同 Persona 下的写法对比，以及如何用**情绪物理倍率**（缩短气泡字数、省掉语音条、未读连击沉默衰减）代替爹味说教和 RPG 括号动作，详见 [`references/persona-and-mood-physics.md`](skills/chat-human/references/persona-and-mood-physics.md)。

## 安装

通过 Agent Skills CLI（`skills.sh`）安装：

```bash
npx skills add <owner>/chat-human
```

或将 [`skills/chat-human`](skills/chat-human) 直接复制 / 软链到对应 Agent 运行时的技能目录：

```bash
# 通用 / OpenAI Codex / Kiki
cp -r skills/chat-human ~/.agents/skills/chat-human

# Claude Code
cp -r skills/chat-human ~/.claude/skills/chat-human
```

## 内置零依赖校验与门禁脚本（Python 3.10+ 标准库）

### 1. `bubble_lint.py` —— 多气泡与语音互补确定性检查器

自动拦截常见工程与文风翻车：多气泡顶层误挂 `voice_mode` 导致连发多条语音、字数超限小作文、气泡内混入 Markdown 表格/标题/编号墙、RPG 剧本括号动作（`*sighs*`、`（叹气）`）、管家下属套话（`向你汇报`、`顺手替你`）、句尾 `啦` 字刷屏，以及文字泡与 `tts.text` 同义复读或漏转口语数字：

```bash
python skills/chat-human/scripts/bubble_lint.py --json <<'EOF'
{
  "messages": [
    { "text": "周四下午窗口约上了" },
    {
      "text": "四页打印稿放桌面，过去签个字就行",
      "voice_mode": "append",
      "tts": {
        "text": "外头今天风挺大的。下午出门记得顺手带件外套呀。"
      }
    }
  ]
}
EOF
```

若当前 Persona 明确允许使用 Unicode Emoji，可传 `--persona-emoji allow`。

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
│       │   ├── bubble-and-voice-patterns.md      # 8 大机器病灶对照、多气泡结构与语音口语规范
│       │   └── persona-and-mood-physics.md       # Persona 优先级、4 类人设同题对比与情绪物理倍率
│       └── scripts/
│           ├── bubble_lint.py                    # 多气泡结构与 TTS 互补校验脚本
│           └── proactive_gate.py                 # 免打扰与沉默衰减主动唤醒门禁
├── tests/
│   └── test_scripts.py                           # 标准库单元测试
├── README.md
├── README.zh-CN.md
└── LICENSE
```

## 运行测试

```bash
python -m unittest discover -s tests -v
```

## 开源协议

[MIT](LICENSE)
