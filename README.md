The reason it “doesn’t convert” on GitHub is almost always one of these:

1) a code block was started with ``` but **never closed**, so everything after it stays “code”  
2) missing a blank line before a fenced code block  
3) pasted content includes stray backticks/quotes

In your snippet, the code fence under **Run Locally** is missing the closing triple backticks.

Below is a **clean, fully valid README.md** you can paste directly into GitHub (it will render correctly). Replace the demo link placeholder.

```markdown
# Autonomous Data Science Agent (Day 100 Capstone)

An end-to-end **Streamlit** app that turns any uploaded CSV into an “autonomous data scientist”.

Upload a dataset → ask questions in natural language → the agent writes Python code → executes it in a **sandbox (E2B)** → auto-fixes errors (up to 3 retries) → returns results + charts with transparent logs.

**Demo video:** https://<your-demo-link>

---

## What this project does

- **Chat-driven analysis:** ask questions like “show missing values”, “plot correlations”, “train a baseline model”.
- **Autonomous code generation:** Gemini generates runnable Python for EDA, visualization, and modeling.
- **Sandboxed execution (E2B):** code runs in an isolated environment (safer than running on the Streamlit host).
- **Self-correction loop:** on runtime errors, the agent reads stderr, fixes the code, and retries automatically (max 3).
- **Charts included:** Matplotlib/Seaborn plots are captured and rendered in Streamlit.
- **Transparency:** executed code + stdout/stderr are shown in an expander.

---

## Tech Stack

- UI: **Streamlit**
- LLM: **Google Gemini** via `langchain_google_genai`
- Execution: **E2B Sandbox** (`e2b-code-interpreter`)
- Data & Viz: **Pandas**, **Matplotlib**, **Seaborn**

---

## Architecture (high level)

1. User uploads a CSV in Streamlit  
2. File is written into the **E2B sandbox** (`/home/user/data/...`)  
3. User asks a question in chat  
4. Gemini generates a single Python code block  
5. Code executes in E2B → returns logs + charts  
6. If error occurs → send error back to Gemini → retry (max 3)  
7. Render final answer + charts + executed code  

---

## Setup

### Prerequisites
You need API keys:
- `GOOGLE_API_KEY` (Google AI Studio / Gemini)
- `E2B_API_KEY` (E2B)

> Important: Never commit keys to GitHub. Add secrets files to `.gitignore`.

---

## Run Locally

### 1) Install dependencies
```bash
pip install -U pip
pip install -r requirements.txt
```

### 2) Add Streamlit secrets
Create a file at `.streamlit/secrets.toml`:
```toml
GOOGLE_API_KEY="YOUR_KEY"
E2B_API_KEY="YOUR_KEY"
```

### 3) Run the app
```bash
streamlit run app.py
```

---

## Run in Google Colab (optional)

Colab often has pinned system packages. Use an isolated environment.

### 1) Create venv + install deps
```bash
python3 -m pip install -U virtualenv
python3 -m virtualenv /content/ads_venv

/content/ads_venv/bin/python -m pip install -U pip setuptools wheel
/content/ads_venv/bin/python -m pip install -U \
  streamlit \
  langchain langchain-core langchain-google-genai google-generativeai \
  "e2b-code-interpreter==2.4.1" \
  pandas matplotlib seaborn \
  "pillow<12"
```

### 2) Add secrets in Colab
```bash
mkdir -p /content/.streamlit
cat > /content/.streamlit/secrets.toml <<'EOF'
GOOGLE_API_KEY="YOUR_KEY"
E2B_API_KEY="YOUR_KEY"
EOF
```

### 3) Run Streamlit
```bash
nohup /content/ads_venv/bin/python -m streamlit run /content/app.py \
  --server.address 0.0.0.0 --server.port 8501 \
  > /content/streamlit.log 2>&1 &
tail -n 50 /content/streamlit.log
```

### 4) (Optional) Expose via Cloudflare quick tunnel
```bash
wget -q https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64 \
  -O /usr/local/bin/cloudflared
chmod +x /usr/local/bin/cloudflared

/usr/local/bin/cloudflared tunnel --url http://localhost:8501 --no-autoupdate --loglevel info
```

---

## Example Demo Prompts

- “Show shape, dtypes, sample rows, missing values summary, and duplicates.”
- “Plot correlations and summarize the strongest relationships.”
- “Build a baseline model and report metrics + confusion matrix.”
- “Explain the results in plain English with supporting charts.”

---

## Project Structure

```text
.
├── app.py
├── requirements.txt
└── .streamlit/
    └── secrets.toml   # not committed
```

---

## Notes / Troubleshooting

- If Gemini model returns `404 NOT_FOUND`, switch to a model your key supports (e.g. `models/gemini-flash-latest`).
- E2B `Sandbox()` may fail on some SDK versions; this project provisions with `Sandbox.create()` for `e2b-code-interpreter==2.4.1`.
- Cloudflare quick tunnel URLs expire when the process stops—restart cloudflared to get a new link.

---

## License

Add a `LICENSE` file (MIT recommended for portfolios).
```

If you paste what you currently have in GitHub’s editor (especially around the first broken code block), I can point out the exact missing/extra backticks causing the render issue.
