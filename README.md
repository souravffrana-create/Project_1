# Project_1
import streamlit as st
import pickle
import requests
from pathlib import Path

# ── Page config ───────────────────────────────────────────────────────────────
st.set_page_config(
    page_title="CineMatch",
    page_icon="🎬",
    layout="wide",
)

# ── Load models ───────────────────────────────────────────────────────────────
BASE_DIR  = Path(__file__).resolve().parent
MODEL_DIR = BASE_DIR / "models"

@st.cache_resource
def load_models():
    movies     = pickle.load(open(MODEL_DIR / "movies.pkl",     "rb"))
    similarity = pickle.load(open(MODEL_DIR / "similarity.pkl", "rb"))
    return movies, similarity

movies_df, similarity = load_models()

# ── TMDB API ──────────────────────────────────────────────────────────────────
TMDB_API_KEY = "46641e0b65de6f89911eb679745ea2d9"

@st.cache_data(show_spinner=False)
def fetch_movie_info(movie_id):
    try:
        url = (
            f"https://api.themoviedb.org/3/movie/{movie_id}"
            f"?api_key={TMDB_API_KEY}&language=en-US"
        )
        r    = requests.get(url, timeout=5)
        data = r.json()
        poster = (
            f"https://image.tmdb.org/t/p/w500{data['poster_path']}"
            if data.get("poster_path") else None
        )
        return {
            "poster":   poster,
            "rating":   data.get("vote_average", 0),
            "overview": data.get("overview", ""),
            "year":     data.get("release_date", "")[:4],
            "genres":   ", ".join(g["name"] for g in data.get("genres", [])[:2]),
        }
    except Exception:
        return {"poster": None, "rating": 0, "overview": "", "year": "", "genres": ""}

# ── Recommend ─────────────────────────────────────────────────────────────────
def recommend(movie_title):
    idx    = movies_df[movies_df["title"] == movie_title].index[0]
    scores = sorted(enumerate(similarity[idx]), key=lambda x: x[1], reverse=True)[1:6]
    results = []
    for i, score in scores:
        row      = movies_df.iloc[i]
        info     = fetch_movie_info(int(row["id"]))
        info["title"] = row["title"]
        info["score"] = round(score, 4)
        results.append(info)
    return results

# ── CSS ───────────────────────────────────────────────────────────────────────
st.markdown("""
<style>
@import url('https://fonts.googleapis.com/css2?family=Bebas+Neue&family=Inter:wght@300;400;500;600&display=swap');

html, body, [data-testid="stAppViewContainer"] {
    background: #0a0a0f;
    color: #e8e4dc;
}
[data-testid="stAppViewContainer"] {
    background: radial-gradient(ellipse at 20% 10%, #1a0a2e 0%, #0a0a0f 60%);
}
[data-testid="stHeader"] { background: transparent; }
#MainMenu, footer, header { visibility: hidden; }

/* Hero */
.hero { text-align:center; padding: 3rem 1rem 0.5rem; }
.hero-title {
    font-family: 'Bebas Neue', sans-serif;
    font-size: clamp(3.5rem, 10vw, 7rem);
    letter-spacing: 0.08em;
    line-height: 1;
    background: linear-gradient(135deg, #e8e4dc 30%, #c084fc 100%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
    margin: 0;
}
.hero-sub {
    font-family: 'Inter', sans-serif;
    font-weight: 300;
    font-size: 0.95rem;
    letter-spacing: 0.25em;
    text-transform: uppercase;
    color: #7c6fa0;
    margin-top: 0.4rem;
}
.divider {
    width: 60px; height: 2px;
    background: linear-gradient(90deg, #c084fc, transparent);
    margin: 1.8rem auto;
}

/* Selectbox */
.select-label {
    font-family: 'Inter', sans-serif;
    font-size: 0.7rem; font-weight: 600;
    letter-spacing: 0.2em; text-transform: uppercase;
    color: #7c6fa0; margin-bottom: 0.4rem;
}
[data-testid="stSelectbox"] > div > div {
    background: #13111a !important;
    border: 1px solid #2a2440 !important;
    border-radius: 4px !important;
    color: #e8e4dc !important;
    font-family: 'Inter', sans-serif !important;
}
[data-testid="stSelectbox"] > div > div:hover { border-color: #c084fc !important; }

/* Button */
[data-testid="stButton"] > button {
    background: linear-gradient(135deg, #7c3aed, #c084fc) !important;
    color: #fff !important; border: none !important;
    border-radius: 3px !important;
    font-family: 'Inter', sans-serif !important;
    font-weight: 600 !important; font-size: 0.8rem !important;
    letter-spacing: 0.15em !important; text-transform: uppercase !important;
    padding: 0.65rem 2.5rem !important; width: 100% !important;
    transition: opacity 0.2s !important;
}
[data-testid="stButton"] > button:hover { opacity: 0.85 !important; }

/* Cards */
.results-heading {
    font-family: 'Inter', sans-serif;
    font-size: 0.65rem; font-weight: 600;
    letter-spacing: 0.3em; text-transform: uppercase;
    color: #7c6fa0; text-align: center;
    margin: 2.5rem 0 1.5rem;
}
.movie-card {
    background: #13111a;
    border: 1px solid #2a2440;
    border-radius: 8px;
    overflow: hidden;
    transition: border-color 0.2s, transform 0.2s;
    height: 100%;
}
.movie-card:hover { border-color: #c084fc; transform: translateY(-4px); }
.card-poster {
    width: 100%; aspect-ratio: 2/3;
    object-fit: cover; display: block;
}
.card-poster-placeholder {
    width: 100%; aspect-ratio: 2/3;
    background: #1e1a2e;
    display: flex; align-items: center; justify-content: center;
    font-size: 2.5rem;
}
.card-body { padding: 0.9rem; }
.card-rank {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 1rem; color: #7c3aed; letter-spacing: 0.1em;
}
.card-title {
    font-family: 'Inter', sans-serif;
    font-weight: 600; font-size: 0.9rem;
    color: #e8e4dc; line-height: 1.3;
    margin: 0.2rem 0 0.4rem;
}
.card-meta {
    font-family: 'Inter', sans-serif;
    font-size: 0.7rem; color: #7c6fa0;
    margin-bottom: 0.5rem;
}
.card-genres {
    font-family: 'Inter', sans-serif;
    font-size: 0.65rem; color: #c084fc;
    letter-spacing: 0.08em; text-transform: uppercase;
    margin-bottom: 0.5rem;
}
.card-overview {
    font-family: 'Inter', sans-serif;
    font-size: 0.72rem; color: #7c6fa0;
    line-height: 1.5;
    display: -webkit-box;
    -webkit-line-clamp: 3;
    -webkit-box-orient: vertical;
    overflow: hidden;
}
.score-row {
    display: flex; align-items: center;
    gap: 0.5rem; margin-top: 0.6rem;
}
.score-label {
    font-family: 'Inter', sans-serif;
    font-size: 0.65rem; font-weight: 600;
    color: #c084fc; white-space: nowrap;
}
.score-bar-bg {
    flex: 1; height: 2px;
    background: #2a2440; border-radius: 2px;
}
.score-bar-fill {
    height: 2px;
    background: linear-gradient(90deg, #7c3aed, #c084fc);
    border-radius: 2px;
}
.star { color: #f59e0b; font-size: 0.7rem; }

/* Selected movie banner */
.selected-banner {
    display: flex; gap: 1.5rem; align-items: flex-start;
    background: #13111a; border: 1px solid #2a2440;
    border-radius: 8px; padding: 1.2rem;
    margin-bottom: 2rem;
}
.selected-poster { width: 80px; border-radius: 4px; flex-shrink: 0; }
.selected-info { flex: 1; }
.selected-title {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 1.6rem; letter-spacing: 0.05em;
    color: #e8e4dc; line-height: 1;
}
.selected-meta {
    font-family: 'Inter', sans-serif;
    font-size: 0.75rem; color: #7c6fa0; margin: 0.3rem 0 0.5rem;
}
.selected-overview {
    font-family: 'Inter', sans-serif;
    font-size: 0.78rem; color: #9d93b5; line-height: 1.55;
    display: -webkit-box;
    -webkit-line-clamp: 3;
    -webkit-box-orient: vertical;
    overflow: hidden;
}

/* Empty state */
.empty-state {
    text-align: center; padding: 3rem 1rem;
    color: #2a2440; font-family: 'Inter', sans-serif;
    font-size: 0.9rem; letter-spacing: 0.05em;
}
</style>
""", unsafe_allow_html=True)

# ── Hero ──────────────────────────────────────────────────────────────────────
st.markdown("""
<div class="hero">
    <p class="hero-title">CineMatch</p>
    <p class="hero-sub">Content-based movie recommendations · Powered by TMDB</p>
</div>
<div class="divider"></div>
""", unsafe_allow_html=True)

# ── Controls ──────────────────────────────────────────────────────────────────
col1, col2, col3 = st.columns([1, 2, 1])
with col2:
    st.markdown('<p class="select-label">Choose a movie</p>', unsafe_allow_html=True)
    selected = st.selectbox("", sorted(movies_df["title"].values), label_visibility="collapsed")
    st.markdown("<div style='height:0.6rem'></div>", unsafe_allow_html=True)
    clicked = st.button("Find Similar Movies")

# ── Results ───────────────────────────────────────────────────────────────────
if clicked:
    # Selected movie info banner
    sel_id   = int(movies_df[movies_df["title"] == selected]["id"].values[0])
    sel_info = fetch_movie_info(sel_id)

    poster_html = (
        f'<img class="selected-poster" src="{sel_info["poster"]}">'
        if sel_info["poster"]
        else '<div style="width:80px;height:120px;background:#1e1a2e;border-radius:4px;flex-shrink:0"></div>'
    )
    st.markdown(f"""
    <div class="selected-banner">
        {poster_html}
        <div class="selected-info">
            <div class="selected-title">{selected}</div>
            <div class="selected-meta">
                {'⭐ ' + str(round(sel_info['rating'],1)) + '&nbsp;&nbsp;·&nbsp;&nbsp;' if sel_info['rating'] else ''}
                {sel_info['year']}&nbsp;&nbsp;·&nbsp;&nbsp;{sel_info['genres']}
            </div>
            <div class="selected-overview">{sel_info['overview']}</div>
        </div>
    </div>
    """, unsafe_allow_html=True)

    st.markdown('<p class="results-heading">Top 5 similar movies</p>', unsafe_allow_html=True)

    with st.spinner("Fetching recommendations…"):
        results = recommend(selected)

    cols = st.columns(5, gap="small")
    for col, (rank, movie) in zip(cols, enumerate(results, 1)):
        with col:
            poster_tag = (
                f'<img class="card-poster" src="{movie["poster"]}">'
                if movie["poster"]
                else '<div class="card-poster-placeholder">🎬</div>'
            )
            score_pct = min(int(movie["score"] * 300), 100)
            stars = "⭐" * round(movie["rating"] / 2) if movie["rating"] else ""
            st.markdown(f"""
            <div class="movie-card">
                {poster_tag}
                <div class="card-body">
                    <div class="card-rank">#{rank:02d}</div>
                    <div class="card-title">{movie["title"]}</div>
                    <div class="card-meta">{stars} {round(movie['rating'],1) if movie['rating'] else '—'} &nbsp;·&nbsp; {movie['year']}</div>
                    <div class="card-genres">{movie['genres']}</div>
                    <div class="card-overview">{movie['overview']}</div>
                    <div class="score-row">
                        <span class="score-label">Match {movie['score']:.3f}</span>
                        <div class="score-bar-bg">
                            <div class="score-bar-fill" style="width:{score_pct}%"></div>
                        </div>
                    </div>
                </div>
            </div>
            """, unsafe_allow_html=True)

else:
    st.markdown("""
    <div class="empty-state">
        Select a movie above and hit <strong>Find Similar Movies</strong>
    </div>
    """, unsafe_allow_html=True)

