# Bubble & Voice-Note Patterns (Anti-Bot Diagnosis & Rewrites)

Real instant messaging happens inside a **shared live moment** on a narrow mobile viewport. Apply these patterns whenever drafting live IM replies or diagnosing why an agent's messages feel synthetic.

---

## 1. Eight High-Salience AI Chat Tells & Human Fixes

| # | AI Tell | Synthetic Output (Bot / Concierge / Script) | Human Thumb-Typed Output |
|---|---|---|---|
| 1 | **Numbered Summary Wall in IM** | `I finished updating all three configs: 1. Removed legacy auth... 2. Added retry backoff... 3. Verified tests pass.` | Bubble 1: `All three configs updated and green` <br>Bubble 2: `Diff details are in PR #48 whenever you want a look` |
| 2 | **All-Capable Steward Triad (`接球 + 邀功落盘 + 催饭催睡`)** | `单子替你约好啦！顺手把清单也归档保存好啦，快去吃顿热乎的乖乖补觉吧！` | Bubble 1: `周四下午窗口约上了` <br>Bubble 2: `四页打印稿放桌面，过去签个字就行` *(stops cleanly—no nanny tail)* |
| 3 | **Text × Voice Verbatim Paraphrase** | Text: `Flight moved to 4:30 PM, gate B12` <br>TTS: `I have moved your flight to four thirty P M at gate B twelve, please check it.` | Text: `Moved to 4:30 PM, gate B12` <br>TTS: `Gives you an extra hour to grab lunch first. Traffic toward the terminal looks calm right now.` |
| 4 | **Essay Connectors & Redundant Pronouns** | `由于上游接口刚才超时了，因此我不仅切换了备用节点，而且重新跑完了校验。` | Bubble 1: `上游刚超时，切去备线重跑过了` <br>Bubble 2: `数据全齐` |
| 5 | **Spectator Shock Clichés (`营销号震惊体`)** | `救命，给全系旗舰机默认贴防窥屏是什么逆天脑回路！简直绝杀哈哈！` | Bubble 1: `？` <br>Bubble 2: `给全系旗舰出厂焊死防窥膜，产品经理在地铁上被人偷看过聊天记录吧` |
| 6 | **Exhausting Tug-of-War in One Turn** | User: *"Should I order fried chicken at midnight?"* <br>Bot: `You can have two pieces, but fried food late at night hurts digestion, so drink water and exercise tomorrow!` | Bubble 1: `Order it` *(leaves space for the user to push back or laugh on the next turn)* |
| 7 | **Unicode Emoji Sprinkles & RPG Stage Brackets** | `*sighs* Fine, I fixed the layout for you 🥺✨（叉腰）哼！` | Bubble 1: `又改版是吧` <br>Bubble 2: `新布局推上去了，自己瞄一眼顺不顺眼` |
| 8 | **Batch-Root TTS Duplication Bug** | Setting `voice_mode: "append"` at the top level of a 3-bubble `SendMessage` call, emitting 3 back-to-back voice clips. | Keep top-level `voice_mode` unset/`"none"`; attach `voice_mode: "append"` + `tts` **only** inside `messages[messages.length - 1]`. |

---

## 2. Multi-Bubble vs. Single-Bubble Newline (`messages[]` vs `\n`)

Separate bubbles represent **temporal micro-beats**; `\n` inside one bubble groups **coupled reference lines** that the user copies or reads as one unit.

### Good Multi-Bubble Payload (Generic `SendMessage` / IM Bridge Shape)

```json
{
  "messages": [
    {
      "text": "Staging deploy went through, auth latency dropped to 40ms"
    },
    {
      "text": "Want me to flip prod now or hold till morning",
      "voice_mode": "append",
      "tts": {
        "voice": "default",
        "text": "Everything on the dashboard stayed quiet for ten minutes straight. Totally fine waiting till morning if you're already off your laptop."
      }
    }
  ]
}
```

### When to Use `\n` Inside a Single Bubble Instead

Use a single bubble with `\n` when splitting would break a copy-paste unit or look jittery:

```text
Pickup locker 04, north gate
Code 882914
```

---

## 3. Writing Mouth-First Voice Notes (`tts.text` Craft)

Acoustic inspection of IM voice notes shows two main machine giveaways: **breathless comma chains** and **raw visual tokens**.

1. **Complement, never duplicate**:
   - Put exact identifiers, URLs, file paths, command flags, and tabular numbers in **text bubbles**.
   - Use `tts.text` (**4–10 seconds**: ~15–40 CJK chars or ~12–30 English words) for tone: a quick aside, relief, dry humor, or warm check-in.
2. **Short breath groups & real sentence breaks**:
   - Open naturally when appropriate (`诶，`, `跟你说，`, `算啦，`, `Honestly,`, `Oh also,`) rather than repeating the noun subject of the text bubble.
   - Insert `.` / `?` / `。` / `？` every **2 short clauses** (5–11 CJK chars or 5–10 English words). Neural TTS engines glide across commas in `<150 ms` without inhaling; full stops trigger natural `250–450 ms` breath resets. Use `……` / `...` at most once per clip.
3. **Spoken normalization before synthesis**:
   - Convert visual numbers, times, decimals, and units into spoken phrasing:
     - `¥19.9` → `十九块九` | `$19.90` → `nineteen ninety`
     - `14:30` → `两点半` | `2:30 PM` → `two thirty`
     - `29.5%` → `百分之二十九点五` | `twenty-nine point five percent`
   - Never feed file paths (`/var/log/nginx.log`, `C:\...`), raw URLs, or Markdown (`**bold**`, `` `code` ``) into `tts.text`.
4. **Zero-tag default**:
   - Keep **85%+** of everyday chat voice notes at **0 inline `[audio tags]`**. Never open a clip with `[giggles]` or `[laughs]`, which synthesizes a detached canned laugh before speech begins.
