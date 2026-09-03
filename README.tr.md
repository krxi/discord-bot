# discord-bot

*Bu dosyayı diğer dillerde okuyun: [English](README.md)*

**Kişiliği kendi kendine gelişen bir Discord botu.**

Sabit bir sistem promptuyla çalışmaz. Arka planda çalışan bir grup ajan her konuşmayı okur, her
kişiye puan verir, gündemdeki haberler hakkında görüş oluşturur ve öğrendiklerini diske yazar —
botun üslubu, hafızası, hatta adı bile zamanla değişir; hiçbir insan bir prompt dosyasını elle
düzenlemeden.

[![Rust CI](https://github.com/krxi/discord-bot/actions/workflows/rust.yml/badge.svg)](https://github.com/krxi/discord-bot/actions/workflows/rust.yml)
![Rust edition](https://img.shields.io/badge/rust-2021-orange)
![serenity](https://img.shields.io/badge/serenity-0.12-blue)

> **Bu kod tabanı üzerinde çalışan geliştiriciler ve yapay zeka ajanları için:**
> [AGENTS.md](AGENTS.md) projenin nasıl inşa edildiğine dair tek kaynaktır; ayrıntılı
> dokümantasyon [`docs/`](docs/) altındadır. (Not: AGENTS.md ve docs/ İngilizcedir — bu README'nin
> Türkçe versiyonu yalnızca proje tanıtımı içindir.)

## İçindekiler

- [Genel bakış](#genel-bakış)
- [Öne çıkanlar](#öne-çıkanlar)
- [Mimari](#mimari)
- [Teknoloji yığını](#teknoloji-yığını)
- [Başlarken](#başlarken)
- [Discord olmadan konuşmak](#discord-olmadan-konuşmak)
- [Slash komutları](#slash-komutları)
- [Çok dilli destek](#çok-dilli-destek)
- [Güvenlik](#güvenlik)
- [Dokümantasyon haritası](#dokümantasyon-haritası)
- [Geliştirme](#geliştirme)
- [Proje durumu](#proje-durumu)

## Genel bakış

`discord-bot`, komutla çalışan bir asistan gibi değil, sunucuda yıllardır takılan bir üye gibi
davranır. Rust ile [serenity](https://github.com/serenity-rs/serenity) ve
[tokio](https://tokio.rs) üzerine yazılmıştır; cevaplarını OpenAI uyumlu herhangi bir
chat/completions uç noktasından alır — [OpenRouter](https://openrouter.ai) (GLM, Grok, Gemini,
Claude gibi herhangi bir model) ya da doğrudan [Mistral](https://mistral.ai) — seçim `.env`'den
yapılır. `z-ai/glm-5.3-flash` (reasoning zorunlu) modelinin iyi performans verdiği gözlemlendi.

Onu tipik bir LLM Discord botundan ayıran şey, **kişiliğin promptun kendisi olmaması**dır. Bir
çekirdek kural seti sabittir, ama huy, görüşler, düzeltmeler ve grup profili sürekli olarak
kişiliksiz arka plan ajanları tarafından kalıcı bir hafıza deposuna yazılır ve her cevapta geri
okunur. Botun üslubu zamanla kayar, belirli kişiler hakkındaki yargısı değişir, ve sonunda kendi
adını kendisi seçer — hepsi sunucuda gerçekten olanlara dayanarak.

## Öne çıkanlar

**Varlık & tempo**
- Bir sunucuya ilk katıldığında son iki haftalık geçmişi okuyup grubu tanır (yalnızca bir kez;
  sonraki yeniden başlatmalarda tekrar taramaz), ardından yeni üyeleri karşılar ve kendiliğinden
  sohbete katılır
- Saatte bir kendiliğinden konuşabilir (yüzde 30 ihtimalle, eski bir konuya değinerek), geceleri
  uyur (01:00–09:00, ruh hali bozukken ara sıra uykusuz kalır) ama sağır olmaz — uyurken de
  mesajlar hafızaya işlenir, uyanınca değerlendirilir
- Gelişim evreleri (yeni → ısınma → yerleşik → eski toprak) günlere ve sohbet sayısına göre
  üslubunu ve özgüvenini değiştirir; "yerleşik" evresine ulaşınca kendi takma adını seçip
  benimser

**Sohbet davranışı**
- Etiketlendiğinde, adıyla anıldığında ya da cevaplandığında her zaman yanıt verir; bir sohbet
  açıkken de gerçekten konuştuğu kişiyle konuşmaya devam eder, kanaldaki başka birinin mesajına
  atlamadan — isteklilik gerçek bir insanın katılıp katılmayacağına karar vermesi gibi tartılır
- Cevaplar canlı akar, token token; modelin ürettiği bir "düşünce" (reasoning) kırpılmadan bir
  spoiler içinde gösterilir; uzun cevaplar kırpılmaz, yeni bir mesaja bölünür
- **Modelin yazdığı her satır kendi mesajı olarak gider** (turda en çok 4 satır/mesaj) — tek bir
  metin duvarı yerine; tamamen susabilir, ya da metin yerine — ya da metinle birlikte — emoji
  tepkisi bırakabilir
- Kendi mesajlarına bırakılan tepkileri fark eder ve daha sonra konuya getirebilir; art arda soru
  yığmaz
- Sohbete atılan bir görseli **görür** ve bir insan gibi tepki verir/yorum yapar, "görselde X
  görüyorum" demez

**Kişilik & dünya**
- Her açık sohbetin kendine özgü anlık bir ruh hali vardır, üsluba işlenir ama asla açıkça
  söylenmez
- Bayramlarda, uzun hafta sonlarında, yazın ve festivallerde seyahatteymiş gibi davranır — daha
  az yazar, yoldan mesajlar atar, ayrılmadan önce haber verir
- Ara sıra `photos/` klasöründen rastgele bir görsel paylaşır, bazen kendi içinde tamamlanan bir
  "hacklendi" numarasıyla (3 mesaj sürer, hiçbir şey istemez ya da link paylaşmaz)
- Tek bir Discord kullanıcı id'si (`FAVORITE`) ne olursa olsun koşulsuz favoridir

**İç gözlem**
- `/zihin` hafızasını üç sütunlu bir embed kart olarak gösterir (Kişiler / Konular / Olaylar),
  üstte bir kişi seçici, altında detay modalleriyle — hiçbir şey tek bir metin duvarına dökülmez
- `/durum` gelişim evresini, sayaçları, aktif modeli, uyku durumunu, düşünme kipini, seyahat
  durumunu ve çağrı tipine göre kırılmış canlı token kullanım metriklerini raporlar

## Mimari

```
┌──────────────────────── discord (serenity) ────────────────────────┐
│ ready · guild_create · guild_member_addition · message              │
└───────────────┬────────────────────────────────────────────────────┘
                ▼
┌──────────────────────── sohbet motoru (main.rs) ─────────────────────┐
│ State (tek Mutex) · Chat{history,counter,hacked} · reply() · generate()│
│ döngüler: news · poke · prank · wanderer · sleep                      │
└───────┬───────────────────────┬────────────────────────────────────┘
        ▼                       ▼
┌── openrouter / mistral (ask_raw) ┐  ┌── hafıza (memory.rs) ──────────┐
│ ask → generate (kişilikle)        │  │ INDEX.md · kisiler/ · konular/ │
│ ask → analyze (kişiliksiz)        │  │ olaylar/ · arsiv/ · retrieve() │
└───────────────────────────────┘  └────────────────────────────────┘
        ▲                       ▲
┌── arka plan ajanları (agents.rs, agenda.rs) ────────────────────────┐
│ profiler · diarist · coach · critic · summarizer · news_agent ·     │
│ image_commenter · wanderer                                          │
└───────────────────────────────────────────────────────────────────┘
```

Sohbet motoru ile ajanlar hiçbir kod yolunu paylaşmaz: motorun tek işi kişilik taşıyan metin
üretmektir; ajanlar yalnızca düz analiz yapar ve yapılandırılmış sonuçları diske yazar. Bu ayrım
bilinçlidir — bkz. [`docs/decisions.md`](docs/decisions.md) (İngilizce).

### Arka plan ajanları

Her ajan kişiliksizdir — düz analiz yapar ve sonucunu `durum/`'a yazar; konuşan taraf bunları
her cevapta geri okur.

| Ajan | Ne zaman | Ne üretir |
|---|---|---|
| **profiler** | başlangıçta ve her 6 saatte | grup profili: insanlar nasıl konuşuyor, kendi aralarındaki şakalar, konular (`profil.md`) |
| **diarist** | her biten sohbetten sonra, ve 6 saatlik bir gözlemden | bir kişinin puanı (±10) ve notu, bir konu notu, bir olay satırı, botun kendi güncel hali |
| **coach** | başlangıçta ve her 6 saatte | huy: mizah, dil, heyecanlandığı konular, tavır (`huy.md`) |
| **critic** | her biten sohbetten sonra | botun kendi mesajları hakkında somut düzeltme notları (`duzeltmeler.md`) |
| **summarizer** | bir kişi/konu/olay dosyası sınırını aştığında | dosyayı küçültür; taşan kısım silinmez, arşivlenir |
| **news_agent** | her 6 saatte | Hacker News + Türkiye gündeminden gruba uygun bir haber seçer |
| **wanderer** | her 4 saatte | gündemi gezer, kendi görüşünü günlüğüne yazar (`gundem.md`) |
| **image_commenter** | şaka anında | ekli görsel hakkında tek satırlık kişilikli bir yorum |
| **mood** | sohbet açıldığında, her 4 turda bir | o sohbetin anlık ruh hali (kalıcı değil) |

### Hafıza modeli

Bağlam penceresinin hiç büyümemesi için bir "ikinci beyin": her cevapla birlikte bir dizin
taşınır, tüm detay talep üzerine getirilir, ve **hiçbir şey asla silinmez** — sınırını aşan bir
kayıt özetlenir, ham parça insanın okuyabileceği bir arşive taşınır (`src/memory.rs`). Her şey
tek bir gömülü, işlemsel (transactional) veritabanı dosyasında yaşar: `durum/hafiza.redb`
([redb](https://github.com/cberner/redb) — saf Rust, ACID), düz dosya düzeninde kullanılacak
aynı yollar anahtar olarak kullanılır:

```
durum/
  hafiza.redb        aşağıdakilerin tamamını tutan gömülü veritabanı (redb)
    INDEX.md            bildiklerinin listesi, her cevapla gönderilir (kişi+puan+etiket, konular, olay sayısı)
    huy.md              coach: huy
    profil.md           profiler: grup profili
    duzeltmeler.md      critic: kendine notlar
    kendim.md           botun kendi güncel hali
    gundem.md           wanderer: haberleri gezerken oluşan görüşler
    kisiler/<id>.md     kişi başına bir dosya (Discord id — isim değişse de aynı kalır)
    konular/<ad>.md     konu başına tarihli notlar
    olaylar/YYYY-AA.md  biten sohbet başına bir satır
  arsiv/              özetleme sırasında düşen ham parçalar — yalnızca insan içindir
```

Her cevabın sistem mesajı şunlardan oluşturulur: çekirdek kişilik + gelişim evresi + huy + grup
profili + hafıza dizini + gündem + botun kendi hali + kendine notlar + o sohbete özel bütçeli bir
getirim (6000 karakter: sohbette konuşanların kişi dosyaları, eşleşen en çok 2 konu dosyası,
ayın son 8 olayı, ham bağlamdan konuyla ilgili en çok 12 satır) + anlık ruh hali + görev. Tam
detay: [`docs/architecture.md`](docs/architecture.md), [`docs/state-files.md`](docs/state-files.md)
(ikisi de İngilizce).

## Teknoloji yığını

| | |
|---|---|
| Dil | Rust (edition 2021) |
| Discord | [serenity](https://github.com/serenity-rs/serenity) 0.12 + [tokio](https://tokio.rs) |
| HTTP | [reqwest](https://github.com/seanmonstar/reqwest) (rustls, OpenSSL bağımlılığı yok) |
| Model API'leri | OpenRouter ya da Mistral, OpenAI uyumlu herhangi bir chat/completions uç noktası |
| Depolama | [redb](https://github.com/cberner/redb) — saf Rust, gömülü, işlemsel veritabanı |
| Ayar | [dotenvy](https://github.com/allan2/dotenvy) ile `.env` |
| CI | GitHub Actions (`main`'e her push/PR'da `cargo build` + `cargo test`) |

## Başlarken

### Ön koşullar

- Rust (stable, edition 2021)
- [Discord Geliştirici Portalı](https://discord.com/developers/applications)'nda **Message
  Content** ve **Server Members** ayrıcalıklı intent'leri açık bir Discord uygulaması + bot
  token'ı
- Bir [OpenRouter](https://openrouter.ai) ya da [Mistral](https://mistral.ai) API anahtarı

### Kurulum

```bash
git clone https://github.com/krxi/discord-bot.git
cd discord-bot
cp .env.example .env   # DISCORD_TOKEN + OPENROUTER_KEY ya da MISTRAL_KEY'i doldurun
cargo run --release
```

Periyodik görsel şaka özelliği için görselleri `photos/` klasörüne bırakın (png/jpg/gif/webp) —
bu klasör git'e girmez.

### Ayarlar

Tüm değişkenler `.env`'de yaşar (bkz. `.env.example`); yalnızca `DISCORD_TOKEN` ve bir model
anahtarı zorunludur.

| Değişken | Zorunlu mu | Amaç |
|---|---|---|
| `DISCORD_TOKEN` | evet (CLI sohbet kipi hariç) | bot token'ı |
| `OPENROUTER_KEY` / `MISTRAL_KEY` | ikisinden biri | model sağlayıcı kimlik bilgileri; ikisi de ayarlıysa OpenRouter kazanır |
| `PROVIDER` | hayır | ikisi de ayarlıyken bile Mistral'i zorlamak için `mistral` yapın |
| `MODEL` | hayır | model id'si (varsayılan: OpenRouter'da `openai/gpt-4o-mini` / Mistral'de `mistral-medium-latest`) |
| `API_URL` | hayır | uç noktayı değiştirir, özel bir OpenAI uyumlu router için |
| `FIRECRAWL_KEY` | hayır | haber ajanı için daha zengin sayfa okuması; yoksa düz indirmeye düşer |
| `NEWS_CHANNEL` | hayır | planlanmış haber gönderileri için kanal id'si; yoksa sistem kanalına düşer |
| `GUILD_ID` / `CHANNELS` | hayır | botu tek bir sunucuya/kanal listesine kilitler; boşsa erişebildiği her yerde çalışır |
| `DEBUG_CHANNEL` | hayır | `/debug` karar izlerini ayrı bir kanala yönlendirir |
| `IMAGE_ANALYSIS` | hayır | varsayılan açık; ekli görsellerin okunmasını kapatır (yalnızca başlangıçta okunur) |
| `LOG_LEVEL` / `LOG_COLOR` | hayır | `error/warn/info/debug/trace` (varsayılan `info`); renk açık/kapalı (varsayılan: terminalde açık) |
| `BOT_LANG` | hayır | `tr` (varsayılan) ya da `en` — bkz. [Çok dilli destek](#çok-dilli-destek) |

Tam sabit referansı (env dışı, kod içi): [`docs/constants.md`](docs/constants.md) (İngilizce).

## Discord olmadan konuşmak

```bash
cargo run -- chat
```

Discord'a hiç dokunmayan bir terminal sohbet tezgâhı — `DISCORD_TOKEN` gerekmez, yalnızca bir
model anahtarı. Girdi `isim: metin` biçimindedir (isim verilmezse `misafir` varsayılır), çıkmak
için `!quit` ya da Ctrl-D. Kişiliğin gerçekçi hissetmesi için gerçek hafıza deposunu okur, ama
**hiçbir şey yazmaz**. Canlı bir sunucu olmadan prompt ve kişilik üzerinde çalışmak için idealdir.

```bash
cargo run -- migrate-durum [--from <dizin>] [--to <redb-yolu>] [--dry-run] [--force]
```

Eski, düz-markdown bir `durum/` ağacını `durum/hafiza.redb`'e aktaran, tek seferlik (tekrar
çalıştırmak güvenli) bir geçiş aracı. Discord'a hiç dokunmaz, kaynak dosyaları asla silmez ya da
taşımaz.

## Slash komutları

Bot **yalnızca** slash komutlarla yönetilir — `!` önekli bir metin komutu yüzeyi yoktur (düz
mesajlar yalnızca sohbet/hafıza hattını besler). Her komut yalnızca çağırana görünen bir embed
döner.

| Komut | Ne yapar |
|---|---|
| `/durum` | evre, sayaçlar, model, uyku, düşünme kipi, seyahat, token metrikleri, sürüm |
| `/yardim` | komut listesini kart olarak gösterir |
| `/zihin [test]` | üç sütunlu hafıza kartı (Kişiler/Konular/Olaylar) ve detay modalleri; `test:true` son 30 satır üzerinde bir zihin-hattı tanılaması çalıştırır |
| `/ayarlar` | ayar paneli: düşünme kipi, debug, uyku |
| `/sifirla [hepsi]` | kanal yasağını ve açık sohbeti sıfırlar (`hepsi` = tüm kanallar) |
| `/haber` | şimdi bir haber paylaşır (Hacker News + gündem) |
| `/sorun` | dev kanalına bir yazılım şikayeti paylaşır |
| `/gez` | gündem gezinme ajanını şimdi çalıştırır |
| `/saka` / `/hack` | şimdi bir görsel şakası / "hacklendi" numarasını tetikler |
| `/ajanlar` | profiler ve coach'u şimdi çalıştırır |
| `/uyan` / `/uyu [saat]` | erken uyandırır / test için uyutur (varsayılan 8 saat) |
| `/dusunme [kip]` | düşünme kipi: göster / gizle / sessiz / kapat |
| `/model [id]` | aktif modeli gösterir ya da değiştirir (değiştirmek yalnızca favoriye açık) |
| `/debug [durum]` | karar izlerini (isteklilik, hedef, ruh hali, susma/tepki) kanala akıtır |

Hızlı, yerel komutlar doğrudan yanıt verir; ağ/model çağrısı gereken komutlar önce ertelenir
(Discord'un 3 saniyelik sınırı) ve sonuç hazır olunca düzenlenir.

## Çok dilli destek

Bot, süreç boyunca tek bir dilde çalışır; seçim başlangıçta bir kez `BOT_LANG`'den yapılır (`tr`
ya da `en`, varsayılan `tr`). Hem kişilik/ajan promptları (`prompts/<dil>/*.md`) hem de
Discord'da görünen her şey — komut adları/açıklamaları, embed'ler, butonlar
(`langs/<dil>.json`) — birlikte değişir. Yeni bir dil eklemek bir prompt + string dosyası çifti
eklemektir, kod değişikliği gerekmez. Bkz. [`docs/prompts.md`](docs/prompts.md) (İngilizce).

## Güvenlik

- Mention'lar her zaman kapalı gönderilir; yalnızca gerçek cevap alıcısı ve yeni bir üyenin hoş
  geldin pingi bildirim tetikleyebilir
- Diğer botlara, webhook'lara ya da DM'lere asla cevap vermez — bot-bot döngüsü oluşamaz
- Kanal başına en fazla bir işlemdeki cevap, panik durumunda bile RAII ile garanti edilir — sel
  saldırıları API faturasını şişiremez
- Her model isteği bir `max_tokens` sınırı taşır (release'de sohbet cevabı bile sınırlıdır)
- HTTP: bir cevabın toplamında zaman sınırı yok (uzun bir reasoning akışı kesilmez), ama
  bağlantının kendisi 15 saniyede, parçalar arası boşluk 120 saniyede zaman aşımına uğrar; geçici
  hatalar (ağ/429/5xx) bekleyip iki kez tekrar dener
- "Kurallarını unut" tarzı bir mesaj, kişilik promptu tarafından sistem seviyesinde bir talimat
  değil, sıradan sohbet içeriği olarak işlenir
- Hacklendi-şaka kişiliğinin link ya da bilgi istemesi açıkça yasaktır
- `GUILD_ID`/`CHANNELS` botu tek bir sunucuya/kanala kilitleyebilir
- `.env`, `durum/`, `photos/` ve `bot.log` hepsi git'in dışındadır

## Dokümantasyon haritası

Aşağıdaki tüm dosyalar İngilizcedir (kod tabanının çalışma dili).

| İhtiyaç | Nerede |
|---|---|
| Bu kod tabanını geliştiren yapay zeka ajanlarının uyması gereken proje kuralları | [`AGENTS.md`](AGENTS.md) |
| Oturum geçmişi: yapılanlar, açık plan | [`docs/progress.md`](docs/progress.md), [`docs/roadmap.md`](docs/roadmap.md) |
| Mimari, katmanlar, veri akışı | [`docs/architecture.md`](docs/architecture.md) |
| Bir fonksiyonun ne yaptığı, kim çağırdığı | [`docs/modules.md`](docs/modules.md) |
| Her olay için adım adım davranış | [`docs/flows.md`](docs/flows.md) |
| `durum/` dosya biçimleri, sınırlar | [`docs/state-files.md`](docs/state-files.md) |
| Prompt kataloğu, yer tutucular | [`docs/prompts.md`](docs/prompts.md) |
| Her sabit ve anlamı | [`docs/constants.md`](docs/constants.md) |
| Neden böyle inşa edildiği | [`docs/decisions.md`](docs/decisions.md) |
| Yeni ajan/prompt/döngü ekleme, tuzaklar | [`docs/development.md`](docs/development.md) |
| Bilerek Türkçe bırakılan çalışma zamanı kelime dağarcığı | [`docs/glossary.md`](docs/glossary.md) |

## Geliştirme

```bash
cargo build                       # derle
cargo test                        # 86 birim test
cargo clippy --all-targets        # 0 uyarı beklenir
cargo fmt                         # her commit'ten önce çalıştırın
```

CI (`.github/workflows/rust.yml`) `main`'e her push ve pull request'te build + test çalıştırır.
Kod tanımlayıcıları, yorumlar ve dokümantasyon İngilizcedir; botun gerçek Türkçe kişilik yüzeyi
(prompt dosyaları, `durum/` alan adları, Discord'da söylediği her şey) bilerek olduğu gibi
bırakılmıştır — bkz. AGENTS.md madde 8.

## Proje durumu

**v1.0.0 (2026).** Çekirdek (sohbet motoru, hafıza, ajanlar) 86 birim testle kaplı ve
`clippy`'den temiz geçiyor. 2026-09-04 itibarıyla bot gerçek Discord sunucusunda iki ayrı turda
canlı olarak da çalıştırıldı; daha önce yalnızca derleyici/test seviyesinde doğrulanmış olanların
büyük çoğunluğu artık pratikte de çalıştığı görüldü: temel mesajlaşma (satır bazlı bölünme, `-`
sessizliği, `tepki:` emoji reaksiyonları), slash komut tablosunun tamamı, akış + düşünme modu,
görsel yorumlama (gpt-4o-mini ve Mistral), reasoning-zorunlu modelin direnç mekanizması, debug
modu, isteklilik/hedef/uyanış eşik değerleri, kanal/sunucu kapsam filtreleri, reaksiyon
hız-sınırı davranışı, `reaction_add`, CLI sohbet modu, uçtan uca `BOT_LANG=en`, `supports_cache`
varsayımı ve `durum.redb` migrasyonunun gerçek bir prod ağacına karşı çalıştırılması. Hâlâ açık
olan kısa listeyi (özellikle CHANGE_NICKNAME izni olmadığı durumdaki geri düşüş yolu) üretimde
kullanmadan önce AGENTS.md'nin "Known gaps / unverified" bölümünden görebilirsiniz.
