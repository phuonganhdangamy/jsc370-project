   # Sonic Time Travelers: Can a Song's Identity Place It in Its Era?
   ### JSC370 Project — University of Toronto, 2026

   **Website:** https://phuonganhdangamy.github.io/jsc370-project

   ---

   ## Project Overview

   This project investigates whether a song's identity can reliably place it in its decade of origin, and more interestingly, where it fails to do so. 

   The project is motivated by a cultural observation: just as fashion cycles repeat themselves, music appears to as well. Artists like The Weeknd and Dua Lipa are frequently noted for their retro sonic aesthetics despite being contemporary acts. We ask whether this phenomenon is statistically measurable, and whether it correlates with commercial success.

   ---

   ## Research Question

   > *"Can a song's lyrical identity reliably place it in its era, and where does the model fail? Are modern songs that 'sound old' actually distinguishable from the originals?"*

   ---

   ## Reproducibility

   All data collection scripts are in `midterm.qmd`. To reproduce:

   1. Clone the repository
   2. Create a virtual environment and install dependencies (see **Environment Setup** below)
   3. Set environment variables for API credentials:
      - `SPOTIFY_CLIENT_ID`
      - `SPOTIFY_CLIENT_SECRET`
      - `GENIUS_ACCESS_TOKEN`
   4. Run `quarto render` from the project root

   **Note:** Spotify API is subject to rate limits (development mode). Full data collection takes approximately 2–3 hours across all steps. Intermediate parquet and csv checkpoints are saved throughout so the pipeline is resumable.

   If API credentials are not available, one can skip the data collection section and continue running the remaining cells.

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
   # CPU only (works on all machines)
   pip install torch torchvision --index-url https://download.pytorch.org/whl/cpu

   # If you have a CUDA-capable GPU (optional, faster)
   pip install torch torchvision --index-url https://download.pytorch.org/whl/cu121
   ```

   ### 3. Install remaining dependencies
   ```bash
   pip install -r requirements.txt
   ```

   ### 4. Install Quarto kernel dependencies and register the kernel

   Quarto requires `nbformat`, `nbclient`, and `ipykernel` to be present in the venv. Run:
   ```bash
   pip install nbformat nbclient ipykernel ipython
   python -m ipykernel install --user --name jsc370-venv --display-name "jsc370-venv"
   ```

   ### 5. Render the report
   ```bash
   quarto render
   ```

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

   Amy Dang Phuong -- University of Toronto, JSC370 Data Science II, 2026