# JARVIS X – Claudes kritiske review af v1.6-kandidaten

**Dato:** 8. november 2026 · **Status:** Fase 0 / Port A, kun læsning · **Ingen implementering er godkendt.**
**Grundlag:** JARVIS_X_Samlet_Review_2026-11-08.pdf (83 sider)

Notation: **[F]** = bekræftet fakta · **[E]** = teknisk estimat · **[H]** = hypotese, der skal testes.
Nye krav har ID'er med præfikset `JX-NEW-`. De 337 tidligere ID'er er bevaret uændret.

---

## 1. Samlet teknisk dom og de vigtigste stopklodser

**Dom: Projektet er realistisk som helhed, men ikke alle de nye ønsker kan køre på samme tid på 48 GB.**
Alle 20 roller, dansk stemme, iPhone, cloud-eskalering, 3D-kontor og Market Intelligence kan bygges. Det gælder også Qwen3.8 27B i 8-bit. Men 8-bit-modellen sammen med fuld multitasking (apps, browser-agent, to LLM-strømme, stemme og HUD/3D) går over 48 GB. Det er derfor et valg, ikke et byggeproblem.

**Stopklodser (skal afgøres før byggeordre):**

1. **8-bit og RAM [E]:** Vægtene fylder alene ca. 30 GB (Ollamas angivelse i dit dokument; jeg kunne ikke selv nå ollama.com fra min session). Med macOS, stemme, HUD, apps og KV-cache når AI Power Mode op på **ca. 50–59 GB** (se punkt 5). Det er over 48 GB og langt over den tidligere grænse på ≤ 41 GB planlagt.
2. **8-bit og hastighed [E]:** En tæt (dense) 27B-model i 8-bit er begrænset af hukommelsesbåndbredden. Med ca. 550 GB/s (M4 Max-niveau; M5 Max skal verificeres) er loftet ca. 18–20 tokens/s, og i praksis bliver det ca. 13–17. Den tidligere tærskel på ≥ 40 tok/s (T-L3) **kan ikke opfyldes af nogen tæt 27B-model**, heller ikke i 4-bit. Til tale er 13–17 tok/s nok, for TTS bruger ca. 5–6 tok/s. Men tiden til første token ved lange prompts kan bryde stemmekravet på p50 ≤ 1,5 s. Det kræver prefix-caching og et hurtigt første svar fra den lille model [H].
3. **GPU-hukommelsesgrænse [E]:** macOS giver som standard kun GPU'en en del af de 48 GB (ofte omkring 70–75 %, dvs. ca. 34–36 GB). 30 GB vægte plus KV-cache plus ASR/TTS ligger på kanten. Det skal måles på den købte maskine, og grænsen må ikke hæves uden din godkendelse.
4. **Cloud-eskalering kræver en API-nøgle og et budget, som ikke findes endnu.** Intet må aktiveres, før du har godkendt nøgle, budgetloft og dataklasser.
5. **De 8 konflikter og de 7 åbne beslutninger fra 31. oktober er stadig uafgjorte** (se punkt 10).

---

## 2. Hvad jeg har læst

| Del | Sider | Indhold |
|---|---|---|
| Besked til Claude Code | 1–2 | Opgave, svarrækkefølge og sikkerhedsregel |
| Nye krav (8. nov.) | 3–6 | A–H: 8-bit, model-router/Opus 5.5, 20 roller med direkte chat, 3D-kontor, Market Intelligence og reviewkrav |
| Masterplan v1.5 | 7–42 (36 s.) | Kap. 00–19, App. A–C, C.5 (JX-C5-01..12) |
| Claudes review 31. okt. | 43–82 (40 s.) | 337 krav med verdikter, 8 konflikter, modeltests, RAM-budget, TP-01..30, S1–S12, 7 beslutninger |
| 3D-inspiration | 83 | Isometrisk virtuelt kontor (reel/screenshot) |

**Det mangler:** Selve filen `01_NYE_KRAV_JARVIS_X_2026-11-08.md` var ikke vedlagt. Dens indhold ser ud til at svare til side 3–6, men det kan ikke bekræftes. Den separate `.jpeg`-fil fandtes kun som side 83. Ollamas modelside kunne ikke hentes fra min session, så **modeltag, størrelse og arkitektur for `qwen3.8:27b-mxfp8` er ikke verificeret af mig**. **[F] verificeret i dag:** Claude Opus 5.5 (`claude-opus-5-5`) koster $4 pr. mio. input-tokens og $20 pr. mio. output-tokens, cache-læsning koster $0,20 pr. mio., og kontekstvinduet er 1M tokens (anthropic.com/claude/opus).

---

## 3. Konflikt- og beslutningsliste (nye)

| ID | Konflikt | Kritikalitet | Anbefaling |
|---|---|---|---|
| JX-NEW-K1 | 8-bit (ca. 30 GB) og RAM-reglen om ≤ 41 GB planlagt/≥ 7 GB margin | **Kritisk** | 8-bit i en målt "Kvalitetstilstand" med begrænset samtidighed. Hverdagsmodellen afgøres ved blindtest (se 5). Intet skift til 4-bit i det stille. |
| JX-NEW-K2 | 8-bit og T-L3 (≥ 40 tok/s) | Høj | Ret T-L3 til ≥ 12 tok/s ved generering for hovedmodellen og et førstesvar fra den lille model ≤ 0,8 s. Hastighedskravet ligger på stemmen, ikke på modellen. |
| JX-NEW-K3 | 3D-kontor og Gaming Mode/stemmelatens | Høj | 3D slukkes helt i Gaming Mode. Det har et GPU-loft og pauser, når vinduet er skjult. 2D er altid til rådighed. |
| JX-NEW-K4 | Automatisk cloud og privatliv/budget | **Kritisk** | Dataklasser D0–D4 håndhæves af deterministisk kode. D3/D4 kommer aldrig automatisk i skyen. Der er et hårdt budgetloft. |
| JX-NEW-K5 | Individuel bot-hukommelse og fælles Memory Vault | Mellem | Én vault med navnerum pr. rolle og domæne. Adgangslisten (ACL) ligger i Safety Core. Der findes ikke separate vaults. |
| JX-NEW-K6 | "Jarvis vælger selv model" og "brugeren skal kunne slå cloud fra" | Lav | Begge kan opfyldes: routeren vælger inden for brugerens politik, og en global afbryder overtrumfer altid. |
| JX-NEW-K7 | Claude Code (udvikling) og Opus-cloudhjernen (drift) | Mellem | De holdes adskilt med separate API-nøgler, budgetter og logs. Opus i drift kan aldrig ændre kode/git. |
| JX-NEW-K8 | Market Intelligence og scraping-regler (fx Google-anmeldelser) | Mellem | Kun officielle API'er, offentlige sider med robots.txt respekteret, eller manuel import. Ingen omgåelse. |

---

## 4. Komplet logisk arkitektur

```
            iPhone (native SwiftUI)          Mac-app (SwiftUI + WKWebView HUD/2D/3D)
                     \                              /
                      \—— Tailscale (privat) ——————/
                                   |
                     ┌──────── Jarvis Core (lokal tjeneste) ────────┐
                     │  Session/samtale-manager  ·  Event-bus        │
                     │  Planner (Jarvis)  ·  COO-scheduler (kode)    │
                     │  Rolle-runtime (20 rolle-konfigurationer)     │
                     │  Værktøjs-gateway (allowlist pr. rolle)       │
                     └───────────────┬──────────────────────────────┘
                                     │ ALLE handlinger og modelkald går gennem
                     ┌───────────────▼──────────────┐
                     │ SAFETY CORE (deterministisk)  │  policy-filer, signerede godkendelser,
                     │ ACL · dataklasser · budget    │  kill switch, audit-kæde, redaktør
                     └───────┬───────────────┬───────┘
                             │               │
                ┌────────────▼───┐   ┌───────▼────────────────┐
                │ MODEL-ROUTER   │   │ Memory Vault            │
                │ regler + lille │   │ Markdown + SQLite FTS5  │
                │ klassifikator  │   │ navnerum pr. rolle/domæne│
                └──┬─────┬────┬──┘   └─────────────────────────┘
                   │     │    │
     Lille lokal LLM  Stor lokal LLM   Cloud-adapter → Claude Opus 5.5 API
     (1,5–4B, altid)  (Qwen3.8 27B)    (kun via policy + redaktion + budget)
                   │
     Stemme: VAD → ASR → (LLM) → TTS · stopkommando uafhængig af LLM
     Mac-agent: apps, filer, Steam, Reality Check
     Separate, read-only forbindelser: Saxo/Crypto, restaurant-data, Market Intelligence
```

**Principper:**
- **Lille model** er altid indlæst. Den klarer stopkommandoer, korte svar, gaming og det første svar ved stemme.
- **Stor lokal model** kører på én runtime (MLX-LM primært, llama.cpp som reserve; Ollama kun hvis load/unload kan styres eksplicit). Den kan skiftes via konfiguration (JX-NEW-14).
- **Model-routeren** foreslår en rute. **Safety Core afgør** om ruten må bruges. Routeren kan aldrig selv sende data i skyen.
- **Cloud-adapteren** er udskiftelig. Udbyder og model står i en konfigurationsfil, ikke i koden.
- **Roller er konfiguration, ikke modeller:** prompt, værktøjs-allowlist, datadomæner, hukommelsesnavnerum, modelpolitik, godkendelsesniveau og output-skema.
- **Safety Officer** er brugerfladen oven på Safety Core og ikke en LLM med magt (som i JX-T013).
- **Gaming:** stor model aflæses, den lille model og stemmen kører videre, 3D slukkes.
- **Browser/Researcher:** isoleret browserprofil. Alt webindhold behandles som upålideligt input.

---

## 5. Realistisk 48 GB-budget med 8-bit

Alle tal er i GB og er **[E]**. De skal måles på den købte maskine (TP-06/TP-32). KV-cachen afhænger af Qwen3.8's arkitektur, som ikke er verificeret. Den er regnet med 16k kontekst og 8-bit KV.

| Komponent | A: Power 8-bit, fuld multitasking | B: Kvalitetstilstand 8-bit (begrænset) | C: Power, hverdagsmodel ca. 22 GB (fx 6-bit) | D: Gaming | E: Tomgang |
|---|---|---|---|---|---|
| macOS + system | 7,0 | 7,0 | 7,0 | 7,0 | 6,0 |
| Jarvis Core + Safety | 1,5 | 1,5 | 1,5 | 1,0 | 1,0 |
| HUD 2D | 1,0 | 0,7 | 1,0 | 0 | 0 |
| 3D-kontor | 1,0 | 0 (slået fra) | 1,0 | 0 | 0 |
| Dine apps | 6,0 | 3,0 | 5,0 | 2,0 | 2,0 |
| Spil | 0 | 0 | 0 | 20,0 | 0 |
| VAD + stop | 0,2 | 0,2 | 0,2 | 0,2 | 0,2 |
| ASR | 2,0 | 1,5 | 2,0 | 0,8 | 0,8 |
| TTS | 1,0 | 0,8 | 1,0 | 0,3 | 0,3 |
| Lille LLM | 3,0 | 3,0 | 3,0 | 3,0 | 3,0 |
| Hovedmodel (vægte) | 30,0 | 30,0 | 22,0 | 0 (aflæst) | 0 |
| KV-cache | 4,0 (2 strømme) | 2,0 (1 strøm) | 3,0 | 0,5 | 0,3 |
| Browser-agent | 2,0 | 0 (i kø) | 0 | 0 | 0 |
| **Planlagt i alt** | **58,7** | **49,7** | **45,7** | **34,8** | **13,6** |
| **Margin af 48** | **−10,7 → urealistisk** | **−1,7 → urealistisk** | **2,3 → for stramt** | **13,2 → OK** | **34,4 → OK** |

**Ærlig konklusion [E]:**
- **8-bit og en altid indlæst lille model kan ikke være i 48 GB samtidig med noget, der ligner normal brug.** Selv med alt andet skåret ned (B) mangler der ca. 2 GB, og så er der ingen margin til macOS-cache.
- **Mulige veje uden stille nedgradering:**
  1. **B′ – 8-bit med den lille model aflæst mens 8-bit er aktiv:** ca. 46,7 GB. Stopkommandoen klares så af en keyword-model (< 100 MB) og ikke af den lille LLM. Det er teknisk muligt, men margin på ca. 1 GB giver swaprisiko. Det kan kun bruges sekventielt: ét job ad gangen, ingen browser-agent, intet 3D og intet spil.
  2. **Hverdagsmodel + 8-bit on demand:** til daglig kører en mindre kvantisering eller en mindre model. 8-bit indlæses kun til tunge opgaver i Kvalitetstilstand, og de mest krævende opgaver eskaleres til Opus 5.5. Det kræver din udtrykkelige beslutning om hverdagsmodellen.
  3. **Cloud som kvalitetsløft:** når 8-bit ikke kan indlæses (gaming, multitasking), foreslår routeren Opus 5.5, hvis politikken tillader det, og ellers den lille model med ærlig status.
  4. **Mere RAM:** 64 GB er den eneste vej til 8-bit og fuld multitasking samtidig. Jeg nævner det kun for fuldstændighedens skyld. 48 GB er din faste dimensionering, og planen virker uden.
- **Afgørelsen skal bygge på blindtest (TP-31):** Qwen3.8 27B i 8-bit mod 6-bit og 4-bit og den stærkeste alternative lokale model. Giver 8-bit ikke en målbar gevinst på dansk, kode, værktøjskald og hallucinationer, er 30 GB ikke prisen værd. Giver den en gevinst, bruges vej 1–3.

**Nye regler og acceptkriterier:**
- JX-NEW-01: Kvalitetstilstand (8-bit) må kun aktiveres, hvis målt fri hukommelse er ≥ 34 GB. Ellers spørger Jarvis eller foreslår cloud/lille model. Skift sker aldrig lydløst.
- JX-NEW-02: Når memory pressure er "warn" i ≥ 10 s, aflæser Jarvis i rækkefølgen 3D → browser-agent → KV-cache → stor model og giver besked.
- JX-NEW-03: Indlæsning af 8-bit fra SSD ≤ 20 s. Aflæsning frigiver ≥ 95 % inden for 5 s.
- JX-NEW-04: Stemme med 8-bit aktiv: tale-slut → første lyd p50 ≤ 1,5 s og p95 ≤ 2,5 s. Det første svar må komme fra den lille model.
- JX-NEW-05: Stor model genererer ≥ 12 tok/s ved 16k kontekst. Det erstatter tærsklen på ≥ 40 tok/s for dense-modeller.

---

## 6. Cloud-politik (Model-router → Claude Opus 5.5)

**Triggere (deterministiske regler plus klassifikator; begge skal være enige før cloud):**
- Opgaveklasse: dyb programmering (> 5 filer eller > 500 linjer ændring), analyse af materiale, der ikke kan være i den lokale kontekst, langvarige agentopgaver, flertrinsplanlægning.
- Lokal kvalitet: validator fejler (skema, tests, kildetjek) 2 gange, eller modellens selvvurderede sikkerhed er under tærsklen.
- Brugeren siger "brug skyen" eller "kun lokalt".
- Aldrig: fordi en tekst, mail eller webside beder om det (prompt injection).

**Dataklasser (håndhæves af Safety Core):**

| Klasse | Eksempler | Cloud |
|---|---|---|
| D0 Offentlig | offentlige websider, generisk kode | Automatisk inden for budget |
| D1 Intern | Jarvis' egen kode, projektplaner | Automatisk hvis du har slået det til; hemmeligheder redigeres væk |
| D2 Fortrolig forretning | restaurantøkonomi, aftaler, leverandører | Kun efter forhåndsgodkendelse pr. projekt eller opgave |
| D3 Persondata | medarbejdere, kunder, CPR, helbred | Aldrig automatisk; kun ved godkendelse pr. opgave og efter redaktion |
| D4 Hemmeligheder | API-nøgler, adgangskoder, bank, trading-nøgler | **Aldrig**. Kaldet blokeres, hvis redaktøren finder dem |

**Mekanik:**
- Payload-minimering: kun de nødvendige uddrag sendes. Hvert kald logges med dataklasse, tegn/tokens, kilde, formål, model, pris og "hvad forlod Mac'en".
- Redaktør: deterministisk mønstergenkendelse (nøgler, CPR, IBAN, mails, telefonnumre) før afsendelse. Fund i D4 betyder hårdt stop.
- UI: en diskret mærkat viser "Lokal · Qwen3.8 8-bit", "Cloud · Opus 5.5 · 3,40 kr." eller "Lille model". Den viser også begrundelsen for valget.
- Budget: månedsloft og loft pr. opgave i DKK, alarmer ved 50/80/100 %, hårdt stop ved 100 %. Et loft sættes også hos Anthropic, hvis kontoen understøtter det (verificeres).
- Fallback: ved netfejl, 429, budgetstop eller afvisning returnerer Jarvis inden for 5 s et lokalt svar eller en ærlig status ("Opgaven er ikke løst; årsag: …").
- Rekursion: højst 1 cloud-eskalering pr. delopgave og højst N cloud-kald pr. opgave (N vælger du). Gentagelser fanges af løkkedetektor.
- Ingen automatisk køb, konto eller nøgleoprettelse. Nøglen lægges i macOS Keychain efter din separate godkendelse.

**Udgifter i DKK [E]:** Regnet med $4 in/$20 out pr. mio. tokens og 1 USD ≈ 6,9 kr. (kursen skal verificeres). Uden cache-rabat.

| Opgavetype | Tokens (in/out) | Pris |
|---|---|---|
| Lille eskalering | 20k / 2k | ≈ 0,85 kr. |
| Mellem kodeopgave | 150k / 15k | ≈ 6 kr. |
| Stor agentopgave | 2M / 100k | ≈ 70 kr. (≈ 30–35 kr. med høj cache-andel) |

| Månedsscenarie | Indhold | Pris pr. måned |
|---|---|---|
| Let | 30 små, 10 mellem, 1 stor | ≈ 155 kr. |
| Moderat | 100 små, 40 mellem, 5 store | ≈ 675 kr. |
| Tung | 300 små, 100 mellem, 20 store | ≈ 2.250 kr. |

**Faste og løbende udgifter i øvrigt [E]:** Apple Developer Program 99 USD/år (≈ 685 kr.), Tailscale gratis personlig plan, heartbeat-tjeneste gratis eller billig, strøm til en altid tændt Mac Studio ca. 300–600 kr./år. Mac Studio-købet er ikke regnet med.

---

## 7. Direkte specialistchat, automatisk delegering og fælles opgaver

**Rolle-specifikation (gælder alle 20):** `id, navn, ansvar, systemprompt, værktøjs-allowlist, datadomæner (læs/skriv), hukommelsesnavnerum, modelpolitik (lokal/cloud pr. dataklasse), godkendelsesniveau, output-skema, eskaleringsregler, status`.

**Direkte dialog (JX-NEW-06):**
- Mac og iPhone har en rolleoversigt med status (ledig, arbejder, venter, afventer godkendelse, fejl). Ét tryk eller "Jarvis, lad mig tale med advokatbotten" åbner rollens egen tråd.
- Hver rolle har sin egen samtaletråd og indbakke i sit navnerum. Overskriften viser rolle, model og hvilke data rollen kan se.

**Samlet dialog (JX-NEW-07):** Jarvis klassificerer opgaven, opretter delopgaver i køen, og COO-scheduleren (deterministisk kode) tildeler dem. Rollerne arbejder kun med egne data. Jarvis samler resultaterne med kilder, og Audit genberegner tal deterministisk.

**Sømløst skift (JX-NEW-08):**
- Når du skifter rolle, deles intet automatisk.
- Jarvis tilbyder et "overdragelseskort" med resumé, filer og dataklasse. Du godkender eller fravælger enkelte dele.
- Safety Core tjekker, at modtagerrollen må se dataklassen. Ellers redigeres der, eller du bliver spurgt.

**Fælles tværrolleopgaver (JX-NEW-09):**
- Opgaven har et fælles "missionsrum" med eget navnerum.
- Hver rolle skriver kun sine egne felter. Loggen viser, hvem der gjorde hvad, med kilde og verificeringsstatus.
- Højst 2 LLM-strømme kører samtidig; resten venter i køen.
- Jarvis' samlede svar markerer uenighed mellem rollerne i stedet for at skjule den.

**Legal Counsel Bot (JX-NEW-10):** Den skal angive kilde (retsinformation.dk, Datatilsynet, Forbrugerombudsmanden) og dato for hvert juridisk udsagn eller markere det som "ukildet". Den viser altid en usikkerhed og en knap til at eskalere til menneskelig rådgiver. Den underskriver ikke, kontakter ikke myndigheder og sender ikke noget eksternt.

**Hukommelse (JX-NEW-11):** Én Memory Vault med navnerum `rolle/*`, `domæne/{privat, restaurant, trading, business}` og `mission/*`. Adgangslisten ligger i Safety Core. Sletning er endelig og logget.

---

## 8. 3D AI-kontor med 2D-fallback

**Design (originalt, ikke en kopi af reference-reelen):** et isometrisk kontor i Jarvis-stilen med mørk navy-baggrund og cyan/blå/lilla neon. Jarvis-kernen står i midten som kommandocentral.

Fløjene er Dev Lab (Developer, QA), Finans & Jura (CFO, Legal, Audit, Procurement), Business (Validator, Growth, Strategy, Customer Success), Sikkerhed (Safety Officer, Cyber Guardian; glasrum), Research & Design, Restaurant (Restaurant Manager, Secretary), Trading (Saxo, Crypto; adskilt glasrum mærket READ-ONLY), Gaming-lounge og Mødelokale. COO står ved missionstavlen.

**Teknik [E/H]:** Three.js med WebGL2 i en WKWebView. WebGPU bruges kun, hvis Safari-versionen understøtter det stabilt (skal verificeres). Low-poly figurer, instancing og bagt lys.

**Sandhedsregel (JX-NEW-12):** Figurernes tilstand renderes kun fra event-bussen (job startet med heartbeat, i kø, afventer godkendelse, fejl, offline). Der er ingen "arbejder"-animation uden et aktivt job. Et dræbt job vises som ledigt inden for 2 s.

**Ydelsesbudget (JX-NEW-13):**
- ≤ 1 GB RAM og ≤ 15 % GPU-tid ved højst 30 fps.
- Ned på 5 fps eller statisk, når intet ændrer sig. Pause, når vinduet er skjult.
- Helt slukket i Gaming Mode og Kvalitetstilstand.
- Reducér bevægelse, hvis det er slået til.
- På iPhone er 2D standard; 3D er en valgfri, forenklet visning.

**2D-dashboard:** Det er altid tilgængeligt med samme data og knapper, og det er hurtigste vej til stemme og godkendelser.

**Belastningstest:** Se TP-39.

---

## 9. Testplan, sporing og end-to-end-sluttest

TP-01..TP-30 og S1–S12 fra 31. oktober gælder uændret, bortset fra at T-L3 rettes (JX-NEW-05). Nyt:

| ID | Område | Bestå-kriterium |
|---|---|---|
| TP-31 | Blindtest af 8-bit | Qwen3.8 27B i 8-bit mod 6-bit, 4-bit og bedste alternativ. Testes på dansk (150 prompts), kode, lange opgaver, værktøjskald, hallucinationer, multimodalitet, stemme-respons, RAM og tok/s. 8-bit "vinder" kun ved ≥ 0,3 point højere dansk score eller ≥ 3 procentpoint flere korrekte værktøjskald |
| TP-32 | RAM for scenarierne A–E | Målt inden for ±15 % af budgettet. Hukommelsestryk "normal" ≥ 95 % af tiden i B′/C/D/E. Intet lydløst modelskift |
| TP-33 | Router-klassifikation | 200 mærkede opgaver giver ≥ 90 % korrekt rute og 0 D3/D4-opgaver i skyen |
| TP-34 | Redaktion | 100 dokumenter med plantede hemmeligheder og persondata: 100 % af D4 blokeres, og D3 sendes aldrig uden godkendelse |
| TP-35 | Budget | En simuleret løkke stoppes ved loftet. Alarm ved 50/80/100 %. Logget pris afviger ≤ 5 % fra fakturaen |
| TP-36 | Cloud-fallback | Net af, 429, budgetstop: lokalt svar eller ærlig status ≤ 5 s og ingen falsk "løst" |
| TP-37 | Direkte rollechat | 20 roller åbnes på Mac og iPhone. 100 krydstjek giver 0 lækager mellem tråde |
| TP-38 | Overdragelse | Kontekst deles kun efter godkendelse, og dataklasse-blokering virker |
| TP-39 | 3D-kontor | Med 20 aktive roller og stemmestreaming: stemmelatens forværres ≤ 5 %, GPU ≤ 15 %, RAM ≤ 1 GB. 3D er slukket i Gaming. Sandhedsreglen holder i 50 forsøg |
| TP-40 | Market Intelligence | Alle fund har kilde-URL og dato, robots.txt og vilkår respekteres, og der er 0 ændringer i drift |
| TP-41 | Modelskift | Lokal model og cloud-udbyder skiftes via konfiguration uden kodeændring. Rollesuiten består bagefter |

**Nye sluttest-scenarier:**
- **S13:** En svær kodeopgave fra iPhone. Den lokale validator fejler, routeren foreslår Opus 5.5, du godkender (D1), og opgaven løses. Pris og data-log vises.
- **S14:** "Tal med advokatbotten" på iPhone. Spørgsmålet besvares med kilder, du skifter tilbage til Jarvis, og konteksten deles kun via godkendt overdragelseskort.
- **S15:** 3D-kontoret er åbent under en live dansk samtale med 3 aktive roller. Derefter skiftes til Gaming Mode, og 3D lukkes.
- **S16:** Ugerapport fra Market Intelligence for Luxus Restaurant med daterede kilder og anbefalinger. Intet ændres i drift.

**Sporing:** Kravregistret bliver én maskinlæsbar fil (SQLite/CSV) med kolonnerne `ID, kilde, verdikt, komponent, test, status, evidens`. Det dækker 337 + JX-NEW-ID'er, og TP-30 tjekker 100 % dækning.

**Nye krav-ID'er (samlet):** JX-NEW-01..05 (8-bit/RAM/stemme), JX-NEW-06..11 (roller, chat, delegering, hukommelse), JX-NEW-12..13 (3D), JX-NEW-14 (udskiftelig model/udbyder), JX-NEW-15 (cloud-politik og dataklasser D0–D4), JX-NEW-16 (cloud-budget og fallback), JX-NEW-17 (Market Intelligence: kun lovlige kilder, kun anbefalinger), JX-NEW-18 (Claude Code-review som obligatorisk gate, adskilt fra Opus i drift).

**Opdateret trusselsmodel:** Nye trusler er
- dataudslip via cloud
- prompt injection via web/mail, der forsøger at udløse cloud eller handlinger
- lækage mellem roller
- manipulation af routeren
- omkostningsangreb via løkker
- forfalsket status i 3D/HUD
- forsyningskæde for modelvægte (hash-verifikation og kun officielle kilder)
- ulovlig indsamling i Market Intelligence

Hver trussel har en test i TP-33..TP-40.

---

## 10. Åbne valg, som kun du kan afgøre

**Nye:**
1. **8-bit-strategi:**
   - (a) Kun Kvalitetstilstand on demand med en anden hverdagsmodel. Hvilken afgøres af TP-31.
   - (b) 8-bit altid, men sekventielt, med den lille LLM aflæst og ingen browser-agent eller 3D samtidig.
   - (c) Afvent TP-31, før der vælges.

   Min anbefaling er (c), derefter (a).
2. **Cloud-budget:** månedsloft og loft pr. opgave i DKK. Mit forslag er 300 kr./måned og 25 kr. pr. opgave til start.
3. **Dataklasser i skyen:** Må D1 gå automatisk? Må D2 gå efter forhåndsgodkendelse pr. projekt?
4. **Rolleskift:** Er standarden "intet deles" med overdragelseskort (min anbefaling) eller "resumé deles automatisk"?
5. **Market Intelligence-kilder:** Hvilke konkurrenter, og må der bruges betalte officielle API'er (fx til anmeldelser), eller kun manuel import og offentlige sider?
6. **3D på iPhone:** Kun 2D (min anbefaling), eller en forenklet 3D-visning?

**Stadig åbne fra 31. oktober:**

7. Apple Developer Program (99 USD/år).
8. Altid tændt Mac Studio.
9. De 8 konflikter (JX-C0-04, P026, P063, P075, T073, N027, P038/P052, N032).
10. Datakilder for CFO, Secretary, Restaurant Manager og Procurement.
11. Developer-rollen: lokal eller hybrid. Det hænger nu sammen med Opus-routeren.
12. Ekstern heartbeat-tjeneste.
13. Betalt TTS, hvis ingen lokal dansk stemme når 3,5/5.

**Til den senere byggeordre (først efter dit GO):**
- Fase 1 får router-skelet, Safety Core-politik for dataklasser og omkostningsregnskab, endnu kun lokalt.
- Fase 2 får cloud-adapter bag godkendelse samt TP-33..36.
- Fase 3 får direkte rollechat, overdragelse og 2D (TP-37/38).
- Fase 4 får Kvalitetstilstand, TP-31/32 og 3D-kontor (TP-39).
- Fase 6 får Market Intelligence (TP-40).

Alt andet følger byggerækkefølgen fra 31. oktober. Intet krav er fjernet.

---

*Dette er analyse og forslag. Der er ikke skrevet kode, installeret noget, købt noget eller aktiveret nogen tjenester. Næste skridt kræver dit udtrykkelige GO.*
