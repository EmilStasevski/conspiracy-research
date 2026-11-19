# German Fringe-Media Narrative Classifier — Knowledge Pack for LLMs

**Source:** "Catalogue of Key Information Manipulation Narratives in the German Information Space (2014–2026)" (uploaded PDF; synthesises CeMAS, ISD Germany, FES Mitte-Studien, EEAS 4th Threat Report, Verfassungsschutz, Uni Basel, Leipzig Authoritarianism Study).
**Purpose:** Let an LLM tag German-language Telegram / fringe-media posts with one or more of 25 catalogued narratives (+ `NONE` / `OTHER`), using a closed label set and a fixed JSON output.
**Layout:** §1 task rules → §2 enums → §3 decision procedure → §4 narrative cards (the label set) → §5 confusion guide → §6 output schema → §7 few-shot examples → §8 ready-to-paste system prompt → §9 model recommendation for 16 GB VRAM → §10 commercial-post filter.
**Edit (project):** this copy adds a *commercial filter*: product ads, fundraising and channel self-promotion are detected first (`commercial_type`, §2.7, §3 step 0, §10) and are never labelled with a narrative unless the editorial text itself asserts one.

> **Provenance legend.** Fields marked *(report)* come directly from the PDF. Fields marked *(added)* — mainly `cues_de`, `confusable_with`, `not_this_if`, and the few-shot examples — are my additions to make classification easier; they are not claims from the report and should be validated on your own labelled data.

---

## 1. Task rules (read first)

1. **You classify the *content theme* of a post, not who wrote it.** Never assert that a specific post or channel is Russian-made. The `origin_class` and `evidence_level` fields describe the *narrative in general* (as documented in the report), not the individual message.
2. **Closed label set.** Choose only from the IDs in §4, plus `NONE` and `OTHER`.
3. **Multi-label allowed.** Give one `primary_narrative` and up to 2 `secondary_narratives`. Many posts combine frames (e.g. Great Reset + Climate Dictatorship).
4. **Prefer `NONE` over a weak match.** Legitimate criticism (energy prices, migration policy, Ukraine strategy, media bias, AfD-ban debate, gender pedagogy) is *not* a narrative hit. The report explicitly separates legitimate debate from manipulation. A hit requires the post to assert or strongly imply the **core claim** of a narrative (see `core_claim`), not merely touch the topic.
5. **Quoting/debunking ≠ endorsing.** If the post reports, mocks or debunks a narrative, set `stance` accordingly (see §2).
6. **Language:** posts are mostly German (some English, Russian, mixed, emoji, hashtags, sarcasm, dog whistles). Reason in the source language; output field values in English.
7. **Do not invent facts.** `factual_status` comes from the card, not from your own guess.
8. **Commercial filter first *(project edit)*.** Before looking for a narrative, decide `commercial_type` (§2.7). Ads, shop/product promotion, donation appeals and channel self-promotion are *not* narratives: use `NONE`. Only if the non-commercial part of the post (the editorial text above or around the sales pitch) itself asserts a narrative's core claim may you assign that narrative, and then keep `commercial_type` set so the post can be filtered out later. A fear story used only as a sales hook ("Blackout kommt – jetzt Notstromaggregat bestellen") is `product_ad` + `NONE`.

---

## 2. Enumerations

### 2.1 `origin_class` *(report)*
| value | meaning |
|---|---|
| `RUSSIAN` | directly traced to Russian state/proxy actors |
| `GERMAN` | arises from domestic historical/political traditions |
| `TRANSNATIONAL` | imported (e.g. US alt-right/global conspiracism), localised for Germany |
| `HYBRID` | mixed origin, or a domestic issue hijacked/amplified by foreign actors |

### 2.2 `cluster` *(report)*
| code | theme |
|---|---|
| `a` | Delegitimisation of democracy and state institutions |
| `b` | Migration, identity, demographics |
| `c` | Ukraine war and foreign policy |
| `d` | Energy, climate, economy |
| `e` | Health, biopolitics, science *(no catalogued narrative is primarily `e`; N06 and N16 touch it)* |
| `f` | History and memory politics |
| `g` | Gender and "traditional values" |
| `h` | Antisemitic and elite-conspiracy meta-narratives |

### 2.3 `narrative_type` *(report)*
`DISINFORMATION` (false + intent to deceive) · `MISINFORMATION` (false, no intent) · `MALINFORMATION` (factual core, weaponised out of context) · `PROPAGANDA` · `CONSPIRACY_NARRATIVE` · `IDEOLOGICAL_MASTER_FRAME`

### 2.4 `evidence_level` (attribution strength of the *narrative's* documented origin/amplification) *(report)*
| level | meaning |
|---|---|
| `A` | uncontested; official government / EU sources or multiple independent technical investigations |
| `B` | high probability; peer-reviewed literature or reputable think-tank, transparent methods |
| `C` | probable; investigative journalism with verifiable documents |
| `D` | plausible but contested / single source |
| `E` | highly speculative |

### 2.5 `stance` (how the post relates to the narrative)
`PROMOTES` · `IMPLIES` (dog-whistle / insinuation) · `REPORTS_NEUTRALLY` · `DEBUNKS_OR_CRITICISES` · `SATIRE_OR_UNCLEAR`

### 2.6 `confidence`
`HIGH` (explicit core claim present) · `MEDIUM` (strong implication or typical wording) · `LOW` (topic overlap only → usually choose `NONE`)

### 2.7 `commercial_type` *(project edit)*
| value | meaning | typical cues (German) |
|---|---|---|
| `content` | news, opinion, commentary, calls to act - anything that is not selling, asking for money or promoting the channel | - |
| `product_ad` | promotes or sells a product, service, book, event or discount | "Rabatt", "Code", "jetzt bestellen", "Zum Produkt", "Angebot", prices in €, shop links, "exklusiv bei" |
| `fundraising` | asks for donations, support, membership, crowdfunding | "Spende", "unterstütze uns", "IBAN", "PayPal", "Patreon", "Server eigenfinanzieren" |
| `self_promo` | advertises the channel itself or other channels/platforms; welcome texts; link lists | "abonniere", "folgt uns", "unsere Kanäle", "Newsletter", "Probeabo", "Hier geht es zum Artikel" footers |
A post that mixes editorial content with a trailing footer ("… Kostenlos abonnieren:") is `content`; only classify it as commercial when the commercial part is the post's purpose.

---

## 3. Decision procedure (follow in order)

```
STEP 0  (project edit) Decide commercial_type (§2.7). If it is product_ad, fundraising or self_promo → primary_narrative = NONE,
        unless the editorial text itself asserts a narrative's core claim (then keep commercial_type set). Stop for pure ads.
STEP 1  Is the post about politics, society, war, economy, history, media, health, climate, gender, migration or elections?
        No  → NONE.
STEP 2  Pick the likely CLUSTER (a–h) from §2.2.
STEP 3  Within that cluster, read the narrative cards in §4. For each, ask:
        "Does the post assert or strongly imply the core_claim?"  (not just the topic)
STEP 4  If two cards fit, use §5 (confusion guide) to choose the primary; list the other as secondary.
STEP 5  If the post is a legitimate opinion / news report / debunk with no core claim endorsed → NONE (or set stance = REPORTS_NEUTRALLY / DEBUNKS_OR_CRITICISES with the narrative it discusses).
STEP 6  If a clear disinformation/conspiracy frame is present but matches no card → OTHER (describe in `other_description`).
STEP 7  Fill the JSON in §6. Quote ≤ 25 words of evidence from the post.
```

---

## 4. Narrative cards (the label set)

Format per card: `id` · names · attributes · core claim · mechanism · truth status · carriers · cues · when NOT to assign.

### 4.A Russian-originated

#### N01 — UKRAINE_DENAZIFICATION
- **Names:** EN "Ukraine Denazification" · DE "Entnazifizierung der Ukraine"
- **Attributes:** origin `RUSSIAN` · cluster `c` · type `DISINFORMATION` / `PROPAGANDA` · evidence `A`
- **Carriers *(report)*:** RT DE, Storm-1516, left-nationalists, pro-Russian influencers
- **Core claim:** Ukraine's government/military is structurally controlled by Nazis; Russia's invasion is a legitimate anti-fascist act.
- **Mechanism:** kernel-of-truth inflation (existence of far-right battalions → whole state), exploits German sensitivity to National Socialism.
- **Truth status:** core claim false; Ukraine is a pluralistic, democratically elected state with real but marginal far-right actors.
- **Reach *(report)*:** ~5–10 % of German audience believes the framing (CeMAS).
- **cues_de *(added)*:** "Nazi-Regime in Kiew", "Banderisten", "Asow", "Neonazis in der Ukraine", "Entnazifizierung", "Faschisten in Kiew", Z/V symbols
- **not_this_if:** post merely notes that far-right units exist in Ukraine in a factual/balanced way.

#### N02 — NATO_PROVOCATION
- **Names:** "NATO Provocation" · "NATO-Osterweiterung als Provokation"
- **Attributes:** `RUSSIAN` · `c` · `PROPAGANDA` / frame · `A`
- **Carriers:** AfD, peace-movement milieu, Nachdenkseiten, RT DE remnants
- **Core claim:** NATO's eastward expansion forced Russia into a defensive war; the West is the real aggressor.
- **Mechanism:** whataboutism, victimhood framing; shifts blame from Moscow to Washington/Brussels.
- **Truth status:** highly distorted — NATO enlargement happened via voluntary accession of sovereign states.
- **Reach:** up to ~19 % domestic agreement (CeMAS).
- **cues_de:** "NATO-Provokation", "Russland wurde provoziert", "Ostexpansion", "Stellvertreterkrieg", "der Westen ist schuld", "Versprechen an Gorbatschow"
- **not_this_if:** scholarly/historical discussion of NATO–Russia security dilemma that still assigns responsibility for the 2022 invasion to Russia.

#### N03 — SANCTIONS_BOOMERANG
- **Names:** "Sanctions Boomerang" · "Sanktions-Bumerang"
- **Attributes:** `RUSSIAN` · `c`,`d` · `MALINFORMATION` / `PROPAGANDA` · `A`
- **Carriers:** AfD networks, formerly Compact (banned 2024), Doppelgänger network
- **Core claim:** Western sanctions are destroying Germany's economy while Russia's economy is unaffected.
- **Mechanism:** exploits real economic anxiety (energy prices, inflation, deindustrialisation); scapegoating.
- **Truth status:** distortion — sanctions raised European energy costs, but Russia suffers severe long-term structural damage.
- **cues_de:** "Sanktionen schaden nur uns", "Sanktionen ruinieren Deutschland", "Russland geht es prima", "Wirtschaftskrieg gegen uns selbst", "Gas aus Russland zurück"
- **not_this_if:** economic critique of sanctions design/cost without the "Russia is fine, only we suffer" claim.

#### N04 — BUCHA_DENIAL
- **Names:** "Bucha Denial" · "Butscha-Lüge"
- **Attributes:** `RUSSIAN` · `c` · `DISINFORMATION` · `A`
- **Carriers:** alt-media, Telegram, Querdenken-adjacent conspiracy networks
- **Core claim:** Bucha atrocities were staged (crisis actors, fake corpses) or committed by Ukraine/West.
- **Mechanism:** "firehose of falsehood" — multiple contradictory alternative explanations to paralyse assessment; atrocity denial.
- **Truth status:** entirely false (debunked by UN, forensic journalists, satellite imagery).
- **cues_de:** "Butscha Inszenierung", "Krisenschauspieler", "Leichen bewegen sich", "Fake-Massaker", "Butscha-Lüge", "False Flag Butscha"
- **not_this_if:** post cites Bucha as a verified war crime.

#### N05 — ZELENSKY_HITLER_HOUSE
- **Names:** "Zelensky's Hitler House" · "Hitler-Haus"
- **Attributes:** `RUSSIAN` · `c`,`h` · `DISINFORMATION` · `A`
- **Carriers:** Storm-1516, X, TikTok; shared by some AfD politicians (Feb 2025)
- **Core claim:** Zelensky used Western aid to buy a house once owned by Hitler.
- **Mechanism:** forged documents/fake news sites, bot amplification; reinforces "Ukraine = Nazi" frame.
- **Truth status:** entirely fabricated.
- **cues_de:** "Hitlers Haus gekauft", "Selenskyj Villa", "Zelensky Immobilien", "Berghof", "Steuergelder Villa"
- **not_this_if:** generic corruption allegation without the Hitler-house claim → consider `OTHER`/`NONE`.

#### N06 — DIGITAL_CONCENTRATION_CAMP
- **Names:** "Digital Concentration Camp" · "Digitales KZ"
- **Attributes:** `RUSSIAN` (specific Dec-2024 iteration; broader theme is transnational) · `a` (touches `e`) · `DISINFORMATION` · **evidence `B`**
- **Carriers:** Storm-1516, Querdenken Telegram channels, alt-media proxies
- **Core claim:** The government is secretly building a totalitarian "digital concentration camp" via digital ID, CBDC and health data.
- **Mechanism:** historical relativisation (Holocaust inversion) + technological paranoia.
- **Truth status:** false fabrication exploiting real surveillance concerns.
- **cues_de:** "digitales KZ", "digitale Zwangsjacke", "Bargeldabschaffung", "digitale ID Überwachung", "CBDC Kontrolle", "Social-Credit-System Deutschland", "Impfpass-Kontrolle"
- **confusable_with:** N16 (Great Reset). Use N06 when the *state is building a camp/surveillance cage*; N16 when *WEF/global elites* are the architects.
- **not_this_if:** sober privacy/data-protection criticism of digital ID.

#### N07 — DOPPELGANGER_MEDIA_CLONES
- **Names:** "Faked Media Clones" · "Doppelgänger Fakes"
- **Attributes:** `RUSSIAN` · `a` · `DISINFORMATION` · `A`
- **Carriers:** Social Design Agency (SDA), typo-squatting domains (e.g. spiegel[.]bz), bots, paid Facebook ads, X botnets
- **Core claim (meta-narrative):** Mainstream outlets (SPIEGEL, SZ, Welt…) supposedly publish articles confirming pro-Russian points.
- **Mechanism:** visual deception / institutional mimicry of real media branding.
- **Truth status:** false — the sites are hostile clones.
- **cues_de:** lookalike domains, screenshots of "SPIEGEL/Bild/Welt" articles with pro-Kremlin headlines, "sogar der SPIEGEL gibt zu…", odd TLDs (.bz, .ltd, .pro)
- **not_this_if:** genuine article from a real outlet. Flag as N07 only when the *clone/fake-article* pattern is evident.

### 4.B German-originated (domestic)

#### N08 — SOVEREIGN_STATE_DENIAL
- **Names:** "Sovereign State Denial" · "BRD GmbH"
- **Attributes:** `GERMAN` · `a` · `CONSPIRACY_NARRATIVE` · `A` (domestic)
- **Carriers:** Reichsbürger, Selbstverwalter, esoteric Telegram channels
- **Core claim:** The Federal Republic is not a sovereign state but a corporation under Allied occupation law; the Reich borders (1871/1919) remain valid.
- **Mechanism:** selective bad-faith legal reading (e.g. misuse of a 1973 Constitutional Court ruling); pseudo-legal justification for non-compliance/insurrection (cf. 2022 coup plot).
- **Truth status:** entirely false.
- **cues_de:** "BRD GmbH", "Personal statt Person", "Staatsangehörigkeitsausweis", "Reichsgebiet", "Besatzungsrecht", "kein Friedensvertrag", "Selbstverwalter", "Kaiserreich 1871"
- **not_this_if:** satire; or reporting on Reichsbürger.

#### N09 — LYING_PRESS
- **Names:** "Lying Press" · "Lügenpresse / Systemmedien"
- **Attributes:** `GERMAN` · `a` · `IDEOLOGICAL_MASTER_FRAME` · `B`
- **Carriers:** Pegida, AfD, AUF1, NIUS, Compact (banned)
- **Core claim:** All mainstream media are state-directed propaganda organs that conspire to suppress the truth.
- **Mechanism:** epistemic closure; inoculates audience against fact-checks; positions alt-media as sole arbiter of truth.
- **Truth status:** claim of centrally coordinated conspiracy is false (legitimate media criticism of bias/groupthink remains valid).
- **cues_de:** "Lügenpresse", "Systemmedien", "Staatsfunk", "GEZ-Propaganda", "Lückenpresse", "Mainstream lügt", "gleichgeschaltete Medien"
- **not_this_if:** specific, evidenced critique of an article/outlet.

#### N10 — GREAT_REPLACEMENT
- **Names:** "Great Replacement" · "Großer Austausch / Remigration"
- **Attributes:** `GERMAN (Local)` (concept from France, Renaud Camus 2011) · `b` · `CONSPIRACY_NARRATIVE` · `B`
- **Carriers:** Identitarian Movement (DE/AT), Neue Rechte, Institut für Staatspolitik, radical AfD factions
- **Core claim:** Elites intentionally replace white European populations with non-European migrants; the "solution" is forced mass "Remigration".
- **Mechanism:** existential demographic panic; scapegoating.
- **Truth status:** demographic change is real; coordinated elite plot is false.
- **cues_de:** "Großer Austausch", "Bevölkerungsaustausch", "Umvolkung", "Remigration", "Volkstod", "Messermigration", "Umsiedlung der Deutschen"
- **confusable_with:** N20 (single atrocity story), N18 (antisemitic "who is behind it"). Use N10 when the post claims a *planned demographic engineering*.
- **not_this_if:** discussion of migration policy or integration problems without the "planned replacement" claim.

#### N11 — COALITION_DICTATORSHIP
- **Names:** "Coalition Dictatorship" · "Ampel-Diktatur / Merkel-Diktatur / Corona-Diktatur"
- **Attributes:** `GERMAN` · `a` · `MALINFORMATION` · `B`
- **Carriers:** Querdenken milieu, AfD, Compact, alt-media
- **Core claim:** Democratically elected German governments are de facto dictatorships suppressing opinion and civil liberties.
- **Mechanism:** concept creep / semantic inversion (pluralist democracy relabelled totalitarianism); justifies radical resistance.
- **Truth status:** false — Germany is a functioning constitutional democracy (dissatisfaction with policies is legitimate).
- **cues_de:** "Ampel-Diktatur", "Merkel-Regime", "Corona-Diktatur", "Meinungsdiktatur", "Gesinnungsdiktatur", "Widerstand", "DDR 2.0", "Faeser-Regime"
- **confusable_with:** N14 (cartel parties), N09 (media). N11 = *government as dictatorship*.
- **not_this_if:** criticism of specific laws (e.g. NetzDG) in measured terms.

#### N12 — CLIMATE_EXPROPRIATION
- **Names:** "Climate Expropriation" · "Heizungsgesetz will enteignen"
- **Attributes:** `GERMAN` · `d` · `MALINFORMATION` / `PROPAGANDA` · **`C`**
- **Carriers:** Tichys Einblick, NIUS, AfD, tabloid amplification
- **Core claim:** Green energy policy (Building Energy Act/GEG) is a deliberate plan to bankrupt and expropriate the middle class.
- **Mechanism:** slippery slope + hyperbole, weaponises economic anxiety.
- **Truth status:** distorted — the law had costly mandates (legitimate debate), but "communist expropriation plot" is malinformation.
- **cues_de:** "Heizungsgesetz", "Heizungshammer", "Enteignung", "Wärmepumpen-Zwang", "Öko-Diktatur Hauseigentümer", "Habecks Enteignungsgesetz"
- **confusable_with:** N19 (Climate Dictatorship), N25 (Deindustrialisation). N12 = *housing/heating law specifically*.
- **not_this_if:** substantive cost/feasibility critique of the GEG.

#### N13 — GUILT_CULT
- **Names:** "Guilt Cult" · "Schuldkult"
- **Attributes:** `GERMAN` · `f` · `IDEOLOGICAL_MASTER_FRAME` · `B`
- **Carriers:** far-right, Neue Rechte, some AfD politicians (e.g. "Denkmal der Schande" rhetoric)
- **Core claim:** Holocaust remembrance culture is an imposed psychological weapon that keeps Germany subjugated and blocks healthy national identity.
- **Mechanism:** national narcissism, historical inversion; revisionism.
- **Truth status:** ideological falsehood; remembrance culture is internally developed.
- **cues_de:** "Schuldkult", "Denkmal der Schande", "Vogelschiss", "Erinnerungskultur beenden", "180-Grad-Wende", "Dauerbüßer", "Kollektivschuld"
- **not_this_if:** academic debate on memory-politics design.

#### N14 — CARTEL_PARTIES
- **Names:** "Cartel Parties" · "Altparteienkartell"
- **Attributes:** `GERMAN` · `a` · `IDEOLOGICAL_MASTER_FRAME` · `B`
- **Carriers:** AfD, Compact, AUF1, broader alt-media
- **Core claim:** CDU/SPD/Greens/FDP are one indistinguishable corrupt cartel against the "German people"; AfD is the sole authentic alternative.
- **Mechanism:** false equivalence; "pure people vs corrupt elite" dichotomy.
- **Truth status:** distorted (parties have distinct platforms and voters).
- **cues_de:** "Altparteien", "Blockparteien", "Kartellparteien", "Systemparteien", "Einheitspartei", "die da oben"
- **not_this_if:** criticism of one party or of a specific coalition compromise.

#### N15 — AFD_BAN_DEMOCRACY_DEMISE
- **Names:** "AfD Ban as Democracy Demise" · "AfD-Verbot ist Demokratie-Abbau"
- **Attributes:** `GERMAN` · `a` · `PROPAGANDA` · `B`
- **Carriers:** AfD, right-wing influencers on X/Telegram
- **Core claim:** Cartel parties want to ban the AfD merely to eliminate competition and thereby abolish democracy.
- **Mechanism:** victim/perpetrator reversal; recasts "streitbare Demokratie" (Art. 21 GG) as totalitarianism.
- **Truth status:** distorted — Art. 21 GG allows banning anti-constitutional parties; whether to use it is a legitimately contested question.
- **cues_de:** "AfD-Verbot", "Verbotsantrag", "Demokratie abschaffen", "Oppositionsverbot", "Zensur der Opposition", "Stasi-Methoden gegen AfD"
- **not_this_if:** legal/strategic arguments for or against a ban without the "abolishing democracy" frame.

### 4.C Transnational / imported

#### N16 — GREAT_RESET
- **Names:** "The Great Reset"
- **Attributes:** `TRANSNATIONAL` · `h`,(`e`) · `CONSPIRACY_NARRATIVE` · `B`
- **Carriers:** Querdenken, AUF1, alternative-medicine networks
- **Core claim:** WEF/Klaus Schwab used COVID-19 to install a totalitarian global government that strips property and freedom.
- **Mechanism:** apophenia, elite panic; unites anti-vax, anti-climate, anti-government camps.
- **Truth status:** false — WEF is a lobbying forum without executive power/intent.
- **cues_de:** "Great Reset", "Klaus Schwab", "WEF-Agenda 2030", "Du wirst nichts besitzen und glücklich sein", "Pandemieplan der Eliten", "Plandemie"
- **confusable_with:** N06, N19, N25, N18.

#### N17 — GENDER_MADNESS
- **Names:** "Gender Madness" · "Genderwahn / Woke Mind Virus"
- **Attributes:** `TRANSNATIONAL` · `g` · `IDEOLOGICAL_MASTER_FRAME` · `B`
- **Carriers:** Junge Freiheit, NIUS, AfD, Russian state media ("traditional values")
- **Core claim:** Woke/LGBTQ+/gender studies are elite tools to destroy the family, feminise men and indoctrinate children (*Frühsexualisierung*).
- **Mechanism:** moral panic, disgust elicitation.
- **Truth status:** distorted — debates on gender pedagogy/Gendern are legitimate; the elite-plot claim is an ideological frame.
- **cues_de:** "Genderwahn", "Gender-Gaga", "Frühsexualisierung", "Umerziehung der Kinder", "Woke-Wahn", "Trans-Hype", "Regenbogen-Propaganda", "Traditionelle Familie zerstören"
- **not_this_if:** measured debate on language rules, school curricula or medical guidelines.

#### N18 — GLOBALISTS_OSTKUESTE_SOROS
- **Names:** "Globalists" · "Die Ostküste / Soros"
- **Attributes:** `TRANSNATIONAL` · `h` · `CONSPIRACY_NARRATIVE` (structurally antisemitic) · `B`
- **Carriers:** Reichsbürger, Querdenken, far-right Telegram; QAnon adaptations
- **Core claim:** A hidden cabal of "globalists" (coded: Jewish elites, Soros, Rothschild) controls finance, media and politics.
- **Mechanism:** personified scapegoat; dog-whistling (antisemitic tropes without naming Jews).
- **Truth status:** entirely false.
- **cues_de:** "Globalisten", "Ostküste", "Soros", "Rothschild", "Hintergrundmächte", "Weltfinanz", "Eliten steuern alles", "Kabale", "Khazar", "(((echoes)))", 🐍
- **confusable_with:** N16 (WEF framing, usually without ethnic code). Prefer N18 when coded antisemitic markers are present.
- **not_this_if:** legitimate criticism of a named donor/lobbying activity without cabal/control framing.

#### N19 — CLIMATE_DICTATORSHIP
- **Names:** "Climate Dictatorship / Climate Hoax" · "Klimawahn / Klimadiktatur"
- **Attributes:** `TRANSNATIONAL` · `d` · `CONSPIRACY_NARRATIVE` · `B`
- **Carriers:** AfD, AUF1, libertarian fringe, fossil-fuel-funded lobbying roots
- **Core claim:** Climate change is a hoax/exaggeration by globalists to enforce a "climate dictatorship" (15-minute cities, meat bans, etc.).
- **Mechanism:** denialism + reactance.
- **Truth status:** false — anthropogenic climate change is scientific consensus; 15-minute cities are urban planning, not lockdowns.
- **cues_de:** "Klimalüge", "Klimawahn", "Klimadiktatur", "Klima-RAF", "CO2-Lüge", "15-Minuten-Stadt Gefängnis", "Fleischverbot", "Öko-Faschismus"
- **confusable_with:** N12 (heating law). Use N19 when the claim is *global hoax/dictatorship*.

### 4.D Hybrid / laundered

#### N20 — MIGRANT_ATROCITY_FAKES
- **Names:** "Migrant Atrocity Fakes" · "Fall Lisa → Euskirchen-Fakes (2025)"
- **Attributes:** `HYBRID` · `b` · `DISINFORMATION` · `A`
- **Carriers:** Doppelgänger clones, Russian embassies, local AfD chapters, Telegram
- **Core claim:** Migrants/refugees commit brutal, unprovoked crimes and state/media cover it up (specific fabricated incidents).
- **Mechanism:** atrocity propaganda, emotional hijacking; prototype Lisa Case (2016), replicated 2022 (Euskirchen), 2025 (Storm-1516 "Islamist murder" fake).
- **Truth status:** the specific viral stories were verified fabrications (migrant crime as a topic is legitimate).
- **cues_de:** "vertuscht", "Medien schweigen", "Polizei verschweigt", "Vergewaltigung durch Flüchtlinge" + unverifiable details, "Lisa", "Ukrainer Flüchtling schlägt", viral gory images w/o source
- **not_this_if:** verified crime reporting or a policy debate on crime statistics.

#### N21 — ELECTION_FRAUD_2025
- **Names:** "2025 Election Fraud" · "Wahlbetrug"
- **Attributes:** `HYBRID` · `a` · `DISINFORMATION` · `A`
- **Carriers:** Storm-1516 (AI videos of ballot stuffing), AfD influencers, X, TikTok
- **Core claim:** The Feb-2025 federal election was rigged against the AfD (postal votes, systemic fraud); a "fake re-run".
- **Mechanism:** US "Stop the Steal" playbook + AI deepfakes; pre-bunking democratic outcomes (alleging fraud before the vote).
- **Truth status:** false — decentralised, secure system; viral videos proven AI fabrications.
- **cues_de:** "Wahlbetrug", "Briefwahl manipuliert", "Wahlfälschung", "Neuwahl erzwingen", "Stimmen gestohlen", "gefälschte Stimmzettel", "Wahl wiederholen"
- **not_this_if:** neutral reporting on actual irregularities investigated by authorities (e.g. recount petitions).

#### N22 — PEACE_MOVEMENT_FRAME
- **Names:** "Peace Movement" · "Friedensbewegung / Diplomatie statt Waffen"
- **Attributes:** `HYBRID` · `c` · `MALINFORMATION` / `PROPAGANDA` · **`C`**
- **Carriers:** BSW, AfD, Overton Magazin, RT DE, left-nationalists
- **Core claim:** German arms deliveries to Ukraine are warmongering; ending all military aid = "peace".
- **Mechanism:** semantic manipulation — redefining capitulation as peace; omits that aid cessation without Russian withdrawal means occupation.
- **Truth status:** highly distorted; desire for peace itself is legitimate.
- **cues_de:** "Diplomatie statt Waffen", "Kriegstreiber", "Waffenlieferungen stoppen", "Friedensverhandlungen jetzt", "Ukraine soll aufgeben", "Rüstungslobby"
- **not_this_if:** good-faith advocacy for negotiations that does not blame Western aid as the cause of the war.

#### N23 — US_VASSAL_STATE
- **Names:** "US Vassal State" · "Deutschland als US-Kolonie"
- **Attributes:** `HYBRID` · `c` (geopolitics) · `CONSPIRACY_NARRATIVE` · `B`
- **Carriers:** Compact, Reichsbürger networks, RT DE, Nachdenkseiten
- **Core claim:** Germany is a puppet of the USA, forced into policies (Ukraine aid, energy embargo) against its own interests.
- **Mechanism:** national humiliation frame; merges Cold-War left anti-imperialism with Reichsbürger ideology and Russian multipolar propaganda.
- **Truth status:** false — Germany is sovereign within NATO.
- **cues_de:** "US-Kolonie", "Vasall", "Besatzungsmacht USA", "Ramstein", "Amerikas Statthalter", "transatlantische Netzwerke steuern", "Souveränität zurück"
- **confusable_with:** N08 (BRD GmbH — legal-status claim) and N24.

#### N24 — NORD_STREAM_US_BLAME
- **Names:** "Nord Stream US Blame" · "Nord-Stream-Sabotage durch die USA"
- **Attributes:** `HYBRID` · `c` (geopolitics) · `PROPAGANDA` / speculative · **`C`**
- **Carriers:** AfD, left-nationalist media, Russian diplomatic accounts
- **Core claim:** The USA (often citing Seymour Hersh) blew up Nord Stream to force Germany onto expensive US LNG.
- **Mechanism:** investigative vacuum + "cui bono?" reductionism.
- **Truth status:** speculative/contested; official investigations pointed toward a Ukrainian sabotage group; US-involvement claims lack verified evidence.
- **cues_de:** "Nord Stream", "Hersh", "Biden hat gesprengt", "Cui bono", "LNG-Abhängigkeit", "Terror gegen Verbündete"
- **not_this_if:** neutral reporting on the investigation's status.

#### N25 — INTENTIONAL_DEINDUSTRIALIZATION
- **Names:** "Intentional Deindustrialization" · "Deindustrialisierung als Absicht"
- **Attributes:** `HYBRID` · `d` · `CONSPIRACY_NARRATIVE` · `B`
- **Carriers:** Compact, AUF1, AfD economic spokespersons, alt-media
- **Core claim:** Government/global elites are deliberately destroying German industry (for a green agenda or to weaken the nation).
- **Mechanism:** intentionality bias; turns real structural/energy-cost problems into "treason".
- **Truth status:** false as intent claim — Germany faces real structural economic challenges.
- **cues_de:** "Deindustrialisierung gewollt", "Wirtschaft absichtlich ruinieren", "Morgenthau-Plan", "Verrat am Standort", "Grüne zerstören Industrie", "bewusst abgewickelt"
- **confusable_with:** N03, N12, N16.
- **not_this_if:** economic analysis of competitiveness or energy prices.

### 4.E Special labels
| id | use when |
|---|---|
| `NONE` | no narrative's core claim is asserted/implied; includes legitimate political opinion, ordinary news, **ads / fundraising / channel promotion (see `commercial_type`)**, spam, off-topic |
| `OTHER` | a clear manipulative/conspiratorial frame is present but not in N01–N25 (fill `other_description`; e.g. new COVID/vaccine claims, other conspiracy, antisemitism not fitting N18) |

---

## 5. Confusion guide (pick the primary)

| If the post is mainly about… | choose | not |
|---|---|---|
| Germany is not a real state / GmbH / Reich borders | **N08** | N23 |
| Germany is a US puppet (policy, Ramstein, sanctions) | **N23** | N08 |
| Government = dictatorship / censorship | **N11** | N14, N09 |
| All parties are one cartel | **N14** | N11 |
| AfD ban = end of democracy | **N15** | N14, N11 |
| Media lie / are state-controlled | **N09** | N07 |
| A fake clone of a real outlet / screenshot of fake headline | **N07** | N09 |
| WEF/Schwab/pandemic plan | **N16** | N18 |
| Coded "globalist/Soros/Ostküste/Rothschild" cabal | **N18** | N16 |
| Digital ID / CBDC / "digital KZ" | **N06** | N16 |
| Planned demographic replacement / remigration | **N10** | N20 |
| Specific cover-up story of a migrant crime | **N20** | N10 |
| Heating law = expropriation | **N12** | N19 |
| Climate = hoax / dictatorship | **N19** | N12 |
| Government wants to destroy industry on purpose | **N25** | N03, N12 |
| Sanctions only hurt us, Russia fine | **N03** | N25 |
| Ukraine is Nazi | **N01** | N05 |
| Specific Zelensky–Hitler house claim | **N05** | N01 |
| NATO caused the war | **N02** | N22 |
| Stop weapons = peace | **N22** | N02 |
| Bucha/other atrocity is staged | **N04** | N01 |
| Nord Stream was a US attack | **N24** | N23 |
| Election rigged | **N21** | N11 |
| Holocaust remembrance = guilt cult | **N13** | — |
| Gender/LGBT = elite plot against family | **N17** | — |

---

## 6. Output schema (JSON)

```json
{
  "post_language": "de | en | ru | mixed | other",
  "commercial_type": "content | product_ad | fundraising | self_promo",
  "primary_narrative": "N01..N25 | NONE | OTHER",
  "secondary_narratives": ["N.."],
  "cluster": "a|b|c|d|e|f|g|h|none",
  "stance": "PROMOTES | IMPLIES | REPORTS_NEUTRALLY | DEBUNKS_OR_CRITICISES | SATIRE_OR_UNCLEAR",
  "confidence": "HIGH | MEDIUM | LOW",
  "evidence_quote": "<=25 words copied from the post",
  "reasoning": "1-2 sentences naming the core claim that was matched (or why NONE)",
  "legitimate_debate_flag": true,
  "other_description": "only if primary_narrative == OTHER, else null"
}
```
Rules: valid JSON only; `commercial_type` is decided first; if it is not `content`, `primary_narrative` must be `NONE` unless the editorial text itself asserts a core claim; `secondary_narratives` max 2; if `primary_narrative` = `NONE` then `secondary_narratives` = `[]`. Enforce with a JSON-schema / GBNF grammar (see §9).

---

## 7. Few-shot examples (*synthetic, written by me for illustration*)

**Ex. 1**
Post: "Der Spiegel gibt zu: Sanktionen zerstören Deutschland, Russland boomt! 👉 spiegel[.]bz/wirtschaft/…"
```json
{"post_language":"de","primary_narrative":"N07","secondary_narratives":["N03"],"cluster":"a","stance":"PROMOTES","confidence":"HIGH","evidence_quote":"Der Spiegel gibt zu: Sanktionen zerstören Deutschland, Russland boomt! spiegel[.]bz","reasoning":"Lookalike domain imitating SPIEGEL carrying the 'sanctions only hurt us' claim (clone pattern + N03 content).","legitimate_debate_flag":false,"other_description":null}
```

**Ex. 2**
Post: "Die Heizungspläne der Grünen sind sozialistische Enteignung auf Raten. Erst die Gasheizung, dann das Haus."
```json
{"post_language":"de","primary_narrative":"N12","secondary_narratives":[],"cluster":"d","stance":"PROMOTES","confidence":"HIGH","evidence_quote":"sozialistische Enteignung auf Raten. Erst die Gasheizung, dann das Haus","reasoning":"Frames the heating law as deliberate expropriation (core claim of N12).","legitimate_debate_flag":true,"other_description":null}
```

**Ex. 3**
Post: "Das GEG ist handwerklich schlecht gemacht, die Förderung kommt zu spät und die Kosten für Mieter sind unklar. Hier die Zahlen des IW."
```json
{"post_language":"de","primary_narrative":"NONE","secondary_narratives":[],"cluster":"d","stance":"REPORTS_NEUTRALLY","confidence":"HIGH","evidence_quote":"Das GEG ist handwerklich schlecht gemacht, die Förderung kommt zu spät","reasoning":"Substantive policy criticism; no claim of deliberate expropriation plot.","legitimate_debate_flag":true,"other_description":null}
```

**Ex. 4**
Post: "Wer steckt wirklich hinter Migration, Gender und Klimaagenda? Soros & Co. Die Ostküste lacht sich tot. 🐍"
```json
{"post_language":"de","primary_narrative":"N18","secondary_narratives":["N10","N17"],"cluster":"h","stance":"PROMOTES","confidence":"HIGH","evidence_quote":"Soros & Co. Die Ostküste lacht sich tot. 🐍","reasoning":"Coded 'Ostküste/Soros' cabal steering migration/gender/climate — antisemitic conspiracy frame.","legitimate_debate_flag":false,"other_description":null}
```

**Ex. 5**
Post: "Faktencheck: Die Behauptung, Selenskyj habe Hitlers Haus gekauft, stammt aus einem gefälschten Dokument und wurde von Storm-1516 verbreitet."
```json
{"post_language":"de","primary_narrative":"N05","secondary_narratives":[],"cluster":"c","stance":"DEBUNKS_OR_CRITICISES","confidence":"HIGH","evidence_quote":"Die Behauptung, Selenskyj habe Hitlers Haus gekauft, stammt aus einem gefälschten Dokument","reasoning":"Post discusses the Hitler-house narrative but debunks it; stance is DEBUNKS.","legitimate_debate_flag":false,"other_description":null}
```

**Ex. 6**
Post: "Bundestagswahl 20:00 Uhr: Hier die Hochrechnung und die Koalitionsoptionen. ☕"
```json
{"post_language":"de","primary_narrative":"NONE","secondary_narratives":[],"cluster":"none","stance":"REPORTS_NEUTRALLY","confidence":"HIGH","evidence_quote":"Hier die Hochrechnung und die Koalitionsoptionen","reasoning":"Ordinary election news; no fraud claim.","legitimate_debate_flag":false,"other_description":null}
```

**Ex. 7** *(project edit - product ad)*
Post: "🚨 STÄRKEN SIE IHRE LUNGE! 💨 SPIROSOL stoppt Reizhusten & Schleim. 🎁 Code LUNGE30 = -30 % Rabatt. Zum Produkt - EXKLUSIV bei Heilnatura"
```json
{"post_language":"de","commercial_type":"product_ad","primary_narrative":"NONE","secondary_narratives":[],"cluster":"none","stance":"REPORTS_NEUTRALLY","confidence":"HIGH","evidence_quote":"Code LUNGE30 = -30 % Rabatt. Zum Produkt - EXKLUSIV bei Heilnatura","reasoning":"Sales pitch with discount code; no narrative claim is asserted.","legitimate_debate_flag":false,"other_description":null}
```

**Ex. 8** *(project edit - fundraising)*
Post: "Liebe Zuschauer unseres Kanals! Da wir uns die Server für unser Projekt eigenfinanzieren, würden wir uns über eine kleine Spende freuen. PayPal: …"
```json
{"post_language":"de","commercial_type":"fundraising","primary_narrative":"NONE","secondary_narratives":[],"cluster":"none","stance":"REPORTS_NEUTRALLY","confidence":"HIGH","evidence_quote":"würden wir uns über eine kleine Spende freuen","reasoning":"Donation appeal; no narrative.","legitimate_debate_flag":false,"other_description":null}
```

**Ex. 9** *(project edit - ad that also asserts a narrative)*
Post: "Die Eliten wollen uns mit dem Great Reset alles nehmen – du wirst nichts besitzen. Sichere dich jetzt ab: Goldmünzen mit 15 % Rabatt, nur heute!"
```json
{"post_language":"de","commercial_type":"product_ad","primary_narrative":"N16","secondary_narratives":[],"cluster":"h","stance":"PROMOTES","confidence":"MEDIUM","evidence_quote":"Die Eliten wollen uns mit dem Great Reset alles nehmen – du wirst nichts besitzen","reasoning":"Editorial hook asserts the Great Reset core claim before the sales pitch; keep commercial_type so the post can be filtered.","legitimate_debate_flag":false,"other_description":null}
```

**Ex. 10** *(project edit - channel promotion)*
Post: "📇 NEWSTICKER 📇 Du bist an weiteren Nachrichten interessiert? Dann folge unseren Kanälen: 👉 Hauptkanal 👉 Archiv 👉 Newsletter"
```json
{"post_language":"de","commercial_type":"self_promo","primary_narrative":"NONE","secondary_narratives":[],"cluster":"none","stance":"REPORTS_NEUTRALLY","confidence":"HIGH","evidence_quote":"Dann folge unseren Kanälen","reasoning":"Channel promotion only.","legitimate_debate_flag":false,"other_description":null}
```

---

## 8. Ready-to-paste system prompt (compact)

```
You are a classifier of German-language fringe/alt-media and Telegram posts.
Task: assign narrative labels from a CLOSED list (N01–N25, NONE, OTHER) using the knowledge pack below.
Rules: (0) FIRST set commercial_type (content | product_ad | fundraising | self_promo); ads, fundraising and channel promotion get primary_narrative=NONE unless the editorial text itself asserts a narrative core claim; (1) classify the CONTENT THEME, never the author or origin of this specific post; (2) a label needs the narrative's CORE CLAIM asserted or strongly implied — topic overlap alone → NONE; (3) legitimate criticism (energy costs, migration policy, Ukraine strategy, media critique, AfD-ban debate, gender pedagogy) → NONE with legitimate_debate_flag=true; (4) set stance=DEBUNKS_OR_CRITICISES for fact-checks/reporting; (5) 1 primary + max 2 secondary; (6) output ONLY the JSON schema, evidence_quote ≤25 words copied from the post.
[INSERT §4 narrative cards (shortened: id, core_claim, cues_de, not_this_if) + §5 confusion guide + §6 schema]
```
*Token budget tip:* a shortened card set (id + core_claim + 6 cues + not_this_if) is ≈ 3–4k tokens; keep it as a static prefix so llama.cpp / vLLM can cache it (prompt-prefix caching).

---

## 9. Model recommendation — 16 GB VRAM, German Telegram/fringe-media classification

**Caveat:** I could not find a trustworthy German-specific benchmark for fringe/Telegram text. The recommendations below combine web sources from 2026 (several are SEO-style blogs; treat their numbers as indicative) with general model properties. **Run a 100–300-post labelled pilot (your own gold set) before committing.**

### 9.1 Requirements for this task
1. Strong **German** (slang, dialect, sarcasm, dog-whistles, Denglisch).
2. Reliable **structured output** (JSON via grammar/guided decoding).
3. Long context headroom for a ~4k-token label prompt + few-shots (+ thread context).
4. **Throughput** — thousands of posts; dense ≤14B models at Q6–Q8 are usually faster and more stable than a 27B squeezed into 16 GB.
5. **Refusal-robustness** — some safety-tuned models refuse or sanitise extremist text; test this.

### 9.2 Shortlist
| rank | model (quant) | why | risk |
|---|---|---|---|
| **1 (default)** | **Gemma 4 12B-it (QAT or Q6_K/Q8_0)** | Reported strong multilingual quality; small footprint (~7–10 GB) leaves plenty of room for long context and few-shots; Gemma 2/3 family had a good German reputation | multimodal extras unused; check KV-cache usage with long contexts |
| **2** | **Qwen3.5-9B (Q8_0) or Qwen3-14B (Q4_K_M–Q6_K)** | Strong instruction following + JSON; multilingual; Apache-2.0; very fast; 14B Qwen3 reported to run in ~9–11 GB | German nuance slightly below Gemma in some community reports; thinking mode must be disabled/controlled for speed |
| **3 (quality push)** | **Qwen3.5-27B (IQ4_XS) or Gemma 4 26B-A4B (IQ4_XS ≈ 14 GB)** | Larger models can capture subtle dog-whistles/sarcasm better | Tight fit: context limited (≈18k tokens OK in one benchmark, 64k degrades), slower, speed varies strongly by GPU (one report: ~11 tok/s for 26B-A4B on a 16 GB 5060 Ti) |
| 4 | Mistral Small 3.x 24B (Q4_K_M ≈ 14 GB) | Good European-language coverage, Apache-2.0 | very little context headroom on 16 GB |
| 5 | gpt-oss-20b (≈12.8 GiB native) | Good reasoning/tool use | little headroom; German on fringe slang unproven; refusals possible |

**My recommendation:** start with **Gemma 4 12B** at Q6/Q8 as the production classifier; run **Qwen3.5-9B/Qwen3-14B** as a cross-check (agreement = high-confidence; disagreement → review queue); only move to a 26–27B IQ4 model if the pilot shows it clearly beats the 12B on your gold set.

### 9.3 Practical setup
- **Runtime:** `llama.cpp` (server) or `vLLM`/`SGLang` if your GPU/OS supports it; Ollama works but gives less control over grammars.
- **Constrain output:** JSON-schema / GBNF grammar so the label can only be one of the 27 enums.
- **Temperature 0**, short `max_tokens` (~200), disable "thinking" mode for Qwen3/3.5 for throughput.
- **Two-stage option (cheaper & more accurate):** Stage 1 = cluster `a–h`/`none` with a 1-paragraph prompt; Stage 2 = choose among only that cluster's cards (3–8 options).
- **Alternative/complement:** embed posts (e.g. a multilingual embedding model) → nearest-narrative retrieval of 5 candidate cards → LLM picks. This scales to thousands of posts and keeps prompts short.
- **Context:** include reply-chain/forward-source if available — forwarded Telegram posts often carry the narrative signal.
- **Evaluation:** macro-F1 on `primary_narrative`, plus a separate NONE-vs-hit precision check (over-labelling legitimate debate is the main failure mode).
- **Fine-tuning option:** a LoRA on a 9–14B model with 500–2,000 labelled German posts usually beats prompt-only approaches for subtle classes.

### 9.4 Known limitations
- The taxonomy comes from one report; labels are policy-/research-framed and some (N12, N22, N24 = evidence level C) are explicitly contested border cases. The report itself warns against over-labelling legitimate grievances as disinformation.
- Attribution levels refer to the narrative's documented lineage, **not** to individual posts.
- Narratives evolve; revisit the label set every few months.

---

## 10. Commercial-post filter *(project edit)*

Why: a large share of posts in some channels are repeated ads (water filters, heaters, supplements, books), fundraising blocks and channel promotion ("Aktuelle RT-DE Nachrichten…", "Kostenlos abonnieren:"). They distort narrative counts, topics, entities and sentiment.

Three layers, cheapest first:
1. **Corpus flags (no LLM):** `is_boilerplate` (the same text >= 20 times in a channel) and the footer patterns in `config/clean_patterns.txt` (removed from `clean_text`).
2. **Rule pre-filter (no LLM):** posts whose cleaned text is mostly commercial cues (>= 2 of: Rabatt, Code, jetzt bestellen, Zum Produkt, Angebot, exklusiv bei, Spende, IBAN, PayPal, abonniere, folgt uns, unsere Kanäle, Probeabo, Newsletter, price in €) are labelled `commercial_type` by rule and skip the LLM (`tg-narratives --prefilter`).
3. **LLM field `commercial_type` (§2.7):** the model decides the rest; downstream analyses keep only `commercial_type == content` unless the commercial post asserts a narrative (then filter by `primary_narrative` and inspect manually).

Validation: label 150 posts by hand for `commercial_type`; commercial should reach high agreement (> 0.9 macro-F1 is realistic); if not, add the missed cues to layer 2.
