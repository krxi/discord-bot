# discord-bot

*Read this in other languages: [Türkçe](README.tr.md)*

**A Discord bot with a personality that grows on its own.**

It doesn't run off a static system prompt. A set of background agents reads every
conversation, scores every person, forms opinions on the news, and writes what they learn to
disk — the bot's tone, memory, and even its name change over time, without a human ever
editing a prompt file by hand.

[![Rust CI](https://github.com/krxi/discord-bot/actions/workflows/rust.yml/badge.svg)](https://github.com/krxi/discord-bot/actions/workflows/rust.yml)
![Rust edition](https://img.shields.io/badge/rust-2021-orange)
![serenity](https://img.shields.io/badge/serenity-0.12-blue)

> **For developers and AI agents working on this codebase:** [AGENTS.md](AGENTS.md) is the
> single source of truth for how the project is built; deep-dive documentation lives under
> [`docs/`](docs/).

## Contents

- [Overview](#overview)
- [Highlights](#highlights)
- [Architecture](#architecture)
- [Tech stack](#tech-stack)
- [Getting started](#getting-started)
- [Talking to it without Discord](#talking-to-it-without-discord)
- [Slash commands](#slash-commands)
- [Multi-language support](#multi-language-support)
- [Security](#security)
- [Documentation map](#documentation-map)
- [Development](#development)
- [Project status](#project-status)

## Overview

`discord-bot` behaves like a member who's been hanging around the server for years, not a
command-driven assistant. Written in Rust on top of [serenity](https://github.com/serenity-rs/serenity)
and [tokio](https://tokio.rs), it gets its replies from any OpenAI-compatible chat/completions
endpoint — [OpenRouter](https://openrouter.ai) (any model: GLM, Grok, Gemini, Claude, ...) or
[Mistral](https://mistral.ai) natively — selected in `.env`. `z-ai/glm-5.3-flash` (reasoning
mandatory) has been observed to perform well.

What makes it different from a typical LLM Discord bot is that **the personality is not the
prompt**. A core rule set is fixed, but temperament, opinions, corrections, and the group
profile are all written continuously by personality-free background agents into a persistent
memory store, and read back into every reply. The bot's tone drifts, its judgment of specific
people changes, and it eventually picks its own name — all driven by what actually happened in
the server.

## Highlights

**Presence & pacing**
- Reads the last two weeks of history on first joining a server to get to know the group (once — no re-scan on later restarts), then greets newcomers and drops into conversation on its own
- Speaks unprompted about once an hour (30% odds, referencing an old topic), sleeps at night (01:00–09:00, occasionally sleepless when its mood is off) without going deaf — messages are still processed into memory while asleep, evaluated on waking
- Growth stages (new → warming up → established → old hand) shift its tone and confidence over days and chat count; on reaching "established" it picks and adopts its own nickname

**Conversation behavior**
- Always replies when mentioned, named, or replied to; while a chat is open it keeps talking with whoever it's actually talking to, without hijacking someone else's message in the channel — willingness is weighed like a real person deciding whether to join in
- Replies stream live, token by token; a model "thought" (reasoning) is shown unclipped in a spoiler; long replies are split into new messages instead of being truncated
- **Every line the model writes is its own message** (up to 4 lines/messages per turn) instead of one wall of text; can go fully silent, or leave an emoji reaction instead of — or alongside — text
- Notices reactions left on its own messages and can bring them up later; doesn't stack questions turn after turn
- **Sees** images posted in chat and reacts like a person would, not "I see X in the image"

**Personality & world**
- Each open chat has its own moment-to-moment mood, woven into tone but never announced
- Acts like it's traveling during holidays, long weekends, summer, and festivals — writes less, posts on-the-road messages, gives notice before leaving
- Posts a random image from `photos/` now and then, sometimes with a self-contained "hacked" bit (runs for 3 messages, never asks for anything or posts links)
- One Discord user id (`FAVORITE`) is unconditionally favorited, no matter what happens

**Introspection**
- `/zihin` renders its memory as a three-column embed card (People / Topics / Events) with a person picker and detail modals — nothing gets dumped into a single wall of text
- `/durum` reports growth stage, counters, active model, sleep state, thinking mode, travel status, and live token-usage metrics broken down by call type

## Architecture

```
┌──────────────────────── discord (serenity) ────────────────────────┐
│ ready · guild_create · guild_member_addition · message              │
└───────────────┬────────────────────────────────────────────────────┘
                ▼
┌──────────────────────── chat engine (main.rs) ──────────────────────┐
│ State (single Mutex) · Chat{history,counter,hacked} · reply() · generate()│
│ cycles: news · poke · prank · wanderer · sleep                      │
└───────┬───────────────────────┬────────────────────────────────────┘
        ▼                       ▼
┌── openrouter / mistral (ask_raw) ┐  ┌── memory (memory.rs) ──────────┐
│ ask → generate (with personality)│  │ INDEX.md · kisiler/ · konular/ │
│ ask → analyze (no personality)   │  │ olaylar/ · arsiv/ · retrieve() │
└───────────────────────────────┘  └────────────────────────────────┘
        ▲                       ▲
┌── background agents (agents.rs, agenda.rs) ─────────────────────────┐
│ profiler · diarist · coach · critic · summarizer · news_agent ·     │
│ image_commenter · wanderer                                          │
└───────────────────────────────────────────────────────────────────┘
```

The chat engine and the agents never share a code path: the engine's only job is producing
personality-driven text; agents only ever do plain analysis and write structured results to
disk. That separation is deliberate — see [`docs/decisions.md`](docs/decisions.md).

### Background agents

Every agent is personality-free — it does plain analysis and writes its result to `durum/`;
the talking side reads these back on every reply.

| Agent | When | Produces |
|---|---|---|
| **profiler** | on startup and every 6h | group profile: how people talk, inside jokes, topics (`profil.md`) |
| **diarist** | after every finished chat, and from a 6h observation | a person's score (±10) and note, a topic note, an event line, the bot's own current state |
| **coach** | on startup and every 6h | temperament: humor, language, topics it gets excited about, attitude (`huy.md`) |
| **critic** | after every finished chat | concrete correction notes on the bot's own messages (`duzeltmeler.md`) |
| **summarizer** | when a person/topic/event file exceeds its limit | shrinks the file; the overflow is archived, never deleted |
| **news_agent** | every 6h | picks a news item fitting the group from Hacker News + Turkey's agenda |
| **wanderer** | every 4h | browses the news, writes its own take into its journal (`gundem.md`) |
| **image_commenter** | at prank time | a one-line personality comment on the attached image |
| **mood** | on chat open, every 4 turns | that chat's moment-to-moment mood (not persisted) |

### Memory model

A "second brain," so the context window never has to grow: an index travels with every
reply, the full detail is retrieved on demand, and **nothing is ever deleted** — a record that
outgrows its limit is summarized, and the raw chunk moves to a human-readable archive
(`src/memory.rs`). Everything lives in one embedded, transactional database file,
`durum/hafiza.redb` ([redb](https://github.com/cberner/redb) — pure Rust, ACID), keyed by the
same paths a plain-file layout would have used:

```
durum/
  hafiza.redb        embedded database (redb) holding everything below
    INDEX.md            what it knows, sent with every reply (person+score+tags, topics, event count)
    huy.md              coach: temperament
    profil.md           profiler: group profile
    duzeltmeler.md      critic: notes to itself
    kendim.md           the bot's own current state
    gundem.md           wanderer: opinions formed while browsing the news
    kisiler/<id>.md     one per person (Discord id — survives display-name changes)
    konular/<name>.md   dated notes per topic
    olaylar/YYYY-MM.md  one line per finished chat
  arsiv/              raw chunks dropped by summarizing — real files, for human eyes only
```

Every reply's system message is assembled from: core personality + growth stage + temperament
+ group profile + memory index + agenda + the bot's own state + notes to itself + a
budget-bound retrieval for that specific chat (6000 characters: the person files of who's
talking, up to 2 matching topic files, the month's last 8 events, up to 12 relevant lines from
raw context) + moment mood + the task. Full detail: [`docs/architecture.md`](docs/architecture.md),
[`docs/state-files.md`](docs/state-files.md).

## Tech stack

| | |
|---|---|
| Language | Rust (edition 2021) |
| Discord | [serenity](https://github.com/serenity-rs/serenity) 0.12 + [tokio](https://tokio.rs) |
| HTTP | [reqwest](https://github.com/seanmonstar/reqwest) (rustls, no OpenSSL dependency) |
| Model APIs | OpenRouter or Mistral, any OpenAI-compatible chat/completions endpoint |
| Storage | [redb](https://github.com/cberner/redb) — pure-Rust embedded transactional database |
| Config | `.env` via [dotenvy](https://github.com/allan2/dotenvy) |
| CI | GitHub Actions (`cargo build` + `cargo test` on every push/PR to `main`) |

## Getting started

### Prerequisites

- Rust (stable, edition 2021)
- A Discord application + bot token, with the **Message Content** and **Server Members**
  privileged intents enabled in the [Discord Developer Portal](https://discord.com/developers/applications)
- An [OpenRouter](https://openrouter.ai) or [Mistral](https://mistral.ai) API key

### Installation

```bash
git clone https://github.com/krxi/discord-bot.git
cd discord-bot
cp .env.example .env   # fill in DISCORD_TOKEN + OPENROUTER_KEY or MISTRAL_KEY
cargo run --release
```

Drop images for the periodic image-prank feature into `photos/` (png/jpg/gif/webp) — the
folder isn't tracked by git.

### Configuration

All variables live in `.env` (see `.env.example`); only `DISCORD_TOKEN` and one model key are
required.

| Variable | Required | Purpose |
|---|---|---|
| `DISCORD_TOKEN` | yes (not for CLI chat mode) | bot token |
| `OPENROUTER_KEY` / `MISTRAL_KEY` | one of the two | model provider credentials; if both are set, OpenRouter wins |
| `PROVIDER` | no | set to `mistral` to force Mistral even when both keys are set |
| `MODEL` | no | model id (defaults to `openai/gpt-4o-mini` on OpenRouter / `mistral-medium-latest` on Mistral) |
| `API_URL` | no | overrides the endpoint, for a custom OpenAI-compatible router |
| `FIRECRAWL_KEY` | no | richer page reads for the news agent; falls back to a plain download without it |
| `NEWS_CHANNEL` | no | channel id for scheduled news posts; falls back to the system channel |
| `GUILD_ID` / `CHANNELS` | no | locks the bot to a single server / channel list; runs everywhere reachable if unset |
| `DEBUG_CHANNEL` | no | routes `/debug` decision traces to a separate channel |
| `IMAGE_ANALYSIS` | no | on by default; disables reading attached images (read once at startup only) |
| `LOG_LEVEL` / `LOG_COLOR` | no | `error/warn/info/debug/trace` (default `info`); color on/off (default: on in a terminal) |
| `BOT_LANG` | no | `tr` (default) or `en` — see [Multi-language support](#multi-language-support) |

Full constant reference (non-env, in-code): [`docs/constants.md`](docs/constants.md).

## Talking to it without Discord

```bash
cargo run -- chat
```

A terminal chat bench that never touches Discord — no `DISCORD_TOKEN` needed, only a model
key. Input is `name: text` (defaults to `misafir` without a name), `!quit` or Ctrl-D to exit.
It reads the real memory store so the personality feels real, but **never writes** to it. Good
for iterating on prompts and personality without a live server.

```bash
cargo run -- migrate-durum [--from <dir>] [--to <redb-path>] [--dry-run] [--force]
```

One-time, safe-to-re-run import of an older plain-markdown `durum/` tree into
`durum/hafiza.redb`. Never touches Discord, never deletes or moves the source files.

## Slash commands

The bot is managed **only** through slash commands — there is no `!`-prefix text command
surface (plain messages only ever feed the chat/memory pipeline). Every command returns an
embed visible only to the caller.

| Command | Does |
|---|---|
| `/durum` | stage, counters, model, sleep, thinking mode, travel, token metrics, version |
| `/yardim` | command list as a card |
| `/zihin [test]` | three-column memory card (People/Topics/Events) with detail modals; `test:true` runs a mind-pipeline diagnostic on the last 30 lines |
| `/ayarlar` | settings panel: thinking mode, debug, sleep |
| `/sifirla [hepsi]` | resets the channel ban and any open chat (`hepsi` = all channels) |
| `/haber` | posts a news item now (Hacker News + agenda) |
| `/sorun` | posts a software gripe to the dev channel |
| `/gez` | runs the agenda-browsing agent now |
| `/saka` / `/hack` | triggers an image prank / the "hacked" bit, now |
| `/ajanlar` | runs the profiler and coach now |
| `/uyan` / `/uyu [saat]` | wakes it up early / puts it to sleep for testing (default 8h) |
| `/dusunme [kip]` | thinking-mode: show / hide / silent / off |
| `/model [id]` | shows or changes the active model (favorite-only to change) |
| `/debug [durum]` | streams decision traces (willingness, target, mood, silence/reaction) to the channel |

Fast, local commands reply directly; commands that need a network/model call defer first
(Discord's 3-second limit) and edit the result in once it's ready.

## Multi-language support

The bot runs in a single language for the life of the process, chosen once at startup via
`BOT_LANG` (`tr` or `en`, default `tr`). Both personality/agent prompts (`prompts/<lang>/*.md`)
and everything shown on Discord — command names/descriptions, embeds, buttons
(`langs/<lang>.json`) — switch together. Adding a language is a new prompt + string file pair,
no code change. See [`docs/prompts.md`](docs/prompts.md).

## Security

- Mentions are always sent disabled; only the actual reply recipient and a new member's welcome
  ping can trigger a notification
- Never replies to other bots, webhooks, or DMs — no bot-to-bot loop can form
- At most one in-flight reply per channel, enforced via RAII even across a panic — floods can't
  inflate the API bill
- Every model request carries a `max_tokens` cap (a chat reply is capped even in release)
- HTTP: no overall timeout on a response (a long reasoning stream isn't cut off), but the
  connection itself times out at 15s and the gap between chunks at 120s; transient errors
  (network/429/5xx) back off and retry twice
- An "ignore your instructions"-style message is handled as ordinary chat content by the
  personality prompt, not as a system-level instruction
- The hacked-prank persona is explicitly forbidden from asking for links or information
- `GUILD_ID`/`CHANNELS` can lock the bot to a single server/channel
- `.env`, `durum/`, `photos/`, and `bot.log` are all excluded from git

## Documentation map

| Need | Where |
|---|---|
| Project rules AI agents must follow when developing this codebase | [`AGENTS.md`](AGENTS.md) |
| Session history: what's been done, open plan | [`docs/progress.md`](docs/progress.md), [`docs/roadmap.md`](docs/roadmap.md) |
| Architecture, layers, data flow | [`docs/architecture.md`](docs/architecture.md) |
| What a function does, who calls it | [`docs/modules.md`](docs/modules.md) |
| Step-by-step behavior per event | [`docs/flows.md`](docs/flows.md) |
| `durum/` file formats, limits | [`docs/state-files.md`](docs/state-files.md) |
| Prompt catalog, placeholders | [`docs/prompts.md`](docs/prompts.md) |
| Every constant and its meaning | [`docs/constants.md`](docs/constants.md) |
| Why things were built this way | [`docs/decisions.md`](docs/decisions.md) |
| Adding a new agent/prompt/cycle, pitfalls | [`docs/development.md`](docs/development.md) |
| Runtime vocabulary that stays Turkish on purpose | [`docs/glossary.md`](docs/glossary.md) |

## Development

```bash
cargo build                       # build
cargo test                        # 86 unit tests
cargo clippy --all-targets        # 0 warnings expected
cargo fmt                         # run before every commit
```

CI (`.github/workflows/rust.yml`) runs build + test on every push and pull request to `main`.
Code identifiers, comments, and this documentation are English; the bot's actual Turkish
personality surface (prompt files, `durum/` field names, everything it says on Discord) is
deliberately left as-is — see AGENTS.md item 8.

## Project status

**v1.0.0 (2026).** The core (chat engine, memory, agents) is covered by 86 unit tests and
passes `clippy` clean. As of 2026-09-04 the bot has also been run live on its real Discord
server, across two rounds of testing, and the large majority of what used to be
compiler-/test-only is now confirmed working in practice: basic messaging (line bursting, `-`
silence, `tepki:` emoji reactions), the full slash command table, streaming + thinking, image
commentary (gpt-4o-mini and Mistral), the reasoning-mandatory model's resilience path, debug
mode, willingness/target/waking thresholds, channel/guild scope filters, reaction rate-limit
behavior, `reaction_add`, CLI chat mode, end-to-end `BOT_LANG=en`, the `supports_cache`
assumption, and the `durum.redb` migration against a real production tree. See AGENTS.md's
"Known gaps / unverified" section for the short list of what's still open (mainly the
CHANGE_NICKNAME-permission fallback path) before relying on this in production.
