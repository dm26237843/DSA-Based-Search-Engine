# 🔎 DSA Search Engine

A minimal **DSA problem search engine** that aggregates problems from popular coding sites and lets you search them using a lightweight TF‑IDF based ranking. It ships with a tiny web UI, a Node/Express server, and Puppeteer-based scrapers to collect problem metadata.

> **Status:** Prototype / WIP. Some files in the provided snapshot contain placeholder sections (`...`). This README documents the intended design and how to run what’s present today, plus what to finish next.

---

## ✨ Features

- **Unified corpus** of DSA problems (currently Codeforces + LeetCode in this snapshot).
- **Search** powered by `natural` (TF‑IDF) with simple stopword removal and tokenization.
- **Tiny web UI** (vanilla HTML/CSS/JS) for instant search.
- **Scrapers** built with `puppeteer` that output JSON under `problems/`, then merged to `corpus/all_problems.json`.
- **Zero DB** — everything runs from JSON on disk.

---

## 🗂 Project Structure

```text
.
gitignore.txt
index.html
index.js
package-lock.json
package.json
scrape.js
script.js
styles.css
assets
assets/logos
assets/logos/codechef.png
assets/logos/codeforces.png
assets/logos/leetcode.png
corpus
corpus/all_problems.json
problems
problems/codeforces_problems.json
problems/leetcode_problems.json
utils
utils/merge.js
utils/preprocess.js
```

Key pieces:

- `index.js` — Express server, builds TF‑IDF index from `corpus/all_problems.json` and exposes `/search`.
- `index.html`, `styles.css`, `script.js` — Slim client that posts the search query and renders ranked results.
- `scrape.js` — Puppeteer scripts to fetch problem lists from sources (LeetCode + Codeforces in this snapshot).
- `utils/preprocess.js` — Lowercasing, basic cleanup, English stopword removal (`stopword` package).
- `utils/merge.js` — Merges `problems/*.json` into `corpus/all_problems.json`.

> ⚠️ **Note on placeholders:** several files include `...` which indicates truncated code in this snapshot (e.g., parts of request/response handling, scraping loops). See the **Roadmap / TODO** below to complete them.

---

## 📦 Prerequisites

- **Node.js 18+** (ESM project: `"type": "module"` in `package.json`)
- **Chromium** (handled automatically by Puppeteer on first run; corporate environments may require setting `PUPPETEER_EXECUTABLE_PATH`)

---

## 🚀 Quick Start

1) **Install dependencies**
```bash
npm install
```

2) **(Optional) Scrape sources**  
If you want to regenerate the problem lists:
```bash
# Run the scrapers (headless is set to false in this snapshot; see scrape.js)
node scrape.js
```

This should create / refresh:
```
problems/leetcode_problems.json
problems/codeforces_problems.json
```

3) **Merge into a single corpus**
```bash
npm run merge
# writes corpus/all_problems.json
```

4) **Start the server**
```bash
npm start
# or: npm run dev    # with nodemon (if defined in your package.json)
```

5) **Open the UI**
```
http://localhost:3000
```
Type a topic, tag, or title (e.g., `binary search`, `graph shortest path`, `dp knapsack`) and hit **Search**.

---

## 🧠 How Search Works

- Each problem document is built from `{title, description, url}`.
- Text is **preprocessed** (lowercased, non‑alphanumerics stripped, stopwords removed).
- We build a **TF‑IDF** index (`natural.TfIdf`) over all documents.
- The `/search` endpoint computes TF‑IDF similarity for the given query and returns the top matches.

> The client expects each result to include at least: `title`, `url`, `description`, and a derived `platform` label (computed from the URL). A `score` may also be included by the server for debugging / ordering, but the UI doesn’t require it.

---

## 🧩 API

> The code in this snapshot is partially truncated (…); adjust below if your local `index.js` differs.

### `POST /search`
Search the corpus.

**Body**
```json
{ "query": "two pointers", "topK": 20 }
```
- `query` (string, required) — user query
- `topK` (number, optional) — number of results to return (defaults to ~20)

**Response**
```json
{
  "results": [
    {
      "title": "Two Sum",
      "url": "https://leetcode.com/problems/two-sum",
      "description": "Given an array…",
      "platform": "LeetCode",
      "score": 7.23
    }
  ]
}
```

> Some builds may also support `GET /search?q=...`. Prefer `POST` to match the UI’s default fetch call in `script.js`.

---

## 🕷️ Scrapers

Located in `scrape.js` (Puppeteer). This snapshot contains collectors for:

- **LeetCode** problem set
- **Codeforces** problemset / problem list

Each writes JSON to `problems/`. Then run `npm run merge` to produce `corpus/all_problems.json` consumed by the server.

**Tips**
- Headless mode is currently set to `false` for easier debugging. Change to `true` in CI.
- If Chromium install is blocked, set `PUPPETEER_EXECUTABLE_PATH` to a local Chrome/Chromium binary.
- Be mindful of site **robots/TOS**; scrape responsibly (throttle requests and avoid heavy navigation loops).

---

## ⚙️ Configuration

Environment variables:

- `PORT` — server port (default: `3000`)
- `PUPPETEER_EXECUTABLE_PATH` — (optional) path to Chrome/Chromium for Puppeteer

Project settings live directly in code:
- Tokenization/stopwords: `utils/preprocess.js`
- Ranking & limits: `index.js` (top‑K capping, score formatting, etc.)

---

## 🧪 Testing Search Locally

Once the server is up:

```bash
# Example with curl
curl -X POST http://localhost:3000/search   -H "Content-Type: application/json"   -d '{"query":"shortest path dijkstra","topK":15}'
```

You should receive a JSON list of ranked problems. Open the web UI to see the formatted cards.

---

## 🛣 Roadmap / TODO

- [ ] Replace `...` placeholders in `index.js`, `script.js`, and `scrape.js` with full implementations (request parsing, scoring, scraping loops).
- [ ] Add **CodeChef** as a source (parallel to LeetCode & Codeforces) and include it in `merge.js`.
- [ ] Introduce **faceted filters** (site, difficulty, topic tags) once the scrapers capture those fields.
- [ ] Add **search highlighting** (show matched terms in description/title).
- [ ] Add **unit tests** for preprocessing, indexing, and ranking.
- [ ] Containerize with a minimal `Dockerfile`.
- [ ] Optional: persist corpus in SQLite for richer querying and pagination.

---

## 🤝 Attribution & License

- Packages: `express`, `natural`, `puppeteer`, `stopword`, `nodemon` (dev).
- Problem metadata and titles belong to their respective platforms (LeetCode, Codeforces). Use the data under each site’s Terms.
- License in `package.json` is set to **ISC** (update if you need a different license).

---

## 💡 Notes

- If search returns no results, confirm that `corpus/all_problems.json` exists and contains data (run `npm run merge` after scraping).
- If Puppeteer fails to launch on Linux, install system deps or supply `PUPPETEER_EXECUTABLE_PATH`.
- For production, serve the static UI through a reverse proxy/CDN and run the Node process with a process manager (PM2, systemd).

---

Made with ❤️ for fast DSA discovery.
