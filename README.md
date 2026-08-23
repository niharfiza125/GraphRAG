# GraphRAG — AI Copyright & Governance Knowledge Graph
 
A GraphRAG (Graph-based Retrieval-Augmented Generation) pipeline that builds a knowledge graph from AI copyright and governance sources, then answers complex, multi-part questions using community-based summarization and LLM synthesis — powered by Google's Gemini API.
 
Unlike traditional RAG, which retrieves isolated chunks of text, GraphRAG organizes information into a **knowledge graph**, groups related entities into **communities**, summarizes each community, and then synthesizes across all relevant communities to produce a single, well-reasoned answer. This makes it especially effective for broad or comparative questions (e.g. *"How are different governments approaching AI governance?"*) that a simple similarity search would struggle to answer well.
 
---
 
## How It Works
 
The pipeline runs in four main stages:
 
1. **Data Collection** — Source documents on AI copyright and governance are scraped/gathered (see `scrape_info.ipynb`).
2. **Graph Construction** — Entities and relationships are extracted from the source data and organized into a knowledge graph, stored via `graph_store.pkl` and `graph_data.json`.
3. **Community Detection & Summarization** — The graph is partitioned into communities (clusters of related entities/topics), and each community is summarized using Gemini.
4. **Query Answering** — When a question is asked:
   - Every community summary is checked against the question via an LLM call.
   - Relevant answers from each community are collected.
   - A final synthesis call combines all relevant partial answers into one coherent response.
 
This is implemented in `graphRag.ipynb`, primarily through the `query_graph()` function.
 
---
 
## Project Structure
 
```
GraphRag/
├── scrape_info.ipynb              # Scrapes/collects source data on AI copyright & governance
├── graphRag.ipynb                 # Main pipeline: graph construction, community summarization, querying
├── graph_store.pkl                # Serialized knowledge graph + community summaries
├── graph_data.json                # Graph data in JSON form
├── graph_template.html            # HTML template for graph visualization
├── ai_copyright_graph.html        # Rendered visualization of the knowledge graph
├── ai_copyright_dataset.csv       # Source dataset
├── requirements_old.txt           # Legacy dependency list
├── .env                           # API keys (not committed — see below)
└── .gitignore
```
 
---
 
## Setup
 
### 1. Clone the repository
```bash
git clone https://github.com/<your-username>/GraphRAG.git
cd GraphRAG
```
 
### 2. Create and activate a virtual environment
```bash
python -m venv myenv
myenv\Scripts\activate      # Windows
source myenv/bin/activate   # macOS/Linux
```
 
### 3. Install dependencies
```bash
pip install -r requirements.txt
```
 
### 4. Configure environment variables
Create a `.env` file in the project root with your Gemini API key:
```
GEMINI_API_KEY_4=your_api_key_here
```
Get a free API key from [Google AI Studio](https://aistudio.google.com/apikey).
 
> ⚠️ Never commit your `.env` file. It's already excluded via `.gitignore`.
 
---
 
## Usage
 
Open `graphRag.ipynb` in Jupyter or VS Code and run the cells in order:
 
1. **Load environment & configure Gemini**
2. **Build or load the knowledge graph** (`graph_store`)
3. **Generate community summaries** (if not already built)
4. **Query the graph:**
 
```python
response = query_graph(
    "How are different governments (e.g. EU, US, and UK) approaching AI governance?"
)
print(response)
```
 
---
 
## Rate Limits (Free Tier)
 
This project uses `gemini-3.5-flash-lite` on the **free tier**, which is capped at **15 requests per minute** per project. Because `query_graph()` makes one LLM call per community, larger graphs can take several minutes per query.
 
To stay within the limit, the pipeline:
- Adds a delay between community-level calls (`time.sleep(4.5)`)
- Retries automatically on `429` rate-limit errors with a backoff wait
 
**If you need faster queries:** enable billing on your Google Cloud project to raise the rate limit, or reduce the number of communities queried per question.
 
---
 
## Tech Stack
 
- **Python 3.11**
- **Google Generative AI (`google-generativeai`)** — Gemini API for summarization and synthesis
- **python-dotenv** — environment variable management
- **Jupyter Notebook** — development and experimentation environment
 
---
 
## Notes
 
- `graph_store.pkl` and `graph_data.json` contain the pre-built graph and community summaries. Regenerate them by re-running the graph construction cells if the source data changes.
- The project is structured for experimentation; for production use, consider moving `query_graph()` and related logic into a standalone `.py` module with proper logging and configurable rate limits.
 
---
 
## License
 
*(Add your license here — e.g. MIT, Apache 2.0 — or state "All rights reserved" if unpublished.)*
 
