# Food Intelligence Assistant — Master Implementation Plan

> **Status**: Awaiting Phase 1 approval before any code is written.

---

## Project Summary

An AI-powered **Personal Food Intelligence Assistant** that learns one individual's own food-symptom patterns from their own logged history. It is not a diagnostic tool. Every recommendation traces to stored evidence. If evidence doesn't exist, the system says so plainly.

---

## Phase Gate Order

| # | Phase | Status |
|---|-------|--------|
| 1 | Problem Framing (Research Report) | 🟡 In Progress |
| 2 | Dataset Selection & Data Quality | ⬜ Pending |
| 3 | Knowledge Graph & Ingredient Decomposition | ⬜ Pending |
| 4 | Food Embeddings | ⬜ Pending |
| 5 | User Memory Schema | ⬜ Pending |
| 6 | Twin-State Evidence Engine | ⬜ Pending |
| 7 | Retrieval + Recommendation Pipeline (RAG) | ⬜ Pending |
| 8 | Backend APIs | ⬜ Pending |
| 9 | Product Design | ⬜ Pending |
| 10 | Frontend Implementation | ⬜ Pending |
| 11 | Testing | ⬜ Pending |
| 12 | Deployment | ⬜ Pending |

---

## Technology Stack (Proposed — Subject to Phase 8/9 Approval)

### Backend
| Layer | Choice | Reason |
|-------|--------|--------|
| Runtime | Python 3.11 | ML/data ecosystem, async support |
| API Framework | FastAPI | Async, OpenAPI auto-docs, pydantic validation |
| Database | PostgreSQL 16 | Relational integrity for user memory + structured food data |
| Vector Store | pgvector (PostgreSQL extension) | Co-located with relational data, no extra service |
| Graph Store | NetworkX (in-process) → Neo4j (stretch) | Knowledge graph queries; NetworkX for MVP |
| LLM Integration | Google Gemini 1.5 Pro via API | Multimodal (image of food), long context, free tier |
| Embedding Model | sentence-transformers `all-MiniLM-L6-v2` | Lightweight, open-source, runs locally |
| Evidence Engine | Custom Beta-Binomial (scipy) | No external dependency, explainable stats |
| Task Queue | Celery + Redis | Async meal processing, embedding jobs |
| Auth | JWT (PyJWT) + bcrypt | Stateless, portable |

### Frontend
| Layer | Choice | Reason |
|-------|--------|--------|
| Framework | Next.js 14 (App Router) | SSR/SSG, API routes, React Server Components |
| UI Language | TypeScript | Type safety across component props & API contracts |
| Styling | Vanilla CSS Modules + CSS custom properties | Full control, no Tailwind dependency |
| Animation | Framer Motion | Declared in prompt; best-in-class React animation |
| Charts | Recharts (custom styled) | Composable, themeable, no Chart.js opinionation |
| State Management | Zustand | Lightweight, no boilerplate, works with Next.js |
| Data Fetching | TanStack Query (React Query) | Cache, loading/error states, background refetch |
| Font | Inter (self-hosted from Fontsource) | Open license, premium feel |

### Infrastructure
| Layer | Choice |
|-------|--------|
| Containerisation | Docker + Docker Compose (local dev) |
| CI/CD | GitHub Actions |
| Hosting (MVP) | Vercel (frontend) + Railway/Render (backend + Postgres) |

---

## Phase 1 — Research Report (Delivering Now)

The research report is being generated as a separate artifact: `phase1_research_report.md`.

### Sections
1. Problem Statement
2. Literature Review
3. Existing Solutions
4. Research Gap
5. Novel Contribution
6. Ethical Considerations
7. Limitations
8. Future Work

---

## Phase 2 — Dataset Selection & Data Quality

### Datasets (Priority Order)
1. **INDB** — Indian Nutrient Databank (Vijayakumar et al., 2024)
2. **IFCT 2017** — NIN/ICMR gold-standard reference (citation only; open-access derivative INDB used for actual data)
3. **USDA FoodData Central** — API-accessible, public domain
4. **OpenFoodFacts** — Packaged foods; crowdsourced, noisy
5. **Curated Indian recipe datasets** — Only if source quality can be verified

### Data Quality Pipeline (Before Any Indexing)
```
Raw CSVs/APIs
    ↓
Deduplication (fuzzy name matching, Levenshtein distance)
    ↓
Missing value analysis (per-column null rates, imputation strategy documented)
    ↓
Outlier detection (nutrient values > 3σ flagged for review)
    ↓
Food name normalisation (lowercase, strip diacritics, canonical form)
    ↓
Ingredient normalisation (synonym map: "dal" → "lentil", etc.)
    ↓
Unit normalisation (all weights → grams, all energy → kcal)
    ↓
Regional alias mapping (Curd Rice = Mosaranna = Thayir Sadam)
    ↓
Merge log (every merge documented with conflict resolution strategy)
    ↓
Clean, versioned dataset (stored with hash for reproducibility)
```

---

## Phase 3 — Knowledge Graph & Ingredient Decomposition

### Graph Schema
```
Food Node
    ├─ id, canonical_name, aliases[], cuisine, category
    └─ CONTAINS → Ingredient Node
            ├─ id, canonical_name, category
            ├─ allergen_flags: [dairy, gluten, nuts, soy, egg, shellfish]
            ├─ property_flags: [fermented, high_oil, high_fibre, probiotic]
            ├─ cooking_method: [raw, boiled, fried, fermented, roasted]
            └─ provenance: {source_dataset, confidence, last_verified}
```

### Decomposition Rules
- **Source order**: INDB recipes → USDA SR → OpenFoodFacts ingredient lists → curated mapping table
- **Never**: LLM guessing ingredients
- **Unknown decomposition**: Returns `{ "status": "unknown", "ingredient": null }` — never a hallucinated guess

---

## Phase 4 — Food Embeddings

### Embedding Strategy
- Build embeddings **only after** Phase 3 is complete
- Feature vector per food: `[ingredient_set_tfidf | cuisine_onehot | cooking_method_onehot | nutrient_vector_normalised]`
- Model: `all-MiniLM-L6-v2` via sentence-transformers (text description of food's ingredient profile)
- Index: pgvector HNSW index (cosine distance)
- Similarity threshold: 0.75 (tunable; justify with spot-check examples)

### Spot-Check Requirements (documented before sign-off)
- Pizza vs. Burger: must be closer than Pizza vs. Idli
- Masala Dosa vs. Plain Dosa: close (shared base)
- Paneer Butter Masala vs. Chicken Butter Masala: very close (same sauce profile)
- Idli vs. Dosa: close (same batter)

---

## Phase 5 — User Memory Schema

```sql
-- Core tables (simplified schema — full DDL in phase 5 artifact)
users, meal_logs, symptom_logs, context_logs (sleep/stress/water/exercise),
medicine_logs, ai_recommendations, user_memory_snapshots

-- Every log row has: id, user_id, logged_at, source ('manual'|'voice'|'photo'), origin_device
```

### Confounding Handling
- Co-occurrence flag: if ingredient A always appears alongside ingredient B in a user's logs, the evidence output includes `"co_occurrence_warning": "X always appears alongside Y — individual effect uncertain"`
- Journal UI nudge: prompt user to occasionally log a food alone to resolve ambiguity

---

## Phase 6 — Twin-State Evidence Engine

### Statistical Model: Beta-Binomial
- **Prior**: Beta(1, 1) — uniform, makes no assumptions
- **Update**: For each exposure, increment either `reaction_count` or `no_reaction_count`
- **Posterior**: Beta(1 + reactions, 1 + no_reactions)
- **Credible interval**: 95% HDI from posterior
- **Formula (in report)**:
  ```
  P(reaction | data) ~ Beta(α + k, β + n - k)
  where α=1, β=1 (prior), k=reactions, n=exposures
  95% CI = scipy.stats.beta.interval(0.95, α+k, β+n-k)
  ```

### Per-Ingredient State Record
```json
{
  "ingredient_id": "uuid",
  "exposure_count": 6,
  "reaction_count": 5,
  "no_reaction_count": 1,
  "avg_severity": 2.3,
  "onset_delays_hours": [3.2, 4.1, 3.8, 2.9, 3.5],
  "credible_interval_95": [0.62, 0.97],
  "last_occurrence": "2026-06-11",
  "evidence_count": 6,
  "co_occurrence_flags": ["spice_blend_X"]
}
```

### Red-Flag Symptoms (Safety Backstop)
Symptoms that bypass food correlation entirely and route to "see a doctor":
- Severe or persistent abdominal pain (>6h)
- Blood in stool or vomit
- Difficulty breathing / throat swelling
- Fainting / loss of consciousness
- High fever (>102°F) with GI symptoms

---

## Phase 7 — RAG Recommendation Pipeline

```
User query: "Can I eat Paneer Butter Masala tonight?"
    ↓
[Step 1] Decompose food → ingredients via knowledge graph
    ↓
[Step 2] For each ingredient → retrieve twin-state evidence record
    ↓
[Step 3] Retrieve top-K similar meals from user's history (pgvector)
    ↓
[Step 4] Retrieve symptom logs linked to those meals
    ↓
[Step 5] Construct evidence context (structured JSON)
    ↓
[Step 6] LLM reasoning prompt (evidence-only, no hallucination allowed)
    ↓
[Step 7] JSON response → render as Evidence Card in UI
```

### Prompts (Separate, Deterministic)
- `extraction_prompt.txt` — extract food/symptom entities from free text
- `reasoning_prompt.txt` — reason over retrieved evidence
- `summarisation_prompt.txt` — produce plain-language summary
- `report_prompt.txt` — generate full evidence report
- `recommendation_prompt.txt` — produce actionable recommendation with limitations

---

## Phase 8 — Backend APIs

### Endpoint Groups
```
POST   /api/auth/register
POST   /api/auth/login

POST   /api/journal/meal          # Log a meal
POST   /api/journal/symptom       # Log a symptom
POST   /api/journal/context       # Log sleep/stress/water/exercise

GET    /api/food/search           # Search food database
GET    /api/food/{id}             # Food detail + decomposition
GET    /api/food/{id}/similar     # Similar foods via embedding

GET    /api/memory/meals          # Paginated meal history
GET    /api/memory/symptoms       # Symptom timeline
GET    /api/memory/ingredients    # Ingredient exposure summary

GET    /api/evidence/{ingredient_id}    # Twin-state record
POST   /api/chat                        # AI chat (RAG pipeline)

GET    /api/insights/dashboard    # Dashboard metrics
GET    /api/insights/trends       # Trend analysis
```

---

## Phase 9 — Product Design

### Hero Screens (Full Detail)
1. **Dashboard** — "How am I today?" — digestive score, AI summary, timeline, insight cards
2. **AI Chat with Evidence Panel** — chat left, evidence right, expandable evidence cards
3. **Food Detail** — hero image, decomposition, twin-state evidence, personal history

### Supporting Screens (Clean but Simpler)
- Daily Journal, Food Memory, Insights, Memory Graph (stretch), Settings, Knowledge Base, Help

### Design Tokens
```css
--color-bg-base: #09090B;
--color-bg-card: #111113;
--color-bg-card-hover: #18181B;
--color-border: rgba(255,255,255,0.06);
--color-accent: #10B981;       /* emerald */
--color-accent-blue: #3B82F6;
--color-accent-cyan: #06B6D4;
--color-warning: #F59E0B;      /* amber */
--color-danger: #EF4444;
--color-success: #22C55E;
--color-text-primary: #FAFAFA;
--color-text-secondary: #A1A1AA;
--color-text-muted: #52525B;

--font-sans: 'Inter', system-ui, sans-serif;
--radius-sm: 8px;
--radius-md: 12px;
--radius-lg: 16px;
--radius-xl: 24px;
```

---

## Non-Negotiable Constraints (Enforced Every Phase)

> [!CAUTION]
> - **No diagnosis** — ever, in any form
> - **No fabrication** — no invented confidence scores, citations, ingredients, nutrition values
> - **Evidence-only answers** — every recommendation traces to stored evidence
> - **Honest uncertainty** — credible intervals widen with thin data; summaries must not round uncertainty up to certainty
> - **Red-flag routing** — severe symptoms route to "see a doctor", not food correlation

---

## Folder Structure (Preview — Finalised in Phase 9)

```
Personal_GutHealth_Tracker/
├── research/                  # Phase 1 research report + references
├── data/                      # Raw datasets, processed, versioned
│   ├── raw/
│   ├── processed/
│   └── merge_log.md
├── backend/
│   ├── app/
│   │   ├── api/               # FastAPI routers
│   │   ├── core/              # Config, auth, logging
│   │   ├── db/                # SQLAlchemy models, migrations (Alembic)
│   │   ├── knowledge_graph/   # Graph builder, decomposition
│   │   ├── embeddings/        # Embedding pipeline, pgvector
│   │   ├── evidence/          # Twin-state engine (Beta-Binomial)
│   │   ├── rag/               # Retrieval, ranking, context construction
│   │   ├── prompts/           # All prompts as versioned text files
│   │   └── services/          # Business logic layer
│   ├── tests/
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── app/               # Next.js App Router pages
│   │   ├── components/        # Reusable UI components
│   │   ├── design-system/     # Tokens, CSS, Framer variants
│   │   ├── hooks/             # Custom React hooks
│   │   ├── stores/            # Zustand stores
│   │   └── lib/               # API clients, utilities
│   └── package.json
├── docker-compose.yml
└── README.md
```

---

## Open Questions (For User Approval)

> [!IMPORTANT]
> Before I begin Phase 1 execution, please confirm the following:

1. **LLM Provider** — I've proposed Google Gemini 1.5 Pro (free tier, multimodal). Do you have a Gemini API key, or do you prefer OpenAI GPT-4o? This affects the RAG and chat implementation.
2. **MVP Scope** — Per the prompt's own note: "a small pipeline that's fully working beats a large one that's half-built." For the initial demo, shall I target: ~500 Indian foods from INDB, full twin-state engine, AI chat, and 3 hero screens? Remaining features (full graph coverage, memory graph visualisation) as stretch goals?
3. **Deployment target** — Vercel + Railway is free-tier friendly. Any preference for cloud provider?
4. **Auth requirement** — Single-user (your own data, no multi-tenancy) or multi-user from the start?

---

*Waiting for your approval to proceed to Phase 1 execution.*
