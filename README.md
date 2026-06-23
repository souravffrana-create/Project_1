# Project_1


import streamlit as st
import pickle
from pathlib import Path

# ── Page config ───────────────────────────────────────────────────────────────
st.set_page_config(page_title="CineMatch", page_icon="🎬", layout="wide")

# ── Load models ───────────────────────────────────────────────────────────────
BASE_DIR  = Path(__file__).resolve().parent.parent
MODEL_DIR = BASE_DIR / "models"

@st.cache_resource
def load_models():
    movies     = pickle.load(open(MODEL_DIR / "movies.pkl",     "rb"))
    similarity = pickle.load(open(MODEL_DIR / "similarity.pkl", "rb"))
    return movies, similarity

movies_df, similarity = load_models()

# ── Recommend ─────────────────────────────────────────────────────────────────
def recommend(movie_title):
    idx    = movies_df[movies_df["title"] == movie_title].index[0]
    scores = sorted(enumerate(similarity[idx]), key=lambda x: x[1], reverse=True)[1:6]
    return [(movies_df.iloc[i].title, round(s, 4)) for i, s in scores]

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

.hero { text-align: center; padding: 3rem 1rem 0.5rem; }
.hero-title {
    font-family: 'Bebas Neue', sans-serif;
    font-size: clamp(3.5rem, 10vw, 7rem);
    letter-spacing: 0.08em; line-height: 1;
    background: linear-gradient(135deg, #e8e4dc 30%, #c084fc 100%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text; margin: 0;
}
.hero-sub {
    font-family: 'Inter', sans-serif;
    font-weight: 300; font-size: 0.95rem;
    letter-spacing: 0.25em; text-transform: uppercase;
    color: #7c6fa0; margin-top: 0.4rem;
}
.divider {
    width: 60px; height: 2px;
    background: linear-gradient(90deg, #c084fc, transparent);
    margin: 1.8rem auto;
}
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

[data-testid="stButton"] > button {
    background: linear-gradient(135deg, #7c3aed, #c084fc) !important;
    color: #fff !important; border: none !important;
    border-radius: 3px !important;
    font-family: 'Inter', sans-serif !important;
    font-weight: 600 !important; font-size: 0.8rem !important;
    letter-spacing: 0.15em !important; text-transform: uppercase !important;
    padding: 0.65rem 2.5rem !important; width: 100% !important;
}
[data-testid="stButton"] > button:hover { opacity: 0.85 !important; }

.results-heading {
    font-family: 'Inter', sans-serif;
    font-size: 0.65rem; font-weight: 600;
    letter-spacing: 0.3em; text-transform: uppercase;
    color: #7c6fa0; text-align: center;
    margin: 2.5rem 0 1.5rem;
}

/* Selected movie banner */
.sel-banner {
    display: flex; gap: 1.2rem; align-items: center;
    background: #13111a; border: 1px solid #2a2440;
    border-left: 3px solid #c084fc;
    border-radius: 6px; padding: 1.2rem 1.4rem;
    margin-bottom: 2rem;
}
.sel-icon { font-size: 2.5rem; flex-shrink: 0; }
.sel-title {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 1.8rem; letter-spacing: 0.05em;
    color: #e8e4dc; line-height: 1;
}
.sel-sub {
    font-family: 'Inter', sans-serif;
    font-size: 0.75rem; color: #7c6fa0;
    margin-top: 0.3rem; letter-spacing: 0.05em;
}

/* Cards */
.card-grid {
    display: flex; gap: 1rem; flex-wrap: wrap;
    justify-content: center; padding-bottom: 3rem;
}
.movie-card {
    background: #13111a;
    border: 1px solid #2a2440;
    border-radius: 6px; padding: 1.4rem 1.2rem;
    width: 190px;
    transition: border-color 0.2s, transform 0.2s;
}
.movie-card:hover { border-color: #c084fc; transform: translateY(-3px); }
.card-rank {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 2.2rem; color: #2a2440; line-height: 1;
    margin-bottom: 0.4rem;
}
.card-title {
    font-family: 'Inter', sans-serif;
    font-weight: 600; font-size: 0.95rem;
    color: #e8e4dc; line-height: 1.35;
    margin-bottom: 0.8rem;
}
.card-score {
    font-family: 'Inter', sans-serif;
    font-size: 0.7rem; font-weight: 600;
    letter-spacing: 0.1em; color: #c084fc;
}
.score-bar-bg {
    height: 2px; background: #2a2440;
    border-radius: 2px; margin-top: 0.35rem;
}
.score-bar-fill {
    height: 2px;
    background: linear-gradient(90deg, #7c3aed, #c084fc);
    border-radius: 2px;
}
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
    <p class="hero-sub">Content-based movie recommendations · TF-IDF + Cosine Similarity</p>
</div>
<div class="divider"></div>
""", unsafe_allow_html=True)

# ── Controls ──────────────────────────────────────────────────────────────────
col1, col2, col3 = st.columns([1, 2, 1])
with col2:
    st.markdown('<p class="select-label">Choose a movie</p>', unsafe_allow_html=True)
    selected = st.selectbox(
        "Choose a movie",
        sorted(movies_df["title"].values),
        label_visibility="collapsed"
    )
    st.markdown("<div style='height:0.6rem'></div>", unsafe_allow_html=True)
    clicked = st.button("Find Similar Movies")

# ── Results ───────────────────────────────────────────────────────────────────
if clicked:
    # Selected movie banner
    st.markdown(f"""
    <div class="sel-banner">
        <div class="sel-icon">🎬</div>
        <div>
            <div class="sel-title">{selected}</div>
            <div class="sel-sub">Finding movies similar to this selection…</div>
        </div>
    </div>
    """, unsafe_allow_html=True)

    st.markdown('<p class="results-heading">Top 5 similar movies</p>',
                unsafe_allow_html=True)

    results = recommend(selected)

    cards_html = '<div class="card-grid">'
    for rank, (title, score) in enumerate(results, 1):
        pct = min(int(score * 300), 100)
        cards_html += f"""
        <div class="movie-card">
            <div class="card-rank">0{rank}</div>
            <div class="card-title">{title}</div>
            <div class="card-score">Match · {score:.4f}</div>
            <div class="score-bar-bg">
                <div class="score-bar-fill" style="width:{pct}%"></div>
            </div>
        </div>"""
    cards_html += "</div>"
    st.markdown(cards_html, unsafe_allow_html=True)

else:
    st.markdown("""
    <div class="empty-state">
        Select a movie above and hit <strong>Find Similar Movies</strong>
    </div>
    """, unsafe_allow_html=True)
