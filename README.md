# Multi-Modal Image & Audio AI Assistant (Notebook)

This repository contains a Gradio-powered multi-modal AI assistant implemented in the
`multi-modal_image_audio.ipynb` notebook. The assistant integrates chat, tool calls
(SQLite ticket-price lookup), image generation (Diffusers), and text-to-speech (Kokoro).

**Quick links**
- Notebook: [multi-modal_image_audio.ipynb](multi-modal_image_audio.ipynb)

**Overview**
- **Purpose:** Demonstrates a multi-modal assistant that:
  - Answers user queries with an LLM (via Ollama/OpenAI-compatible client),
  - Calls a local SQLite tool to fetch ticket prices,
  - Generates images for destinations using a text-to-image pipeline,
  - Produces spoken replies using an open-source TTS pipeline.

**Requirements**
- **Python:** 3.10+ recommended.
- **Core packages:** `requests`, `python-dotenv`, `openai` (or OpenAI-compatible client used here),
  `gradio`, `diffusers`, `Pillow`, `soundfile`, `kokoro`, `sqlite3` (stdlib).
- **Models / Services:**
  - Ollama or Ollama-compatible local server running at `http://localhost:11434` (the notebook sets `OLLAMA_BASE_URL`).
  - A text-to-image model (the notebook loads `stabilityai/sd-turbo` by default).

**Setup**
1. Create a virtual environment and install dependencies, e.g.:

```bash
python -m venv .venv
source .venv/Scripts/activate    # Windows: .venv\\Scripts/activate
pip install -r requirements.txt  # or install packages manually
```

2. Ensure Ollama (or your OpenAI-compatible endpoint) is running locally. The notebook uses:

```python
OLLAMA_BASE_URL = "http://localhost:11434/v1"
ollama = OpenAI(base_url=OLLAMA_BASE_URL, api_key='ollama')
```

3. Prepare the SQLite database `prices.db` in the repo root with a simple table:

```sql
CREATE TABLE IF NOT EXISTS prices (city TEXT PRIMARY KEY, price REAL);
INSERT OR REPLACE INTO prices (city, price) VALUES ('paris', 399.0);
```

4. If you plan to run image generation or TTS locally, ensure the required models are available and you have sufficient hardware (GPU recommended). Running SDXL or other large models on CPU is extremely slow.

**Running the Notebook**
- Open the notebook in Jupyter or VS Code and run cells sequentially. The Gradio UI is launched at the end of the notebook with basic HTTP auth:

```python
ui.launch(inbrowser=True, auth=("jaya", "shree"))
```

**Notable implementation details**
- Chat flow: the notebook builds `messages` including a `system_message` and passes them to the local Ollama client. Tool calls are supported via a `tools` definition and a `handle_tool_calls` function that queries the SQLite DB.
- Image generation: the `artist(city)` function uses a `AutoPipelineForText2Image` pipeline (SD Turbo by default).
- TTS: the `talker()` function uses Kokoro and writes `speech.wav` which Gradio plays.

**Security & Notes**
- The Gradio app in this notebook uses simple HTTP auth — do not expose it publicly without proper safeguards.
- Large image models are GPU-optimized; expect very slow CPU performance. Prefer lighter models (e.g., `sd-turbo`) or a GPU/Colab runtime.

**Troubleshooting**
- If the notebook can't reach Ollama, confirm the service is running and reachable at `http://localhost:11434`.
- If image generation runs out of memory on CPU, switch to a smaller model or use a GPU environment.

**Next steps (suggested)**
- Add a `requirements.txt` or update `pyproject.toml` with runtime deps.
- Add a small script to seed `prices.db` for demo purposes.

If you want, I can:
- Generate `requirements.txt` from the notebook imports, or
- Create a small `seed_db.py` to populate `prices.db` with example data.
