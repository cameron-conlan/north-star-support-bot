# North Star Outdoor Co. — Support Bot

A customer support chatbot for a small e-commerce business selling outdoor apparel and camping gear.

---

## How to run it

Open `index.html` in any browser.

That's it. No API keys, no subscriptions, no accounts, no build step, no internet connection. Everything — the intent engine, the conversation state machine, the mock order data — is contained in that single file.

---

## Why rule-based and not an LLM

The brief requires evaluators to test the bot without adding API keys or completing implementation steps. Any LLM-backed bot either exposes a key in client-side code or asks the reviewer to supply one. A deterministic intent engine avoids both, and has a second advantage: **it cannot hallucinate the business data.** Order statuses, the return policy, and shipping windows are returned verbatim from a single constants block, so accuracy is guaranteed rather than probable.

---

## Required use cases

| Use case | Where it lives | Behaviour |
|---|---|---|
| **Order tracking** | `resolveOrder()` | Asks for the order number, or extracts it inline from phrases like *"Where's order 111?"* |
| **Returns & exchanges** | `RETURN_POLICY` | 30-day window, unused items, original packaging, plus the returns link |
| **Product recommendations** | `startRecommendation()` | Two clarifying questions, then a product-category recommendation |
| **Human handoff** | `handoff()` | Triggered by explicit request or by three consecutive fallbacks |

## Mock order data

| Order | Status |
|---|---|
| `#111` | Shipped, arriving tomorrow |
| `#222` | Processing, ships in 24 hours |
| `#333` | Delivered — bot asks a follow-up question |
| anything else | Invalid order |

## Customer support information

- **Return policy** — 30-day returns · items must be unused · original packaging required
- **Standard shipping** — 3–5 business days
- **Expedited shipping** — 1–2 business days

---

## Intent recognition

Each intent owns a set of weighted regex patterns. An utterance is scored against every intent and the highest score wins, with a minimum threshold to avoid false positives on loose keywords.

Weighting is what resolves collisions. `"Can I return my order?"` contains *order*, but `return` scores 3 and `order` scores 2, so the bot answers with the return policy rather than asking for an order number. Likewise `"return to the main menu"` scores 6 for `menu` against 3 for `returns`.

Variations handled per intent, e.g. order tracking:

> Where is my order? · Track my package · What's my order status? · When will it arrive? · Wheres order 111 · Track #222

## Conversation state machine

```
MAIN MENU ──┬─▶ AWAITING ORDER NO. ──┬─▶ (111/222) ──────────▶ MAIN MENU
            │                        ├─▶ (333) ─▶ DELIVERY FOLLOW-UP ─┬─▶ MAIN MENU
            │                        │                                └─▶ LIVE AGENT
            │                        └─▶ (invalid) ─▶ retry / escalate
            │
            ├─▶ RECOMMEND Q1 ─▶ RECOMMEND Q2 ─▶ category ────▶ MAIN MENU
            ├─▶ returns / shipping (single turn) ────────────▶ MAIN MENU
            └─▶ LIVE AGENT ─┬─▶ "Back to main menu" ─────────▶ MAIN MENU
                            └─▶ "Continue with the bot" ─────▶ MAIN MENU
```

Every flow returns the user to the main flow once it resolves. The header displays the current state, the last recognised intent, and the consecutive fallback count — so a reviewer can watch the machine work rather than take it on trust.

## Fallback handling

1. **First miss** — "I didn't understand that", followed by the four things the bot can do.
2. **Second miss** — acknowledges the repeat, offers options *or* escalation.
3. **Third miss** — automatic transition to the Live Agent state.

The counter resets on any successfully recognised intent.

## Human handoff

Explicit escalation (*"talk to a human"*, *"live agent"*, *"real person"*) is honoured from **any** state, including mid-flow. On handoff the interface shifts to a trail-blaze red accent, the agent's messages carry a distinct left rule and a `Dana R. · Live Agent` label, and the user is given two ways back: return to the main menu, or continue with the bot.

---

## Video demo script (~2:30)

1. **Order tracking** — type `Where is my order?` → `111`. Then `track #222` to show inline number extraction. Then `333` → answer `No, I never got it` → auto-escalation to a live agent → `Back to main menu`.
2. **Returns** — type `it doesn't fit, can I send it back?` (no keyword match on "return") → policy + link.
3. **Shipping** — type `how long does expedited take?`
4. **Recommendations** — type `help me find a tent` → `Camping gear` → `A weekend`.
5. **Fallback** — type `asdkjh`, then `purple monkey dishwasher`, then `qqqq` → watch the fallback counter climb and the bot escalate on the third.
6. Point at the header strip while doing all of this. It shows the state and the matched intent live.

---

## Files

- `index.html` — the complete chatbot
- `README.md` — this file
