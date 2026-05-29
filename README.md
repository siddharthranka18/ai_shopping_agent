# 🛒 AI Shopping Assistant

An intelligent, conversational shopping agent powered by **Groq LLMs** and **LangChain**. It lets users search products via text or image, check ratings, and place orders — all through a natural language chat interface built with Streamlit.

---

## Features

- 🔍 **Product Search** — keyword search with optional price cap and organic filter
- ⭐ **Rating Lookup** — fetches average star rating and review count per product
- 🧾 **One-step Checkout** — places and records orders in a local SQLite database
- 🖼️ **Image-based Search** — upload a product photo; a vision LLM identifies it and finds matches
- 💬 **Conversational UI** — Streamlit chat interface with sidebar image uploader

---

## Project Structure

```
shopping_agent/
├── app.py              # Streamlit web UI
├── shopping_agent.py   # LangChain agent with tools (search, rate, checkout, vision)
├── reviws_api.py       # SQLite helper – reads aggregated ratings from reviews table
└── store.db            # SQLite database (products, reviews, orders tables)
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| LLM (reasoning) | `qwen/qwen3-32b` via Groq |
| LLM (vision) | `meta-llama/llama-4-scout-17b-16e-instruct` via Groq |
| Agent framework | LangChain (`create_agent`) |
| UI | Streamlit |
| Database | SQLite 3 |

---

## Setup

### 1. Prerequisites

- Python 3.11+
- A [Groq API key](https://console.groq.com/)

### 2. Install dependencies

From the `shopping_agent/` directory (or the repo root if a shared `requirements.txt` is used):

```bash
pip install streamlit langchain langchain-groq python-dotenv
```

### 3. Configure environment

Create a `.env` file inside `shopping_agent/` (or the project root):

```env
GROQ_API_KEY=your_groq_api_key_here
```

### 4. Run the app

```bash
streamlit run app.py
```

The app opens at `http://localhost:8501` by default.

---

## How It Works

### Agent Tools

| Tool | Description |
|---|---|
| `search_products` | SQL query on `products` table — filters by keyword, price, organic flag |
| `get_rating` | Reads `reviews` table via `reviws_api.py` — returns avg rating + count |
| `checkout` | Inserts a row into `orders` table — returns a confirmation message |
| `describe_product_image` | Base64-encodes uploaded image → vision LLM → returns JSON attributes |

### Conversation Flow

```
User text query
  └─► search_products ─► get_rating (per result) ─► present numbered list
         └─► user confirms ─► checkout ─► order confirmation

User uploads image
  └─► describe_product_image ─► search_products ─► (same as above)
```

The agent **never places an order without explicit user confirmation** and always resolves product IDs from its own prior message.

---

## Running in Script Mode

`shopping_agent.py` can be run directly for a quick CLI test:

```bash
python shopping_agent.py
```

This invokes the agent with a hardcoded sample query (`"I want to buy milk with price less than $4"`) and prints the response.

---

## Database Schema (store.db)

| Table | Key Columns |
|---|---|
| `products` | `id`, `name`, `category`, `price`, `description`, `is_organic` |
| `reviews` | `product_id`, `rating` |
| `orders` | `id`, `product_id`, `product_name`, `price` |
