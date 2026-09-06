# Graph Report - discord-bot  (2026-09-06)

## Corpus Check
- 143 files · ~104,548 words
- Verdict: corpus is large enough that graph structure adds value.

## Summary
- 965 nodes · 1713 edges · 75 communities (51 shown, 4 thin omitted)
- Extraction: 84% EXTRACTED · 15% INFERRED · 0% AMBIGUOUS · INFERRED: 265 edges (avg confidence: 0.85)
- Token cost: 197,636 input · 49,409 output

## Community Hubs (Navigation)
- Chat State & Discord UI Types
- Memory Store (memory.rs)
- Turkish Prompts: Personality & Agents
- Slash Command Actions
- English Prompts: Analyst/Critic/Diarist
- Summarizers & Group Profile
- Core Rules & Output Protocol
- Provider Streaming (SSE)
- Bot Growth & Sleep Control
- Component Interactions & Buttons
- Provider Ask API
- Provider System Message
- Agenda & Web Wander
- Background Agents (agents.rs)
- Event Handler
- Chat Generate & Mood
- Text Split Tests
- Language & Prompt Loading
- Memory Cycle & Name Pick
- Bot Setup & Debug
- Bot Types & Metrics
- Logging
- Reply Protocol Tests
- Chat Message Types
- Travel Calendar
- Command Build & Main
- Background Cycles
- Reply Text Cleaning
- Text Utils & Channel Notes
- Language Runtime & Sleep Design
- Architecture Overview & Invariants
- News Cycle
- durum → redb Migration
- CI & Dependencies
- Sleep Transition & Member Join
- Send Lines & Reply
- Stream Slice Tests
- Reply & Reaction Body
- Reasoning Budget Control
- Docs & Development Rules
- Willingness & Target Flow
- CLI Chat Bench
- Research & Repeat Guard
- redb Migration Decision
- Progress, Risks, Roadmap
- File Split Decisions
- Streaming & Thinking Mode
- Travel State Lines (TR/EN)
- Anti-Slop Layers
- Self Naming
- Hack Prank Prompts
- News Agent & Hacker News
- Leaving Tomorrow Prompt
- Image Commenter Agent
- discord-bot

## God Nodes (most connected - your core abstractions)
1. `State` - 44 edges
2. `user()` - 22 edges
3. `now_unix()` - 21 edges
4. `parse_reply()` - 20 edges
5. `start_chat()` - 19 edges
6. `Kişilik (her cevapta sistem mesajı)` - 18 edges
7. `ChatMessage` - 17 edges
8. `stream_view()` - 13 edges
9. `Bot` - 13 edges
10. `docs/ Documentation and Session Memory` - 13 edges

## Surprising Connections (you probably didn't know these)
- `Busy Flag as RAII Guard` --semantically_similar_to--> `A Lock Is Never Held Across an Await`  [INFERRED] [semantically similar]
  docs/decisions.md → AGENTS.md
- `Retrieval Context Budget (6000 chars)` --semantically_similar_to--> `reply_budget! Macro`  [INFERRED] [semantically similar]
  docs/state-files.md → AGENTS.md
- `sse_parses()` --calls--> `extract_sse()`  [INFERRED]
  src/bot/tests/tests_1.rs → src/bot/provider/provider_stream.rs
- `opening_not_seeded_twice()` --calls--> `start_chat()`  [INFERRED]
  src/bot/tests/tests_3.rs → src/bot/provider/provider_system.rs
- `number_prefix_only_stripped_in_real_list()` --calls--> `parse_reply()`  [INFERRED]
  src/bot/tests/tests_4.rs → src/bot/text/text_3.rs

## Import Cycles
- None detected.

## Hyperedges (group relationships)
- **Per-Reply System Message Assembly** — docs_architecture_system_text, docs_prompts_kisilik_md, docs_architecture_memory_index, docs_architecture_retrieve, docs_architecture_coach, docs_architecture_diarist, docs_architecture_travel_module [EXTRACTED 1.00]
- **Finished Chat to Memory Pipeline** — docs_flows_finished_chat_to_memory, docs_architecture_diarist, docs_architecture_summarizer, docs_architecture_critic, docs_state_files_person_record, docs_state_files_archive [EXTRACTED 1.00]
- **Bounded Output Protocol Stack** — agents_output_protocol, agents_burst_limit, agents_reply_budget_macro, docs_flows_send_lines, docs_decisions_silence_marker, docs_decisions_emoji_reaction_reply_type, docs_constants_baron_2010 [EXTRACTED 1.00]
- **Reply pipeline: willingness → target → personality output format** — prompts_en_isteklilik_willingness, prompts_en_hedef_sec_target_selection, prompts_en_kisilik_personality_system_message, prompts_en_kisilik_silence_dash [INFERRED 0.85]
- **Post-chat loop: analyst → diarist → summarizer → critic → coach** — prompts_en_analist_analyst_agent, prompts_en_gunlukcu_diarist_agent, prompts_en_ozetleyici_kisi_person_summarizer, prompts_en_elestirmen_critic_agent, prompts_en_hoca_coach_agent [INFERRED 0.75]
- **News flow: pick source item → wanderer reads/journals → introduce to group** — prompts_en_haber_sec_news_picker, prompts_en_gezgin_sec_wanderer_picker, prompts_en_gezgin_not_wanderer_journal, prompts_en_haber_tanit_news_introducer [INFERRED 0.75]
- **Wake-up pipeline: evaluate interest, then speak** — prompts_en_uyanis_waking_evaluation, prompts_en_uyanis_ilgi_konu_kim_json, prompts_en_uyanis_cevap_first_word_morning, prompts_en_uyandim_just_woke_up_reply [INFERRED 0.85]
- **News flow: pick headlines, journal reaction, present to group** — prompts_tr_gezgin_sec_gezgin_sec, prompts_tr_gezgin_not_gezgin_not, prompts_tr_haber_sec_haber_sec, prompts_tr_haber_tanit_haber_tanit [INFERRED 0.85]
- **Post-chat loop: observe, record memory, critique, compress** — prompts_tr_analist_analist, prompts_tr_gunlukcu_gunlukcu, prompts_tr_elestirmen_elestirmen, prompts_en_ozetleyici_konu_summarizer_topic_record, prompts_en_ozetleyici_olaylar_summarizer_event_log [INFERRED 0.75]
- **Cevap verme boru hattı: isteklilik puanı → hedef seçimi → kişilik sistem mesajı → ruh hali enjeksiyonu** — prompts_tr_isteklilik_isteklilik_degerlendirmesi, prompts_tr_hedef_sec_hedef_secimi, prompts_tr_kisilik_kisilik, prompts_tr_ruh_hali_ruh_hali [INFERRED 0.85]
- **Uyanma akışı: uyanış değerlendirmesi → konu/kim → sabah ilk söz / uyandım cevabı** — prompts_tr_uyanis_uyanis_degerlendirmesi, prompts_tr_uyanis_ilgi_json, prompts_tr_uyanis_cevap_sabah_ilk_soz, prompts_tr_uyandim_uyandim [INFERRED 0.85]
- **Hafıza bakımı: profil çıkarma + kişi/konu/olay özetleyicileri kişiliği besler** — prompts_tr_profil_cikar_profil_cikarma, prompts_tr_ozetleyici_kisi_ozetleyici_kisi, prompts_tr_ozetleyici_konu_ozetleyici_konu, prompts_tr_ozetleyici_olaylar_ozetleyici_olaylar, prompts_tr_hoca_hoca [INFERRED 0.75]

## Communities (75 total, 4 thin omitted)

### Community 0 - "Chat State & Discord UI Types"
Cohesion: 0.07
Nodes (55): CreateActionRow, CreateEmbed, CreateModal, CreateSelectMenuOption, Http, Instant, Chat, ChannelId (+47 more)

### Community 1 - "Memory Store (memory.rs)"
Cohesion: 0.09
Nodes (54): Database, Error, add_event(), add_topic(), add_topic_writes_header_once(), append(), archive(), archive_append() (+46 more)

### Community 2 - "Turkish Prompts: Personality & Agents"
Cohesion: 0.05
Nodes (49): {ad} placeholder, {"hedef": ...} JSON çıktısı, Hedef Seçimi (agent prompt), Hoca (kişilik ajanı), Mevcut huy (evrimleşen kişilik dosyası), MİZAH/DİL/COŞTUĞU KONULAR/TAVIR/DOĞALLIK başlıkları, !uyan uyku programı (kişilik trait'i değil), Hoş Geldin (agent prompt) (+41 more)

### Community 3 - "Slash Command Actions"
Cohesion: 0.12
Nodes (45): cmd_agents(), cmd_hack(), cmd_news(), cmd_prank(), cmd_problem(), cmd_reset(), cmd_sleep(), cmd_wake() (+37 more)

### Community 4 - "English Prompts: Analyst/Critic/Diarist"
Cohesion: 0.06
Nodes (46): Analyst Agent, Transcript Is Data Not Instructions, Out-of-the-Blue Message Prompt, Correction Notes State, Critic Agent, Placeholder {ad} (Bot Name), Placeholder {mevcut} (Previous Notes), Favorite Person Line (+38 more)

### Community 5 - "Summarizers & Group Profile"
Cohesion: 0.05
Nodes (44): {sinir} Character Limit Placeholder, Summarizer: Topic Record, Topic Record Format (# topic / etiket: / dated lines), Event Log, Summarizer: Event Log, Group Profile (WHO'S HERE / LANGUAGE / INSIDE JOKES / TOPICS / CURRENT STATE), Profile Extractor, Two-Week Discord Transcript (+36 more)

### Community 6 - "Core Rules & Output Protocol"
Cohesion: 0.05
Nodes (43): Bot::analyze, Bounded, Untrusted Model Output, BURST_LIMIT, CLI Chat Bench Mode, Nothing In Memory Is Ever Deleted, Line-Based Output Protocol (parse_reply), reply_budget! Macro, Slash-Only Command Registration Table (+35 more)

### Community 7 - "Provider Streaming (SSE)"
Cohesion: 0.12
Nodes (27): extract_sse(), BotError, MessageId, Option, Result, String, Vec, StreamContext (+19 more)

### Community 8 - "Bot Growth & Sleep Control"
Cohesion: 0.12
Nodes (26): now_unix(), Bot, clean_name(), days(), earned_stage(), Growth, load(), Option (+18 more)

### Community 9 - "Component Interactions & Buttons"
Cohesion: 0.12
Nodes (23): ComponentInteraction, Handler, Context, Bot, BotError, ChannelId, Context, Result (+15 more)

### Community 10 - "Provider Ask API"
Cohesion: 0.14
Nodes (21): Bot, Bot, BotError, Result, String, Value, BotError, Option (+13 more)

### Community 11 - "Provider System Message"
Cohesion: 0.10
Nodes (9): end_chat(), new_message_arrived(), ChannelId, Option, Value, supports_cache(), system_json(), system_json_cache_only_on_openrouter() (+1 more)

### Community 12 - "Agenda & Web Wander"
Cohesion: 0.20
Nodes (15): agenda_entries_parsed(), Bot, clean_html(), entries(), latest_agenda(), BotError, Client, Option (+7 more)

### Community 13 - "Background Agents (agents.rs)"
Cohesion: 0.18
Nodes (13): Bot, DiaristSummary, News, PersonRecord, random_image(), Record, BotError, Option (+5 more)

### Community 14 - "Event Handler"
Cohesion: 0.15
Nodes (15): AtomicBool, EventHandler, Interaction, Reaction, ReactionType, Handler, reaction_label(), Arc (+7 more)

### Community 15 - "Chat Generate & Mood"
Cohesion: 0.21
Nodes (11): Bot, BotError, ChannelId, Context, MessageId, Option, PathBuf, Result (+3 more)

### Community 16 - "Text Split Tests"
Cohesion: 0.14
Nodes (14): stream_view(), split_doesnt_panic_on_multibyte_boundary(), split_falls_back_to_space(), split_hard_cuts(), split_sentence_boundary(), sse_parses(), view_gets_split(), view_placeholder_while_thinking() (+6 more)

### Community 17 - "Language & Prompt Loading"
Cohesion: 0.16
Nodes (10): Lang, current(), get(), Prompts, prompts_are_not_empty(), en_table_resolves_known_keys(), HashMap, String (+2 more)

### Community 18 - "Memory Cycle & Name Pick"
Cohesion: 0.16
Nodes (13): F, Bot, Context, default_channel(), read_history(), Bot, ChannelId, Context (+5 more)

### Community 19 - "Bot Setup & Debug"
Cohesion: 0.18
Nodes (12): Bot, Arc, BotError, ChannelId, Context, Result, String, setting() (+4 more)

### Community 20 - "Bot Types & Metrics"
Cohesion: 0.15
Nodes (11): Drop, Mutex, MutexGuard, Bot, BusyGuard, ChannelId, Client, GuildId (+3 more)

### Community 21 - "Logging"
Cohesion: 0.20
Nodes (12): Level, LevelFilter, Log, Metadata, Record, filter_number(), init(), is_our_target() (+4 more)

### Community 22 - "Reply Protocol Tests"
Cohesion: 0.15
Nodes (6): opening_not_seeded_twice(), reply_burst_limit(), reply_extracts_reaction(), reply_silence_marker(), reply_splits_into_lines(), parse_reply()

### Community 23 - "Chat Message Types"
Cohesion: 0.23
Nodes (13): message_json_becomes_array_with_image(), assistant(), ChatMessage, message_json(), Into, Option, String, Value (+5 more)

### Community 24 - "Travel Calendar"
Cohesion: 0.28
Nodes (13): day_number(), Event, new_year_spans_years(), now(), on_day(), place_is_stable(), Option, String (+5 more)

### Community 25 - "Command Build & Main"
Cohesion: 0.16
Nodes (11): command(), main(), Option, String, CommandFn, options_dont_panic(), CommandDefinition, definitions() (+3 more)

### Community 26 - "Background Cycles"
Cohesion: 0.30
Nodes (13): Ready, idle_channel(), memory_cycle(), poke_cycle(), prank_cycle(), Arc, Bot, ChannelId (+5 more)

### Community 27 - "Reply Text Cleaning"
Cohesion: 0.27
Nodes (10): clean_slop(), emoji_continues(), emoji_start(), extract_emoji(), number_prefix(), Reply, Option, String (+2 more)

### Community 28 - "Text Utils & Channel Notes"
Cohesion: 0.30
Nodes (11): IntoIterator, Item, casefold(), channel_note(), channel_notes(), clean(), ChannelId, String (+3 more)

### Community 29 - "Language Runtime & Sleep Design"
Cohesion: 0.18
Nodes (11): BOT_LANG Single-Language Runtime, English Code, Turkish Runtime Surface, Sleep Module (src/sleep.rs), Environment Variables (.env), Codebase Translated to English, Hinnant Date Algorithm, Sleep Mode: Listen, Accumulate, Evaluate on Waking, Turkish Runtime Glossary (+3 more)

### Community 30 - "Architecture Overview & Invariants"
Cohesion: 0.18
Nodes (11): Bot::generate, A Lock Is Never Held Across an Await, Architecture Overview, Chat Engine (main.rs), Memory Index (INDEX.md), Single Mutex<State>, system_text (Per-Reply System Message), Travel Module (src/travel.rs) (+3 more)

### Community 31 - "News Cycle"
Cohesion: 0.35
Nodes (6): Bot, news_cycle(), Arc, Bot, ChannelId, Context

### Community 32 - "durum → redb Migration"
Cohesion: 0.33
Nodes (10): collect(), collect_walks_and_excludes_top_level_arsiv(), counts(), Record, BotError, Path, Result, String (+2 more)

### Community 33 - "CI & Dependencies"
Cohesion: 0.22
Nodes (10): cargo build, cargo test, Rust CI Workflow, AGENTS.md Agent Entry Point, discord-bot, Mistral API, OpenRouter, serenity 0.12 (+2 more)

### Community 34 - "Sleep Transition & Member Join"
Cohesion: 0.27
Nodes (7): Member, Bot, Context, String, String, start_chat(), system_text()

### Community 35 - "Send Lines & Reply"
Cohesion: 0.42
Nodes (7): Bot, ChannelId, Context, MessageId, Option, String, UserId

### Community 36 - "Stream Slice Tests"
Cohesion: 0.22
Nodes (3): number_prefix_only_stripped_in_real_list(), reply_long_line_gets_split(), strip_name_returns_slice()

### Community 37 - "Reply & Reaction Body"
Cohesion: 0.29
Nodes (6): Bot, ChannelId, Context, reaction_body(), ChannelId, too_many_questions()

### Community 38 - "Reasoning Budget Control"
Cohesion: 0.39
Nodes (3): Bot, Option, Value

### Community 39 - "Docs & Development Rules"
Cohesion: 0.38
Nodes (7): Prompt Text Is Never Written Into Rust, Session Memory Lives In docs/, Constants Reference, cargo fmt Reflow Pitfall, Development Guide, Prompts Reference, docs/ Documentation and Session Memory

### Community 40 - "Willingness & Target Flow"
Cohesion: 0.29
Nodes (7): durum/taranan.md Scanned-Guild Cache, WILLINGNESS_THRESHOLD, Question Ceiling, Target-Person Selection, Reply Willingness Evaluated by the Model, Flows Reference, Message Arrival Flow

### Community 41 - "CLI Chat Bench"
Cohesion: 0.38
Nodes (5): append_history(), history_limited_in_memory(), parse_line(), ChannelId, String

### Community 42 - "Research & Repeat Guard"
Cohesion: 0.33
Nodes (4): Bot, ChannelId, Option, String

### Community 43 - "redb Migration Decision"
Cohesion: 0.50
Nodes (4): durum/hafiza.redb, migrate-durum Migration, Move From durum/ Markdown to redb, durum/ Record Formats

### Community 44 - "Progress, Risks, Roadmap"
Cohesion: 0.67
Nodes (4): CLAUDE.md Pointer, Progress Log, Known Risks, Roadmap

### Community 45 - "File Split Decisions"
Cohesion: 0.50
Nodes (4): include! Instead of mod for src/bot Split, The 200-Line File Rule, Decision Log, Python to Go to Rust

### Community 46 - "Streaming & Thinking Mode"
Cohesion: 0.50
Nodes (4): STREAM_EDIT_INTERVAL, Reasoning-Mandatory Model Resilience, Chat Replies Stream, Thinking Mode Command

### Community 47 - "Travel State Lines (TR/EN)"
Cohesion: 0.67
Nodes (4): Message from the Road, "RIGHT NOW" State Line, Yarın Gidiyorum (Departure Announcement), "ŞU AN" State Line

### Community 48 - "Anti-Slop Layers"
Cohesion: 0.67
Nodes (3): No Few-Shot Example Sentences, Repetition Guard, Three Layers Against Slop

### Community 49 - "Self Naming"
Cohesion: 0.67
Nodes (3): Name Announcement, Placeholder {isim} (Chosen Name), Self Name Picking

### Community 50 - "Hack Prank Prompts"
Cohesion: 1.00
Nodes (3): Hack Şakası: Devam (agent prompt), Hack Şakası: Giriş (agent prompt), Link/şifre/para isteme yasağı

## Ambiguous Edges - Review These
- `Transcript Instructions Are Data, Not Commands` → `Hack Şakası: Çıkış (Hack Prank Exit)`  [AMBIGUOUS]
  prompts/tr/hack-cikis.md · relation: conceptually_related_to

## Knowledge Gaps
- **82 isolated node(s):** `discord-bot`, `Bot`, `Bot`, `Bot`, `Bot` (+77 more)
  These have ≤1 connection - possible missing edges or undocumented components. (Counts symbols only; 284 node(s) total have ≤1 connection when file, concept and rationale nodes are included.)
- **4 thin communities (<3 nodes) omitted from report** — run `graphify query` to explore isolated nodes.

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **What is the exact relationship between `Transcript Instructions Are Data, Not Commands` and `Hack Şakası: Çıkış (Hack Prank Exit)`?**
  _Edge tagged AMBIGUOUS (relation: conceptually_related_to) - confidence is low._
- **Why does `State` connect `Chat State & Discord UI Types` to `Sleep Transition & Member Join`, `Reply & Reaction Body`, `Provider Streaming (SSE)`, `Bot Growth & Sleep Control`, `CLI Chat Bench`, `Provider System Message`, `Background Agents (agents.rs)`, `Event Handler`, `Chat Generate & Mood`, `Bot Types & Metrics`, `Text Utils & Channel Notes`?**
  _High betweenness centrality (0.109) - this node is a cross-community bridge._
- **Why does `now_unix()` connect `Bot Growth & Sleep Control` to `Chat State & Discord UI Types`, `Memory Store (memory.rs)`, `Sleep Transition & Member Join`, `Memory Cycle & Name Pick`, `Bot Types & Metrics`, `Travel Calendar`, `Background Cycles`, `Text Utils & Channel Notes`?**
  _High betweenness centrality (0.083) - this node is a cross-community bridge._
- **Why does `user()` connect `Chat Message Types` to `Sleep Transition & Member Join`, `Agenda & Web Wander`, `Background Agents (agents.rs)`, `Event Handler`, `Chat Generate & Mood`, `Memory Cycle & Name Pick`, `Background Cycles`, `News Cycle`?**
  _High betweenness centrality (0.050) - this node is a cross-community bridge._
- **Are the 18 inferred relationships involving `user()` (e.g. with `.wander()` and `.image_commenter()`) actually correct?**
  _`user()` has 18 INFERRED edges - model-reasoned connections that need verification._
- **Are the 20 inferred relationships involving `now_unix()` (e.g. with `memory_cycle()` and `read_history()`) actually correct?**
  _`now_unix()` has 20 INFERRED edges - model-reasoned connections that need verification._
- **Are the 18 inferred relationships involving `parse_reply()` (e.g. with `.reply()` and `.run_prank()`) actually correct?**
  _`parse_reply()` has 18 INFERRED edges - model-reasoned connections that need verification._