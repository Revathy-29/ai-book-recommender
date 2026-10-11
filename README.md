# AI Book Recommendation System

A web application that recommends books from a database of about 52,000 titles.
It supports recommendations by **genre, author, rating, similar book, first letter
and year of publication**, plus a **smart search** box that understands plain English,
for example: `fantasy like Dune after 1990 rated above 4`.

- **Database (DBMS):** SQLite, with indexes on title, author, genre, rating and year
- **AI:** text embeddings (TF-IDF + SVD by default, sentence-transformers optional),
  similarity search and Bayesian weighted-rating ranking
- **Web:** Flask + HTML/CSS (light and dark theme)

Full documentation: [PID](PID.md), [Technology Stack](TECH_STACK.md),
[Design Specification](DESIGN_SPECIFICATION.md).

## Quick start

1. Install Python 3.12 or later.
2. Install the libraries:
   ```
   pip install -r requirements.txt
   ```
3. Create the database (choose one):

   **Option A - small real sample (2,000 most-rated books, quick):**
   ```
   python book_db.py load-sql sample_books.sql
   ```
   **Option B - full dataset (about 52,000 books):**
   download `books_1.Best_Books_Ever.csv` from https://zenodo.org/records/4265096, then
   ```
   python book_db.py import books_1.Best_Books_Ever.csv
   ```
   (Or download the ready-made `books.db` from this repository's Releases page and
   place it next to `app.py`.)
4. Start the app:
   ```
   python app.py
   ```
   Open http://127.0.0.1:5000 in a browser. The first start builds the AI index
   (about 5 seconds for the sample, about 45 seconds for the full dataset).

## Files

| File | Purpose |
|------|---------|
| `app.py` | Flask web application |
| `ai_book_recommender.py` | AI engine: embeddings, similarity, ranking, query parser |
| `book_db.py` | SQLite schema, CSV importer, SQL export/load |
| `sample_books.sql` | Sample database script (2,000 real books) |

## Dataset

Full data: "Best Books Ever" Goodreads dataset (Zenodo, CC BY-NC 4.0). Used here for
non-commercial, academic purposes with attribution.
`sample_books.sql` holds the 2,000 most-rated books from that dataset.

## Author

<Revathy V, NSSCE>
