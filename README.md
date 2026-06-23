# Project_1


# 🎬 CineMatch — Movie Recommendation System

A content-based movie recommender that suggests similar movies based on what a movie is actually about — not just its genre.

---

## What it does

You pick a movie, it gives you 5 similar ones. That's it.

It works by combining each movie's overview, genres, keywords, cast, and director into a single tag, then using TF-IDF and Cosine Similarity to find the closest matches.

---

## Stack

- **ML** — scikit-learn, NLTK, Pandas
- **Backend** — FastAPI + Pydantic
- **Frontend** — Streamlit
- **Data** — TMDB 5000 Movies Dataset

---

## Setup

```bash
# 1. Clone
git clone https://github.com/souravffrana-create/movie_recommendation_system.git
cd movie_recommendation_system

# 2. Install
pip install -r requirements.txt

# 3. Generate similarity matrix
python src/train.py

# 4. Run backend
uvicorn api.main:app --reload

# 5. Run frontend
streamlit run frontend/app.py
```

> `similarity.pkl` is not included (175MB). Run `train.py` to generate it locally.

---

## API

```
POST /recommend
{"Title": "Inception"}

→ {"Recommendation": ["The Dark Knight", "Interstellar", ...]}
```

---

Built by **Sourav Rana** · souravffrana@gmail.com

