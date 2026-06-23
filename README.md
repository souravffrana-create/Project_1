# Project_1



import streamlit as st
import requests
import pickle
from pathlib import Path

# ── Config ────────────────────────────────────────────────────────────────────
API_URL = "http://localhost:8000"

st.set_page_config(page_title="CineMatch", page_icon="🎬", layout="wide")

# ── Load movie list directly from pickle (for selectbox) ──────────────────────
BASE_DIR  = Path(__file__).resolve().parent.parent
MODEL_DIR = BASE_DIR / "models"

@st.cache_resource
def load_movies():
    movies = pickle.load(open(MODEL_DIR / "movies.pkl", "rb"))
    return sorted(movies["title"].tolist())

# ── Check API health ──────────────────────────────────────────────────────────
def check_health():
    try:
        r = requests.get(f"{API_URL}/health", timeout=3)
        return r.json().get("Model Status") == "OK"
    except Exception:
        return False

# ── Recommend via API ─────────────────────────────────────────────────────────
def get_recommendations(movie_title):
    r = requests.post(
        f"{API_URL}/recommend",
        json={"input_movie": movie_title},   # ← tera Pydantic model
        timeout=10
    )
    if r.status_code == 200:
        return r.json().get("Recommendation", [])
    raise Exception(r.text)

# ── CSS ───────────────────────────────────────────────────────────────────────
st.markdown("""
<style>
@import url('https://fonts.googleapis.com/css2?family=Bebas+Neue&family=Inter:wght@300;400;500;600&display=swap');

html, body, [data-testid="stAppViewContainer"] { background: #0a0a0f; color: #e8e4dc; }
[data-testid="stAppViewContainer"] {
    background: radial-gradient(ellipse at 20% 10%, #1a0a2e 0%, #0a0a0f 60%);
}
[data-testid="stHeader"] { background: transparent; }
#MainMenu, footer, header { visibility: hidden; }

/* Hero */
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
    font-family: 'Inter', sans-serif; font-weight: 300;
    font-size: 0.95rem; letter-spacing: 0.25em;
    text-transform: uppercase; color: #7c6fa0; margin-top: 0.4rem;
}
.divider {
    width: 60px; height: 2px;
    background: linear-gradient(90deg, #c084fc, transparent);
    margin: 1.8rem auto;
}

/* Status badge */
.badge {
    display: inline-block;
    font-family: 'Inter', sans-serif;
    font-size: 0.65rem; font-weight: 600;
    letter-spacing: 0.15em; text-transform: uppercase;
    padding: 0.3rem 0.8rem; border-radius: 20px;
    margin-bottom: 0.5rem;
}
.badge-ok  { background: #052e16; color: #4ade80; border: 1px solid #166534; }
.badge-err { background: #1a0a0a; color: #f87171; border: 1px solid #7f1d1d; }

/* Selectbox */
.select-label {
    font-family: 'Inter', sans-serif; font-size: 0.7rem;
    font-weight: 600; letter-spacing: 0.2em;
    text-transform: uppercase; color: #7c6fa0; margin-bottom: 0.4rem;
}
[data-testid="stSelectbox"] > div > div {
    background: #13111a !important; border: 1px solid #2a2440 !important;
    border-radius: 4px !important; color: #e8e4dc !important;
    font-family: 'Inter', sans-serif !important;
}
[data-testid="stSelectbox"] > div > div:hover { border-color: #c084fc !important; }

/* Button */
[data-testid="stButton"] > button {
    background: linear-gradient(135deg, #7c3aed, #c084fc) !important;
    color: #fff !important; border: none !important;
    border-radius: 3px !important; font-family: 'Inter', sans-serif !important;
    font-weight: 600 !important; font-size: 0.8rem !important;
    letter-spacing: 0.15em !important; text-transform: uppercase !important;
    padding: 0.65rem 2.5rem !important; width: 100% !important;
}
[data-testid="stButton"] > button:hover { opacity: 0.85 !important; }

/* Selected banner */
.sel-banner {
    display: flex; align-items: center; gap: 1rem;
    background: #13111a; border: 1px solid #2a2440;
    border-left: 3px solid #c084fc; border-radius: 6px;
    padding: 1rem 1.3rem; margin-bottom: 1.5rem;
}
.sel-title {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 1.6rem; letter-spacing: 0.04em; color: #e8e4dc;
}
.sel-sub { font-size: 0.7rem; color: #7c6fa0; margin-top: 0.2rem; }

/* Results heading */
.results-heading {
    font-size: 0.65rem; font-weight: 600;
    letter-spacing: 0.3em; text-transform: uppercase;
    color: #7c6fa0; text-align: center;
    margin: 1.5rem 0 1.2rem;
}

/* Movie cards */
.movie-card {
    background: #13111a; border: 1px solid #2a2440;
    border-radius: 6px; padding: 1.2rem; height: 100%;
}
.movie-card:hover { border-color: #c084fc; }
.card-rank {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 2rem; color: #2a2440; line-height: 1; margin-bottom: 0.3rem;
}
.card-title {
    font-family: 'Inter', sans-serif; font-weight: 600;
    font-size: 0.9rem; color: #e8e4dc;
    line-height: 1.35; min-height: 2.8em;
}
.card-line {
    height: 1px; background: #2a2440;
    margin: 0.7rem 0;
}
.card-tag {
    font-size: 0.62rem; font-weight: 600;
    letter-spacing: 0.12em; text-transform: uppercase;
    color: #c084fc;
}

/* Error box */
.error-box {
    background: #1a0a0a; border: 1px solid #7f1d1d;
    border-radius: 6px; padding: 1rem 1.3rem;
    font-size: 0.85rem; color: #f87171; text-align: center;
}

/* Empty state */
.empty {
    text-align: center; padding: 4rem 1rem;
    color: #2a2440; font-size: 0.85rem;
    letter-spacing: 0.06em;
}
</style>
""", unsafe_allow_html=True)

# ── Hero ──────────────────────────────────────────────────────────────────────
st.markdown("""
<div class="hero">
    <p class="hero-title">CineMatch</p>
    <p class="hero-sub">Content-based recommendations · TF-IDF + Cosine Similarity</p>
</div>
<div class="divider"></div>
""", unsafe_allow_html=True)

# ── API Health badge ──────────────────────────────────────────────────────────
col_l, col_m, col_r = st.columns([1, 2, 1])
with col_m:
    api_ok = check_health()
    if api_ok:
        st.markdown('<div style="text-align:center"><span class="badge badge-ok">● API Online</span></div>',
                    unsafe_allow_html=True)
    else:
        st.markdown('<div style="text-align:center"><span class="badge badge-err">● API Offline — run uvicorn api.main:app</span></div>',
                    unsafe_allow_html=True)

# ── Movie selectbox ───────────────────────────────────────────────────────────
with col_m:
    st.markdown("<div style='height:0.8rem'></div>", unsafe_allow_html=True)
    st.markdown('<p class="select-label">Choose a movie</p>', unsafe_allow_html=True)

    try:
        movies = load_movies()
    except Exception:
        movies = ["API not connected"]

    selected = st.selectbox(
        "Choose a movie",
        movies,
        label_visibility="collapsed"
    )
    st.markdown("<div style='height:0.6rem'></div>", unsafe_allow_html=True)
    clicked = st.button("Find Similar Movies", disabled=not api_ok)

# ── Results ───────────────────────────────────────────────────────────────────
if clicked:
    with st.spinner("Finding similar movies…"):
        try:
            results = get_recommendations(selected)

            # Selected banner
            st.markdown(f"""
            <div class="sel-banner">
                <span style="font-size:1.8rem">🎬</span>
                <div>
                    <div class="sel-title">{selected.upper()}</div>
                    <div class="sel-sub">Showing 5 content-based recommendations via FastAPI</div>
                </div>
            </div>
            """, unsafe_allow_html=True)

            st.markdown('<p class="results-heading">Top 5 Similar Movies</p>',
                        unsafe_allow_html=True)

            # 5 cards
            cols = st.columns(5, gap="small")
            for col, (rank, title) in zip(cols, enumerate(results, 1)):
                with col:
                    st.markdown(f"""
                    <div class="movie-card">
                        <div class="card-rank">0{rank}</div>
                        <div class="card-title">{title}</div>
                        <div class="card-line"></div>
                        <div class="card-tag">Similar Match</div>
                    </div>
                    """, unsafe_allow_html=True)

        except requests.exceptions.ConnectionError:
            st.markdown("""
            <div class="error-box">
                ❌ FastAPI server nahi chal raha!<br>
                Terminal mein run karo: <strong>uvicorn api.main:app --reload</strong>
            </div>
            """, unsafe_allow_html=True)

        except Exception as e:
            st.markdown(f"""
            <div class="error-box">❌ Error: {str(e)}</div>
            """, unsafe_allow_html=True)

else:
    st.markdown("""
    <div class="empty">
        🎞️ &nbsp; Select a movie and hit <strong>Find Similar Movies</strong>
    </div>
    """, unsafe_allow_html=True)
