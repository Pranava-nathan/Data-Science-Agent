```markdown
# Autonomous Data Science Agent (Day 100 Capstone)

An end-to-end **Streamlit** app that turns any uploaded CSV into an “autonomous data scientist”.

Upload a dataset → ask questions in natural language → the agent writes Python code → executes it in a **sandbox (E2B)** → auto-fixes errors (up to 3 retries) → returns results + charts with transparent logs.

**Demo video:** *(paste your demo link here — YouTube/Drive/LinkedIn post)*

---

## What this project does

- **Chat-driven analysis:** ask questions like “show missing values”, “plot correlations”, “train a baseline model”.
- **Autonomous code generation:** Gemini generates runnable Python for EDA, visualization, and modeling.
- **Sandboxed execution (E2B):** code runs in an isolated environment (safer than running on the Streamlit host).
- **Self-correction loop:** on runtime errors, the agent reads stderr, fixes the code, and retries automatically.
- **Charts included:** Matplotlib/Seaborn plots are captured and rendered in Streamlit.
- **Transparency:** executed code + stdout/stderr are shown in an expander.

---

## Tech Stack

- UI: **Streamlit**
- LLM: **Google Gemini** via `langchain_google_genai`
- Execution: **E2B Code Interpreter / Sandbox** (`e2b-code-interpreter`)
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
Create:
```bash
mkdir -p .streamlit
```

Create `.streamlit/secrets.toml`:
```toml
GOOGLE_API_KEY="YOUR_KEY"
E2B_API_KEY="YOUR_KEY"
```

### 3) Start the app
```bash
streamlit run app.py
```

---

## Run in Google Colab (recommended for demo)

Colab often has pinned system packages. Use an isolated environment.

### 1) Create isolated env + install
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

### 4) Expose the app (Cloudflare quick tunnel)
```bash
wget -q https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64 \
  -O /usr/local/bin/cloudflared
chmod +x /usr/local/bin/cloudflared

/usr/local/bin/cloudflared tunnel --url http://localhost:8501 --no-autoupdate --loglevel info
```

> Keep the cloudflared process running during your demo. Quick tunnel URLs expire when the process stops.

---

## Supported Gemini Models

Model availability depends on your API key. This project uses model IDs like:

- `models/gemini-flash-latest` (recommended default for demos)
- `models/gemini-pro-latest`
- `models/gemini-2.5-flash`
- `models/gemini-2.5-pro`

If you get `404 NOT_FOUND`, choose a model that your key supports.

---

## Example Demo Prompts (Titanic CSV)

- “Can you show the dataset shape, dtypes, sample rows, and missing values summary?”
- “What’s the survival rate overall and by Sex and Pclass? Plot both.”
- “Clean missing Age/Embarked and turn Cabin into a HasCabin feature. Show before/after missing values.”
- “Build a baseline Logistic Regression model with preprocessing. Report ROC-AUC and plot confusion matrix + ROC curve.”
- “Try plotting survival rate by the ‘Embark’ column.” *(intentional error to show self-fix)*

---

## Troubleshooting

### E2B sandbox init error (common)
If you see an error like:
`SandboxBase.__init__() missing positional arguments ...`

You must provision with:
`Sandbox.create()`  
Direct `Sandbox()` construction fails with `e2b-code-interpreter==2.4.1`.

### Gemini returns content as a list
Gemini can return “parts” as a list. The app normalizes this into a string before parsing code blocks.

### Cloudflare URL “DNS address could not be found”
The tunnel likely stopped. Restart cloudflared and use the newly printed URL.

---

## Project Structure

```
.
├── app.py
├── requirements.txt
└── .streamlit/
    └── secrets.toml   # not committed
```

---

## License
Choose a license (MIT recommended for portfolios) and add a `LICENSE` file.
```

If you want, paste your current `requirements.txt` (or `pip freeze`) and I’ll generate a clean, minimal `requirements.txt` aligned with your working Colab/venv versions.
