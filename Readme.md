# 🌿 Personal Gut Health Tracker & Evidence Engine

> **Twin-State Bayesian Evidence Engine for Personalized Ingredient-Level Food-Symptom Correlation with Explainable RAG-Grounded Recommendations**

---

## 📑 Research & Academic Publication

This repository contains the complete implementation, database pipelines, Bayesian evidence engine, and user interface for the academic research paper:

> **"Twin-State Bayesian Evidence Engine for Personalized Ingredient-Level Food-Symptom Correlation with Explainable RAG-Grounded Recommendations"**  
> **Author**: Basava S H  
> **Affiliation**: Department of Computer Science & Engineering, Ramaiah Institute of Technology, Bengaluru, India  
> **Contact**: `basavabasava5585@gmail.com`  

### IEEE Initial Submission
The manuscript has been submitted for peer review to an IEEE Conference.

![IEEE Initial Submission Confirmation](./iee_initial_submission.jpg)

### IEEE Initial Submission Status
Awaiting for Associate Editor (AE) Assignment.

![IEEE Submission_Status](./iee_admin.jpg)
---

## 📸 System Showcase

| **1. Regional Food Search & Atomic Deconstruction** | **2. Gut Health Insights Dashboard (User 2)** |
|:---:|:---:|
| ![Food Search & Ingredient Breakdown](./food_search_Image.jpg) | ![Gut Health Insights Dashboard for User 2](./insight_image_of_user_2.jfif) |
| *Deconstructs composite dishes into raw ingredients, allergens, and nutritional profiles in real time.* | *Visualizes Bayesian trigger probabilities, credible intervals, and evidence exposure counts.* |

| **3. Grounded Zero-Hallucination AI Chat (User 2)** | **4. IEEE Conference Submission Confirmation** |
|:---:|:---:|
| ![AI Chat Summary & Grounded RAG](./ai_Chat_Summary_for_user2.jfif) | ![IEEE Initial Submission](./iee_initial_submission.jpg) |
| *Retrieval-Augmented Generation that only answers using mathematically computed personal evidence.* | *Official submission confirmation of research paper to IEEE conference.* |

---

## 💡 The Problem & Research Gap

Traditional food-symptom logging applications suffer from four foundational flaws:
1. **The Composite Meal Dilemma**: When a user logs a dish (e.g., *Masala Dosa*, *Chicken Biryani*, or *Sambar*), they consume 8 to 15 distinct ingredients simultaneously. Standard tracking apps treat the meal as an indivisible black box, unable to isolate whether the culprit was fermented batter, nightshades, alliums, or cooking oil.
2. **Collinearity & Confounders**: In real-world diets, certain ingredients almost always co-occur (e.g., onion and garlic in curry bases; rice and lentils in South Indian cuisine). Naive statistical models falsely accuse harmless companion ingredients.
3. **Reporting Bias & Spurious Correlations**: Users disproportionately log on days they feel unwell. Frequently eaten safe staples (like plain rice or water) correlate spuriously with symptom events.
4. **Dangerous LLM Hallucinations**: Commercial generative AI chatbots guess diagnoses, fabricate food allergies, or prescribe restrictive elimination diets with false confidence.

---

## 🔬 Core Novelties & Scientific Contributions

### 1. Atomic Ingredient Decomposition & Regional Alias Resolution
- Employs the **INDB (Indian Nutrient Databank)** knowledge graph containing over 300 curated regional dishes and 170+ atomic ingredients.
- Built-in phonetic and colloquial alias mapper translates vernacular dish names (e.g., *Mosaranna*, *Thayir Sadam*, *Daddojanam* → `Curd Rice`; *Saaru*, *Charu* → `Rasam`) before resolving into granular constituents.

### 2. Twin-State Bayesian Evidence Engine (Beta-Binomial Model)
- Rather than calculating a single misleading correlation percentage, every ingredient exposure is evaluated as a conjugate Bayesian model:
  $$\text{Prior: } \text{Beta}(\alpha_0, \beta_0) \quad \xrightarrow{+\text{Exposures}, +\text{Reactions}} \quad \text{Posterior: } \text{Beta}(\alpha_0 + k, \beta_0 + n - k)$$
- Generates **$95\%$ Credible Intervals** $[\theta_{\text{lower}}, \theta_{\text{upper}}]$. When data is thin, intervals widen transparently to reflect genuine uncertainty.

### 3. Co-Occurrence & Collinearity Disentanglement
- Tracks pairwise co-occurrence frequencies (`co_occurring_counts`) across all meal events.
- Employs dynamic logging density and baseline reaction rates to suppress false positives caused by shared curry bases or habitual staple pairings.

### 4. Zero-Hallucination Grounded RAG Architecture
- LLM inference is strictly constrained: the model is forbidden from drawing upon generic training data for health advice.
- Prompts enforce strict retrieval of the user's mathematically computed `IngredientEvidence` records. If statistical evidence is insufficient, the system explicitly answers: *"Insufficient evidence logged to establish a correlation."*

---

## 🏗️ System Architecture

```mermaid
flowchart TD
    User([👤 User]) <-->|Next.js 14 Web App| FE[Frontend UI / Tailwind & Framer Motion]
    FE <-->|REST API / JWT Auth| BE[FastAPI Backend Server]
    
    subgraph Data & Inference Engine
        BE -->|Alias & Decomposition| KG[(INDB Knowledge Base)]
        BE -->|Semantic Embeddings| EMB[Sentence-Transformers all-MiniLM-L6-v2]
        BE -->|Meal & Symptom Logs| EE[Twin-State Bayesian Evidence Engine]
        EE -->|Beta-Binomial Posterior| STATS[Credible Intervals & Risk Analysis]
        BE -->|Grounded Evidence Context| RAG[RAG Layer / Google Gemini 1.5]
    end

    subgraph Persistence Layer
        BE <-->|Async Motor / Beanie ODM| DB[(MongoDB 7.0 Document Store)]
        DB -.-> Coll1[users]
        DB -.-> Coll2[foods & ingredients]
        DB -.-> Coll3[meal_logs & symptom_logs]
        DB -.-> Coll4[ingredient_evidence]
        DB -.-> Coll5[chat_sessions & messages]
    end
```

---

## 💻 Tech Stack

| Layer | Technology | Description |
|:---|:---|:---|
| **Frontend** | **Next.js 14+ (App Router)** | Modern React 19 framework with Server & Client components |
| **Styling & UI** | **Tailwind CSS + Framer Motion** | Glassmorphism design tokens, micro-interactions, responsive dashboard |
| **Backend** | **FastAPI (Python 3.11)** | High-performance asynchronous REST API with OpenAPI autodocs |
| **Database** | **MongoDB 7.0 + Beanie ODM** | Asynchronous document ODM with Motor driver |
| **Statistical Engine** | **Scipy (Beta-Binomial)** | Conjugate Bayesian updating, quantile credible intervals |
| **Embeddings** | **Sentence-Transformers** | `all-MiniLM-L6-v2` generating 384-dimensional semantic vectors |
| **LLM Orchestration** | **Google Gemini Flash** | Provider-abstracted RAG pipeline (Gemini, OpenAI, Anthropic, DeepSeek) |
| **Containerization** | **Docker & Docker Compose** | Multi-service local and cloud deployment orchestration |

---

## 🗄️ Database Schema Highlights

```mermaid
erDiagram
    %% Core Entities
    User ||--o{ MealLog : logs
    User ||--o{ SymptomLog : logs
    User ||--o{ ContextLog : logs
    User ||--o{ IngredientEvidence : has
    User ||--o{ ChatSession : initiates

    %% Food Knowledge Graph
    Food ||--o{ FoodIngredientEntry : embeds
    Ingredient ||--o{ IngredientFlags : embeds
    Food ||--|| Provenance : embeds
    Ingredient ||--|| Provenance : embeds
    Food ||--|| NutritionInfo : embeds
    Ingredient ||--|| NutritionInfo : embeds
    Food ||--o| FoodEmbedding : possesses

    %% Journal & Logging
    MealLog ||--o| Food : references
    SymptomLog }|--o{ MealLog : links_to
    
    %% Twin-State Evidence Engine
    IngredientEvidence ||--|| Ingredient : tracks

    %% AI & Chat
    ChatSession ||--o{ ChatMessage : contains
    ChatMessage ||--o| AIRecommendation : references

    User {
        string id PK
        string email
        string full_name
        string hashed_password
        dict profile
        bool is_active
        bool is_admin
    }

    Food {
        string id PK
        string canonical_name
        list aliases
        string cuisine
        string category
        string decomposition_status
        bool is_verified
        bool is_vegetarian
        bool is_vegan
    }

    FoodIngredientEntry {
        string ingredient_id
        string ingredient_name
        float quantity_g
        string source_dataset
    }

    Ingredient {
        string id PK
        string canonical_name
        list aliases
        string category
        list allergens
    }

    FoodEmbedding {
        string id PK
        string food_id FK
        list embedding
        string embedding_text
    }

    IngredientEvidence {
        string id PK
        string user_id FK
        string ingredient_id FK
        string ingredient_name
        int evidence_count
        int exposure_count
        int reaction_count
        int no_reaction_count
        float avg_severity
        float credible_interval_lower
        float credible_interval_upper
        datetime last_occurrence
        datetime last_exposure_at
        datetime last_reaction_at
        list co_occurring_ingredient_ids
        list co_occurring_ingredient_names
        dict co_occurring_counts
    }

    MealLog {
        string id PK
        string user_id FK
        string food_id FK
        string food_name_raw
        string meal_type
        bool has_triggered_reaction
        datetime logged_at
    }

    SymptomLog {
        string id PK
        string user_id FK
        string symptom_name
        int severity
        bool is_red_flag
        list linked_meal_ids
        datetime logged_at
    }

    IngredientFlags {
        bool is_dairy
        bool is_gluten
        bool is_fermented
        bool is_high_oil
        bool is_high_fibre
        bool is_probiotic
        bool is_nightshade
        bool is_allium
    }

    ChatSession {
        string id PK
        string user_id FK
        string title
    }

    ChatMessage {
        string id PK
        string session_id FK
        string user_id FK
        string role
        string content
    }
```

The system is powered by MongoDB collections structured via Beanie ODM models:
- **`User`**: User credentials (bcrypt), profile preferences, dietary restrictions, admin flags.
- **`Food`**: Canonical names, dialectal aliases, regional cuisine, embedded `FoodIngredientEntry` list, and nutritional macros per 100g.
- **`Ingredient`**: Atomic ingredients with biochemical classification flags (`is_dairy`, `is_gluten`, `is_fermented`, `is_nightshade`, `is_allium`, `is_high_oil`).
- **`MealLog` & `SymptomLog`**: Timestamped meal and symptom event documents linked by chronological ingestion windows.
- **`IngredientEvidence`**: Twin-state evidence counters (`exposure_count`, `reaction_count`, `no_reaction_count`), average severity, Bayesian credible intervals (`credible_interval_lower`, `credible_interval_upper`), and co-occurrence frequency matrices.
- **`ChatSession` & `ChatMessage`**: Grounded chat logs linking AI recommendations directly to stored evidence snapshots.

---

## 🚀 Quickstart & Installation

### Option 1: One-Command Setup with Docker Compose (Recommended)

Ensure Docker Desktop is installed and running:

```powershell
# 1. Clone the repository
git clone https://github.com/Basava05/Twin-State-Bayesian-Evidence-Engine.git
cd Twin-State-Bayesian-Evidence-Engine

# 2. Build and start all services
docker compose up --build
```
- **Frontend**: [http://localhost:3000](http://localhost:3000)
- **Backend API**: [http://localhost:8000](http://localhost:8000)
- **Interactive Swagger Docs**: [http://localhost:8000/docs](http://localhost:8000/docs)

---

### Option 2: Local Manual Setup

#### 1. Start MongoDB
Ensure MongoDB is running locally on port `27017` (or provide your MongoDB Atlas cloud URI in `.env`):
```powershell
docker run -d --name food_mongo -p 27017:27017 mongo:7.0
```

#### 2. Configure Backend
```powershell
cd backend

# Create & activate Python virtual environment
python -m venv venv
.\venv\Scripts\activate   # On Linux/macOS: source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Configure environment variables
cp .env.example .env     # Ensure MONGODB_URL and GEMINI_API_KEY are set
```

#### 3. Seed Knowledge Base & Ingest Data
```powershell
# Ingest INDB foods, ingredients, recipes, and semantic embeddings
python -m data_pipeline.ingest_indb

# Heal missing ingredient decompositions
python -m data_pipeline.heal_food_ingredients

# (Optional) Inject synthetic benchmark user with 30-day meal/symptom history
python scripts/inject_synthetic_user.py
```

#### 4. Launch Backend API Server
```powershell
uvicorn app.main:app --reload --port 8000
```

#### 5. Launch Frontend (In a separate terminal)
```powershell
cd frontend
npm install
npm run dev
```
Navigate to **[http://localhost:3000](http://localhost:3000)**.

---

## 🔑 Default Credentials

The application auto-seeds a system administrator account on initial startup:
- **Email**: `admin@example.com`
- **Password**: `admin123`

*(You can also register a new personal account on the [Sign Up page](http://localhost:3000/signup).)*

---

## 📡 Key API Endpoints

| Category | Method | Endpoint | Description |
|:---|:---|:---|:---|
| **Auth** | `POST` | `/api/auth/register` | Register new user account |
| **Auth** | `POST` | `/api/auth/login` | Authenticate and obtain JWT bearer token |
| **Journal** | `POST` | `/api/journal/meal` | Log a meal with automatic ingredient decomposition |
| **Journal** | `POST` | `/api/journal/symptom` | Log a symptom event with severity score (1–10) |
| **Food** | `GET` | `/api/food/search?q={query}` | Search foods across canonical names and regional aliases |
| **Food** | `GET` | `/api/food/{id}` | Retrieve full food profile, ingredients, and nutrition |
| **Food** | `GET` | `/api/food/{id}/similar` | Find semantically similar dishes using embedding vectors |
| **Insights** | `GET` | `/api/insights/dashboard` | Retrieve Bayesian evidence summary, safe foods, and triggers |
| **Chat** | `POST` | `/api/chat/` | Send message to explainable RAG pipeline grounded in user evidence |

---

## 🧪 Benchmark Evaluation (10 Synthetic Personas)

To validate statistical robustness without violating patient privacy, the engine was tested against 10 synthetic user archetypes (`synthetic_10_users.json`) covering challenging edge cases:
1. **Clean Single Trigger**: Isolated ingredient sensitivity (e.g., lactose in paneer).
2. **Strict Collinearity**: Inseparable ingredient pairs (e.g., chickpea + chole masala) to evaluate confounding resolution.
3. **Sparse / Under-Logged**: Irregular diary keeping to test Bayesian credible interval widening.
4. **Symptom-Only Logging Bias**: Users who only record meals when feeling distress.
5. **Multi-Symptom / Delayed Response**: Non-linear latency windows between meal ingestion and symptom onset.

---

## 🛡️ Ethical Principles & Medical Disclaimer

- **No Medical Diagnosis**: This application does not diagnose medical conditions, irritable bowel syndrome (IBS), or allergies.
- **Pure Observational Statistics**: It computes statistical associations strictly from personal logged events.
- **Honest Uncertainty**: If an ingredient has only been consumed 1 or 2 times, the system refuses to draw conclusions.
- **Disclaimer**: *Always consult a qualified gastroenterologist, physician, or registered dietitian before undertaking dietary eliminations.*

---


