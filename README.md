# 🎯 ATS Resume Scorer

A FastAPI and Streamlit web application that scores how well a resume matches a job description and provides actionable feedback. It uses spaCy and Sentence Transformers for NLP and the Groq API for LLM-generated suggestions.

---

## Features
- **Resume Parsing:** Supports PDF, DOC, and DOCX files.
- **ATS Scoring:** Evaluates matching based on skills, keyword density, content/grammar, formatting, and ATS compatibility.
- **Semantic Matching:** Compares resume content with the job description using Sentence Transformers (`all-MiniLM-L6-v2`).
- **AI Suggestions:** Generates recommendations using Groq (`llama-3.3-70b-versatile`).
- **History & Auth:** Tracks user analyses using Supabase Auth & Database.
- **PDF Export:** Exports report results as PDF using WeasyPrint.

---

## Tech Stack
- **Frontend:** Streamlit
- **Backend:** FastAPI (Python)
- **NLP:** spaCy (`en_core_web_md`), Sentence Transformers (`all-MiniLM-L6-v2`)
- **LLM:** Groq API (`llama-3.3-70b-versatile`)
- **Database & Authentication:** Supabase
- **PDF Export:** WeasyPrint & Jinja2 Templates

---

## Project Structure
```
ATS_SCORER/
├── backend/              # FastAPI app, NLP engines, and database services
├── frontend/             # Streamlit pages, views, and auth components
├── jupyter notebooks/    # Research and dataset preparation
└── requirements.txt      # Python dependencies
```

---

## Setup & Installation

### 1. Prerequisites (WeasyPrint Dependencies)
- **macOS**: `brew install cairo pango gdk-pixbuf libffi`
- **Ubuntu/Debian**: `sudo apt install -y libcairo2 libpango-1.0-0 libpangoft2-1.0-0 libffi-dev`
- **Fedora**: `sudo dnf install -y cairo pango gdk-pixbuf2 libffi`

### 2. Install Packages
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
python -m spacy download en_core_web_md
```

### 3. Environment Config

Create a `.env` in the root:
```env
GROQ_API_KEY=your_groq_key
SUPABASE_URL=your_supabase_url
SUPABASE_KEY=your_service_role_key
SUPABASE_ANON_KEY=your_anon_key
SUPABASE_JWT_SECRET=your_jwt_secret
```

Create `frontend/.streamlit/secrets.toml`:
```toml
[supabase]
url = "your_supabase_url"
anon_key = "your_anon_key"

[backend]
url = "http://localhost:8000"
```

### 4. Database Setup (Supabase)
Run the following SQL in your Supabase SQL Editor:
```sql
create table public.analyses (
    id uuid default gen_random_uuid() primary key,
    user_id uuid not null references auth.users(id) on delete cascade,
    filename text not null,
    ats_score numeric not null,
    keyword_match numeric not null,
    missing_keywords jsonb not null default '[]'::jsonb,
    created_at timestamp with time zone default timezone('utc'::text, now()) not null,
    analysis_result jsonb not null
);

-- Enable RLS and add basic policies
alter table public.analyses enable row level security;
create policy "Users can manage their own analyses" on public.analyses 
    for all using (auth.uid() = user_id);
```

---

## Running the App

1. **Start Backend (FastAPI):**
   ```bash
   uvicorn backend.main:app --reload --host 0.0.0.0 --port 8000
   ```
2. **Start Frontend (Streamlit):**
   ```bash
   streamlit run frontend/streamlit_app.py
   ```
