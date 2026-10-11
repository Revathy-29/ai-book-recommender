# Technology Stack

**Project:** AI Book Recommendation System

This document lists every technology used, why it was chosen, and what alternatives exist.

---

## 1. Stack Overview

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Language | Python 3.12 | All application and AI code |
| Database (DBMS) | SQLite 3 | Stores the 52,390-book catalogue |
| Data processing | pandas, NumPy | Loading, cleaning and filtering data; vector maths |
| AI / ML | scikit-learn (TF-IDF, TruncatedSVD) | Default text embeddings (works offline) |
| AI / ML (optional upgrade) | sentence-transformers (`all-MiniLM-L6-v2`) | Higher-quality semantic embeddings |
| Web framework | Flask 3 | Web server and routing |
| Templating | Jinja2 (built into Flask) | HTML pages with automatic escaping |
| Front end | HTML5 and CSS3 | Responsive interface with light and dark theme |
| Dataset | Public CSV (for example Kaggle "Best Books Ever") | Source of book records |

Versions the project was tested with: Python 3.12, Flask 3.1, pandas 3.0, scikit-learn 1.8, SQLite 3.45.

## 2. Component Details and Rationale

### 2.1 Python
- Standard language for data science and AI, with mature libraries.
- One language for the database layer, the AI engine and the web server keeps the project simple.

### 2.2 SQLite (DBMS)
- A real relational database with SQL, indexes and unique constraints.
- Needs no installation or server, which suits a student project.
- Comfortably handles 52,390 rows.
- Indexed columns: `title`, `author`, `genre`, `rating`, `year`.
- **Alternatives:** MySQL or PostgreSQL if the project must run as a multi-user server. Only the connection code and SQL placeholders in `book_db.py` would change.

### 2.3 pandas and NumPy
- pandas reads CSV files in chunks, cleans messy fields (genre lists, dates) and filters results.
- NumPy performs the fast vector operations behind similarity search (one matrix-vector product scores all books).

### 2.4 scikit-learn: default embedding model
- **TF-IDF** converts each book's text (title, author, genre, description) into weighted word features.
- **TruncatedSVD (LSA)** compresses those features into dense 256-dimension vectors that capture topic similarity.
- Runs on any CPU with no downloads, so the project works anywhere.

### 2.5 sentence-transformers: optional upgrade
- Pre-trained neural model that understands meaning, not just shared words.
- Install with `pip install sentence-transformers`. The engine detects it and switches automatically.
- Needs internet on first use to download the model (about 90 MB).
- Slower to index 52,390 books on a CPU (minutes), but results are noticeably better.

### 2.6 Flask and Jinja2
- Lightweight and easy to explain in a viewing or viva.
- Jinja2 escapes all output automatically, protecting against cross-site scripting.
- **Alternatives:** Streamlit (faster to build but less control over layout), FastAPI (better if a separate mobile app will call the engine as an API).

### 2.7 HTML and CSS
- A single server-rendered page with tabs; no JavaScript framework is needed.
- CSS variables provide a light theme and an automatic dark theme.
- Responsive layout works on desktop and phone browsers.

## 3. Libraries Required

```
flask
pandas
numpy
scikit-learn
sentence-transformers   # optional, for better AI quality
```

Install:
```
pip install flask pandas scikit-learn
pip install sentence-transformers   # optional
```

## 4. Development Tools
- Any editor (VS Code recommended)
- Git for version control (recommended)
- Web browser for testing (Chrome, Edge or Firefox)
- DB Browser for SQLite (optional) to inspect `books.db`

## 5. Hardware and Environment Requirements

| Item | Minimum | Recommended |
|------|---------|-------------|
| CPU | Dual-core | Quad-core |
| RAM | 4 GB | 8 GB |
| Disk | 500 MB | 1 GB |
| GPU | Not required | Optional, speeds up sentence-transformers |
| OS | Windows, macOS or Linux | Any |

## 6. Measured Performance (52,390 books, default model)
- Index build at start-up: about 43 seconds
- Typical search: about 0.02 seconds
- Memory for embeddings: about 50 MB

These figures were measured on generated test data of the same size. Real data and the sentence-transformers model will take longer to build.

## 7. Technology Decisions Summary

| Decision | Choice | Main reason |
|----------|--------|-------------|
| Database | SQLite | Real DBMS, zero setup |
| Web | Flask web app, not mobile | Simpler and faster to build; matches requirement |
| Default AI | TF-IDF + SVD | Works offline on any laptop |
| Quality AI | sentence-transformers | Best accuracy with minimal code |
| Ranking | Similarity blended with Bayesian rating | Prevents low-vote books from topping lists |
