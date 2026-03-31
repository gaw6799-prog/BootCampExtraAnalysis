# Trump Iran Market Manipulation Forensics

**NYU Data Bootcamp — Midterm Project**

## Introduction

On March 23, 2026, Donald Trump posted on Truth Social claiming "productive conversations" with Iran were underway. Oil prices dropped over 13% as markets anticipated de-escalation. Hours later, Iran's Foreign Ministry explicitly denied any talks had occurred, accusing Trump of using "fake news to manipulate the financial and oil markets."

This project builds a forensic case for market manipulation by analyzing Trump's Iran-related Truth Social posts (October 2024 – March 2026), oil price movements around those posts, Polymarket prediction market activity, and AI-classified intent and fabrication risk.

**Central thesis**: Trump used an oscillation strategy — posting escalation threats to spike oil prices, then posting de-escalation claims (often fabricated) to crash them — generating volatility that benefited insiders positioned before announcements.

## Research Questions

1. **Oscillation**: Did Trump systematically alternate between escalation and de-escalation posts, creating artificial price volatility?
2. **Fabrication**: Were specific claims fabricated (later denied by official sources), and did these false claims produce larger price movements?
3. **Causality**: Is there statistical evidence that posts preceded (rather than followed) price movements?
4. **Insider Signals**: Do Polymarket prediction markets show anomalous betting patterns before Trump's Iran announcements?
5. **Intent**: Can AI classification detect market-manipulation intent in post language and timing?

## Data Sources

| Source | Data | Method | Rows |
|--------|------|--------|------|
| [trumpstruth.org](https://trumpstruth.org) | Trump Truth Social posts | BeautifulSoup scraping | 21,013 posts |
| [FRED](https://fred.stlouisfed.org) | Brent & WTI daily crude prices | CSV download | 374 trading days |
| [Yahoo Finance](https://finance.yahoo.com) | USO ETF 5-min intraday bars | yfinance library | 4,557 bars |
| [Polymarket](https://polymarket.com) | Iran prediction markets | Gamma/CLOB API | 23 markets ($30M+) |
| [Google Gemini](https://aistudio.google.com) | AI post classification | API with Search grounding | 274 classified posts |
| [EODHD](https://eodhd.com) | Intraday crude oil futures | REST API (optional) | Key dates |

The master dataset contains **379 rows** (one per trading day, Oct 2024 – Mar 2026) with **31 features** of mixed numeric and categorical types.

## Methodology: Five Detection Vectors

| Vector | Weight | Description |
|--------|--------|-------------|
| Oscillation | 20% | Threat → reversal pattern detection within 72-hour windows |
| Fabrication | 25% | Claims denied by official sources × price impact |
| Causality | 20% | Volume anomaly + directional consistency + magnitude |
| Polymarket | 20% | Insider betting indicators from prediction markets |
| Gemini Intent | 15% | AI-scored market mover probability × timing suspicion |

### Composite Score Formula

```
COMPOSITE = Oscillation×0.20 + Fabrication×0.25 + Causality×0.20 + Polymarket×0.20 + Intent×0.15
VERDICT: >70 = HIGH | 50-70 = ELEVATED | <50 = LOW
```

## Key Findings

### Statistical Results

| Test | Result | p-value | Significant? |
|------|--------|---------|-------------|
| Post-day volatility vs non-post | 1.41x higher | 0.009 | Yes (p < 0.01) |
| Escalation → price increase | 57.3% accuracy | 0.020 | Yes (p < 0.05) |
| Volume anomaly on post days | z = 0.050 | 0.028 | Yes (p < 0.05) |
| Granger causality (lag 1-3) | F = 1.87 | 0.172 | No |

### Top Suspicious Days

| Date | Score | Verdict | Key Event |
|------|-------|---------|-----------|
| March 9, 2026 | 64.0 | ELEVATED | $38 intraday swing — largest crude range in history |
| March 6, 2026 | 62.4 | ELEVATED | "Unconditional surrender" — Iran did not surrender |
| March 23, 2026 | 62.2 | ELEVATED | "Productive conversations" fabrication — Iran denied |
| March 2, 2026 | 60.6 | ELEVATED | Escalation with high fabrication risk |
| March 3, 2026 | 57.4 | ELEVATED | Continued escalation pattern |

11 total days scored ELEVATED for manipulation risk.

### The Smoking Gun: March 23, 2026

1. Trump posts claiming "productive conversations" with Iran
2. Oil drops 13%+ as markets anticipate de-escalation
3. Iran's Foreign Ministry explicitly denies any talks
4. Iran accuses Trump of "fake news to manipulate financial and oil markets"
5. Fabrication score: 100/100 | Market mover: 81/100

### Polymarket Evidence

- $30M+ traded across 23 Iran-related prediction markets
- 57% of volume concentrated in military action markets
- Multiple markets showed anomalous activity around key dates

## Visualizations

8 publication-quality figures in `figures/`:

1. Oil price timeline with manipulation-score-colored post markers
2. March 2026 crisis annotated timeline
3. Post-day vs non-post-day volatility (box plots + histograms)
4. Volatility anomaly z-scores
5. Manipulation score decomposition (stacked bars, top 10 days)
6. Polymarket Iran market volumes and suspicion distribution
7. Feature correlation heatmap (13 variables)
8. Key date intraday deep-dives (USO ETF 5-minute bars)

## Notebook Structure

| Notebook | Purpose |
|----------|---------|
| 01_data_collection | Scrape posts, download oil prices, fetch Polymarket data |
| 02_cleaning_merging | Clean, merge into 379-row master dataframe with 31 features |
| 03_gemini_classification | AI classification of 274 posts across 5 dimensions |
| 04_polymarket_forensics | Prediction market analysis and suspicion scoring |
| 05_composite_scoring | Weighted manipulation scoring across all 5 vectors |
| 06_statistical_analysis | t-tests, binomial tests, Granger causality, Mann-Whitney U |
| 07_visualizations | 8 publication-quality figures |

## Project Structure

```
DataBootCamp/
├── README.md
├── requirements.txt
├── .env                              # API keys (not committed)
├── data/
│   ├── raw/                          # Scraped and API data
│   │   ├── truth_posts.csv           # 21,013 Trump posts
│   │   ├── brent_daily.csv           # Brent crude daily prices
│   │   ├── wti_daily.csv             # WTI crude daily prices
│   │   ├── uso_intraday.csv          # USO ETF 5-min bars
│   │   └── polymarket_markets.csv    # 23 Iran prediction markets
│   └── processed/                    # Cleaned datasets
│       ├── master.csv                # 379 rows, 31 features
│       ├── iran_posts_cleaned.csv    # 2,695 Iran-related posts
│       ├── gemini_classifications.csv # 274 AI classifications
│       └── polymarket_suspicion.csv  # Market suspicion scores
├── figures/                          # Publication-quality PNGs
├── notebooks/
│   ├── 01_data_collection.ipynb
│   ├── 02_cleaning_merging.ipynb
│   ├── 03_gemini_classification.ipynb
│   ├── 04_polymarket_forensics.ipynb
│   ├── 05_composite_scoring.ipynb
│   ├── 06_statistical_analysis.ipynb
│   └── 07_visualizations.ipynb
└── scripts/                          # Development scripts (not committed)
```

## How to Reproduce

```bash
# 1. Clone and set up environment
git clone <repo-url>
cd DataBootCamp
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# 2. Add API keys
echo "GEMINI_API_KEY=your_key" > .env
echo "EODHD_API_KEY=your_key" >> .env  # optional

# 3. Run notebooks in order (01 → 07)
```

## Limitations

- **Correlation vs. causation**: While post-day volatility is statistically significant, this alone does not prove intentional manipulation.
- **Keyword classification**: Initial post direction uses keyword matching; Gemini AI provides more nuanced classification but only covers 274 posts.
- **Intraday data**: USO ETF is a proxy for crude oil; yfinance limits intraday data to the last 60 days.
- **Polymarket**: Historical price data for resolved markets is limited; analysis relies primarily on volume and market metadata.
- **Fabrication assessment**: Only 4 events are manually confirmed as fabrications; Gemini's fabrication_risk score is probabilistic.
- **Granger causality**: Not significant at conventional thresholds, suggesting the relationship may be contemporaneous rather than lagged.

## API Keys Required

- **GEMINI_API_KEY**: Required for Notebook 03 (Google Gemini AI classification with Google Search grounding)
- **EODHD_API_KEY**: Optional for intraday crude oil futures data (falls back to USO ETF via yfinance)

## Ethical Considerations

All data sourced from public archives and open APIs. AI classification methodology is transparent and reproducible. Market manipulation claims are supported by statistical evidence but should be understood as forensic indicators, not legal conclusions. Iran's explicit accusation of market manipulation is documented as evidence, not editorial opinion.
# BootCampExtraAnalysis
