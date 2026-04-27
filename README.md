# Sonic Time Travelers: Can a Song's Identity Place It in Its Era?
### JSC370 Project — University of Toronto, 2026

**Website:** https://phuonganhdangamy.github.io/jsc370-project
**Presentation:** https://drive.google.com/file/d/1NMrsqgEzYZIpcpZ-DrLGA_QxFhKcTFv8/view?usp=drive_link 

---

## Project Overview

This project investigates whether a song's identity can reliably place it in its decade of origin, and more interestingly, where it fails to do so.

The project is motivated by a cultural observation: just as fashion cycles repeat themselves, music appears to as well. Artists like The Weeknd and Dua Lipa are frequently noted for their retro sonic aesthetics despite being contemporary acts. We ask whether this phenomenon is statistically measurable, and whether it correlates with commercial success.

Data is collected live from three APIs (Spotify, Genius, MusicBrainz), lyrics are embedded using `all-MiniLM-L6-v2`, and six classifier configurations (Logistic Regression, XGBoost, Random Forest × PCA / full features) are compared. The best model — XGBoost on full 389-dimensional features — achieves a macro-averaged F1 of 0.538 on the held-out test set. An interactive browser-based classifier on the site lets visitors paste any lyrics and get a real-time decade prediction, running the full XGBoost model client-side with no backend.

---

## Research Question

> *"Can a song's lyrical identity reliably place it in its era, and where does the model fail? Are modern songs that 'sound old' actually distinguishable from the originals?"*

---

## Repository Structure

```
├── midterm.qmd          # Midterm progress
├── era-classifier.qmd   # Data collection pipeline (Spotify, Genius, MusicBrainz) and feature engineerings
├── era-classifier.qmd   # Feature engineering, model training, SHAP, UMAP
├── results.qmd          # Model results and interpretation (rendered page)
├── visualizations.qmd   # Interactive EDA plots (rendered page)
├── try-it.qmd           # Interactive era classifier widget (rendered page)
├── index.qmd            # Landing page
├── final_report.qmd     # Drafts
├── _quarto.yml          # Site configuration
├── requirements.txt
├── data/
│   ├── raw/             # Spotify, Genius, MusicBrainz API outputs
│   └── processed/       # Merged, cleaned, and embedded datasets
├── models/              # Trained .joblib model files (see note below)
└── docs/                # Rendered website (served by GitHub Pages)
    └── models/          # Browser-side JSON artifacts for the classifier widget
```

---

## Reproducibility

All data collection scripts are in `data-collection.qmd`. To reproduce:

1. Clone the repository
2. Create a virtual environment and install dependencies (see **Environment Setup** below)
3. Set environment variables for API credentials:
   - `SPOTIFY_CLIENT_ID`
   - `SPOTIFY_CLIENT_SECRET`
   - `GENIUS_ACCESS_TOKEN`
4. Run `quarto render` from the project root

**Note:** Spotify API is subject to rate limits (development mode). Full data collection takes approximately 2–3 hours across all steps. Intermediate parquet and CSV checkpoints are saved throughout so the pipeline is resumable.

If API credentials are not available, skip the data collection section of `data-collection.qmd` and continue from the saved checkpoints in `data/`.

---

## Environment Setup

Python 3.12 is required. The project uses a local virtual environment.

### 1. Create and activate the virtual environment
```bash
python -m venv venv

# Windows
venv\Scripts\activate

# macOS/Linux
source venv/bin/activate
```

### 2. Install PyTorch

`torch` and `torchvision` must be installed from PyTorch's own index before the rest of the requirements, or version mismatches will cause import errors.
```bash
# CPU only 
pip install torch torchvision --index-url https://download.pytorch.org/whl/cpu

# If you have a CUDA-capable GPU
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu121
```

### 3. Install remaining dependencies
```bash
pip install -r requirements.txt
pip install nbformat nbclient ipykernel ipython
python -m ipykernel install --user --name jsc370-venv --display-name "jsc370-venv"
```

### 4. Render the site

```bash
quarto render
```

This renders all pages to `docs/`. Individual pages can also be rendered on their own, e.g. `quarto render results.qmd`.

---

## Model Training Note

> **`era-classifier.qmd` takes over 1.5 hours to render** due to 5-fold cross-validated grid searches for XGBoost and Random Forest over ~9,000 tracks.

This file trains all six model configurations, saves them to `models/` as `.joblib` files, runs SHAP analysis and UMAP, and exports the browser-side JSON artifacts to `docs/models/`.

### Skipping model training — download pre-trained artifacts

Because the trained model files are too large for GitHub (XGBoost: ~10 MB, Random Forest: ~107 MB each), they are not committed to this repository. Download the pre-computed artifacts from Google Drive:

> **[Download pre-trained models and artifacts](INSERT_GOOGLE_DRIVE_LINK_HERE)**

Unzip and place files as follows:

| Downloaded folder | Destination |
|---|---|
| `models/` | `models/` (project root) |
| `processed/` | `data/processed/` |
| `docs_models/` | `docs/models/` |

Once in place, skip `era-classifier.qmd` and render only the remaining pages:

```bash
quarto render index.qmd
quarto render visualizations.qmd
quarto render results.qmd
quarto render try-it.qmd
```

---

## Known Limitations

- **Spotify development mode** restricts request limits and blocks access to certain editorial playlists. Data was collected via the Search API rather than curated playlist seeding as originally planned.
- **Survivor bias**: in older decades, only songs that remained popular enough to appear in Spotify search results are included, which may overrepresent canonical/classic tracks from the 1980s and 1990s.
- **Lyrics coverage drops for older tracks**: Genius skews toward contemporary music, so the 1980s decade has a lower lyrics match rate than the 2020s.
- **Genre keyword sampling**: there could be bias toward the genres we queried (rock, pop, hip hop, etc.) and underrepresents niche or regional genres.
- **Pageview proxy**: Genius pageviews measure lyric-lookup traffic, not streaming popularity or commercial success.

---

## Dependencies

See `requirements.txt`. Key libraries: `spotipy`, `requests`, `beautifulsoup4`, `sentence-transformers`, `scikit-learn`, `xgboost`, `shap`, `umap-learn`, `pandas`, `plotly`, `kaleido`.

---

## Author

Amy Dang Phuong — University of Toronto, JSC370 Data Science II, 2026
