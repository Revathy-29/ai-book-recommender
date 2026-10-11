# Project Initiation Document (PID)

**Project title:** AI Book Recommendation System
**Document version:** 1.0
**Project type:** Academic / portfolio project (web application)

---

## 1. Project Background

Readers face a very large number of books and often struggle to find the next one to read. Traditional library or store search relies on exact keywords and cannot understand intent, such as "fantasy books like Dune published after 1990 rated above 4".

This project builds an AI-based web application that recommends books from a database of about 52,390 titles. It supports six types of request, each usable on its own, plus a natural-language search box that combines them.

## 2. Project Objectives

| # | Objective | Measure of success |
|---|-----------|--------------------|
| O1 | Recommend a large number of books for a given **genre** | Returns up to 500 ranked books for a genre |
| O2 | Recommend books for a given **author** | Returns the author's books plus books by similar authors |
| O3 | Recommend books for a given **rating** (or range) | Returns only books inside the requested range |
| O4 | Recommend **similar books** for a given book name | Returns the closest books by AI similarity; tolerates typos |
| O5 | Recommend books by **first letter** of title | Returns only titles starting with that letter |
| O6 | Recommend books by **year of publication** (or range) | Returns only books in the requested year(s) |
| O7 | Store all book data in a **DBMS** | Data held in a relational database with indexes |
| O8 | Provide a **web** interface | Accessible in a browser, one tab per request type |
| O9 | Use **AI** at the core of recommendation | Embeddings, semantic search and learned-quality ranking drive all results |

## 3. Scope

### 3.1 In scope
- Dataset of approximately 52,390 books stored in a relational database (SQLite)
- CSV import tool to load a public dataset (for example Goodreads-style data)
- AI engine: text embeddings, similarity search, Bayesian weighted-rating ranking, natural-language query parsing
- Web interface with separate tabs for genre, author, rating, similar books, first letter, year and smart search
- Adjustable number of results (1 to 500)
- Documentation: PID, Tech Stack, Design Specification

### 3.2 Out of scope (version 1.0)
- Native mobile application
- User accounts, login and personal reading history
- Collaborative filtering (needs per-user rating data)
- Book cover images and purchase or borrowing links
- Cloud deployment and multi-user scaling

## 4. Deliverables

| Deliverable | File |
|-------------|------|
| AI recommendation engine | `ai_book_recommender.py` |
| Database layer and CSV importer | `book_db.py` |
| Web application | `app.py` |
| Project Initiation Document | `PID.md` |
| Technology Stack document | `TECH_STACK.md` |
| Design Specification | `DESIGN_SPECIFICATION.md` |

## 5. Stakeholders

| Role | Responsibility |
|------|----------------|
| Student / developer | Builds, tests and presents the system |
| Project guide / examiner | Reviews and evaluates the project |
| End users (readers) | Use the web app to find books |

## 6. Assumptions
- A public dataset of roughly 52,390 books (for example "Best Books Ever" from Kaggle) will be downloaded by the developer and checked for size and column names before import.
- The project runs on a standard laptop with Python 3.12 or later.
- Internet access is available during setup to install libraries (and optionally a higher-quality AI model).

## 7. Constraints
- Zero budget: only free and open-source tools.
- Single developer.
- Dataset limited to about 52,390 books.
- No GPU assumed. The default AI model must run on a normal CPU.

## 8. Project Plan (indicative)

| Phase | Activities | Duration |
|-------|-----------|----------|
| 1. Initiation | Requirements, PID, tech selection | 2 days |
| 2. Data | Download dataset, import to database, clean data | 2-3 days |
| 3. AI engine | Embeddings, similarity, ranking, query parsing | 4-5 days |
| 4. Web interface | Flask routes, tabs, results table | 3-4 days |
| 5. Testing | Functional, performance and edge-case tests | 2-3 days |
| 6. Documentation and demo | Final documents, presentation | 2 days |
| **Total** | | **about 3-4 weeks** |

## 9. Risks and Mitigation

| ID | Risk | Likelihood | Impact | Mitigation |
|----|------|-----------|--------|-----------|
| R1 | Dataset column names differ from expected | Medium | Low | Importer recognises common header aliases; add more as needed |
| R2 | Poor recommendation quality with the basic offline model | Medium | Medium | Upgrade to sentence-transformers embeddings |
| R3 | Dataset has missing descriptions or genres, reducing quality | High | Medium | Clean at import; fall back to title and author text |
| R4 | Slow index build with the high-quality model on CPU | Medium | Low | Build once at start-up; allow several minutes for 52,390 books |
| R5 | Ambiguous natural-language queries parsed incorrectly | Medium | Low | Each request type also has its own dedicated tab |
| R6 | Fuzzy title matching returns the wrong book | Low | Low | Page shows which book was matched |

## 10. Quality and Acceptance Criteria

The project is accepted when:
1. All six request types return correct results, individually and via smart search.
2. All book data is read from the database.
3. A search on 52,390 books responds in under one second.
4. Invalid input (for example letters where a number is expected) shows a clear error and does not crash the application.
5. The application runs from the documented commands on a clean machine.

## 11. Success Criteria

- Functional completeness against objectives O1 to O9
- Fast, stable behaviour on 52,390 books
- Clear documentation that lets a reviewer set up and run the project

## 12. Known Limitations (version 1.0)
- Recommendation quality depends on the dataset's description and genre quality.
- Without the optional sentence-transformers model, the AI uses a simpler statistical embedding (TF-IDF with SVD).
- Recommendations are not personalised per user.
- Natural-language parsing is rule-based and covers common phrasing only.

## 13. Future Enhancements
- Personalised recommendations from user reading history
- Collaborative filtering using user rating data
- Book covers and external links
- Mobile app using the same engine behind an API
- Deployment to a cloud host
