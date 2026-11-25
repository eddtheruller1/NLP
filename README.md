# Sentiment de la presse financière marocaine et MASI

Can the tone of the Moroccan financial press tell us something about the Casablanca stock market (the MASI index)?

We collect six years of French-language Moroccan financial headlines and train a small French language model, **DistilCamemBERT**, to read their tone. From that tone we build the **Net Optimism Oscillator (I_Net)**, turn it into weekly **Buy / Sell / Hold** signals, label the market's **phases**, and backtest the resulting **strategy** against simply holding the MASI.

## The results in five lines

1. **The model reads tone well.** On 941 headlines it never saw, it scores F1 = 0.76, against 0.62 for FinBERT (on translated headlines) and 0.45 for DistilCamemBERT without training.
2. **The press is optimistic by default.** The raw I_Net is above +20% in 92% of weeks and never below −20%, so fixed thresholds say "buy" almost all the time.
3. **Centred on its own past year**, the oscillator gives rare, clear signals: 15 buy and 8 sell weeks out of 289.
4. **The signals do not forecast next week's return** (p = 0.63), and the strategy does not beat buy-and-hold on return (7.3% against 7.2% a year, p = 0.97).
5. **It does lower risk.** Volatility falls from 14.2% to 12.3% and the worst fall from −30% to −16%: it sold before the 2022 decline. That rests on only 5 trades.

## How to read this folder

Start with the slides, then go deeper only where you need to.

| Order | Open | What it is |
|---|---|---|
| 1 | `deck/presentation_FR.pptx` (or `_EN`) | The presentation: 17 slides. |
| 2 | `deck/econometrie_FR.pptx` (or `econometrics_EN`) | Appendix: each formula and test, with a plain-language box. |
| 3 | `results/charts/` | The dual-axis charts (I_Net vs MASI) and the market-phase chart. |
| 4 | `results/*.json` | Every number in the slides, exactly as the code produced it. |
| 5 | The scripts below | The full pipeline, if you want to check or rerun it. |

## Folder map

```
collect.py          1. collect headlines from Google News (Oct 2020 – Oct 2026)
clean.py            2. clean them and date each one to the next trading session
label.py            3. a large language model (Gemini) labels each headline
train.py            4. fine-tune DistilCamemBERT on those labels, score every headline
compare_models.py   5. compare with FinBERT (on French and on translated headlines)
barometer.py        6. the Net Optimism Oscillator, signals, phases, strategy backtest, charts
econ.py, extra.py   7. side analysis: volatility, correlations, forecasts (one slide in the talk)
llm.py              a small Gemini client used by label.py and compare_models.py
data/               headlines (raw, clean, labelled, scored) and market prices (data/market/)
results/            every result as JSON, and the charts
deck/               the presentation and the appendix (French and English)
```

## Run it yourself

```
pip install -r requirements.txt
python clean.py            # rebuilds data/clean_headlines.csv from data/raw_headlines.csv
python barometer.py        # oscillator, phases, strategy, charts  -> results/barometer.json, results/charts/
python econ.py             # volatility side analysis              -> results/econ_results.json
python extra.py            # chart data                            -> results/extra.json
```

These four run in a minute on any laptop and reproduce every number in the slides from the files in `data/` (they need the price files described under Data).

The longer steps need more:

- `collect.py` needs internet access (about 5 minutes).
- `label.py` and `compare_models.py` need a Gemini API key: copy `.env.example` to `.env` and fill it in.
- `train.py` needs `torch` (CPU is enough, about 15 minutes); the trained model is saved in `models/`, which is not committed.

## Rules we fixed before looking at the results

- A headline counts at the first trading session after its date, so no future information is used.
- Every average, threshold and model weight uses past weeks only.
- The 941 test headlines were never used in training.
- Three hypotheses, each tested against H0 ("no effect"), rejected only if p < 5%.

## Data

- MASI daily closes: TradingView. Share prices: Médias24, checked against the Casablanca Stock Exchange's official figures.
- The price files (`data/market/masi_daily.csv`: date, close; `data/market/shares_daily.csv`: ticker, date, close, value) are not in this public repository, because of their sources' terms of use. To rerun `clean.py`, `barometer.py`, `econ.py` or `extra.py`, put your own copies there with those columns. Every result they produce is already in `results/`.
- Headlines: Google News, Morocco edition, in French; 25 searches; 6,368 headlines from 241 outlets after cleaning.
- Limits: headlines only (not full articles), French-language press only, fewer headlines before 2023, labels given by a language model.
