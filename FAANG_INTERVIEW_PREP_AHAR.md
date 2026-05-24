# 🍽️ AHAR — Smart Food Waste Prediction & Kitchen Management
## FAANG Interview Preparation Guide (Beginner-Friendly Edition)

> **How to use this document:** Read it like a story. Every section explains **WHY** things were done, not just what they are. If you see a new word, look right after it — it will be explained simply.

---

# Table of Contents

1. [Project Overview](#1-project-overview)
2. [Folder Structure & What Each File Does](#2-folder-structure--what-each-file-does)
3. [System Design — Big Picture](#3-system-design--big-picture)
4. [Database Design (Deep Dive)](#4-database-design-deep-dive)
5. [API Design — How the App Talks to Itself](#5-api-design--how-the-app-talks-to-itself)
6. [Interview Questions by Topic](#6-interview-questions-by-topic)
   - [Data Structures & Algorithms](#61-data-structures--algorithms)
   - [Object-Oriented Design](#62-object-oriented-design)
   - [System Design](#63-system-design)
   - [Database & Backend](#64-database--backend)
   - [Frontend](#65-frontend)
   - [DevOps & Deployment](#66-devops--deployment)
7. [How Multitasking Works (VERY IMPORTANT)](#7-how-multitasking-works-very-important)
8. [Behavioral + Project Questions](#8-behavioral--project-questions)
9. [Bonus — Tricky Follow-up Questions](#9-bonus--tricky-follow-up-questions)

---

# 1. Project Overview

## What does AHAR actually do? (Plain English)

Imagine you run a school canteen. Every day you have to decide:
- **How many plates of food** should I cook today?
- **Do I have enough ingredients** in my storage?
- **How much food was wasted** yesterday, last week?
- If I cook too much and have leftovers, **where can I donate it?**

AHAR answers all these questions automatically. It's a **smart kitchen management web app** that:

1. **Predicts demand** — guesses how many meals to prepare using past data + weather + events
2. **Tracks inventory** — keeps a live count of every ingredient in stock
3. **Logs consumption** — records what was cooked vs what was actually eaten
4. **Shows analytics** — daily/weekly charts showing how much food was wasted
5. **Helps donate** — finds nearby NGOs on a map when there's surplus food
6. **Scans expiry** — an image upload feature that classifies ingredients as Fresh / Near-Expiry / Spoiled

Think of it as **a smart assistant for anyone who runs a large kitchen** (hospital, school, hotel, restaurant).

---

## Tech Stack — What's Used and WHY

### 🖥️ Frontend (what the user sees)

| Technology | What it is (simple) | Why we chose it | Problem it solves |
|---|---|---|---|
| **React** | A library to build interactive web pages as small reusable pieces ("components") | Makes large UIs manageable and fast to build | Without it, updating one part of the page would reload everything |
| **Vite** | A tool that starts a fast local development server | Much faster than older tools like webpack | Developers waste time waiting for the code to "compile"; Vite does it almost instantly |
| **React Router** | Handles which page you see when you go to `/inventory`, `/analytics`, etc. | Controls navigation without full page reload | Without it, changing pages would refresh the whole browser |
| **Axios** | A helper to make web requests from the browser to the backend | Simpler than plain browser `fetch` | Reduces repetitive code for headers, error handling, etc. |
| **Clerk** | A ready-made login/signup service | Handles user authentication so we don't build it from scratch | Authentication (login security, tokens, sessions) is complex and must be done right |

### 🧠 Backend (the "manager" that enforces rules)

| Technology | What it is (simple) | Why we chose it | Problem it solves |
|---|---|---|---|
| **Node.js** | Runs JavaScript on a server (not in a browser) | Great for handling many requests that mostly wait for the database | If many users click at once, Node.js can handle them without freezing |
| **Express** | A lightweight framework on top of Node.js | Makes it easy to define what URL does what | Without it, managing routes and requests is very manual |
| **express-validator** | Checks incoming data is correct before processing | Catches bad input early | Prevents bad data from crashing the app or corrupting the database |
| **CORS** | Allows the frontend (on a different URL) to talk to the backend | Browsers block cross-domain requests by default | Without CORS config, the frontend simply cannot reach the backend |
| **Morgan** | Logs every request that arrives at the server | Helps in debugging | Without logs, it's hard to know what's happening |

### 🗄️ Database (the "storage room")

| Technology | What it is (simple) | Why we chose it | Problem it solves |
|---|---|---|---|
| **MongoDB** | A database that stores data like JSON documents (not like Excel rows) | Flexible — easy to change data shape without painful migrations | In fast development, requirements change often; SQL would require rewriting table definitions |
| **Mongoose** | A helper that defines a "shape" (schema) for MongoDB data | Adds rules and automatic indexes | Raw MongoDB has no rules; bad data sneaks in without Mongoose |

### 🤖 ML Service (the "prediction advisor")

| Technology | What it is (simple) | Why we chose it | Problem it solves |
|---|---|---|---|
| **FastAPI (Python)** | A Python web framework for building APIs very fast | Perfect for exposing ML models through HTTP endpoints like `/predict` | Lets the Node.js backend "ask for a prediction" like calling any other website |
| **scikit-learn** | A Python library with ready-made machine learning tools | Industry standard for training and using ML models | Training a prediction model from scratch is very hard |
| **pandas** | A Python library for working with data tables | Makes it easy to format input data for the ML model | ML models expect specific data shapes; pandas helps prepare that |
| **joblib** | Saves/loads trained ML models to/from disk (`.pkl` files) | Models can be trained once and reused | Without this, the model would have to retrain every time the server restarts |

### 🗺️ Maps & Donation

| Technology | What it is | Why |
|---|---|---|
| **Google Maps JS** | Interactive map in the browser | Familiar, easy to use, shows NGO pins clearly |
| **Overpass API** | A free search engine for OpenStreetMap data | Finds nearby NGOs/charities without needing our own NGO database |

---

## Key Features (What to Demo in Interviews)

- ✅ **Demand prediction** using historical data + weather + special events (with fallback logic if ML is down)
- ✅ **Inventory tracking** with low-stock alerts and reorder tracking
- ✅ **Menu management** with ingredient requirements per dish
- ✅ **Consumption logging** that automatically decrements stock
- ✅ **Analytics dashboard** — daily waste totals, weekly trends, dish-wise waste comparison (via MongoDB aggregations)
- ✅ **Donation locator** — real-time nearby NGO search on a map
- ✅ **Expiry scan** — image upload with mock ML classification
- ✅ **Prediction history** — saved to the database so dashboard survives restarts

---

## Simple Architecture (Easy Analogy)

Think of AHAR like a **restaurant with 3 rooms and a kitchen advisor**:

- 🧑‍💼 **Frontend (React)** = The reception desk. Talks to the customer (user), takes orders (clicks), shows results.
- 👨‍💼 **Backend (Node.js + Express)** = The manager's office. Checks rules, writes records, decides what happens.
- 🗄️ **MongoDB** = The filing cabinet. Stores all records permanently.
- 🔬 **ML Service (FastAPI)** = The outside consultant. The manager calls the consultant when they need a prediction. If the consultant is busy, the manager makes a rough guess themselves.

```
User clicks a button
       ↓
Frontend sends an HTTP request (a web request carrying JSON data)
       ↓
Backend receives the request → validates data → applies business rules
       ↓
Backend reads/writes MongoDB (stores or fetches data)
       ↓ (sometimes)
Backend calls ML Service to get a prediction
       ↓
Backend sends a JSON response back to Frontend
       ↓
Frontend displays the result on screen
```

---

# 2. Folder Structure & What Each File Does

> **Why this matters in interviews:** Interviewers often ask *"Walk me through your project structure"* or *"Where would you add feature X?"* Knowing exactly what each folder and file does shows that you **truly understand your own project** — not just that you built it.

---

## Top-Level Overview

```
Reckon 7.0/
├── code/                    ← All source code lives here
│   ├── backend/             ← Node.js + Express API server
│   ├── frontend/            ← React (Vite) web application
│   └── ml-service/          ← Python FastAPI ML prediction server
├── models/                  ← Trained ML model files (.pkl)
├── FAANG_INTERVIEW_PREP_AHAR.md   ← This file!
├── FWPS_Hackathon.pptx      ← Presentation slides
└── package.json             ← Root-level scripts (to start all services)
```

**Why split into 3 separate folders?**
Each service is completely independent:
- The frontend is just static files — it doesn't need Python or Node.js to deploy
- The backend can restart without affecting the ML service
- The ML model can be upgraded without touching the frontend

This is called a **microservices-style architecture** — each part has its own job and can be deployed/scaled independently.

---

## 📁 Backend Folder (Node.js + Express)

```
code/backend/
├── server.js                ← Entry point: connects to MongoDB, starts the server
├── app.js                   ← Sets up Express: registers routes, middleware, CORS
├── .env                     ← Secret values (DB URL, ML URL, Clerk keys) — NOT in Git
├── .env.example             ← Template showing which env variables are needed
├── package.json             ← Lists all npm packages the backend depends on
│
├── config/
│   └── db.js                ← MongoDB connection logic (connect + retry on failure)
│
├── models/                  ← Database schema definitions (Mongoose)
│   ├── Ingredient.js        ← Defines the "ingredients" collection shape
│   ├── Dish.js              ← Defines the "dishes" collection with embedded ingredients
│   ├── ConsumptionLog.js    ← Defines the daily cooked/consumed/leftover records
│   ├── PredictionLog.js     ← Saves every demand prediction to DB (for history)
│   ├── EventAdjustment.js   ← Special events (festivals, holidays) with demand multipliers
│   └── Ngo.js               ← NGO locations with geo-coordinates for map search
│
├── controllers/             ← The "brain" — handles the actual business logic
│   ├── inventoryController.js   ← Create/read/update/delete ingredients + stock calculation
│   ├── menuController.js        ← Create/read/update/delete dishes
│   ├── consumptionController.js ← Log a meal service: saves log + decrements stock
│   ├── analyticsController.js   ← MongoDB aggregations: daily waste, weekly trends, dish waste
│   ├── predictionController.js  ← Demand prediction + waste prediction + dish recommendations
│   ├── donationController.js    ← Find nearby NGOs via Overpass API or DB cache
│   └── imageController.js       ← Handle food image upload for expiry classification
│
├── routes/                  ← URL mapping: "Which URL goes to which controller?"
│   ├── inventoryRoutes.js   ← Maps GET/POST/PUT/DELETE /api/inventory → inventoryController
│   ├── menuRoutes.js        ← Maps /api/menu endpoints → menuController
│   ├── consumptionRoutes.js ← Maps /api/consumption → consumptionController
│   ├── analyticsRoutes.js   ← Maps /api/analytics/* → analyticsController
│   ├── predictionRoutes.js  ← Maps /api/predict/* → predictionController
│   ├── donationRoutes.js    ← Maps /api/donations/* → donationController
│   └── imageRoutes.js       ← Maps /api/image/scan-expiry → imageController
│
├── middleware/              ← Code that runs BEFORE the controller on every request
│   ├── errorHandler.js      ← Catches ALL unhandled errors, sends clean JSON response
│   └── validateRequest.js   ← Runs express-validator checks; rejects bad input early
│
├── services/                ← Reusable business logic (shared across controllers)
│   └── stockService.js      ← reduceStockForCooking(): decrements ingredient stock atomically
│
└── scripts/                 ← One-off utility scripts (e.g., seed sample data into DB)
    └── seedData.js
```

### Key File Explanations

**`server.js` — The starting gun**
```js
// Connects to MongoDB, then starts listening for requests
mongoose.connect(DB_URL).then(() => app.listen(PORT));
```
Why separate from `app.js`? `app.js` defines *what* the server does. `server.js` defines *how* to start it. This makes `app.js` importable in tests without actually starting a server.

**`app.js` — The traffic director**
Registers all routes and middleware in order:
```
CORS → JSON parser → Morgan logger → Routes → Error handler
```
The **error handler must be last** — Express only sends errors to it when all routes have been tried.

**`services/stockService.js` — The smart stock updater**
```js
// Uses Promise.all() to update ALL ingredients in parallel (not one by one)
const updates = dish.ingredients.map(async (item) => {
  await Ingredient.findOneAndUpdate(
    { _id: item.ingredientId, kitchenId },
    { $inc: { stockQuantity: -requiredAmount } }  // atomic decrement
  );
});
await Promise.all(updates);
```
Why a separate `services/` folder? This logic is used by the consumption controller. If you put it directly in the controller, you can't easily reuse it or test it. Services are the "helper functions" of the backend.

**`middleware/errorHandler.js` — The safety net**
```js
const errorHandler = (err, req, res, next) => {
  const statusCode = err.statusCode || 500;
  res.status(statusCode).json({ success: false, message: err.message });
};
```
Every controller calls `next(error)` when something fails. This one function handles ALL errors in one place — no duplication.

---

## 📁 Frontend Folder (React + Vite)

```
code/frontend/
├── index.html               ← The single HTML file — React mounts inside this
├── vite.config.js           ← Vite settings (dev server port, build output, etc.)
├── package.json             ← Frontend npm dependencies
├── vercel.json              ← Vercel deployment config (rewrites for React Router)
├── .env                     ← Frontend secrets (API base URL, Clerk publishable key)
│
└── src/                     ← All the actual React source code
    ├── main.jsx             ← Entry point: wraps app in ClerkProvider, renders <App/>
    ├── App.jsx              ← Defines all page routes using React Router
    ├── i18n.jsx             ← Internationalization (multi-language support strings)
    │
    ├── pages/               ← One file = one full page of the app
    │   ├── DashboardPage.jsx       ← Main home dashboard (prediction history, charts, KPIs)
    │   ├── Dashboard.jsx           ← Dashboard subcomponent (summary cards)
    │   ├── InventoryPage.jsx       ← Ingredient list, add/edit/delete, stock levels
    │   ├── InventoryHub.jsx        ← Tabbed wrapper: Inventory + Requirements calculator
    │   ├── MenuManagement.jsx      ← Add/edit dishes + their ingredient amounts
    │   ├── ConsumptionForm.jsx     ← Form to log daily cooked vs consumed meals
    │   ├── PredictionPage.jsx      ← Input form → calls /api/predict/demand → shows results
    │   ├── AnalyticsPage.jsx       ← Embeds charts from analytics API (daily/weekly waste)
    │   ├── DonationLocatorPage.jsx ← Google Map + Overpass API nearby NGO finder
    │   ├── RecommendationsPage.jsx ← Dish recommendations from ML + filter controls
    │   ├── ImageUploadPage.jsx     ← Upload food image → expiry scan result
    │   ├── PaymentPage.jsx         ← Razorpay payment integration page
    │   ├── GuidePage.jsx           ← User guide / help page
    │   ├── SignInPage.jsx          ← Clerk sign-in component wrapper
    │   └── SignUpPage.jsx          ← Clerk sign-up component wrapper
    │
    ├── components/          ← Reusable UI pieces (used across multiple pages)
    │   ├── Layout.jsx       ← App shell: sidebar navigation + top header + main content area
    │   └── ui/              ← Generic styled building blocks
    │       ├── Button.jsx   ← Reusable button with variants (primary, danger, outline)
    │       ├── Card.jsx     ← White box container with shadow — used for every section
    │       ├── Alert.jsx    ← Colored banner for success/error/warning messages
    │       ├── Badge.jsx    ← Small colored pill label (e.g., "High Risk", "Low Stock")
    │       ├── Field.jsx    ← Form input + label combination
    │       ├── PageHeader.jsx ← Consistent page title + description header
    │       └── StatChip.jsx ← Small stat display chip (e.g., "Total Waste: 42 kg")
    │
    ├── services/
    │   └── api.js           ← Creates an Axios instance pre-configured with the backend URL
    │                           (all pages import this — keeps base URL in one place)
    │
    └── styles/              ← CSS stylesheets for the app
```

### Key File Explanations

**`main.jsx` — The entry point**
```jsx
// Wraps the entire app in Clerk's auth provider
<ClerkProvider publishableKey={CLERK_KEY}>
  <App />
</ClerkProvider>
```
Clerk needs to wrap everything so any component can check if the user is logged in.

**`App.jsx` — The router map**
```jsx
<Routes>
  <Route path="/" element={<DashboardPage />} />
  <Route path="/inventory" element={<InventoryHub />} />
  <Route path="/prediction" element={<PredictionPage />} />
  <Route path="/analytics" element={<AnalyticsPage />} />
  ...
</Routes>
```
Think of this as the **address book for all pages**. When the URL changes, React Router reads this file to know what to display.

**`components/Layout.jsx` — The app shell**
This renders the sidebar + top bar that appears on EVERY page. Individual pages are plugged into the `<main>` content area. Why? So you only write the navigation once, not in every single page file.

**`services/api.js` — The API connector**
```js
import axios from 'axios';
export default axios.create({ baseURL: import.meta.env.VITE_API_URL });
```
Every page imports this instead of calling `axios` directly with a hardcoded URL. If the backend URL changes, you change it **in one place only**.

**`vercel.json` — The SPA fix**
```json
{ "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }] }
```
React Router handles navigation in the browser. But if you refresh the page at `/inventory`, Vercel looks for an `inventory.html` file — which doesn't exist. This tells Vercel: *"For any URL, always serve `index.html` and let React handle the rest."*

---

## 📁 ML Service Folder (Python + FastAPI)

```
code/ml-service/
├── app.py               ← Main FastAPI application: all endpoints (/health, /predict, /recommend)
├── requirements.txt     ← Python package dependencies (fastapi, scikit-learn, pandas, joblib)
├── README.md            ← How to set up and run the ML service
└── src/
    ├── __init__.py      ← Makes `src` a Python package (required for imports)
    └── utils.py         ← Helper functions shared across the ML service
```

### Key File Explanations

**`app.py` — The entire ML brain**
Three endpoints, all in one file (small service = fine to keep simple):

| Endpoint | What it does |
|---|---|
| `GET /health` | Reports if model files are loaded and present |
| `POST /predict` | Accepts kitchen features → returns predicted food waste number |
| `POST /recommend` | Accepts filters → ranks candidate dishes → returns top K |

**Why lazy-load models?**
```python
_food_waste_bundle: Optional[dict] = None

def _get_food_waste_bundle() -> dict:
    global _food_waste_bundle
    if _food_waste_bundle is None:                   # only load once
        _food_waste_bundle = _load_bundle(MODEL_PATH)
    return _food_waste_bundle
```
Loading a `.pkl` model from disk is slow. By loading it **only on first use** and keeping it in memory, every subsequent prediction request is fast.

**`requirements.txt` — The dependency list**
```
fastapi
uvicorn           ← the web server that runs FastAPI
scikit-learn      ← the ML library
pandas            ← data formatting
joblib            ← loads .pkl model files
```
When deploying, `pip install -r requirements.txt` installs everything the service needs.

**Why separate `src/utils.py`?**
Utility functions (e.g., input normalization helpers) that could be used by multiple files go here. Keeps `app.py` focused on the API layer only.

---

## 📁 Models Folder (at root level)

```
Reckon 7.0/models/
├── food_waste_model.pkl      ← Trained scikit-learn LinearRegression model
│                                (predicts food waste from kitchen features)
└── dish_recommender_model.pkl ← Trained model for ranking dishes
                                 (includes candidate_dishes + feature_cols in bundle)
```

**What is a `.pkl` file?**
`.pkl` (pickle) is a Python format for saving any Python object — including a trained ML model — to disk. `joblib.dump(model, "file.pkl")` saves it; `joblib.load("file.pkl")` brings it back. The model doesn't need to be retrained every time the service starts.

**The bundle format (important interview point):**
```python
bundle = {
  "model": trained_sklearn_model,
  "feature_names": ["occupancy_rate", "temperature_c", "weather_rain", ...],
  # For recommender also:
  "feature_cols": [...],
  "candidate_dishes": [{ "dish_name": "...", "cuisine": "...", ... }]
}
```
Why bundle everything together? Because the model file and the feature list it was trained on must stay in sync. If the feature list is stored separately and someone updates one without the other, predictions break.

---

## Interview Q&A — Folder Structure

### Q: "Why do you have a `services/` folder separately from `controllers/`?"

> "Controllers handle HTTP requests and responses — they receive input, call the right logic, and send back a response. Services hold the **reusable business logic** that doesn't deal with HTTP at all.
>
> For example, `stockService.js` reduces ingredient stock when a meal is cooked. This same logic could be needed by a batch job, a scheduled task, or an admin override endpoint — not just the consumption route. By putting it in a service, I can call it from anywhere without copy-pasting code."

---

### Q: "Why is there a `vercel.json` in the frontend?"

> "React apps are Single Page Applications (SPAs). There's only ONE actual HTML file (`index.html`). React Router simulates different pages by changing the URL in the browser without actually loading a new file. But when you deploy to Vercel and refresh the page at `/inventory`, Vercel looks for a real file called `inventory.html` — which doesn't exist — and returns a 404 error. The `vercel.json` rewrite rule tells Vercel: 'For ANY URL path, always serve `index.html` and let React handle routing.'"

---

### Q: "Why is the ML model a separate service and not just imported into Node.js?"

> "Node.js can't run Python code directly. ML libraries like scikit-learn, pandas, and numpy are Python-only. Instead of trying to integrate Python into a JavaScript runtime (which would be very messy), we run the ML model as a completely separate HTTP service. The Node.js backend calls it just like any other API — with a POST request and a JSON response. This also means the ML service can be deployed on a GPU server while the backend lives on a cheaper CPU server."

---

# 3. System Design — Big Picture

## High-Level Design (3-Tier Architecture)

```
┌─────────────────────────────────────────────┐
│              TIER 1: Frontend               │
│   React (Vite)  ─  Clerk Auth  ─  Axios    │
│   Pages: Inventory, Analytics, Prediction   │
└────────────────────┬────────────────────────┘
                     │  HTTPS / REST API
┌────────────────────▼────────────────────────┐
│              TIER 2: Backend                │
│   Node.js + Express API server              │
│   Routes → Middleware → Controllers         │
│   Timeout + Fallback logic for ML           │
└──────────┬──────────────────┬───────────────┘
           │                  │
  ┌────────▼──────┐  ┌────────▼──────────────┐
  │  TIER 3:      │  │  ML Microservice       │
  │  MongoDB      │  │  FastAPI (Python)      │
  │  (Database)   │  │  /predict  /recommend  │
  └───────────────┘  └───────────────────────┘
```

**Important point:** The ML service is a **separate microservice**. This means:
- It can be deployed independently
- It can be scaled independently (ML needs more CPU; Node.js doesn't)
- If it crashes, the backend still works using fallback logic

---

## Inside the Backend — The "Layers" Pattern

Every feature in the backend is broken into 4 layers. Let's explain each simply:

```
Request arrives
      ↓
1. Route      → "Which URL? Which HTTP method?"
      ↓
2. Middleware → "Is the data valid? Is the user logged in?"
      ↓
3. Controller → "Do the main work. Call DB or ML. Send response."
      ↓
4. Model      → "Define the database table shape and query DB."
```

**Real example — when a user logs consumption:**

| Layer | File | Job |
|---|---|---|
| Route | `consumptionRoutes.js` | Maps `POST /api/consumption` to the right controller |
| Middleware | `express-validator` rules | Checks `cooked ≥ 0`, `dishId` is valid, etc. |
| Controller | `consumptionController.js` | Calculates leftover, saves log, decrements stock |
| Model | `ConsumptionLog.js`, `Ingredient.js` | Defines MongoDB schema; runs DB queries |

---

# 3. Database Design (Deep Dive)

## Why MongoDB? (SQL vs NoSQL — Beginner Friendly)

**SQL databases** (like MySQL/PostgreSQL) store data like a spreadsheet:
- Every row must have the same columns
- Changing the structure requires "migrating" the table (can be risky and slow)
- Very good for strict relationships and complex joins

**NoSQL databases** (like MongoDB) store data like JSON files:
- Each document can have different fields
- Adding a new field = just start storing it (no migration)
- Great when requirements change often during development

**Why MongoDB fits AHAR:**
- During a hackathon/fast build, requirements change often
- A `Dish` document naturally wants to embed its ingredients as a list — this is awkward in SQL (needs a separate join table)
- JavaScript and JSON are the same shape, so data flows naturally from frontend → backend → database

---

## Database Schema (Plain English)

### Collection 1: `ingredients`
```
Ingredient {
  kitchenId:     "kitchen-nyc-001"    // which kitchen owns this
  name:          "Rice"               // ingredient name
  stockQuantity: 50                   // how much is in stock (in "unit" units)
  unit:          "kg"                 // measurement unit
  reorderDays:   3                    // days ahead to order more
  createdAt, updatedAt                // automatic timestamps
}
```
**Unique rule:** No kitchen can have two ingredients with the same name (`kitchenId + name` must be unique). This is a **database-level unique index** — the database enforces it, not just our code.

### Collection 2: `dishes`
```
Dish {
  kitchenId: "kitchen-nyc-001"
  name:      "Chicken Fried Rice"
  ingredients: [                   // EMBEDDED LIST (no separate table!)
    { ingredientId, name, amountPerMeal: 0.2, unit: "kg" },
    { ingredientId, name, amountPerMeal: 0.05, unit: "kg" }
  ]
  quantityPerPerson: 0.4           // portion size
}
```
**Design choice:** Ingredients are **embedded** inside Dish (not a separate collection). Why?
- When we show a dish, we always need its ingredients too
- Embedding means one DB read instead of two
- The tradeoff: if an ingredient's name changes, it's not automatically updated inside dishes

### Collection 3: `consumptionlogs`
```
ConsumptionLog {
  kitchenId: "kitchen-nyc-001"
  dishId:    ObjectId (points to a Dish)
  cooked:    120                   // meals cooked
  consumed:  105                   // meals eaten
  leftover:  15                    // meals wasted (cooked - consumed)
  date:      2024-04-15
}
```
**Composite index:** `{ kitchenId: 1, date: 1 }` — because analytics always filters by both kitchen AND date. Without this index, MongoDB scans every record.

### Collection 4: `predictionlogs`
Records every demand prediction made. Why save this?
- The dashboard shows prediction **history**
- After a server restart, history would be lost if only stored in the browser
- Saving to DB = survives restarts ✅

### Collection 5: `ngos`
```
Ngo {
  name, address,
  location: { type: "Point", coordinates: [longitude, latitude] }
}
```
**Geo index:** `{ location: "2dsphere" }` — this special index lets MongoDB find NGOs within X km of the user's location. Without it, finding nearby NGOs would require checking every NGO one by one.

### Collection 6: `eventadjustments`
```
EventAdjustment {
  kitchenId, name,
  specialDemandMultiplier: 1.3   // 30% more demand on this day
  holidayFlag: true
}
```
Used in demand prediction: if a festival is happening, multiply the base prediction by 1.3.

---

## What is an Index? (Simple Example)

Imagine a library with 10,000 books. Without a catalog (index), you search shelf by shelf. With a catalog (index), you go directly to the right shelf.

**Indexes in AHAR:**
- `{ kitchenId: 1 }` on Ingredient → fetch all ingredients for one kitchen fast
- `{ kitchenId: 1, name: 1 }` unique on Ingredient → fast duplicate check
- `{ kitchenId: 1, date: 1 }` on ConsumptionLog → fast analytics queries
- `{ location: "2dsphere" }` on NGO → fast "find NGOs near me" queries

**The tradeoff:** Indexes make reads faster but writes slightly slower (because the index must also be updated on every write). This is usually worth it for read-heavy analytics data.

---

# 4. API Design — How the App Talks to Itself

## What is an API? (Very Simple)

An API (Application Programming Interface) is a set of URLs your code can call to get things done.

Think of it like a **menu at a restaurant**:
- The menu lists what you can order (the endpoints)
- You place your order (send a request with data)
- The kitchen prepares it and brings it back (the response)

## HTTP Methods — The "Verbs"

| Verb | Meaning | Example |
|---|---|---|
| `GET` | Fetch data (don't change anything) | Get list of ingredients |
| `POST` | Create something new | Add a new dish |
| `PUT` | Update an existing thing | Edit an ingredient's stock |
| `DELETE` | Remove something | Delete an old dish |

## AHAR's API Endpoints

| Endpoint | Method | What it does |
|---|---|---|
| `/api/health` | GET | "Is the server alive?" health check |
| `/api/inventory` | GET/POST/PUT/DELETE | Manage ingredients |
| `/api/inventory/requirements` | POST | Calculate ingredient shortages for predicted meals |
| `/api/menu` | GET/POST/PUT/DELETE | Manage dishes |
| `/api/consumption` | POST/GET | Log cooked/consumed meals |
| `/api/analytics/waste-dashboard` | GET | Daily waste totals, weekly trends, dish-wise waste |
| `/api/analytics/weekly-sustainability` | GET | This week vs last week waste comparison |
| `/api/predict/demand` | POST | Predict how many meals to cook |
| `/api/predict/waste` | POST | Predict expected food waste (calls ML service) |
| `/api/predict/recommend` | POST | Recommend dishes (calls ML service) |
| `/api/predict/history` | GET | Get past predictions from DB |
| `/api/donations/ngos` | GET | Find nearby NGOs |
| `/api/image/scan-expiry` | POST | Upload ingredient photo for expiry classification |

## Request/Response Example — Demand Prediction

**Frontend sends this (a POST request with JSON):**
```json
{
  "kitchenId": "kitchen-nyc-001",
  "pastConsumption": [120, 130, 115, 140, 128],
  "dayOfWeek": "Friday",
  "expectedPeople": 145,
  "events": ["Founders Day"],
  "weather": "Rainy"
}
```

**Backend does this:**
1. Fetches event adjustments from DB (`{"name": "Founders Day"}` → multiplier `1.3`)
2. Calculates: `base = avg(past) * 0.7 + expectedPeople * 0.3`
3. Applies: `predicted = base × eventMultiplier(1.3) × weatherMultiplier(1.05)`
4. Rounds the result
5. Saves a `PredictionLog` to MongoDB
6. Returns:

```json
{
  "predictedQuantity": 178,
  "surplusRisk": true,
  "donationRecommended": true,
  "adjustmentFactors": {
    "eventMultiplier": 1.3,
    "weatherMultiplier": 1.05,
    "dayOfWeek": "Friday"
  }
}
```

---

# 5. Interview Questions by Topic

## 5.1 Data Structures & Algorithms

> **Quick reminder:**
> - **Time complexity** = "how much slower does it get as data grows?"
> - **O(1)** = always same speed | **O(n)** = grows with data | **O(n log n)** = grows but manageable
> - **Space complexity** = "how much extra memory does it need?"

---

### Q1: How is leftover food (waste) calculated?

**Simple answer:**
```
leftover = cooked - consumed
```
From the actual `ConsumptionLog` model:
```js
leftover: { type: Number, required: true, min: 0 }
```
The `min: 0` constraint means if someone enters `consumed > cooked`, the DB rejects it.

- Time: `O(1)` — just a subtraction
- Space: `O(1)` — no extra memory needed

---

### Q2: How do you calculate required ingredients for a predicted number of meals?

**Real code from `inventoryController.js`:**
```js
const neededMap = new Map();

dishDocs.forEach((dish) => {
  dish.ingredients.forEach((ingredient) => {
    const current = neededMap.get(String(ingredient.ingredientId)) || { required: 0, ...ingredient };
    current.required += ingredient.amountPerMeal * predictedMeals;
    neededMap.set(String(ingredient.ingredientId), current);
  });
});
```

**Why a Map (dictionary)?**
- A Map lets you quickly look up "how much Rice do we need so far?" by ingredientId
- Much faster than searching through a list every time

**Then it compares with stock:**
```js
const shortage = Math.max(0, item.required - stockItem.stockQuantity);
```

- Time: `O(R)` where R = total recipe-ingredient pairs across all selected dishes
- Space: `O(I)` where I = number of unique ingredients

---

### Q3: How does the analytics dashboard find the top wasted dishes?

**Real MongoDB aggregation from `analyticsController.js`:**
```js
ConsumptionLog.aggregate([
  { $group: { _id: '$dishId', totalLeftover: { $sum: '$leftover' } } },
  { $lookup: { from: 'dishes', localField: '_id', foreignField: '_id', as: 'dish' } },
  { $sort: { totalLeftover: -1 } }
])
```

**What is an aggregation? (Simple)**
Think of it like Excel formulas on your data:
- `$group` = group all rows by dishId and add up their leftovers (like a SUM in Excel)
- `$lookup` = join with the dishes collection to get dish names (like VLOOKUP in Excel)
- `$sort` = sort by most wasted first

This all happens **inside the database**, not in Node.js code — so it's very efficient even with millions of records.

- Time: `O(N)` to scan logs + sorting the grouped results
- Space: `O(D)` where D = distinct dishes in the time range

---

### Q4: How do you find duplicate ingredients?

**Database-level unique index (real code from `Ingredient.js`):**
```js
ingredientSchema.index({ kitchenId: 1, name: 1 }, { unique: true });
```

This means MongoDB **automatically rejects** any attempt to insert a duplicate `(kitchenId, name)` pair. The code catches this:
```js
if (error?.code === 11000) {   // 11000 = MongoDB duplicate key error
  return res.status(409).json({ message: "Ingredient already exists" });
}
```

- No big algorithm needed — the database constraint handles it

---

### Q5: How is the demand prediction formula calculated?

**Real code from `predictionController.js`:**
```js
const calculateBaseDemand = (pastConsumption, expectedPeople) => {
  const avgPast = pastConsumption.reduce((sum, v) => sum + v, 0) / pastConsumption.length;
  return (avgPast * 0.7) + (expectedPeople * 0.3);
};
```

**Why this formula?**
- Historical average (`avgPast`) tells us what's happened before — weighted at 70%
- Expected headcount (`expectedPeople`) tells us about today — weighted at 30%
- It's a weighted average: past data matters more than the stated headcount (people often mis-estimate attendance)

Then event and weather multipliers are applied:
```js
const predictedQuantity = Math.round(
  calculateBaseDemand(pastConsumption, expectedPeople) * eventMultiplier * weatherMultiplier
);
```

- Time: `O(n)` where n = length of pastConsumption array
- Space: `O(1)`

---

### Q6: How do you avoid running out of stock after logging consumption?

**The flow:**
1. When consumption is logged, the backend calculates how many ingredients were used per dish
2. It groups ingredient usage in a map (same pattern as Q2)
3. Then decrements stock using MongoDB's `$inc` operator

Why `$inc` instead of read-then-write?
```
❌ Wrong way: read stock (say 50), subtract in code (50 - 5 = 45), write back 45
✅ Right way: MongoDB does atomic: { $inc: { stockQuantity: -5 } }
```
**Atomic** means the database does it as one single operation — no risk of two requests interfering.

---

### Q7: How do you prevent negative stock?

**Two-layer defense:**
1. **Schema-level**: `stockQuantity: { min: 0 }` — Mongoose rejects updates that would set it below 0
2. **Application-level**: check before decrementing; if stock would go negative, return a warning

---

### Q8: How do you quickly find NGOs near a user?

**Geo index on the NGO model:**
```js
ngoSchema.index({ location: '2dsphere' });
```

Then query with:
```js
Ngo.find({
  location: { $near: { $geometry: userLocation, $maxDistance: 10000 } }
})
```

This uses the special geospatial index. Without it, finding "within 10 km" would require calculating distance to every single NGO in the database.

---

## 5.2 Object-Oriented Design

### Q1: What design principle does AHAR's folder structure follow?

**Single Responsibility Principle (SRP)** — "each module/file should do only one main job."

| File/Module | Its ONE job |
|---|---|
| `routes/inventoryRoutes.js` | Maps URLs to controller functions |
| `middleware/errorHandler.js` | Catches and formats all errors |
| `controllers/analyticsController.js` | Handles analytics logic, sends responses |
| `models/ConsumptionLog.js` | Defines the database schema |
| `services/` | Reusable business logic (e.g., stock decrement) |

**Why this matters in interviews:**
> "If a bug is in the analytics, you only look in `analyticsController.js`. If the DB schema is wrong, you look in the model. No file does everything."

---

### Q2: Why is the ML service a separate service (microservice)?

**Separation of Concerns** — different types of work should be separated.

**Benefits:**
- Node.js handles web logic; Python handles ML. Each uses the best language for the job.
- You can update the ML model without touching the backend
- You can scale the ML service separately (ML is CPU-heavy; Node.js is not)
- If ML goes down, the backend can still serve other features

---

### Q3: What design pattern handles ML failure gracefully?

**The "Try-Fallback" pattern.** Real code from `predictionController.js`:

```js
try {
  // Try calling the ML service
  const { data } = await axios.post(modelUrl, { features }, { timeout: 5000 });
  return res.status(200).json({ predictedWaste: data.prediction, ml: { provider: 'ml-service' } });
} catch (mlError) {
  // ML is down — calculate a rough estimate ourselves
  const estimatedWaste = Math.max(0, mealsPrepared - (prev7DayAvg * occupancyRate));
  return res.status(200).json({ predictedWaste: estimatedWaste, ml: { provider: 'fallback' } });
}
```

**Key point:** The `timeout: 5000` means if ML doesn't reply within 5 seconds, we don't wait forever — we use the fallback. This keeps the app **responsive even when a dependency is slow**.

---

### Q4: How would you make this code easier to test (unit tests)?

**Dependency Injection (simple version):**

Instead of calling `axios.post(mlUrl, ...)` directly inside a controller, put that logic in a helper service:
```js
// services/mlService.js
const getPrediction = async (features) => axios.post(ML_URL, features);
```

In tests, replace `mlService` with a **fake version** that returns controlled data. Controllers stay clean and testable.

---

## 5.3 System Design

### Q1: What happens if 10,000 users use AHAR at the same time?

**Current single-server setup:**
```
10,000 users → 1 backend server → MongoDB
```
This will get slow or crash.

**Solution: Horizontal Scaling**
```
10,000 users → Load Balancer → [Server 1, Server 2, Server 3] → MongoDB
```

- A **load balancer** is like a traffic police officer that directs each car (request) to the least-busy lane (server)
- All servers share the same MongoDB database
- MongoDB itself can be scaled with **replica sets** (copies of the DB for more reads)

---

### Q2: What is caching and why would AHAR benefit from it?

**Caching = keeping a copy of a result nearby so you don't recalculate it repeatedly.**

Restaurant analogy: A chef prepares the "soup of the day" once in the morning and serves that same soup all day — not cooking it fresh for every customer.

**Where caching helps in AHAR:**
- The analytics dashboard (`/api/analytics/waste-dashboard`) aggregates ALL logs — slow on large data
- The menu list doesn't change often — no need to query DB on every page load
- NGO results near a fixed location stay the same for hours

**How to add it:**
```
Cache (Redis) ← check: is result already stored?
      ↓ No
MongoDB → compute result → store in cache → return result
      ↓ Yes
Return cached result immediately (very fast)
```

---

### Q3: How would you handle 1 million predictions per day efficiently?

**Problems at scale:**
1. Each prediction saves a `PredictionLog` to MongoDB — 1M writes/day → DB gets slow
2. Analytics aggregations on 1M records → slow queries

**Solutions:**
- **Async queue** for prediction logging: Put logs in a queue (like Redis or RabbitMQ); a background worker writes to DB in batches
- **Pre-computed rollups**: Instead of aggregating on every dashboard load, run a scheduled job every hour that pre-computes daily/weekly summaries and stores them
- **Read replicas**: Separate servers for reading analytics, primary server for writes

---

### Q4: What is database sharding? (Simple)

**Sharding = splitting the database across multiple servers.**

In AHAR, `kitchenId` is the natural "shard key":
- Kitchen A's data lives on Server 1
- Kitchen B's data lives on Server 2
- Queries for kitchen A never touch Server 2

**Why:** As data grows, no single server can hold it all. Sharding keeps queries fast by keeping each server's dataset small.

---

### Q5: How would you redesign AHAR if it needed to process expiry scan images at scale?

**Current:** Image uploaded → backend does simple classification → returns mock status

**At scale:**
```
User uploads image
       ↓
Backend saves file to S3 (cloud storage)
       ↓
Backend pushes a "classify this image" job to a queue
       ↓
ML worker picks the job, runs the real model, saves result
       ↓
User gets a webhook/socket notification when done
```

Why this is better: The web request returns immediately (no waiting for ML). The ML work happens in the background.

---

## 5.4 Database & Backend

### Q1: What is a transaction? Why does AHAR need them?

**Transaction = "all steps happen together perfectly, or none of them happen."**

ATM analogy:
- Debit your account: ✅
- Credit the recipient: ❌ (fails)
- Without transaction: money is lost
- With transaction: both steps are rolled back

**In AHAR — the consumption log + stock decrement problem:**
```
Step 1: Save ConsumptionLog → ✅ succeeds
Step 2: Decrement Ingredient stock → ❌ crashes

Without transaction: Log exists but stock is not decremented → DATA MISMATCH
With transaction: Both roll back → data stays consistent
```

MongoDB supports transactions on replica sets. For production, this should be used.

---

### Q2: What is idempotency? Why does it matter here?

**Idempotency = doing the same operation twice gives the same result as doing it once.**

**Problem in AHAR:** User clicks "Submit Consumption" and the internet is slow. They click again. Now the server receives 2 requests. Stock gets decremented twice!

**Fix:**
1. Frontend sends a unique `requestId` (e.g., a UUID) with the request
2. Backend checks: "Have I processed this `requestId` before?"
3. If yes: return the same response without processing again
4. If no: process, save the `requestId`, return response

---

### Q3: How does the backend handle errors?

**Centralized error handler** in `app.js`:
```js
app.use(errorHandler); // at the bottom — catches ALL unhandled errors
```

Inside `errorHandler.js`:
- All controllers call `next(error)` when something goes wrong
- The error handler formats the error and sends a clean JSON response
- The user never sees a raw server crash

**Why centralize?** Without it, every controller would have copy-pasted error handling code — a maintenance nightmare.

---

### Q4: What is input validation and why is it critical?

**Validation = checking that data is what you expect before trusting it.**

Example: The prediction endpoint expects `expectedPeople` to be a positive number. If someone sends `-500` or `"hello"`, the app would break.

**AHAR uses `express-validator`:**
```js
body('expectedPeople').isInt({ min: 1 })
body('pastConsumption').isArray({ min: 1 })
```

**Why this matters for security:**
- Prevents **injection attacks** (malicious data that tricks the DB or breaks logic)
- Prevents crashes from unexpected data types
- Documents the expected API contract

---

## 5.5 Frontend

### Q1: How is the React UI structured?

```
src/
├── pages/          ← One file per page (Inventory, Analytics, Prediction, etc.)
├── components/     ← Reusable UI pieces (cards, buttons, charts)
├── services/       ← API call functions (so pages don't have axios everywhere)
└── styles/         ← CSS files
```

**Why separate pages from components?**
- If you need a "chart card" in 3 different pages, you put it in `components/` and reuse it
- Changes to the card's style apply everywhere automatically

---

### Q2: What is "state" in React? (Very Simple)

**State = the data that the page is currently "remembering."**

Examples:
- The list of ingredients loaded from the backend (state)
- Whether the loading spinner is showing (state)
- The form data the user is typing (state)

In React: `const [ingredients, setIngredients] = useState([]);`

When `setIngredients(newData)` is called, React automatically updates the screen.

---

### Q3: What is the problem with calling the Overpass API on every keystroke?

If the user types a location letter by letter, each letter triggers an API call. That's 10+ calls for "Bangalore" — expensive and slow!

**Fix: Debouncing**
- Wait 500ms after the user stops typing
- Only then make the API call

```js
// pseudocode
useEffect(() => {
  const timer = setTimeout(() => callOverpassAPI(location), 500);
  return () => clearTimeout(timer); // cancel if user types again
}, [location]);
```

---

### Q4: How would you add global state management at scale?

Currently: each page fetches its own data with `useState` + `useEffect`.

**At scale, use React Query or SWR:**
- Automatically caches results (no duplicate API calls if you visit a page twice)
- Handles loading and error states
- Refetches on window focus
- Reduces boilerplate significantly

---

## 5.6 DevOps & Deployment

### Q1: What is CI/CD? (Simple)

**CI/CD = an automatic pipeline that builds, tests, and deploys your code.**

Without CI/CD (manual):
```
Developer writes code → manually runs tests → manually builds → manually deploys
```

With CI/CD:
```
Developer pushes to GitHub → pipeline auto-runs tests → auto-builds → auto-deploys
```

**Benefits:** Faster releases, fewer human mistakes, consistent deployments.

---

### Q2: Why use Docker?

**Docker = packing your app with everything it needs to run, in a portable box.**

Problem without Docker:
- "It works on my laptop but not on the server!" (different Node.js version, missing packages, etc.)

With Docker: The app + its exact Node.js version + all packages are in one "image". The server just runs the image — same result everywhere.

**For AHAR (3 services = 3 containers):**
```yaml
# docker-compose.yml (concept)
services:
  frontend:  build: ./frontend
  backend:   build: ./backend
  ml:        build: ./ml-service
  mongo:     image: mongo:latest
```

---

### Q3: How would you deploy AHAR step by step?

```
Step 1: Frontend
   → Build: npm run build (creates static HTML/JS/CSS files)
   → Deploy: Vercel or Netlify (free, fast, CDN-distributed)

Step 2: Backend
   → Deploy on: Railway, Render, or AWS EC2
   → Environment variables in .env (NEVER commit .env to GitHub)
   → Reverse proxy with Nginx

Step 3: ML Service
   → Deploy as Docker container (Python environment is tricky without Docker)
   → On: Railway, Google Cloud Run, or AWS Lambda

Step 4: Database
   → Use MongoDB Atlas (managed cloud MongoDB — no server maintenance)
   → Enable backups and IP allowlisting for security

Step 5: Environment & Secrets
   → DATABASE_URL, ML_DEMAND_URL, CLERK_SECRET_KEY go in environment variables
   → Never hardcode!
```

---

# 6. How Multitasking Works (VERY IMPORTANT)

## The Restaurant Story 🍴

Imagine a very busy restaurant:
- **Customers** = users making requests
- **Waiters** = the backend server
- **Kitchen** = the database (MongoDB)
- **Special consultant** = the ML service

When 100 customers arrive at once, the restaurant needs a strategy. Let's see two different approaches:

---

## Approach 1: One Waiter, One Task at a Time (Bad)

```
Customer A orders → Waiter goes to kitchen → Waits 5 minutes → Comes back → Serves A
Customer B can now order → Waiter goes to kitchen → Waits 5 minutes...
```

100 customers × 5 minutes = **500 minutes total wait**. Everyone is **angry**.

This is like a **single-threaded synchronous** server — it does one thing at a time.

---

## Approach 2: One Smart Waiter, Many Orders (Node.js approach) ✅

```
Customer A orders → Waiter notes it down, goes to kitchen, WAITS
While waiting for A → Customer B arrives → Waiter takes B's order, goes to kitchen
While A and B are cooking → Customer C arrives → Waiter takes C's order too
A's food is ready → Waiter serves A → B's food is ready → Waiter serves B → etc.
```

The waiter **never sits idle**. While waiting for one thing, they handle other things.

This is exactly **how Node.js works**!

---

## The Event Loop — The Waiter's Notebook

Node.js has an **event loop** — think of it as the waiter's notebook and decision system:

```
1. New request arrives → write it in the notebook
2. Start processing (send request to DB)
3. While DB is thinking → pick up next request from the notebook
4. DB responds → event loop picks up that task and continues it
5. Repeat forever
```

**Node.js is single-threaded** (one waiter!) but **non-blocking** (never waits idle). This is why it can handle thousands of requests without needing thousands of threads.

---

## Threads vs Async — What's the Difference?

| | Threads (multi-threaded) | Async (Node.js style) |
|---|---|---|
| **Analogy** | 100 waiters, one per customer | 1 smart waiter managing 100 orders |
| **How** | OS creates separate "workers" | One worker switches between tasks |
| **Good for** | CPU-heavy work (video encoding) | I/O-heavy work (waiting for DB) |
| **Problem** | Threads use lots of memory; coordination is hard | Not good for pure CPU work |
| **AHAR uses** | Node.js async for backend, Python processes for ML | ML uses separate workers (CPU-heavy) |

**AHAR's backend is I/O-heavy** (mostly waiting for MongoDB or the ML service), so Node.js's async model is a perfect fit.

---

## Step-by-Step: What Happens When a User Logs Consumption

```
👤 User clicks "Submit" on the consumption form
   ↓
📱 Frontend (React): axios.post('/api/consumption', { kitchenId, dishId, cooked: 120, consumed: 105 })
   ↓
🌐 HTTP Request travels over the internet to the backend
   ↓
🔐 Backend Middleware 1: CORS check → "Is this allowed origin?"
   ↓
✅ Backend Middleware 2: express-validator → "Is cooked a number ≥ 0? Is dishId valid?"
   ↓
🧠 Controller begins:
   - leftover = 120 - 105 = 15
   - Save ConsumptionLog({ cooked:120, consumed:105, leftover:15 }) to MongoDB
   [Event loop sends this to MongoDB and moves on to other requests while waiting]
   ↓
📦 MongoDB saves the log → signals back "Done"
   ↓
🧠 Controller continues:
   - Fetch the Dish's ingredients from MongoDB
   - Calculate total ingredient usage: rice: 0.2kg × 120 = 24kg
   - Decrement stock: Ingredient.updateOne({ $inc: { stockQuantity: -24 } })
   ↓
✅ Backend sends 201 Created response
   ↓
📱 Frontend: shows "Logged successfully" toast message, refreshes the inventory display
```

**Total time: ~100-300ms** for a single user. Node.js handles other requests while any MongoDB step is in-progress.

---

## How the System Avoids Crashing

| Threat | Protection |
|---|---|
| Bad input data | `express-validator` middleware rejects before processing |
| ML service is slow | `timeout: 5000ms` — if ML takes >5s, use fallback formula |
| ML service is down | Try-catch fallback in `predictWaste` and `recommendDishes` |
| MongoDB is down at startup | `server.js` has retry logic — keeps trying to connect |
| Too many requests | Rate limiting (recommended: `express-rate-limit`) |
| Server crash | PM2 process manager restarts the server automatically |

---

## Race Conditions — When Two Things Collide

**Race condition = two operations happening at the same time, messing each other up.**

**Example in AHAR:**

````
Time 0ms: User A clicks Submit (stock = 50 kg)
Time 1ms: User B clicks Submit too (stock is still 50 kg — hasn't updated yet)
Time 50ms: A's update → stock = 50 - 20 = 30
Time 51ms: B's update → stock = 50 - 20 = 30  ← WRONG! Should be 10!
````

Stock should be 10 (50 - 20 - 20) but instead it's 30. Stock is now incorrect.

**Fix: MongoDB's Atomic `$inc` operator:**
```js
Ingredient.updateOne({ _id: id }, { $inc: { stockQuantity: -20 } })
```

`$inc` is **atomic** — the database handles the subtraction internally as one unbreakable step. Two simultaneous `$inc -20` operations on stock=50 will always give stock=10. ✅

**Other fixes:**
- **Idempotency key**: If user submits twice, the second request is detected and ignored
- **Transactions**: For multi-step operations that must all succeed-or-fail together

---

## Queues and Load Balancing — When You Really Scale

### Load Balancer (Traffic Police 🚦)

```
1000 users/second
      ↓
  [Load Balancer]
    /     |     \
Server1  Server2  Server3
   ↓         ↓        ↓
         MongoDB
```

The load balancer checks which server is least busy and sends the next request there. Popular tools: Nginx, AWS ELB.

### Queue (Waiting Line 🧾)

**When a task is too heavy to do immediately:**
```
User uploads expiry scan image
       ↓
Backend: "Got it! Job ID: ABC123" → returns immediately
       ↓
Job ABC123 goes into a queue (Redis/RabbitMQ)
       ↓
ML Worker picks up job → runs classification model → saves result → notifies user
```

**What could be queued in AHAR:**
- Daily waste report emails
- Analytics rollup computations
- Expiry scan image processing
- Heavy ML batch predictions

---

# 7. Behavioral + Project Questions

## Why did you choose this tech stack?

> "We chose React because it lets us build a complex dashboard UI with reusable components — the page updates automatically when data changes, without refreshing the whole screen.
>
> Node.js + Express for the backend because it handles many simultaneous requests efficiently, especially for I/O-heavy work like database reads — we're mostly waiting on MongoDB, not doing heavy computation.
>
> MongoDB because during a hackathon, requirements change fast. With a NoSQL document store, we didn't need to create migrations every time we added a new field. Plus, embedding ingredient lists inside dish documents felt very natural.
>
> FastAPI for the ML service because Python has the best ML library ecosystem, and FastAPI is the fastest Python framework. Keeping ML separate means we can upgrade the model anytime without touching the main backend."

---

## What problems did you face and how did you solve them?

> "The biggest challenge was making the app reliable when external services are down. The ML service could be slow or unavailable. So I added a `timeout: 5000ms` on ML calls, and if the call fails, the backend returns a fallback estimate calculated from historical averages. The user still gets a useful answer — they don't even know the ML was unavailable.
>
> Another challenge was the sklearn version compatibility issue. The saved `.pkl` model file was trained with an older version of scikit-learn. The loading worked, but running the preprocessing pipeline failed on newer versions. I solved this by manually encoding the features (one-hot encoding categorical variables like weather type, menu type) in Python — avoiding the saved pipeline altogether.
>
> We also had a data persistence issue: the prediction history was stored in browser localStorage, so it was lost on server restart. I added a `PredictionLog` MongoDB collection and saved every prediction there so the dashboard always shows full history."

---

## How did you improve performance?

> "I used MongoDB aggregation pipelines for analytics instead of fetching all logs to Node.js and computing in JavaScript. Running aggregations database-side means only the summarized result travels over the network — not thousands of raw records.
>
> I added compound indexes like `{ kitchenId: 1, date: 1 }` on ConsumptionLog so analytics queries are fast even with large datasets.
>
> I used `$inc` atomic operations for stock updates to avoid race conditions without needing full transactions.
>
> I also used `map()` to batch ingredient usage updates — instead of updating stock record-by-record, I aggregate all decrements first and make one update per ingredient."

---

## What will you change if 1 million users use it?

> "First, I'd run multiple backend instances behind a load balancer (Nginx or AWS ELB). They all share the same MongoDB Atlas cluster.
>
> Second, I'd add Redis caching for read-heavy endpoints — the analytics dashboard and menu list don't change every second, so caching them for 5-10 minutes would cut DB load dramatically.
>
> Third, I'd use a job queue (Bull/BullMQ with Redis) for heavy tasks like analytics rollup, expiry scan processing, and prediction logging. These don't need to block the HTTP response.
>
> Fourth, I'd add MongoDB replica sets for high read availability, and consider sharding by `kitchenId` once a single server can't handle the write volume.
>
> Fifth, I'd make the ML service horizontally scalable with multiple containers behind its own load balancer."

---

# 8. Bonus — Tricky Follow-up Questions

### "How do you stop one kitchen from seeing another kitchen's data?"

**Answer:** Every query is filtered by `kitchenId`:
```js
Ingredient.find({ kitchenId: req.user.kitchenId })  // ← never expose other kitchens
```
The `kitchenId` comes from the authenticated user's session (Clerk token), not from the request body. Even if a malicious user sends `kitchenId: "other-kitchen"` in the body, the backend uses only the token-derived `kitchenId`.

At scale: add **row-level security** and audit logging for access attempts.

---

### "What happens if the same consumption log is submitted twice?"

**Current behavior:** A second identical submission creates a second log and decrements stock twice — wrong!

**Fix:**
1. Add an **idempotency key** field to `ConsumptionLog`
2. Frontend generates a UUID when form is shown
3. Backend checks: "Has this `idempotencyKey` been processed before?"
4. If yes: return the original response
5. If no: process and save the key alongside the log

---

### "What if Overpass API is rate-limited or down?"

**Current behavior:** Donation locator fails silently or shows an error.

**Better approach:**
```js
try {
  const ngos = await callOverpass(location);
  return res.json({ ngos, source: 'overpass' });
} catch (err) {
  // Fallback: return our own cached NGO list from MongoDB
  const cachedNgos = await Ngo.find({}).limit(10);
  return res.json({ ngos: cachedNgos, source: 'cache' });
}
```

Also: cache Overpass results in Redis for 1 hour — a given location's nearby NGOs don't change that frequently.

---

### "How do you roll out a new ML model safely?"

**Blue-Green Deployment:**
- Keep the old ML model running ("blue")
- Deploy the new model on a separate URL ("green")
- Send 5% of traffic to "green", 95% to "blue"
- Monitor: if the new model's predictions look correct, increase to 50/50 then 100%
- If it breaks: instantly switch all traffic back to "blue"

This avoids a big-bang risky release.

---

### "What if the database connection is lost in the middle of a request?"

**Answer:**
- Mongoose has automatic **connection pooling and retry** logic
- In `server.js`, implement startup retry: if MongoDB isn't ready, wait and try again
- In production: Use MongoDB Atlas with automatic failover (primary goes down → secondary is promoted automatically within seconds)

---

### "How do you handle time zones in AHAR?"

**Common mistake:** storing local time in the database.

**Correct approach:**
- Always store dates in **UTC** (Universal Time Coordinated) — the "world clock"
- Convert to local time **only at the display layer** (on the frontend)
- Why: analytics comparing "today" across kitchens in different time zones would be wrong if local times are mixed

```js
// ✅ Correct: store in UTC
date: new Date().toISOString()  // "2024-04-15T18:30:00.000Z" ← always UTC

// ❌ Wrong: store local time
date: new Date().toLocaleDateString()  // "4/16/2024" ← ambiguous, no timezone info
```

---

### "What's the biggest architectural weakness of the current AHAR design?"

**Honest answer (interviewers love honesty):**

> "The current design is well-structured for a hackathon but has some production gaps:
>
> 1. **No transactions** on the consumption log + stock decrement step — a crash between steps leaves data inconsistent
> 2. **No queue** for heavy background tasks — analytics rollup and image processing block the request
> 3. **Frontend-only auth checks** — Clerk auth should be validated on EVERY backend endpoint, not just relied on in the Frontend
> 4. **No rate limiting** — a bad actor could spam the prediction endpoint
> 5. **No monitoring** — production needs alerts when ML is down, DB is slow, or error rates spike (tools like Datadog or Grafana)
>
> I know how to fix each of these and would implement them before a production launch."

---

*Document generated for AHAR — Smart Food Waste Prediction & Kitchen Management System*  
*Stack: React + Vite | Node.js + Express | MongoDB | FastAPI (Python) | scikit-learn | Clerk | Google Maps*
