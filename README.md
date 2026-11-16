# Northside Dental RAG CLI

A beginner-friendly Retrieval-Augmented Generation (RAG) demo. Ask questions in plain English; the application searches a local Northside Dental knowledge base, sends the most relevant passages to Google Gemini, and returns a structured JSON answer with source citations.

> **Fictional demonstration only.** Northside Dental Clinic and all clinic details in this repository are invented. This project is not a real clinic service, not medical or dental advice, and is not approved for use with real patients. Do not enter patient names, contact details, health records, or other personal information.

## What it does

1. Loads a local JSON knowledge base (31 prepared topic chunks).
2. Finds relevant passages with a local BM25 keyword retriever.
3. Sends only the selected passages and the question to Gemini.
4. Requests a typed JSON response from Gemini.
5. Validates the answer and attaches citations from the local knowledge base (the model cannot invent source-page metadata).
6. Returns an answer, source IDs, section/page details, a retrieval relevance score, and an `answerable` flag.

The CLI is the main interface. A small FastAPI endpoint is also available as an optional secondary interface. No GUI or hosted backend is required.

## Requirements

- Windows 10/11, macOS, or Linux.
- Python 3.11 or newer. Python 3.12 or 3.13 is recommended for a broadly supported setup.
- An internet connection for Gemini requests.
- A Google Gemini API key. You can create one in [Google AI Studio](https://aistudio.google.com/app/apikey).

The default model is `gemini-3.5-flash-lite`. Google's model documentation lists structured output support for this model. Free-tier access and rate limits depend on Google's current availability, your project, region, and account. Check the [model page](https://ai.google.dev/gemini-api/docs/models/gemini-3.5-flash-lite), [pricing/free-tier information](https://ai.google.dev/gemini-api/docs/pricing), and [rate limits](https://ai.google.dev/gemini-api/docs/rate-limits) before use. Free access is not an unlimited-use guarantee.

## Quick start — Windows PowerShell

These instructions deliberately use the virtual environment's Python executable directly. **You do not need to activate the virtual environment**, so PowerShell's script-execution policy will not block setup.

### 1. Install Python and Git

Install Python 3.11+ from [python.org](https://www.python.org/downloads/) and Git from [git-scm.com](https://git-scm.com/downloads). During Python installation, enable the Python Launcher if the installer offers that option.

Open a new PowerShell window and check:

```powershell
py --version
git --version
```

### 2. Download the project

Run:

```powershell
git clone https://github.com/rahil565665-afk/RAG-CLI.git
cd RAG-CLI
```

After changing directory, verify that the project files are present:

```powershell
Get-ChildItem
```

You should see `requirements.txt`, `.env.example`, the `app` folder, and the `data` folder.

### 3. Create a virtual environment

```powershell
py -3 -m venv .venv
```

This creates an isolated Python environment in the project's `.venv` folder.

### 4. Install the dependencies

Run these commands exactly:

```powershell
.\.venv\Scripts\python.exe -m pip install --upgrade pip
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

Wait until pip reports that the packages were installed successfully. This project intentionally uses plain `uvicorn`, not `uvicorn[standard]`, to avoid optional native build dependencies that can cause installation failures on some Python/Windows combinations.

### 5. Create your local environment file

```powershell
Copy-Item .env.example .env
notepad .env
```

In Notepad, replace the placeholder with your own Gemini API key:

```dotenv
GEMINI_API_KEY=PASTE_YOUR_OWN_KEY_HERE
GEMINI_MODEL=gemini-3.5-flash-lite
TOP_K=4
MIN_RELEVANCE=0.22
RATE_LIMIT_PER_MINUTE=30
```

Save the file and close Notepad.

**Keep your key private.** The real `.env` file is ignored by Git. Never paste your key into source code, README files, screenshots, issue reports, or chat messages. Do not commit `.env`. If a key is exposed, revoke it in Google AI Studio and create a replacement.

### 6. Run your first question

```powershell
.\.venv\Scripts\python.exe -m app.cli "What are the clinic hours?"
```

A successful response is JSON shaped like this (wording and scores vary):

```json
{
  "answer": "The clinic is open ...",
  "sources": [
    {
      "chunk_id": "NSDC-01-003",
      "section_id": "01",
      "section_title": "Clinic profile",
      "subsection": "Opening hours and holiday/weather closures",
      "source_document": "Northside_Dental_Clinic_Knowledge_Base.pdf",
      "source_pages": [2],
      "relevance_score": 0.46
    }
  ],
  "confidence_score": 0.46,
  "confidence_basis": "Retrieval-based relevance heuristic (BM25-normalized score); not a calibrated probability.",
  "answerable": true
}
```

The exact values above are illustrative. The response is generated from the fictional local data.

## Asking questions

### One question at a time

```powershell
.\.venv\Scripts\python.exe -m app.cli "What is the clinic name?"
.\.venv\Scripts\python.exe -m app.cli "What does the clinic do?"
.\.venv\Scripts\python.exe -m app.cli "Does the clinic accept insurance?"
```

Always run a question through `python -m app.cli`. Do not type plain-English questions directly at the PowerShell prompt; PowerShell will interpret them as commands.

### Interactive mode

Start the CLI without a question:

```powershell
.\.venv\Scripts\python.exe -m app.cli
```

At the `Question>` prompt, type a question and press Enter. Type `exit` or `quit` to leave.

### Useful options

```powershell
# Compact, one-line JSON
.\.venv\Scripts\python.exe -m app.cli --compact "What are the clinic hours?"

# Retrieve up to five knowledge-base passages
.\.venv\Scripts\python.exe -m app.cli --top-k 5 "What services are available?"

# See command-line help
.\.venv\Scripts\python.exe -m app.cli --help
```

The number of chunks can be set from 1 to 5. The `TOP_K` and `MIN_RELEVANCE` defaults can also be changed in your local `.env` file.

## Run automated tests

Tests do not call Gemini and do not require an API key. They use local test data and mocked model responses.

```powershell
.\.venv\Scripts\python.exe -m pytest -q
```

The test suite covers retrieval, knowledge-base integrity, abstention, emergency phrase handling, citation filtering, the API endpoint, and the Gemini structured-output schema.

## Optional: run the HTTP API

The command-line interface is the primary way to use this project. To start the optional local FastAPI server:

```powershell
.\.venv\Scripts\python.exe -m uvicorn app.main:app --reload
```

Then open:

- API documentation: http://127.0.0.1:8000/docs
- Health endpoint: http://127.0.0.1:8000/health

This server is local to your computer by default. It is not deployed to the internet.

## macOS and Linux

From Terminal, install Python 3.11+ and Git, then run:

```bash
git clone https://github.com/rahil565665-afk/RAG-CLI.git
cd RAG-CLI
python3 -m venv .venv
./.venv/bin/python -m pip install --upgrade pip
./.venv/bin/python -m pip install -r requirements.txt
cp .env.example .env
```

Open `.env` in a text editor, add your own Gemini API key, and save it. Then run:

```bash
./.venv/bin/python -m app.cli "What are the clinic hours?"
./.venv/bin/python -m pytest -q
```

## Project structure

```text
RAG-CLI/
├── app/
│   ├── cli.py              # Command-line interface
│   ├── config.py           # Reads local .env settings
│   ├── gemini_client.py    # Gemini API and structured JSON generation
│   ├── main.py             # Optional FastAPI interface
│   ├── rag_service.py      # Orchestrates retrieval, generation and citations
│   ├── retrieval.py        # Local BM25 retrieval
│   └── schemas.py          # Pydantic response schemas
├── data/
│   ├── northside_dental_knowledge_base.json
│   └── northside_dental_knowledge_base_section_index.md
├── tests/                  # Offline automated tests
├── .env.example            # Safe configuration template (no real key)
├── .gitignore              # Excludes .env and local environment files
├── requirements.txt
└── README.md
```

## Configuration

| Setting | Default | Purpose |
|---|---:|---|
| `GEMINI_API_KEY` | required | Your private Google Gemini API key |
| `GEMINI_MODEL` | `gemini-3.5-flash-lite` | Gemini model used to generate answers |
| `TOP_K` | `4` | Maximum number of retrieved passages (1–5) |
| `MIN_RELEVANCE` | `0.22` | Minimum retrieval relevance required before calling Gemini |
| `RATE_LIMIT_PER_MINUTE` | `30` | Local application request limit |

The app loads `.env` from the project root. Environment variables already set in the shell take precedence over `.env`.

## Troubleshooting

### PowerShell says running scripts is disabled

You do not need to activate the virtual environment. Use the explicit interpreter path in the commands above, for example:

```powershell
.\.venv\Scripts\python.exe -m app.cli --help
```

This avoids running `Activate.ps1` and avoids changing your machine's execution policy.

### `requirements.txt` cannot be found

Make sure the PowerShell working directory is the repository root (the folder named `RAG-CLI`), not its parent folder:

```powershell
Get-Location
Get-ChildItem requirements.txt
```

### API key is missing

- Confirm the file is named exactly `.env`, not `.env.txt`.
- Confirm `.env` is in the same folder as `requirements.txt`.
- Confirm the line starts with `GEMINI_API_KEY=` and contains your own key.
- Do not include angle brackets or the words `PASTE_YOUR_OWN_KEY_HERE`.

### Gemini returns 400 with `additional_properties`

The `GeminiDraft` schema in `app/schemas.py` must keep `ConfigDict(extra="ignore")`. Gemini's response-schema endpoint does not accept the extra `additionalProperties` field generated by a strict Pydantic schema. Do not change that setting back to `extra="forbid"`.

### Gemini returns 401 or 403

Check that the key is valid, the corresponding Google AI Studio project can use the Gemini API, and the selected model is enabled for the project. If the key was exposed, revoke it and create a new one.

### Gemini returns 404 or says the model is unavailable

Check the current [Gemini model list](https://ai.google.dev/gemini-api/docs/models) and confirm the model ID is available to your project. You can change `GEMINI_MODEL` in `.env`. Model availability can change over time and may vary by account or region.

### Gemini returns 429 / quota exceeded

This means a rate or usage limit was reached. Check the project's current [Gemini API rate limits](https://ai.google.dev/gemini-api/docs/rate-limits) and usage in Google AI Studio. Wait for the relevant quota window to reset or use a project/model with available quota. Do not assume free tier means unlimited requests.

### The answer says the knowledge base does not contain enough information

The local retriever uses BM25 keyword matching, not semantic embeddings. Try a more specific question using terms likely to appear in the clinic document, or try `--top-k 5`. A very low relevance threshold can introduce unrelated passages, so change `MIN_RELEVANCE` carefully.

### The model responds but the CLI reports a provider error

Check the model ID, network connection, key, and quota. The app intentionally hides low-level provider errors in normal CLI output so it does not accidentally expose request details or credentials. Run the automated tests to verify the local installation separately:

```powershell
.\.venv\Scripts\python.exe -m pytest -q
```

## How confidence and citations work

- `sources` contains only passages retrieved from the local knowledge base and cited by the model.
- Each source includes the chunk ID, section code/title, subsection, source document, source page(s), and retrieval relevance.
- `confidence_score` is the highest relevance score among the valid cited passages.
- This score is a BM25-normalized retrieval heuristic, **not** a probability that the answer is correct, not clinical certainty, and not a medical risk score.
- If relevant evidence is not retrieved, the app abstains rather than asking Gemini to invent an answer.

## Safety and limitations

- All Northside Dental content is fictional demonstration data.
- Do not use this project to answer real patient questions or make treatment decisions.
- Do not enter or upload real patient information.
- The app does not diagnose, prescribe, interpret X-rays, or provide individualized clinical advice.
- Its emergency phrase check is a narrow deterministic backstop, not a diagnostic tool and not a substitute for local emergency services.
- BM25 is lexical retrieval; it can miss synonyms and paraphrases.
- No appointment booking, live insurance verification, or patient-record integration is included.
- Google Gemini API usage is subject to Google's terms, privacy practices, model availability, and rate limits.

## License

No license has been added to this repository. Unless a license is added by the repository owner, assume the code is not granted for redistribution or reuse beyond the rights allowed by applicable law.
