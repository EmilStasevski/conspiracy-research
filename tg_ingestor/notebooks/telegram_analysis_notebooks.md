# Telegram Channel Dynamics & Discourse Analysis (2020–2026)

This repository contains Jupyter notebooks, data processing pipelines, and analytical findings examining post volume, audience reactions, and narrative shifts across German-language alternative Telegram channels from **January 1, 2020 to October 21, 2026**.

---

## 📂 Notebooks Overview

| Notebook | Focus Area | Description |
| :--- | :--- | :--- |
| `01_corpus_diagnostics.ipynb` | **Data Hygiene & Corpus Validation** | Channel footprints, missing channel audits, text vs. media shifts, forward topologies, and scrape-gap checks. |
| `02_reactions_and_emojis.ipynb` | **Engagement & Sentiment Artifacts** | Channel reaction constraints, hostile vs. disapproving engagement, emoji codebook cleanup, and audience scale. |
| `03_period_and_trajectory.ipynb` | **Temporal Trends & Trajectories** | Longitudinal activity across 8 distinct periods, channel lifecycle shifts, and panel modeling with fixed effects. |
| `04_hypothesis_testing.ipynb` | **Pre-registered Event & Topic Tests** | Event studies (Leipzig, Reichstag), narrative substitution (COVID to Ukraine), and keyword-based tracking. |

---

## 1. Corpus Diagnostics (`01_corpus_diagnostics.ipynb`)

### Dataset at a Glance
- **Total volume:** 869,307 posts across 19 channels.
- **Coverage:** Jan 1, 2020 – Oct 21, 2026.
- **Power-law distribution:** Five channels generate ~70% of the entire corpus:
  - Eva Herman (~224k)
  - Oliver Janich (~149k)
  - Druschba FM (~84k)
  - Epoch Times (~78k)
  - satellit_de (~71k)

### Key Dynamics & Methodological Caveats
* **The "21 Channels" Discrepancy:** The initial research design called for ~21 channels. Two expected channels are missing from this scrape; verify target lists before drawing macro-level conclusions.
* **Format Shift (Media over Text):** Overall post frequency per active channel peaked in Q4 2020 (~34 posts/day) and settled at roughly 18 posts/day from 2024 onward. Concurrently, purely visual/media posts (posts lacking text bodies) doubled from 5–7% in 2020 to ~13% from 2022 onward.
* **Forwarding Behaviors:** 
  - ~21% of Eva Herman’s output consists of self-forwards (rebroadcasting content from her own channel).
  - Janich heavily amplifies external actors, providing an ideal anchor for a network diffusion analysis.
* **Data Gaps to Audit:** Three channels show extended quiet periods:
  - `icic_law` (510-day hiatus)
  - `Fuellmich` (213-day gap in 2022)
  - `RKI_Files` (missing interval)
  *Action item:* Cross-check Telegram API limits and scraper logs to determine if these are genuine channel pauses or interrupted scrapes.

---

## 2. Reactions & Sentiment (`02_reactions_and_emojis.ipynb`)

Reactions in Telegram carry distinct affordances and structural constraints that complicate straightforward sentiment analysis.

* **Reactions Are Not Universally Available:** 15 of 19 channels had reactions active. Reactions remained permanently disabled on *AllesAusserMainstream*, *NachDenkSeiten*, *RKI_Files*, and *kenjebsen*.
* **Admin-Curated Reaction Palettes:** Admins choose which emojis to enable. For example, Eva Herman and Compact explicitly disabled the standard 👍 emoji, while 31% of Epoch Times’ total reactions are 👎. Comparing aggregate approval ratios *across* channels introduces severe structural bias; evaluations should always be normalized *within* each channel.
* **AUF1 Dominance:** AUF1 heavily skews cross-channel aggregates, averaging ~3,000 reactions per post. Ranking popularity globally without channel normalization obscures smaller community dynamics.
* **"Disapproval" vs. Shared Outrage:** Posts receiving high concentrations of 🤮, 🤬, and 🤡 are overwhelmingly directed at external political figures and institutions (e.g., Merz, Antifa, NATO, Habeck). These emojis reflect **shared hostility toward the target of the post**, not disapproval of the channel itself. The category has been renamed from *Disapproval* to **Hostile Reactions**.
* **Emoji Misclassification in Text:** Standard emotion dictionaries fail on this dataset because the most frequent emojis function purely as structural bullet points or attention hooks (👉, ‼, ❗, 💥). Similarly, a mid-2020 spike categorized as "Peace/Hope" is an artifact of ~1,460 recurring daily prayer posts (`#abendgebet` with 🙏).

---

## 3. Periodization & Channel Trajectories (`03_period_and_trajectory.ipynb`)

### Macro Trends vs. Channel-Level Reality
Aggregate daily posts increased from ~129/day (Pre-COVID) to ~400/day (Ukraine invasion). However, this aggregate growth is primarily driven by **channel cohort entry** (only 7 channels were tracked Pre-COVID, with new channels joining over time) rather than increased individual output.

When normalized to **posts per active channel-day**, activity tells a steadier story across the 8 studied periods:
$$\text{Posts/Active Day}: 18 \rightarrow 27 \rightarrow 32 \rightarrow 27 \rightarrow 26 \rightarrow 21 \rightarrow 22 \rightarrow 19$$

### Divergent Lifecycle Archetypes
Channel activity trajectories frequently move in opposite directions:
* **COVID Peakers:** Janich peaked at 108 posts/day during the peak COVID winter, later cooling to ~42/day. Epoch Times declined from 58 to 28 posts/day.
* **Geopolitical Pivot Channels:** Druschba FM and satellit_de started near zero before 2022 and surged to 40–55 posts/day as geopolitical conflict expanded.

> **Statistical Note:** A naive Kruskal-Wallis test across periods yields a negligible effect size ($\epsilon^2 = 0.002$), largely because channel-level daily medians remain tightly centered around 9–11 posts. Because the active channel mix changes across periods, **channel fixed-effects models** are required to estimate true period effects.

---

## 4. Hypothesis Testing Results (`04_hypothesis_testing.ipynb`)

| Hypothesis | Expected Effect | Observed Outcome | Status | Notes |
| :--- | :--- | :--- | :--- | :--- |
| **H2** (Mobilization Spikes) | $\ge 2.0\times$ baseline volume during key protests | **$1.6\times - 1.8\times$** baseline | **Partial / Below Pre-Reg** | Clearly elevated above placebo control dates. Reichstag mobilization reverted to baseline by Day 14; Leipzig remained elevated ($1.27\times$). |
| **H3** (Narrative Pivot: COVID $\rightarrow$ War) | $\ge 60\%$ drop in COVID share; rise in Ukraine share | **$-43\%$** COVID share; **$+180\%$ to $+250\%$** Ukraine share | **Partial** | Pivot change-point sharply localized to **Feb 21–22, 2022**. However, 0 of 14 channels met the strict $60\%$ decline threshold. |
| **H5** (QAnon De-platforming Drop) | $\ge 50\%$ drop in engagement post-ban | Posts/day: **$0.45$** ratio; Views: **$0.96$** ratio | **Not Supported** | Tested on *QlobalChange*. While publishing frequency dropped by half, audience view counts per post held steady. |
| **H8** (Discourse Diffusion) | Statistically significant adoption of test terms | — | **Inconclusive** | Lexicon contained only two keyword targets; insufficient breadth to draw a definitive conclusion. |
| **H9** (Persistent Lexical Shift) | Sustained post-event lexical elevation | **$10\times$** baseline at 2 weeks; **$4\times$** at 30–45 days | **Directionally Supported** | Trend sustained while H2 returned to baseline ($p = 0.08$ vs. placebo). Note: low baseline ($0.36$ posts/day) means raw counts are sensitive. |

---

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

### Key Python Dependencies
- `polars` / `pandas` – High-throughput post and interaction processing
- `statsmodels` – Panel regression and channel fixed-effects models
- `matplotlib` / `seaborn` – Longitudinal heatmaps and timeline visualizations
- `emoji` – Lexicon tokenization and layout-glyph filtering