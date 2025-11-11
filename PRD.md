# Product Requirements Document: Kenyan Meal Recommendation System

**Project Codename:** ChakuLa AI    
**Owner:** Tumaini  
**Last Updated:** November 12, 2025

---

## 1. Executive Summary

A conversational AI-powered meal recommendation system that solves decision fatigue around meal planning, with deep understanding of Kenyan cuisine and meal component structures. The system reduces the cognitive load of cooking by handling the planning phase intelligently.

### Success Metrics (v1)
- [ ] Can suggest relevant meals in <3 seconds
- [ ] 80%+ of suggestions feel contextually appropriate (Kenyan/fusion)
- [ ] User completes 5+ meal queries in a session
- [ ] System remembers and respects dietary preferences

---

## 2. Problem Statement

**Target User:** Busy professionals who struggle with meal planning and cooking motivation

**Core Problems:**
1. Decision paralysis when choosing what to eat
2. Lack of culturally relevant (Kenyan) meal suggestions
3. Overwhelming meal planning process kills cooking motivation
4. Limited understanding of how to construct meals from components
5. Ends up with unhealthy choices or skipping meals

---

## 3. Solution Overview

A **hybrid recommendation system** combining:
1. **Component-based selection UI** - User selects base, protein, vegetables, time, meal type
2. **Conversational AI assistant** - Ollama-powered chat for clarifications, substitutions, cooking tips
3. **Vector search backend** - Semantic matching of user preferences to recipes
4. **Learning system** - Tracks history to avoid repetition and improve suggestions

**Primary User Flow:**
```
Landing Page
    ↓
Component Selector (5 filters)
    ↓
AI processes selection → Vector search → Ranked results
    ↓
3-5 Recipe Cards displayed
    ↓
User clicks card → Recipe Detail View
    ↓
Optional: Chat with AI (substitutions, cooking guidance)
    ↓
Mark as "Cooked" → Stored in history
```

**Secondary Flow (Chat-first):**
```
User types: "Quick dinner with beef"
    ↓
AI extracts: protein=beef, time=quick
    ↓
Sets component filters automatically
    ↓
Shows results + allows refinement
```

---

## 4. MVP Feature Specification

### 4.1 Core Features (MUST HAVE for v1)

#### Feature 1: Component-Based Recipe Discovery (PRIMARY INTERACTION)
- [ ] **5 Filter System:**
  - Base: Rice, Ugali, Chapati, Pasta, Bread, Potatoes, None (dropdown/pills)
  - Protein: Beef, Chicken, Fish, Eggs, Beans, Lentils, None/Vegetarian (dropdown/pills)
  - Vegetables: Sukuma Wiki, Cabbage, Tomatoes, Spinach, Carrots, None (multi-select)
  - Time: Quick (<30 min), Medium (30-60), Long (60+) (radio buttons)
  - Meal Type: Breakfast, Lunch, Dinner, Snack (radio buttons)
- [ ] Real-time results update as filters change
- [ ] "Clear all filters" functionality
- [ ] Visual feedback on selected filters

#### Feature 2: Conversational AI Assistant (SECONDARY INTERACTION)
- [ ] Persistent chat widget (bottom-right corner OR sidebar)
- [ ] Accept natural language queries:
  - "What should I cook for dinner?" → AI extracts context, sets filters
  - "Quick lunch with chicken" → Sets protein=chicken, time=quick
  - "I have beef and rice" → Sets base=rice, protein=beef
- [ ] Cooking guidance during recipe view:
  - "Can I substitute X with Y?"
  - "How do I know when it's done?"
  - "Make it less spicy"
- [ ] Context-aware: Knows current filters, selected recipe, user preferences

#### Feature 3: Smart Recipe Recommendations
- [ ] Vector search across recipe embeddings (title + description + ingredients)
- [ ] Ranking algorithm:
  1. Exact component matches (highest priority)
  2. Partial component matches (medium priority)
  3. Semantic similarity (lower priority)
  4. User history penalty (avoid recent meals)
- [ ] Return 3-5 ranked suggestions per query
- [ ] Include for each result:
  - Recipe title and description
  - Cooking time and difficulty
  - Component breakdown (visual tags)
  - Nutrition summary (calories, protein, carbs)

#### Feature 4: Recipe Detail View
- [ ] Full ingredient list with quantities
- [ ] Step-by-step cooking instructions
- [ ] Cooking time breakdown (prep + cook)
- [ ] Difficulty indicator
- [ ] Component tags (base, protein, vegetables)
- [ ] Nutritional information panel
- [ ] "I cooked this" button (adds to history)
- [ ] Share button (copy link)

#### Feature 5: User Preferences & History
- [ ] One-time onboarding (skippable):
  - Dietary restrictions (vegetarian, halal, allergies)
  - Dislikes (ingredients to avoid)
  - Cooking skill level
- [ ] Store in localStorage and sync to backend
- [ ] Meal history:
  - Track last 14 days of cooked meals
  - Avoid suggesting same meal within 3 days
  - "Recently cooked" section on homepage
- [ ] Preference editing (settings page)

### 4.2 Nice-to-Have (Defer to v2)
- Weekly meal planning
- Shopping list generation
- Pantry inventory tracking
- Image upload for ingredient identification
- Recipe rating and reviews
- Cooking mode (hands-free step-through)
- Multi-language support (Swahili)

---

## 5. Technical Architecture

### 5.1 System Architecture Diagram

```
┌─────────────────────────────────────────────────────────┐
│                     USER INTERFACE                      │
│              (React + Vite + Tailwind CSS)              │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│                  FASTAPI BACKEND                        │
│  ┌──────────────────────────────────────────────────┐   │
│  │           Conversation Handler                   │   │
│  │  - Parse user query                              │   │
│  │  - Extract intent & entities                     │   │
│  └──────────────────┬───────────────────────────────┘   │
│                     │                                   │
│  ┌──────────────────▼───────────────────────────────┐   │
│  │         LLM Service (Ollama/Groq)                │   │
│  │  - Intent classification                         │   │
│  │  - Entity extraction                             │   │
│  │  - Response generation                           │   │
│  └──────────────────┬───────────────────────────────┘   │
│                     │                                   │
│  ┌──────────────────▼───────────────────────────────┐   │
│  │     Recommendation Engine                        │   │
│  │  ┌─────────────────────────────────────────┐     │   │
│  │  │ Vector Search (Chroma)                  │     │   │
│  │  │ - Semantic recipe search                │     │   │
│  │  │ - Embedding: sentence-transformers      │     │   │
│  │  └─────────────────┬───────────────────────┘     │   │
│  │                    │                             │   │
│  │  ┌─────────────────▼───────────────────────┐     │   │
│  │  │ Reranking & Filtering                   │     │   │
│  │  │ - Apply user preferences                │     │   │
│  │  │ - Filter by history (avoid repetition)  │     │   │
│  │  │ - Diversity optimization                │     │   │
│  │  └─────────────────────────────────────────┘     │   │
│  └──────────────────────────────────────────────────┘   │
└────────────────────┬────────────────────────────────────┘
                     │
        ┌────────────┴────────────┐
        │                         │
        ▼                         ▼
┌──────────────┐          ┌──────────────┐
│  Vector DB   │          │   SQL DB     │
│  (Chroma)    │          │  (SQLite)    │
│              │          │              │
│ - Recipes    │          │ - Users      │
│ - Embeddings │          │ - Prefs      │
│              │          │ - History    │
└──────────────┘          └──────────────┘
```

### 5.2 Data Models

#### Recipe Schema (Hybrid: Vector DB + SQLite)

**ChromaDB (Vector Store):**
```python
# Embedded document for semantic search
{
  "id": "recipe_15003",
  "embedding": vector(384),  # from sentence-transformers
  "metadata": {
    "title": "Kaimati (Fried Dumplings)",
    "components": {
      "base": "bread",  # closest match from CSV
      "protein": "none",
      "vegetables": []
    },
    "time_category": "medium",  # derived from cooking time
    "meal_types": ["breakfast", "snack"],
    "cuisine": "kenyan"
  }
}
```

**SQLite (Structured Data):**
```python
Recipe:
  id: str  # e.g., "recipe_15003" (from CSV index)
  title: str  # from CSV
  page: int  # from CSV
  about: str  # description from CSV
  cuisine_type: str  # Kenyan, Fusion (derived)
  
  # Component mappings (derived from ingredients + title)
  base_type: str  # rice, ugali, chapati, pasta, bread, potatoes, none
  protein_type: str  # beef, chicken, fish, eggs, beans, lentils, none
  vegetable_types: List[str]  # sukuma_wiki, cabbage, tomatoes, etc.
  
  # Time and difficulty (derived from CSV)
  prep_time_minutes: int  # extracted from "Preparation X minutes"
  cook_time_minutes: int  # extracted from "Cooking X minutes"
  total_time_minutes: int  # prep + cook
  time_category: str  # quick (<30), medium (30-60), long (60+)
  difficulty: str  # Easy, Medium, Hard (default: Medium)
  serves: int  # from CSV "Serves X"
  
  # Recipe content (from CSV)
  ingredients: str  # CSV "ingridients" column (raw)
  ingredients_list: List[str]  # parsed from CSV
  preparation: str  # CSV "preparation" column (raw)
  instructions_list: List[str]  # parsed from CSV
  
  # Nutrition (from CSV)
  nutrition_per_100g: str  # raw from CSV
  energy_kcal: float  # from CSV
  fat_g: float
  carbohydrates_g: float
  proteins_g: float
  fibre_g: float
  vitamin_a_mcg: float
  iron_mg: float
  zinc_mg: float
  
  # Metadata
  tags: List[str]  # quick, traditional, vegetarian, etc. (derived)
  created_at: datetime
  updated_at: datetime

# Helper table for component taxonomy
ComponentMapping:
  id: int
  csv_ingredient: str  # e.g., "white rice", "beef", "sukuma wiki"
  base_type: str  # normalized: rice, ugali, etc.
  protein_type: str  # normalized: beef, chicken, etc.
  vegetable_type: str  # normalized: sukuma_wiki, cabbage, etc.
```

#### User Schema (SQLite)
```python
User:
  id: uuid
  dietary_restrictions: List[str]  # vegetarian, halal, etc.
  dislikes: List[str]  # ingredients to avoid
  allergies: List[str]
  cooking_skill_level: str  # beginner, intermediate, advanced
  created_at: datetime

MealHistory:
  id: uuid
  user_id: uuid  # foreign key (default user for MVP)
  recipe_id: str  # matches Recipe.id
  cooked_at: datetime
  rating: int  # 1-5 (optional, defer to v2)

UserPreferences:
  id: uuid
  user_id: uuid
  max_cooking_time: int  # minutes
  preferred_cuisines: List[str]
  updated_at: datetime
```

### 5.3 Technology Stack

| Component | Technology | Rationale |
|-----------|-----------|-----------|
| **Frontend** | Streamlit | Rapid prototyping, Python-native, no JS needed |
| **Backend** | FastAPI | Fast, async, easy API creation |
| **LLM** | Ollama (llama3.2) or Groq API | Free, local control OR fast cloud inference |
| **Embeddings** | sentence-transformers (all-MiniLM-L6-v2) | Lightweight, good quality, free |
| **Vector DB** | ChromaDB | Embedded, no setup, perfect for prototyping |
| **Relational DB** | SQLite | Zero config, sufficient for single user |
| **LLM Framework** | LangChain (optional) | Helps with prompt management, can skip for v1 |
| **Deployment** | Docker | Reproducible environment |

### 5.4 Alternative Tech Considerations

**If you want faster LLM responses:**
- Use **Groq API** (free tier) instead of local Ollama
- Trade-off: Requires internet, API key management

**If you need multi-user from day 1:**
- Replace SQLite with PostgreSQL
- Add simple auth (Clerk or Auth0 free tier)

**If Streamlit feels limiting:**
- Switch to Gradio (better for chat interfaces)
- Or Next.js + FastAPI (more work, better UX)

---

## 6. Implementation Plan (7-Day Sprint)

### Day 1: Data Processing & Backend Setup

**Morning: Environment & CSV Processing**
- [ ] Install Ollama: `ollama pull llama3.2:3b`
- [ ] Set up Python backend environment
  - [ ] Install: fastapi, chromadb, sentence-transformers, pandas, uvicorn
- [ ] Create CSV parser script:
  - [ ] Read RecipesImp.csv (160+ recipes ready!)
  - [ ] Parse ingredient lists (comma-separated)
  - [ ] Parse instructions (line breaks with "?")
  - [ ] Extract prep/cook times from text
  - [ ] Derive serves/yields from text
  - [ ] Clean nutrition data

**Afternoon: Component Mapping & Database**
- [ ] Build component taxonomy:
  - [ ] Map CSV ingredients to base types (ugali, rice, chapati, etc.)
  - [ ] Map to protein types (beef, chicken, fish, etc.)
  - [ ] Map to vegetable types (sukuma wiki, cabbage, etc.)
  - [ ] Derive time categories from total_time
  - [ ] Derive meal types from title/description
- [ ] Initialize SQLite:
  - [ ] Create Recipe table
  - [ ] Create User, MealHistory, UserPreferences tables
  - [ ] Load all 160+ recipes
- [ ] Generate embeddings:
  - [ ] Create embedding text: title + about + ingredients
  - [ ] Use sentence-transformers to embed all recipes
  - [ ] Store in ChromaDB with metadata
- [ ] Test vector search with sample queries

### Day 3-4: Core Backend
- [ ] **FastAPI application skeleton**
  - [ ] Create main.py with basic routes
  - [ ] `/query` endpoint - handle meal queries
  - [ ] `/recipes/{id}` endpoint - get recipe details
  - [ ] `/preferences` endpoint - set user preferences
  
- [ ] **Recommendation engine**
  - [ ] Implement semantic search using ChromaDB
  - [ ] Add filtering logic (preferences, history)
  - [ ] Implement reranking algorithm
  - [ ] Test with sample queries
  
- [ ] **LLM integration**
  - [ ] Set up Ollama locally OR Groq API
  - [ ] Create prompt templates for:
    - Query understanding
    - Entity extraction (cuisine, time, ingredients)
    - Response formatting
  - [ ] Test intent classification accuracy

### Day 5-6: Frontend & Integration
- [ ] **Streamlit UI**
  - [ ] Chat interface for meal queries
  - [ ] Display recipe recommendations as cards
  - [ ] Recipe detail view with components
  - [ ] User profile/preferences form
  
- [ ] **Connect frontend to backend**
  - [ ] API client setup
  - [ ] Handle loading states
  - [ ] Error handling and user feedback
  
- [ ] **Preference learning**
  - [ ] Track selected recipes
  - [ ] Update meal history
  - [ ] Implement "avoid recent meals" logic

### Day 7: Testing & Polish
- [ ] **End-to-end testing**
  - [ ] Test 20+ different query types
  - [ ] Verify recommendation quality
  - [ ] Check preference persistence
  
- [ ] **Polish & Documentation**
  - [ ] Add example queries in UI
  - [ ] Write README with setup instructions
  - [ ] Document API endpoints
  - [ ] Create sample .env file
  
- [ ] **Deployment preparation**
  - [ ] Create Dockerfile
  - [ ] Test Docker build
  - [ ] Document deployment steps

---

## 7. Success Criteria & Testing

### Functional Tests
- [ ] System returns relevant recipes for 10 test queries
- [ ] Recommendations respect dietary restrictions 100% of time
- [ ] No duplicate recipes suggested within 3-day window
- [ ] Response time < 3 seconds for each query
- [ ] System gracefully handles ambiguous queries

### User Experience Tests
- [ ] Can complete full flow (query → recommendation → recipe view) without errors
- [ ] Component breakdown is clear and helpful
- [ ] At least 2/3 suggestions feel culturally appropriate
- [ ] Conversation feels natural, not robotic

### Technical Tests
- [ ] Vector search returns semantically similar recipes
- [ ] Database operations complete without errors
- [ ] LLM consistently extracts key entities from queries
- [ ] System handles 100+ recipes without performance issues

---

## 8. Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| LLM hallucinations in recipe suggestions | High | Use retrieval-augmented generation (RAG); LLM only formats results |
| Limited recipe dataset | Medium | Start with 30 quality recipes; expand iteratively |
| Local LLM too slow | Medium | Fall back to Groq API (free tier) |
| Preferences don't improve recommendations | Medium | Start with simple rules; add ML ranking in v2 |
| Over-engineering for v1 | High | Strict scope adherence; defer nice-to-haves |

---

## 9. Future Roadmap (Post-MVP)

### v2 Features (Week 2-4)
- Weekly meal planner with variety optimization
- Shopping list generation with ingredient consolidation
- Pantry inventory management
- Cooking mode (hands-free voice interaction)

### v3 Features (Month 2-3)
- Community recipe contributions
- Recipe ratings and reviews
- Image-based ingredient identification
- Nutritional goal tracking
- Multi-language support (Swahili)

### Long-term Vision
- Mobile app (React Native)
- Marketplace integration (grocery delivery)
- Social features (share meal plans)
- Advanced ML personalization

---

## 10. Getting Started Checklist

### Environment Setup
- [ ] Clone/create project repository
- [ ] Set up Python virtual environment
- [ ] Install dependencies from requirements.txt
- [ ] Configure environment variables (.env)
- [ ] Test basic imports

### Development Tools
- [ ] IDE setup (VS Code recommended)
- [ ] Install Ollama (if using local LLM)
- [ ] Set up database migrations (Alembic)
- [ ] Configure linting/formatting (Black, Ruff)

### First Tasks
- [ ] Create 10 recipe entries manually
- [ ] Test embedding generation
- [ ] Test vector search with sample queries
- [ ] Build "Hello World" FastAPI endpoint
- [ ] Build basic Streamlit page

---

## 11. Resources & References

### Kenyan Cuisine References
- Popular meals: Ugali + Sukuma Wiki, Pilau, Nyama Choma, Githeri, Mukimo
- Common ingredients: Maize flour, kale, beans, beef, chicken, tomatoes, onions
- Flavor profiles: Pilau masala, royco, tangawizi (ginger), coconut

### Technical Resources
- ChromaDB docs: https://docs.trychroma.com
- LangChain cookbook: https://github.com/langchain-ai/langchain
- Sentence transformers: https://www.sbert.net
- FastAPI tutorial: https://fastapi.tiangolo.com

### Recommended Reading
- "Building Recommendation Systems with Python" (book)
- "Designing Data-Intensive Applications" (Chapter on search)

---

## Appendix A: Sample Recipe Entry

```json
{
  "id": "recipe_001",
  "name": "Ugali with Sukuma Wiki and Beef Stew",
  "description": "Classic Kenyan comfort meal with maize ugali, sautéed kale, and tender beef in tomato gravy",
  "cuisine_type": "Kenyan",
  "components": {
    "base": "Ugali",
    "protein": "Beef",
    "vegetables": ["Sukuma Wiki"],
    "flavor_profile": "Tomato-based, savory"
  },
  "cooking_time_minutes": 45,
  "difficulty": "Easy",
  "ingredients": [
    {"name": "Maize flour", "amount": "2 cups"},
    {"name": "Sukuma wiki", "amount": "1 bunch"},
    {"name": "Beef stew meat", "amount": "500g"},
    {"name": "Tomatoes", "amount": "3 large"},
    {"name": "Onions", "amount": "2 medium"}
  ],
  "instructions": [
    "Boil water for ugali...",
    "Cut beef into chunks...",
    "Sauté onions until golden..."
  ],
  "nutrition": {
    "calories": 650,
    "protein_g": 35,
    "carbs_g": 70,
    "fat_g": 20
  },
  "tags": ["traditional", "filling", "comfort-food", "beginner-friendly"]
}
```

---

**Document Version:** 1.0  