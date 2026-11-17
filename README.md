# chat-human

**`humanizer` strips AI prose tells from essays. `chat-human` (+ [`nb-speak`](https://github.com/NB-Corp/nb-speak)) stops your Personal Agent (OpenClaw, AstrBot, Telegram, WhatsApp, WeChat, QQ, Claude Code) from turning chat windows into corporate memos and raw pipe-table dumps.**

**English** · [简体中文](README.zh-CN.md) · [Skill (`skills/chat-human/SKILL.md`)](skills/chat-human/SKILL.md) · [PA Plugins & Protocol (`NB-Corp/nb-speak`)](https://github.com/NB-Corp/nb-speak) · [Live Split-Stage](https://nb-corp.github.io/nb-speak/) · [llms.txt](llms.txt)

---

Prose "humanizer" skills are built for blog posts and articles. The moment you wire an LLM into a **Personal Agent** on Telegram, WhatsApp, WeChat, QQ, Feishu/Lark, Discord, Slack, or a terminal coding agent, article-level style rules do not stop it from:

1. **Dumping raw `|---|---|` Markdown pipe tables** that shatter into unreadable pipe soup on narrow phone screens (`openclaw#36323`);
2. **Sending 200-word numbered corporate memos** (`### Summary` + `1. **Item**:` + `"Hope this helps! Let me know if you need anything else!"`) when two short thumb-typed bubbles would do;
3. **Gluing formal periods onto pickup codes or commands** (`663920.`), narrating RPG stage brackets (`*sighs*`, `（叹气）`), or—when optional voice notes are enabled—reading its own text bubble word-for-word aloud.

`chat-human` is a zero-dependency, persona-agnostic Agent Skill (`v0.2.0`) that separates **Layer 1 Universal Mobile IM Bubble Physics** (pure text short bubbles + clean code/table offload by default; `enableVoiceNotes = false` unless explicitly opted in) from **Layer 2 Pluggable Persona & Mood Adaptation**.

## Quick Start

### 1. Install the Agent Skill (`skills.sh` · Claude Code · Codex · Cursor · OpenClaw)

```bash
# Install globally across Claude Code, Codex, Cursor, Gemini CLI, Windsurf, and 70+ agents
npx skills add NB-Corp/chat-human --global --agent '*'

# Or target Claude Code / Codex specifically
npx skills add NB-Corp/chat-human --global --agent claude-code
npx skills add NB-Corp/chat-human --global --agent codex
```

### 2. Mount Native Personal Agent & Bot Plugins (`NB-Corp/nb-speak`)

Prompt instructions alone can regress after context compaction. Pair `chat-human` with the deterministic outbound gates and MCP server in **[`NB-Corp/nb-speak`](https://github.com/NB-Corp/nb-speak)**:

| Ecosystem / Client | Package / Repo | One-Line Setup |
| --- | --- | --- |
| **Claude Code Marketplace** | `.claude-plugin/` | `/plugin marketplace add NB-Corp/chat-human` -> `/plugin install chat-human@chat-human` |
| **MCP Clients** (Cursor / Cherry / Cline) | `@nb-corp/nb-speak-mcp` | `npx -y github:NB-Corp/nb-speak mcp` |
| **OpenClaw** (TG / WhatsApp / Discord) | `@nb-corp/nb-speak-openclaw` | `npx --yes github:NB-Corp/nb-speak openclaw --apply` |
| **AstrBot** (QQ / WeChat / Feishu) | [`NB-Corp/astrbot_plugin_nb_speak`](https://github.com/NB-Corp/astrbot_plugin_nb_speak) | Paste repo URL in AstrBot WebUI Plugin Manager |
| **LangBot v4+** (WeChat / QQ / DingTalk) | [`NB-Corp/langbot_plugin_nb_speak`](https://github.com/NB-Corp/langbot_plugin_nb_speak) | Paste repo URL in LangBot WebUI Plugin Space |
| **Koishi v4 / NoneBot2** | `koishi-plugin-nb-speak` · `nonebot-plugin-nb-speak` | `npm i koishi-plugin-nb-speak` · `nb plugin install nonebot-plugin-nb-speak` |
| **OpenWebUI / OpenAI Bridge** | `nb_speak_filter.py` · `@nb-corp/nb-speak` | Import `nb_speak_filter.py` or run `npx --yes github:NB-Corp/nb-speak serve --port 18090` |

---

## Before vs. After on Mobile IM

| Real Personal Agent Scenario | Vanilla Agent (`assistant.content`) | With `chat-human` (+ `nb-speak` Gate) |
| --- | --- | --- |
| **Telegram / WhatsApp status & table** | Dumps a 180-word intro + raw `\| Task \| Status \| Priority \|` pipe soup wrapping across 9 lines + `"Let me know if you need anything!"` | **Bubble 1**: `Auth routing is live, docs still in progress`<br>**Card**: `- Fix route — Status: Done · Prio: High` / `- Docs — Status: WIP · Prio: Med` |
| **SRE canary check (plain-reply chat)** | *"You can proceed to 100%. However, 25 minutes only covers one slice; I recommend staging to 50% first and pre-defining a 0.1% rollback threshold."* | `Error rate held at 0.02%, p99 down to 38ms`<br><br>`Pool looks steady, good to push 100%` |
| **3rd spec change** *(companion / teammate persona)* | *"Frequent requirement changes reduce efficiency, so please plan ahead next time! \*sighs\* Here is the updated layout for your review."* | **Bubble 1**: `Third "final" layout on your desk`<br>**Bubble 2**: `Take a look and tell me this one sticks` |
| **Task handoff** *(with optional voice note opted in)* | Single 65-word nanny paragraph + 16s TTS reading the exact same text aloud. | **Bubble 1**: `Thursday afternoon slot is booked`<br>**Bubble 2**: `Four-page packet is on your desktop, just needs a signature`<br>*(Opt-in 6s voice aside, final bubble only)*: *"Wind picked up outside. Grab a jacket before heading over."* |

## Architecture: Two Orthogonal Layers

```text
  User Instruction / Active Persona / SOUL.md  (or Neutral Peer Default)
                                │
                                ▼
  ┌───────────────────────────────────────────────────────────────────┐
  │ Layer 2 · Persona & Tone Adapter                                  │
  │ Pronouns · Warmth vs. Dryness · Wit · Emoji Policy · Mood Physics │
  └─────────────────────────────┬─────────────────────────────────────┘
                                │
                                ▼
  ┌───────────────────────────────────────────────────────────────────┐
  │ Layer 1 · Universal Bubble Physics (Always Active)                │
  │ 1–4 Short Beats · Code/Table Offload · Zero Trailing Periods      │
  │ Anti-Lecture · Zero `*sighs*` · Opt-In Complementary Voice Notes  │
  └───────────────────────────────────────────────────────────────────┘
```

- **Works with or without a bubble tool**: Calls `send_message` / `send_bubble` (`end_turn: true`) when available, or emits `\n\n`-separated micro-beats directly in plain-reply chats (preserving coupled 2-line pickup codes like `Locker 07\nCode 663920` without leaking `{"bubbles":...}` JSON or `Bubble 1:` labels).
- **Voice notes off by default**: Pure text bubbles and clean code/table cards are the default. Optional voice notes activate only when explicitly enabled in the host config or requested by the user, compatible with any generic OpenAI `/v1/audio/speech` endpoint, local TTS, or browser Web Speech.
- **Pluggable Persona precedence**: **(1)** Current-turn explicit user request > **(2)** Active `SOUL.md` / `PERSONA.md` / AstrBot or OpenClaw persona > **(3)** Channel context (1-on-1 DM vs. group chat) > **(4)** **Neutral Peer Baseline** (equal-footing teammate, zero uninvited roleplay or servility).

## Bundled Zero-Dependency Utilities (Python 3.10+ Stdlib)

### 1. `bubble_lint.py` — Deterministic 8-Rule IM Bubble & Plain-Reply Linter

Catches leaked pipe tables/headers/code fences inside bubbles (`bubble.markdown_structure`), over-budget walls, trailing periods on short beats or after copyable codes (`style.trailing_period`), RPG stage brackets (`style.stage_brackets`), concierge clichés, plain-reply JSON/label leaks (`plain.meta_leak`), and verbatim overlap on opt-in voice notes (`tts.verbatim_overlap`):

```bash
python skills/chat-human/scripts/bubble_lint.py --surface plain --json <<'EOF'
Error rate held at 0.02%, p99 down to 38ms

Pool looks steady, good to push 100%
EOF
```

### 2. `proactive_gate.py` — Quiet-Hours & Silence-Decay Outreach Gate

For Personal Agent heartbeat/cron hooks (`OpenClaw`, `AstrBot`, custom bots) so proactive check-ins respect quiet hours (`01:30-10:00`), cooldowns, daily caps, and exponential **silence decay** when pokes go unanswered:

```bash
python skills/chat-human/scripts/proactive_gate.py --state-file .tmp/gate.json check --ignore-quiet --force
python skills/chat-human/scripts/proactive_gate.py --state-file .tmp/gate.json ack-user
```

## Verify Locally

```bash
python -B -m unittest discover -s tests -v
```

## License

[MIT](LICENSE) © 2026 NB-Corp
