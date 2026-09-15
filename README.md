# meta-prompt-engine

A single-file browser tool that engineers structured prompts from a task description. Describe what you are working on, select a model, and get a prompt with visible architecture — role definition, context block, task instruction, output format, chain hook, and guard rails — ready to paste into the next session or run in parallel.

Works with Claude (Anthropic), GPT-4o (OpenAI), and Gemini 2.0 Flash (Google). Streams responses in real time. No build step, no server, no dependencies.

---

## What it does

You provide a task description and configure:
- **Domain** — General, Code, Writing/Docs, Data, Research, System Design, Creative, QA
- **Prompt use** — Follow-up, Parallel, Decompose, Review, or Chain
- **Target model** — Claude, GPT-4o, Gemini, or Model-Agnostic
- **Components** — which structural blocks to include in the output

Guard rails are opt-in — deselected by default. Enable them when you need explicit exclusions and failure-mode constraints in the generated prompt.

The engine sends a meta-prompt to your chosen model — a prompt that instructs it to reason about prompt architecture and return a structured result. The output is displayed two ways: an anatomy view where each block is labeled with the engineering decision behind it, and a raw view with the full prompt ready to copy.

Model-Agnostic mode requires no API key. It assembles a structured prompt locally from the meta-prompt architecture without making any network requests.

---

## Prompt architecture

Every generated prompt is composed of up to six typed blocks:

| Block | Purpose |
|---|---|
| `ROLE` | Sets the model's identity, expertise, and authority domain for the task |
| `CONTEXT` | Frames the situation, prior work completed, and known state inherited from the previous task |
| `TASK` | The primary instruction — precise, unambiguous, scoped to a single deliverable |
| `FORMAT` | Output structure, length, level of detail, and any required markup or labeling |
| `CHAIN` | A hook that makes the output of this prompt directly usable as input to the next prompt in the workflow |
| `GUARD` | Explicit exclusions, failure modes to avoid, and quality bars the output must meet |

The anatomy view shows each block with a one-sentence note explaining the engineering decision behind it — not just what the block says, but why it is structured that way.

---

## How the meta-prompt works

The system prompt sent to the model is itself an engineered prompt. It instructs the model to:

1. Identify the prompt use case (follow-up, parallel, decompose, review, or chain) and frame the output accordingly
2. Apply model-specific syntax conventions — XML tags for Claude, markdown headings for GPT-4o, concise structure for Gemini
3. Return only valid JSON matching a strict schema — `title`, `purpose`, and a `blocks` array where each block has `type`, `content`, and `note`
4. Make every block substantive and specific to the described task, not generic placeholders

The system prompt adapts to the selected model and domain before every request. Changing the target model changes the structural conventions in the generated prompt, not just a label.

---

## API setup

Each model requires its own API key, entered in the tool. Keys are held in memory for the session only and sent exclusively to the respective provider's API — nowhere else.

| Model | Provider | Get a key |
|---|---|---|
| Claude (claude-sonnet-4-6) | Anthropic | [console.anthropic.com](https://console.anthropic.com/settings/keys) |
| GPT-4o | OpenAI | [platform.openai.com](https://platform.openai.com/api-keys) |
| Gemini 2.0 Flash | Google | [aistudio.google.com](https://aistudio.google.com/app/apikey) |

All three providers bill per token. A typical generation is 500–1,200 tokens total (input + output). Check each provider's current pricing before use.

---

## Running it

**Locally:**
```
git clone https://github.com/BleedingCodes/meta-prompt-engine
open meta-prompt-engine.html
```
Open in any modern browser. No install, no server, no build step.

**Hosted:**
Drag `meta-prompt-engine.html` to [netlify.com/drop](https://netlify.com/drop) or push to a GitHub Pages repo. The file is fully self-contained.

**Browser requirements:** Chrome, Firefox, Safari, or Edge — any version from the last two years. Requires `fetch` with streaming (`ReadableStream`) support, which all modern browsers provide.

---

## Streaming

All three API callers implement SSE streaming via `ReadableStream`. The response renders in real time as it arrives — the anatomy view attempts a live parse on each chunk and falls back to raw text display until the JSON is complete. Each provider's SSE format is handled separately:

- **Anthropic** — `content_block_delta` events with `text_delta` type
- **OpenAI** — `choices[0].delta.content` from chat completion chunks  
- **Google** — `candidates[0].content.parts[0].text` from Gemini SSE

---

## Project structure

```
meta-prompt-engine/
├── meta-prompt-engine.html   # entire application — HTML, CSS, JS in one file
└── README.md
```

No `node_modules`. No `package.json`. No framework. The only external resources loaded are two Google Fonts families (IBM Plex Mono and IBM Plex Sans) — the tool works without them if fonts are unavailable, falling back to system monospace and sans-serif.

---

## License

MIT — see LICENSE file.

---

Built by [MainbyteLabs](https://github.com/MR-MainbyteLabs) — Python tooling and technical documentation for electronics labs and hardware teams.
