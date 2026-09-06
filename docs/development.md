# Development guide

## Before you start
1. Read `AGENTS.md`. 2. Read the relevant section of `docs/modules.md` for the module your
work touches. 3. Check that `cargo test && cargo clippy` are green. 4. If you're going to do
a text-matching patch, read the file's current state first (`cargo fmt` reflows lines and
shifts aligned comments).

## Before calling it done
`cargo fmt` → `cargo clippy` (0 warnings) → `cargo test` → `cargo build --release` → update
the relevant `docs/` file (modules/flows/constants/prompts) → reasoning into
`docs/decisions.md` → commit (in Turkish, what+why) → push.

## Recipes

### New prompt
1. Write `prompts/tr/<name>.md`; first line `# Title`; placeholders `{x}`. (This file stays
   in Turkish.)
2. In `src/prompts.rs`'s `tr` submodule, add
   `pub const NAME: &str = include_str!("../prompts/tr/<name>.md");` (the constant name is
   English), plus a matching field in the `Prompts` struct and the `TR` constant.
3. At the call site: `prompts::current().field_name.replace("{x}", ..)`.
4. Add a row to the `docs/prompts.md` table.

### New Discord-facing text (command name/description, embed, button)
1. Add `"key": "value"` to `langs/tr.json` (dot-separated namespace, e.g. `cmd.x.name`).
2. At the call site: `strings::t("key")`; if there's a placeholder, `.replace("{x}", ..)`.
3. If it's a command option/choice: the **value** (the wire protocol registered with
   Discord, the second argument of `option_text`/`add_string_choice`) is never translated,
   only the displayed **label** comes from `strings::t`.

### New language
1. A `prompts/<lang>/` folder: a translation of every file in `prompts/tr/`, with the same
   file names.
2. `langs/<lang>.json`: a translation of every key in `langs/tr.json` (keys and
   `{placeholders}` stay exactly the same).
3. A new variant in `src/lang.rs`'s `Lang` enum + a match arm in `parse`; one `match` arm
   each in `src/prompts.rs`'s `get` and `src/strings.rs`'s `table`.

### New agent (background evaluation)
1. In `src/agents.rs`: `impl Bot { pub async fn <name>(&self, ...) }`. Input is cloned from
   `State` under the lock, the lock is released, `self.analyze(text, instruction,
   max_tokens).await`, then the result goes back into `State` under the lock and to the file
   via `memory::write`.
2. Set a limit: `max_tokens`, `clamp` if it's a number, `memory::over_limit` logic if it's a
   file.
3. Hook it into the schedule: inside the 6-hourly `news_cycle` round, or at the end of
   `reply`.
4. Add a section to `system_text` (if needed) and to the system message list in
   `docs/architecture.md`.
5. Read it at startup via `State::load`.

### New cycle
`async fn <name>_cycle(bot: Arc<Bot>, ctx: Context)`; `loop { sleep(..).await; if
!sleep::is_awake(&bot.state()) { continue; } ... }`; `tokio::spawn` in `ready` (only on the
first `started`). Consider the travel effect (`travel::now()`).

### New state file
Use `memory::read/write` (path relative to `durum/`). Define a limit and archiving rule; add
a row to the `docs/state-files.md` table.

### Changing personality behavior
A fixed rule → `prompts/kisilik.md`. Something that should change over time → add it to
`hoca.md`'s decision area (the code doesn't change). A post-chat correction →
`elestirmen.md`.

### Changing a constant
Top of `src/bot/types/types_settings.rs`; update `docs/constants.md`.

## Pitfalls
- **MutexGuard held across an await:** the compiler gives a `Send` error, or you get a
  deadlock. Close the lock in a `{ }` block, keep `.await` outside it.
- **`state.chats.get_mut` followed by `end_chat(&mut state)`:** the borrows clash. Compute
  the bool first, then call it (see the `reply` example).
- **`content_safe`** turns mentions into `@name`; don't use raw `msg.content`.
- **Serenity `Guild::member`** takes `&ctx` (CacheHttp). `ctx.cache.current_user()` returns a
  guard; copy the id out, don't carry the guard across an await.
- **Ready fires more than once** (reconnects). `started` guards the cycles.
- **`guild_create` fires on every connect.** The `scanned` set prevents re-scanning.
- **`include_str!` paths are relative to `src/`:** `"../prompts/x.md"`.
- **Identifiers must be English and ASCII** (code side, see AGENTS.md item 8); rustc's
  uncommon_codepoints lint will warn otherwise. `prompts/*.md` and `durum/` file fields are
  exempt from this rule and stay Turkish — they need to match the model's JSON output /
  existing on-disk data.
- **`cargo fmt` breaks long `let … else { continue };`** lines into multiple lines; this
  breaks patch matching.
- **Firecrawl response** errors if `data.markdown` is missing; wander falls back to a
  summary.
- **`<link>` in RSS** sometimes collides with `<atom:link/>`; `tag_content` looks for both
  `<link>` and `<link `.
- **Travel table**: holidays are year-specific; add next year at the end of every year.

## Testing
- Unit: `cargo test` (memory: date, slug, person format, keyword, raw fetch; agenda: rss,
  html, entries; travel: day number, year rollover, holiday, place consistency; output
  protocol: line splitting, `tepki:`, the silence marker, slop prefixes, burst limit,
  question ceiling, `message_json`; chat_cli: line parsing, memory history limit).
- Protocol test bench: `cargo run -- chat` (no Discord, just a model key) — to try line
  bursting, `tepki:`, and `-` behavior against a live model.
- Live: fill in `.env`, `cargo run --release`, look in the log for "logged in", "<server>: N
  messages read" (`read history`), "profiler", "coach". `durum/INDEX.md` should get created.
  When a chat ends, `mind: diarist ... written` and a file under `kisiler/`.
- Sleep test: temporarily change `TIMEZONE_OFFSET` or `build_plan`'s hours; or print `plans`
  after `sleep::update`.
- Travel test: via `travel::on_day(day_number(y,m,d))` unit tests.

## Not done / ideas
- Custom (server-uploaded) emoji reactions aren't supported: `extract_emoji` filters out the
  `:kekw:` form, only Unicode emoji get sent. If needed, this would require
  `ReactionType::Custom` plus validation against the server's emoji list.
- ILGI/keyword hook: there's no path to skip the willingness call and jump straight in when
  a specific word appears (the bot's own pet topics); for now everything goes through the
  willingness score.
- Simple stemming for keywords (Turkish suffixes).
- `plans` and `posted_news` aren't written to disk; they reset on restart.
- No voice channel events.

## Knowledge graph (graphify)

`graphify-out/graph.json` maps every Rust symbol, doc concept, and prompt role plus their
edges (EXTRACTED / INFERRED / AMBIGUOUS). Use it instead of grepping across modules:

```
/graphify query "why does State bridge so many modules"
/graphify path "parse_reply" "send_lines"
/graphify explain "reply_budget! Macro"
/graphify . --update      # after code/doc changes; only changed files are re-extracted
graphify export html      # regenerate graph.html locally (ignored by git)
```

Commit `graph.json`, `GRAPH_REPORT.md`, `manifest.json` with the change that made them stale.
