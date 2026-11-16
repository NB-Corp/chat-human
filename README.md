# chat-human

**Thumb-typed IM bubble mechanics, plain-reply beat pacing, complementary voice notes, and pluggable persona adaptation for AI agents.**

**English** · [简体中文](README.zh-CN.md) · [Skill (`skills/chat-human/SKILL.md`)](skills/chat-human/SKILL.md) · [Full Protocol & Bridge (`NB-Corp/nb-speak`)](https://github.com/NB-Corp/nb-speak) · [Live Playground](https://nb-corp.github.io/nb-speak/)

---

Prose "humanizer" prompts are built for essays and articles. When you connect an agent to Telegram, WeChat, Feishu/Lark, Discord, Slack, WhatsApp, a multi-message `SendMessage` tool, or a live plain-reply chat stream, prose rules do not stop the agent from sending 180-word bulleted memos, lecturing colleagues on unrequested edge cases, repeating the concierge *"Got it! + Here is everything I did! + Go get some rest!"* triad, or reading its own text bubble word-for-word in a TTS voice note.

`chat-human` is a standalone, persona-agnostic Agent Skill that separates **how humans physically text and send voice notes on a phone** from **who the agent is supposed to be**—across both multi-bubble tool harnesses (`SendMessage` / `messages[]`) and plain-reply chat harnesses where no bubble tool exists.

```text
  User Instruction / Active Persona / SOUL.md  (or Neutral Peer Default)
                                │
                                ▼
  ┌───────────────────────────────────────────────────────────────────┐
  │ Layer 2 · Persona & Tone Adapter                                  │
  │ Pronouns · Warmth vs. Dryness · Wit · Emoji Policy · TTS Voice    │
  └─────────────────────────────┬─────────────────────────────────────┘
                                │
                                ▼
  ┌───────────────────────────────────────────────────────────────────┐
  │ Layer 1 · Universal Bubble Physics (Always Active)                │
  │ SendMessage[] or Plain `\n\n` Beats · Ellipsis · Zero Periods     │
  │ Anti-Lecture · Artifact Offload · Complementary TTS · Mood Decay  │
  └───────────────────────────────────────────────────────────────────┘
```

## Before vs. After

| Scenario | Typical Chat Agent | With `chat-human` |
| --- | --- | --- |
| **Task handoff + TTS** | **Single 65-word bubble**: *"I have finished booking your Thursday appointment! 1. Reserved the afternoon slot. 2. Prepared the 4-page packet on your desktop. Now please go eat something warm and get some rest! ✨"* <br>**Voice note**: Reads the exact same paragraph aloud in 16 seconds. | **Bubble 1**: `Thursday afternoon slot is booked` <br>**Bubble 2**: `Four-page packet is on your desktop, just needs a signature` <br>**Voice note (6s, final bubble only)**: *"Wind picked up pretty hard outside. Grab a jacket before you head over."* |
| **SRE canary check (plain-reply harness)** | *"You can proceed to 100%. However, 25 minutes only covers one traffic slice; I recommend staging to 50% first, watching connection-pool wait queues, and pre-defining a 0.1% rollback threshold."* | `Error rate held at 0.02%, p99 down to 38ms`<br><br>`Pool looks steady, good to push 100%` |
| **3rd spec change** *(with companion/buddy persona)* | *"Frequent requirement changes reduce efficiency, so please plan ahead next time! *sighs* Here is the updated layout."* | **Bubble 1**: `Third "final" layout on your desk` <br>**Bubble 2**: `Take a look and tell me this one sticks` |

## Works With or Without a Bubble Tool

- **Multi-bubble tool harnesses (`SendMessage` / `messages[]`)**: Delivers every turn—including short casual acknowledgments—through `messages[]` (`end_turn: true` on the final send), attaches optional `tts` strictly on the final bubble, and falls back to clean text without leaking backend/plugin errors if TTS is unavailable.
- **Plain-reply chat harnesses (CLI, web chat, single-reply webhooks)**: Applies the same mobile brevity, subject ellipsis, and zero-period physics directly in ordinary replies using `\n\n`-separated micro-beats (keeping coupled copy-paste lines like locker + pickup code on a single `\n`), never leaking raw `{"messages": ...}` JSON or `Bubble 1:` labels.

## Pluggable Persona: One Mechanics Layer, Any Voice

`chat-human` never locks your agent into a hardcoded personality. It resolves tone in strict priority order:

1. **Current-turn user instruction** (*"draft a polite reply to Director Li with a handshake emoji"*)
2. **Active Persona / `PERSONA.md` / `SOUL.md` / System Prompt** (partner, dry engineer coworker, gamer friend, calm SRE relay)
3. **Channel context** (1-on-1 DM vs. shared group channel)
4. **Neutral Peer Baseline** (clean, equal-footing colleague—concise and human, zero uninvited roleplay or servility)

See [`references/persona-and-mood-physics.md`](skills/chat-human/references/persona-and-mood-physics.md) for side-by-side examples of the exact same deployment update rendered across four distinct personas, pronoun consistency rules, and **Mood Physics** (expressing pushback or silence decay via beat count and voice-note omission rather than lecturing).

## Install

Install with the Agent Skills CLI (`skills.sh`):

```bash
npx skills add NB-Corp/chat-human
# Or install from the unified nb-speak protocol + bridge + DSH plugin monorepo:
npx skills add NB-Corp/nb-speak
```

Or copy/symlink [`skills/chat-human`](skills/chat-human) directly into your agent runtime's skill directory:

```bash
# Universal / OpenAI Codex / Kiki
cp -r skills/chat-human ~/.agents/skills/chat-human

# Claude Code
cp -r skills/chat-human ~/.claude/skills/chat-human
```

## Bundled Utilities (Python 3.10+ Stdlib, Zero Dependencies)

### 1. `bubble_lint.py` — Deterministic Bubble, Plain-Reply & TTS Linter

Catches batch-root TTS duplication bugs, plain-reply JSON/label leaks (`--surface plain`), over-budget walls, Markdown tables/lists/fences inside bubbles, trailing periods on short beats or after copyable codes, RPG stage brackets (`*sighs*`, `（叹气）`), concierge clichés, backend TTS error leaks, and verbatim overlap between text bubbles and `tts.text`:

```bash
python skills/chat-human/scripts/bubble_lint.py --surface json --json <<'EOF'
{
  "end_turn": true,
  "messages": [
    { "text": "Staging deploy went through, auth latency dropped to 40ms" },
    {
      "text": "Flip prod tonight or hold till morning",
      "voice_mode": "append",
      "tts": {
        "text": "Dashboard stayed quiet for ten minutes straight. Totally fine waiting till morning if you already closed your laptop."
      }
    }
  ]
}
EOF
```

Lint a plain-reply stream (`--surface plain`) or pass `--persona-emoji allow` when the active persona or turn explicitly uses Unicode emoji.

### 2. `proactive_gate.py` — Probabilistic & Silence-Decay Outreach Gate

For scheduled heartbeat hooks that let an IM agent reach out organically without spamming or repeating pre-baked prompts:

- Enforces local quiet hours (`--quiet-hours 01:30-10:00`, exit code `3`), minimum cooldowns, and daily caps (exit code `4`).
- Applies exponential **silence decay** when consecutive proactive pokes go unanswered (`ack-user` resets the streak on user reply).
- On trigger (exit code `0`), prints pure state metadata (`<proactive_tick ... />`) with zero canned topic text.

```bash
python skills/chat-human/scripts/proactive_gate.py --state-file .tmp/gate.json check --ignore-quiet --force
python skills/chat-human/scripts/proactive_gate.py --state-file .tmp/gate.json ack-user
```

## Repository Layout

```text
chat-human/
├── skills/
│   └── chat-human/
│       ├── SKILL.md                              # Core 2-layer workflow & pre-send check
│       ├── agents/openai.yaml                    # UI metadata
│       ├── references/
│       │   ├── bubble-and-voice-patterns.md      # 10 AI chat tells, dual surfaces & TTS prosody
│       │   └── persona-and-mood-physics.md       # Persona precedence, 4 archetypes & mood physics
│       └── scripts/
│           ├── bubble_lint.py                    # Outbound bubble, plain-reply & TTS linter
│           └── proactive_gate.py                 # Quiet-hours & silence-decay wake gate
├── tests/
│   └── test_scripts.py                           # Stdlib unit tests
├── README.md
├── README.zh-CN.md
└── LICENSE
```

## Verify

```bash
python -B -m unittest discover -s tests -v
```

## License

[MIT](LICENSE)
