# Design Specification

**Project:** AI Book Recommendation System
**Version:** 1.0

---

## 1. Introduction

This document describes how the system is designed: its architecture, data model, AI design, module interfaces, web interface and testing approach. Requirements and scope are in `PID.md`; technologies are in `TECH_STACK.md`.

## 2. System Architecture

The system has three layers.

```
 +--------------------------------------------------------+
 |  Presentation layer   (app.py - Flask + Jinja2 + CSS)   |
 |  Tabs: Smart | Genre | Author | Rating | Similar |       |
 |        First letter | Year                               |
 +----------------------------+---------------------------+
                              |  method calls
 +----------------------------v---------------------------+
 |  AI engine   (ai_book_recommender.py)                    |
 |  Embedder -> vectors -> similarity search -> ranking     |
 |  + natural-language query parser                         |
 +----------------------------+---------------------------+
                              |  load at start-up
 +----------------------------v---------------------------+
 |  Data layer   (book_db.py - SQLite, books.db)            |
 |  Schema, indexes, CSV import, loading                    |
 +--------------------------------------------------------+
```

**Start-up flow:** the web app loads all books from the database, builds embeddings once, and keeps them in memory. Each search then runs in milliseconds.

## 3. Data Design

### 3.1 Database table `books`

| Column | Type | Constraint | Description |
|--------|------|-----------|-------------|
| `id` | INTEGER | Primary key, auto-increment | Unique book ID |
| `title` | TEXT | NOT NULL | Book title |
| `author` | TEXT | | Author name(s) |
| `genre` | TEXT | | Genres, comma separated |
| `rating` | REAL | Default 0 | Average rating, 0 to 5 |
| `ratings_count` | INTEGER | Default 1 | Number of ratings |
| `year` | INTEGER | Default 0 | Publication year (0 = unknown) |
| `description` | TEXT | | Book summary |

- **Unique constraint:** (`title`, `author`) prevents duplicate books on import.
- **Indexes:** `title`, `author`, `genre`, `rating`, `year`.

### 3.2 Data cleaning rules (import)
- Column headers matched case-insensitively against known aliases (for example `averageRating` to `rating`, `numRatings` to `ratings_count`).
- Genre lists such as `['Fantasy', 'Young Adult']` become `Fantasy, Young Adult`.
- Year is extracted from date text (for example `09/14/2008` becomes 2008).
- Rating counts with commas (`12,345`) become integers.
- Missing author becomes "Unknown"; missing numbers become 0 or 1.
- Rows without a title are skipped; malformed CSV lines are skipped.
- Import runs in chunks of 52,390 rows to limit memory use.

## 4. AI Design

### 4.1 Overview
Every book and every query is converted into a vector (an embedding). Books whose vectors point in similar directions are similar in meaning. Results are then re-ranked to favour well-rated, widely rated books.

### 4.2 Book representation
Each book becomes one text string:

```
title. author. genre. genre. description
```

The genre is repeated to give it extra weight.

### 4.3 Embedding models
| Model | When used | Output |
|-------|-----------|--------|
| TF-IDF + TruncatedSVD (LSA) | Default, offline | 256-dimension normalised vectors |
| sentence-transformers `all-MiniLM-L6-v2` | If installed | 384-dimension normalised vectors |

All vectors are L2-normalised, so a dot product equals cosine similarity.

### 4.4 Similarity search
For a query vector `q`, the similarity of every book is `emb @ q`, a single matrix-vector product over all books. Top results are selected with a partial sort.

### 4.5 Quality score (Bayesian weighted rating)
A raw average is misleading when a book has few votes. The engine uses:

```
WR   = ( v / (v + m) ) * R  +  ( m / (v + m) ) * C
qual = WR / 5
```

- `R` = the book's average rating
- `v` = the book's number of ratings
- `m` = the 60th percentile of rating counts (minimum votes to be trusted)
- `C` = mean rating across all books

### 4.6 Final ranking score
```
score = 0.75 * similarity + 0.25 * qual
```
When a request has no semantic part (for example rating only, year only, or first letter only), similarity is skipped and results are ranked by `qual`.

### 4.7 Natural-language query parser
The smart search box extracts structured conditions with pattern matching, then sends the remaining words to the embedding model.

| Phrase pattern | Extracted condition |
|----------------|--------------------|
| `after / since / from 1990` | year >= 1990 |
| `before / until 2000` | year <= 2000 |
| `in / published in 1997` | year = 1997 |
| `between 2000 and 2012`, `2000-2012` | year range |
| `rated above 4`, `4+`, `4 stars`, `at least 4.5` | rating >= value |
| `starting with T` | title starts with "t" |
| `by Tolkien` | author contains "tolkien" |
| `books like X`, `similar to X` | similar-book search for X |
| Remaining words (for example "dark fantasy") | semantic query |

### 4.8 Function design for each request type

| Request | Method | Logic |
|---------|--------|-------|
| Genre | `by_genre(genre, k)` | Exact genre tag match as a filter, ranked semantically; if no tag matches, pure semantic search |
| Author | `by_author(author, k)` | The author's books, plus books nearest to the centroid of that author's embeddings (labelled "readers of this author also like"); unknown names fall back to semantic search |
| Rating | `by_rating(min, max, k)` | Range filter, ranked by quality score |
| Similar | `similar_to(title, k)` | Find the title (exact, prefix, contains, then semantic fallback), return nearest neighbours, excluding the book itself |
| First letter | `by_first_letter(letter, k)` | Prefix filter on lower-cased title, ranked by quality |
| Year | `by_year(from, to, k)` | Range filter, ranked by quality |
| Smart | `search(text, k)` | Parse, filter, then rank semantically |

An extra method, `for_reader(liked_titles)`, recommends from the average of several liked books. It is included in the engine for future personalisation but has no web tab in version 1.0.

## 5. Web Interface Design

### 5.1 Routes
| Route | Method | Parameters | Description |
|-------|--------|-----------|-------------|
| `/` | GET | `mode`, `q`, `lo`, `hi`, `n` | Shows the selected tab and, if input is present, the results |

`mode` is one of `smart`, `genre`, `author`, `rating`, `similar`, `letter`, `year`. `n` (number of results) is limited to 1 to 500.

### 5.2 Page layout
1. **Header:** title, number of books in the database, and the AI model in use
2. **Tab bar:** seven rounded buttons, the active one highlighted
3. **Input form:** fields depend on the active tab (see table below), plus "How many" and a Recommend button
4. **Results card:** note line, result count and table

### 5.3 Form fields per tab
| Tab | Fields |
|-----|--------|
| Smart search | One text box for plain-English requests |
| Genre | Genre text |
| Author | Author name |
| Rating | Minimum rating, maximum rating |
| Similar books | Book title |
| First letter | One letter (maximum length 1) |
| Year | Year (or start year), optional end year |

### 5.4 Results table
Columns: **#, Title, Author, Genre, Rating, Year**. For author searches an extra **Why** column shows "by this author" or "readers of this author also like". The table scrolls sideways on narrow screens. Unknown years display as "-".

### 5.5 Visual design
- Clean card-based layout, maximum width about 980 px, centred.
- System font stack for fast loading.
- One accent colour for the active tab and buttons.
- Automatic dark mode through `prefers-color-scheme`.
- Responsive: forms wrap and tables scroll on phones.

### 5.6 Error and empty states
- Invalid numbers show the message "Please enter valid numbers."
- No matches show "No books found. Try different input."
- A similar-book search shows which book was matched, so a wrong fuzzy match is visible.

## 6. Module Design

| Module | Key elements |
|--------|--------------|
| `book_db.py` | `connect()`, `insert_books()`, `import_csv()`, `load_books()`, `count()`, command-line `import`, `seed`, `count` |
| `ai_book_recommender.py` | `Embedder`, `AIBookRecommender`, `synthetic_books()`, `demo_data()`, text interface |
| `app.py` | `create_app()`, page template, route logic |

## 7. Non-Functional Design

### 7.1 Performance
- Index is built once at start-up and cached in memory.
- Measured with 52,390 generated books and the default model: build about 43 s, search about 0.02 s.
- Memory for 52,390 vectors of 256 floats is about 50 MB.

### 7.2 Security
- All SQL uses parameterised queries; user input is never concatenated into SQL.
- Jinja2 auto-escapes output, preventing cross-site scripting.
- Input limits: result count capped at 500; letter field limited to one character.
- The development server runs with debug mode off.
- Version 1.0 has no login. It is intended for local use, not public hosting.

### 7.3 Reliability
- Bad CSV lines are skipped during import.
- Duplicate books are ignored through the unique constraint.
- Invalid user input returns an error message instead of crashing.

### 7.4 Scalability
- Search is a single vector-matrix product, so it scales to millions of books with enough memory.
- Beyond that, an approximate nearest-neighbour index (such as FAISS) can replace the product without changing the interface.

## 8. Test Plan

| ID | Test | Expected result |
|----|------|----------------|
| T1 | Genre "fantasy", 100 results | 100 rows, fantasy books first |
| T2 | Author with known books | Their books plus labelled similar-author books |
| T3 | Rating 4.5 to 5 | Every row rated 4.5 or higher |
| T4 | Similar to a known title | Related books; the matched title is shown |
| T5 | Similar to a misspelled title | Still matches the intended book |
| T6 | First letter "G" | Every title starts with G |
| T7 | Year range 1990 to 2000 | Every year within the range |
| T8 | Smart: "horror after 2000 rated above 4 starting with G" | All conditions satisfied |
| T9 | Year field contains letters | Error message, no crash |
| T10 | Empty result (unknown genre or author) | "No books found" or semantic fallback |
| T11 | CSV import with Goodreads-style headers | Rows imported with cleaned genres, years and counts |
| T12 | Re-import same CSV | No duplicates added |
| T13 | 52,390 books | Search completes in under one second |

Tests T1 to T13 were run against generated data. They should be repeated on the real dataset.

## 9. Known Limitations
- Quality of recommendations depends on dataset quality.
- The default model matches topics and shared words, not deep meaning; the sentence-transformers option improves this.
- Smart search handles common phrasing, not every possible sentence.
- No personalisation or user accounts.
- The interface was verified through automated page requests, not on every browser.

## 10. Future Design Extensions
- Web tab for personalised recommendations using `for_reader`
- User accounts and reading history stored in new database tables
- Collaborative filtering from user rating data
- REST API (FastAPI) to support a mobile app
- Cover images and external links
- Approximate nearest-neighbour index for millions of books
