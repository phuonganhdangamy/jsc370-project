# jsc370-project
midterm and final project

# Sonic Time Travelers: Can a Song's Identity Place It in Its Era? 

## Project overview


## Data Collection
Step 1: Spotify Search API
        → Genre + year keyword queries across 5 decades
        → TODO raw tracks collected

Step 2: Deduplication + Cleaning
        → Deduplicate on track_id
        → Filter to 1980–2026 only
        → Remove remasters/live versions via regex
        → Keep minimum release_year per song (true era rule)
        → Cap at 2,000 songs per decade (stratified)

Step 3: MusicBrainz Enrichment
        → Query all unique artists for country + tags
        → Merge on artist_id
        → Output: musicbrainz.parquet

Step 4: Genius Lyrics Scraping
        → Search + scrape per track
        → Clean lyrics text
        → Expected TODO tracks with lyrics
        → Output: lyrics_raw.parquet

Step 5: Lyrical Embeddings
        → sentence-transformers: all-MiniLM-L6-v2
        → 384-dimensional vector per song
        → Output: embeddings.parquet

Step 6: Master Dataset
        → Merge audio + embeddings on track_id
        → Output: master.parquet (~6,500 complete observations)


# Sonic Time Travelers: Can a Song's Identity Place It in Its Era?
### JSC370 Midterm Project — University of Toronto, 2026

---

## Project Overview

This project investigates whether a song's sonic and lyrical identity can reliably place it in its decade of origin — and more interestingly, where it fails to do so. We build two parallel machine learning classifiers: one trained on audio features and one trained on lyrical embeddings, both predicting which decade (1980s–2020s) a song belongs to. The disagreements between the two models reveal songs that are "Sonic Time Travelers" — modern songs that sound like they belong to a different era.

The project is motivated by a cultural observation: just as fashion cycles repeat themselves, music appears to as well. Artists like The Weeknd and Dua Lipa are frequently noted for their retro sonic aesthetics despite being contemporary acts. We ask whether this phenomenon is statistically measurable, and whether it correlates with commercial success.

---

## Research Question

> *"Can a song's sonic and lyrical identity reliably place it in its era — and where does the model fail? Are modern songs that 'sound old' actually distinguishable from the originals?"*

---

## Hypotheses

**Primary Hypothesis:** Modern songs that mimic 80s/90s sonics (e.g. The Weeknd, Dua Lipa) will be misclassified by the Audio Model as "Vintage" but correctly identified by the Lyric Model as "Modern," due to contemporary vocabulary and themes in their lyrics.

**Secondary Hypothesis:** Misclassified modern songs — which we call "Neo-Retro" hits — will have a significantly higher Spotify Popularity Score than correctly classified modern songs, supporting a "nostalgia sells" theory.

---

## Data Sources

### 1. Spotify Web API
**Purpose:** Audio features and track metadata  
**Documentation:** https://developer.spotify.com/documentation/web-api  
**What we collect:** Track ID, track name, artist name, album ID, release date, popularity score, and 11 audio features per song: danceability, energy, loudness, acousticness, valence, tempo, speechiness, instrumentalness, liveness, key, and mode.  
**Method:** Search API queries by decade and genre keyword (e.g. `"rock year:1980-1989"`), pulling up to 1,000 results per query in batches of 10. A custom rate limiter (10 requests per 30-second rolling window) is used to respect Spotify's API limits.

### 2. Genius API + Web Scraping
**Purpose:** Song lyrics for lyrical embedding  
**Documentation:** https://docs.genius.com  
**What we collect:** Raw lyrics for each track, scraped from Genius song pages after searching by artist name and track name via the Genius search API.  
**Method:** Two-step pipeline — first query the Genius search API to find the lyrics URL, then scrape the lyrics page using BeautifulSoup. Lyrics are cleaned by removing section headers (`[Verse 1]`, `[Chorus]`, etc.), non-ASCII characters, and excess whitespace. Expected match rate is 60–70% of tracks.

### 3. MusicBrainz API
**Purpose:** Cultural enrichment — artist country of origin, community genre tags  
**Documentation:** https://musicbrainz.org/doc/MusicBrainz_API  
**What we collect:** Artist country (`mb_country`), area of origin (`mb_area`), city of origin (`mb_begin_area`), community tags, and genre tags.  
**Method:** Two-step query per artist — first search by artist name, then fetch the full artist record with tags. Rate limited to 1 request per second per MusicBrainz policy. Match confidence score (`mb_score`) is retained to filter low-quality matches.

---

## Data Collection Pipeline

```
Step 1: Spotify Search API
        → Genre + year keyword queries across 5 decades
        → ~15,000 raw tracks collected

Step 2: Deduplication + Cleaning
        → Deduplicate on track_id
        → Filter to 1980–2026 only
        → Remove remasters/live versions via regex
        → Keep minimum release_year per song (true era rule)
        → Cap at 2,000 songs per decade (stratified)
        → Output: spotify_tracks.parquet (~10,000 tracks)

Step 3: MusicBrainz Enrichment
        → Query all unique artists for country + tags
        → Merge on artist_id
        → Output: musicbrainz.parquet

Step 4: Genius Lyrics Scraping
        → Search + scrape per track
        → Clean lyrics text
        → Expected ~6,000–7,000 tracks with lyrics
        → Output: lyrics_raw.parquet

Step 5: Lyrical Embeddings
        → sentence-transformers: all-MiniLM-L6-v2
        → 384-dimensional vector per song
        → Output: embeddings.parquet

Step 6: Master Dataset
        → Merge audio + embeddings on track_id
        → Output: master.parquet (~6,500 complete observations)
```

## Key Variables

| Variable | Source | Description |
|----------|--------|-------------|
| `track_id` | Spotify | Unique Spotify track identifier |
| `release_date` | Spotify | Album release date |
| `decade` | Derived | Target variable: 1980s / 1990s / 2000s / 2010s / 2020s |
| `popularity` | Spotify | Spotify popularity score (0–100) |
| `danceability` | Spotify | How suitable for dancing (0–1) |
| `energy` | Spotify | Perceptual intensity (0–1) |
| `loudness` | Spotify | Overall loudness in dB |
| `acousticness` | Spotify | Confidence of acoustic sound (0–1) |
| `valence` | Spotify | Musical positiveness (0–1) |
| `tempo` | Spotify | Estimated beats per minute |
| `lyrics` | Genius | Cleaned song lyrics text |
| `embedding` | Derived | 384-dim sentence embedding of lyrics |
| `mb_country` | MusicBrainz | Artist country of origin |
| `mb_tags` | MusicBrainz | Community genre/style tags |

---

## Modeling Plan (Final Project)

Three parallel classifiers will be trained to predict `decade` (5-class):

1. **Audio Model** — 11 Spotify audio features → logistic regression baseline + XGBoost
2. **Lyric Model** — PCA(50) on 384-dim embeddings → XGBoost
3. **Combined Model** — concatenated audio + lyric features → XGBoost

**Train/validation/test split:** 70/15/15, stratified by decade, with artist-level leakage prevention (all songs by one artist go to the same split).

**Key metric:** Macro F1 (accounts for class imbalance across decades).

**"Sonic Time Traveler" score:** Absolute difference in decades between the audio model's prediction and the lyric model's prediction. Songs with high scores are the core finding of the project.

---

## Reproducibility

All data collection scripts are in `midterm.qmd`. To reproduce:

1. Clone the repository
2. Create a virtual environment: `python -m venv venv && venv\Scripts\activate`
3. Install dependencies: `pip install -r requirements.txt`
4. Set environment variables for API credentials:
   - `SPOTIFY_CLIENT_ID`
   - `SPOTIFY_CLIENT_SECRET`
   - `GENIUS_ACCESS_TOKEN`
5. Run all cells in `midterm.qmd` sequentially

**Note:** Spotify API is subject to rate limits (development mode). Full data collection takes approximately 2–3 hours across all steps. Intermediate parquet and csv checkpoints are saved throughout so the pipeline is resumable.

If API credentials are not available, one can skip the data collection section and continue running the remaining cells.

---

## Known Limitations

- **Spotify development mode** restricts request limits and blocks access to certain editorial playlists. Data was collected via the Search API rather than curated playlist seeding as originally planned.
- **Survivor bias**: in older decades, only songs that remained popular enough to appear in Spotify search results are included, which may overrepresent canonical/classic tracks from the 1980s and 1990s.
- **Lyrics coverage drops for older tracks**: Genius skews toward contemporary music, so the 1980s decade has a lower lyrics match rate than the 2020s.
- **Genre keyword sampling**: there could be bias toward the genres we queried (rock, pop, hip hop, etc.) and underrepresents niche or regional genres.

---

## Dependencies

See `requirements.txt`. Key libraries: `spotipy`, `requests`, `beautifulsoup4`, `sentence-transformers`, `scikit-learn`, `xgboost`, `pandas`, `plotly`, `umap-learn`.

---

## Author

Amy Dang Phuong — University of Toronto, JSC370 Data Science II, 2026