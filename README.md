# AI Voice Customer Support

Live app: https://ai-voice-customer-support.streamlit.app/

A Streamlit app that answers questions about any documentation website, in text **and** in a spoken voice. Give it a documentation URL, it reads the pages, and then you ask questions in plain English. It replies with a written answer, the source pages it used, and an MP3 you can play or download.

## What it does

1. Crawls the documentation site you point it at.
2. Stores the pages in a vector database so they can be searched by meaning, not just keywords.
3. Finds the pages most relevant to your question.
4. Has an AI model write a short, easy-to-say answer from those pages.
5. Turns the answer into speech in the voice you chose.

## How it works

```
Documentation URL
      |
      v
Firecrawl (crawl, max 5 pages) --> FastEmbed (embeddings) --> Qdrant (vector store)
                                                                   |
Your question --> FastEmbed --> search top 3 pages ----------------+
                                       |
                                       v
                      Documentation Processor agent (GPT-4o): writes the answer
                                       |
                                       v
                      Text-to-Speech agent (GPT-4o): writes delivery instructions
                                       |
                                       v
                      OpenAI gpt-4o-mini-tts --> MP3 + text answer + sources
```

| Step | Tool | Details |
|---|---|---|
| Crawl | Firecrawl | Reads up to 5 pages per site (set by `limit` in `crawl_documentation`) and keeps the Markdown, falling back to HTML |
| Embed | FastEmbed | Default `TextEmbedding` model; each page is stored as one vector, with its URL, title, description, language and crawl date |
| Store and search | Qdrant | Collection `docs_embeddings`, cosine similarity, top 3 matches per question |
| Answer | OpenAI Agents SDK, `gpt-4o` | The "Documentation Processor" agent answers from the retrieved pages and cites their URLs |
| Speech | `gpt-4o-mini-tts` | The second agent writes speaking instructions (pacing, tone), and the spoken MP3 is made in your chosen voice |

## What you need

Three outside services, each with its own API key. All of them can charge for usage.

| Service | Used for | Where to get it |
|---|---|---|
| OpenAI | Answers and text-to-speech | OpenAI Platform |
| Qdrant Cloud | Vector database (needs the cluster **URL** and an API key) | Qdrant Cloud |
| Firecrawl | Crawling the documentation | Firecrawl |

## Run it yourself

```bash
git clone https://github.com/Guru1315/AI-Voice-Customer-Support.git
cd AI-Voice-Customer-Support
pip install -r "requirements .txt"
streamlit run customer_support_voice_agent.py
```

The requirements file has a space before `.txt`, so keep the quotes. Python 3.11 is what the dev container uses.

## How to use it

1. Open the sidebar and enter your **Qdrant URL**, **Qdrant API Key**, **Firecrawl API Key** and **OpenAI API Key**.
2. Enter the **Documentation URL** you want to ask about, for example `https://docs.example.com`.
3. Pick a voice. The choices are `alloy`, `ash`, `ballad`, `coral` (default), `echo`, `fable`, `onyx`, `nova`, `sage`, `shimmer` and `verse`.
4. Click **Initialize System**. It connects to Qdrant, crawls the pages, stores them and starts the agents. All five fields are required.
5. Type a question, for example *How do I authenticate API requests?* You get the text answer, an audio player with a download button, and a list of the source URLs.

The question box stays disabled until initialization has finished.

## Project structure

```
customer_support_voice_agent.py   The whole app (UI, crawling, embeddings, search, agents, speech)
requirements .txt                 Python dependencies
.devcontainer/devcontainer.json   Python 3.11 dev container / Codespaces setup, serves on port 8501
README.md                         This file
```

## Notes and limits

- **Keys stay in the session.** Keys typed into the sidebar are held in Streamlit's session state and are not written to disk. The app calls `load_dotenv()`, but the sidebar fields are not filled from a `.env` file, so you still have to type them in.
- **Only 5 pages.** The crawl is capped at 5 pages, so answers only cover those pages. Raise the `limit` value in `crawl_documentation` to read more.
- **No chunking.** Each page is stored as a single vector, so long pages are represented coarsely.
- **Re-initializing adds duplicates.** Every click on **Initialize System** stores the pages again in the same `docs_embeddings` collection with new ids, so the same page can appear several times. Delete the collection in Qdrant to start clean.
- **Answers are limited to the top 3 pages found.** If nothing is found, you get an error instead of an answer.
- **Dependencies are not pinned.** If the Firecrawl or OpenAI Agents calls start failing after a library update, check the installed versions first.
- **Dev container.** `.devcontainer/devcontainer.json` installs from `requirements.txt`, but the file in this repository is named `requirements .txt`, so only Streamlit is installed automatically in Codespaces. Run the `pip install` command above there, or rename the file.
