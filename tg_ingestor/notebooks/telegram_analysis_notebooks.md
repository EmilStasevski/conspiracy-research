# Telegram Channel Dynamics & Discourse Analysis (2020–2026)

This repository contains Jupyter notebooks, data processing pipelines, and analytical findings examining post volume, audience reactions, and narrative shifts across German-language alternative Telegram channels from **January 1, 2020 to October 21, 2026**.

## 📂 Notebooks Overview

| Notebook | Focus Area | Description | 
| ----- | ----- | ----- | 
| `01_corpus_diagnostics.ipynb` | **Data Hygiene & Corpus Validation** | Channel footprints, missing channel audits, text vs. media shifts, forward topologies, and scrape-gap checks. | 
| `02_reactions_and_emojis.ipynb` | **Engagement & Sentiment Artifacts** | Channel reaction constraints, hostile vs. disapproving engagement, emoji codebook cleanup, and audience scale. | 
| `03_period_and_trajectory.ipynb` | **Temporal Trends & Trajectories** | Longitudinal activity across 8 distinct periods, channel lifecycle shifts, and panel modeling with fixed effects. | 
| `04_hypothesis_testing.ipynb` | **Pre-registered Event & Topic Tests** | Event studies (Leipzig, Reichstag), narrative substitution (COVID to Ukraine), and keyword-based tracking. | 

## 1. Corpus Diagnostics (`01_corpus_diagnostics.ipynb`)

### Dataset at a Glance

* **Total volume:** 869,307 posts across 19 (out of 21 planned) channels.
* **Coverage:** Jan 1, 2020 – Oct 21, 2026.
* **Power-law distribution:** Five channels generate \~70% of the entire corpus:
  * Eva Herman (\~224k)
  * Oliver Janich (\~149k)
  * Druschba FM (\~84k)
  * Epoch Times (\~78k)
  * satellit_de (\~71k)

### Channel Directory & Metadata

Below is the directory of tracked channels, their geopolitical orientation, and the background of their operators. *(Note: Subscriber counts reflect approximate parsed totals at the end of the scrape).*

| Channel / Handle | Est. Subs | Origin / Type | Background / Lead Context |
| :--- | :--- | :--- | :--- |
| **`auf1tv`** | ~240k | Pure German (AT) | Austrian right-wing alternative media network (Stefan Magnet). |
| **`EvaHermanOffiziell`** | ~200k | Pure German | Former German TV host Eva Herman, now operating from Canada. |
| **`neuesausrussland`** | ~190k | Mixed (GER / RU) | Alina Lipp, German influencer living in Russia/Donbas; pro-Russian war coverage. |
| **`AntiSpiegel`** | ~105k | Mixed (GER / RU) | Thomas Röper, German blogger living in St. Petersburg; pro-Russian. |
| **`oliverjanich`** | ~145k | Pure German | Oliver Janich, libertarian/conspiracy author operating from the Philippines. |
| **`attilahildmann`** | ~50k | Pure German | Former vegan chef turned extremist, hiding in Turkey (highly banned/censored). |
| **`RT_DE`** / `rt_de_live` | ~600k+ | Pure Russian | Russian state media. Banned in EU, relies heavily on mirror channels. |
| **`satellit_de`** | ~71k | Pure Russian | Successor channel to SNA/Sputnik Germany (Russian state media). |
| **`CompactMagazin`** | ~85k | Pure German | Right-wing magazine led by Jürgen Elsässer (faces state bans). |
| **`freiesachsen`** | ~150k | Pure German | Right-wing extremist regional party mobilizing in Saxony. |
| **`kenjebsen`** / `apolut` | ~120k | Pure German | Founded by Ken Jebsen, later transitioned to the "Apolut" network. |
| **`dr_reiner_fuellmich`**<br>`icic_law_official` | ~75k | Pure German | Reiner Füllmich (Corona Committee lawyer), arrested late 2023. |
| **`AllesAusserMainstream`** | ~130k | Pure German | Bodo Schiffmann, former German ENT doctor now operating from Tanzania. |
| **`RKI_Files`** | ~65k | Pure German | Single-issue channel focused on leaked German COVID (RKI) protocols. |
| **`QlobalChange`** | ~120k | Mixed (GER / US) | Primary German translation hub for US-originated QAnon narratives. |
| **`NachDenkSeiten_DE`** | ~110k | Pure German | Alternative left-wing blog and podcast network. |
| **`epochtimesde`** | ~35k | Mixed (GER / INTL) | German branch of the Falun Gong-affiliated international network. |
| **`DruschbaFM`** | ~55k | Mixed (GER / RU) | German-Russian "friendship" channel amplifying Russian state narratives. |

### Key Dynamics & Methodological Caveats

* **Format Shift (Media over Text):** Overall post frequency per active channel peaked in Q4 2020 (\~34 posts/day) and settled at roughly 18 posts/day from 2024 onward. Concurrently, purely visual/media posts doubled from 5–7% in 2020 to \~13% from 2022 onward.
* **Forwarding Behaviors:** \~21% of Eva Herman’s output consists of self-forwards. Janich heavily amplifies external actors, providing an ideal anchor for a network diffusion analysis.
* **Data Gaps to Audit:** Three channels show extended quiet periods (`icic_law` 510 days; `Fuellmich` 213 days; `RKI_Files` missing intervals). *Action item:* Cross-check whether these gaps are real real-world channel pauses (e.g., Füllmich's arrest) or scraper failures.

## 2. Reactions & Sentiment (`02_reactions_and_emojis.ipynb`)

Reactions in Telegram carry distinct affordances and structural constraints that complicate straightforward sentiment analysis.

* **Reactions Are Not Universally Available:** 15 of 19 channels had reactions active. Reactions remained permanently disabled on *AllesAusserMainstream*, *NachDenkSeiten*, *RKI_Files*, and *kenjebsen*.
* **Admin-Curated Reaction Palettes:** Admins choose which emojis to enable. For example, Eva Herman and Compact explicitly disabled the standard 👍 emoji, while 31% of Epoch Times’ total reactions are 👎. Comparing aggregate approval ratios *across* channels introduces severe structural bias; evaluations should always be normalized *within* each channel.
* **AUF1 Dominance:** AUF1 heavily skews cross-channel aggregates, averaging \~3,000 reactions per post. 
* **"Disapproval" vs. Shared Outrage:** Posts receiving high concentrations of 🤮, 🤬, and 🤡 are overwhelmingly directed at external political figures (e.g., Merz, Antifa, NATO, Habeck). These emojis reflect **shared hostility toward the target of the post**, not disapproval of the channel itself. (Categorized here as **Hostile Reactions**).
* **Emoji Misclassification in Text:** Standard emotion dictionaries fail here because the most frequent emojis function as structural bullet points (👉, ‼, ❗, 💥). Similarly, a mid-2020 spike in "Peace/Hope" is an artifact of \~1,460 daily `#abendgebet` (evening prayer) posts using 🙏.

## 3. Periodization & Channel Trajectories (`03_period_and_trajectory.ipynb`)

### Macro Trends vs. Channel-Level Reality

Aggregate daily posts increased from \~129/day (Pre-COVID) to \~400/day (Ukraine invasion). However, this aggregate growth is primarily driven by **channel cohort entry** (new channels being created) rather than increased individual output.

Normalized to **posts per active channel-day**, activity is much steadier across the 8 studied periods:
$$
\text{Posts/Active Day}: 18 \rightarrow 27 \rightarrow 32 \rightarrow 27 \rightarrow 26 \rightarrow 21 \rightarrow 22 \rightarrow 19
$$

### Divergent Lifecycle Archetypes

Channel activity trajectories frequently move in opposite directions:
* **COVID Peakers:** Janich peaked at 108 posts/day during the COVID winter, later cooling to \~42/day. Epoch Times declined from 58 to 28 posts/day.
* **Geopolitical Pivot Channels:** Druschba FM and satellit_de started near zero before 2022 and surged to 40–55 posts/day as the Ukraine conflict expanded.

> **Statistical Note:** A naive Kruskal-Wallis test across periods yields a negligible effect size ($\epsilon^2 = 0.002$), because channel-level daily medians remain tightly centered around 9–11 posts. **Channel fixed-effects models** are required to estimate true period effects.

## 4. Hypothesis Testing Results (`04_hypothesis_testing.ipynb`)

| Hypothesis | Expected Effect | Observed Outcome | Status | Notes | 
| ----- | ----- | ----- | ----- | ----- | 
| **H2** (Mobilization Spikes) | $\ge 2.0\times$ baseline volume during key protests | $1.6\times - 1.8\times$ baseline | **Partial** | Clearly elevated above placebo control dates. Reichstag mobilization reverted to baseline by Day 14; Leipzig remained elevated ($1.27\times$). | 
| **H3** (Narrative Pivot: COVID $\rightarrow$ War) | $\ge 60\%$ drop in COVID share; rise in Ukraine share | $-43\%$ COVID share; $+180\%$ **to** $+250\%$ Ukraine share | **Partial** | Pivot sharply localized to **Feb 21–22, 2022**. However, 0 of 14 channels met the strict $60\%$ COVID decline threshold. | 
| **H5** (QAnon De-platforming) | $\ge 50\%$ drop in engagement post-ban | Posts/day: $0.45$ ratio; Views: $0.96$ ratio | **Not Supported** | Tested on *QlobalChange*. While publishing frequency dropped by half, audience view counts per post held steady. | 
| **H8** (Discourse Diffusion) | Statistically significant adoption of test terms | — | **Inconclusive** | Lexicon contained only two keyword targets; insufficient breadth for a definitive conclusion. | 
| **H9** (Persistent Lexical Shift) | Sustained post-event lexical elevation | $10\times$ baseline at 2 weeks; $4\times$ at 30–45 days | **Supported** | Trend sustained while H2 returned to baseline. Note: low baseline ($0.36$ posts/day) means raw counts are highly sensitive. | 

## 🛠️ Setup & Replication

```bash
# Clone the repository
git clone https://github.com/your-org/telegram-corpus-analysis.git
cd telegram-corpus-analysis

# Set up environment
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Run notebook server
jupyter lab
```