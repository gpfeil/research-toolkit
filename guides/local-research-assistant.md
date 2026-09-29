<!-- DRAFT drafted with Claude, pending Gretchen's rewrite -->

# Local Research Assistant: Setup Guide

A private, local research assistant for a Mac: a chat window that runs a language model entirely on your own computer, searches the web, reads documents you give it, and is set up from the start to cite where every answer came from.

Written for researchers who want AI help with open-source desk research without sending their working notes to a cloud service. No programming background needed; one step uses Terminal. Last checked September 19, 2026.

## What you're building

Three parts, each a normal app or folder you can remove at any time:

- **LM Studio**, a free Mac app that runs the AI model and provides the chat window.
- **A model**, the "brain" you download once (about 5 to 8 GB).
- **Connectors**, small add-ons (called MCP servers) that let the model use a source such as web search. Adding a new source later means adding one more connector.

Plan on 45 to 60 minutes for the first setup, mostly waiting for downloads. You'll need about 15 GB of free disk space and an internet connection for setup. Nothing here changes your other apps, and quitting LM Studio gives you all your computer's memory back.

One honest expectation: a model this size is a capable research assistant, not an oracle. It reads and summarizes well, but it can be wrong, so this guide builds in the habit of making it cite where each answer came from.

## Step 1: Install LM Studio

1. Go to [lmstudio.ai](https://lmstudio.ai) and download the Mac version.
2. Open the downloaded file and drag LM Studio into your Applications folder.
3. Open LM Studio. If macOS asks whether you're sure you want to open it, choose Open.
4. If it offers a setup walkthrough, follow it. If it asks you to pick a mode, choose the simplest or "User" option; you can change it later.

You'll see a sidebar with icons for chat, model search (a magnifying glass), and settings.

## Step 2: Choose and download a model

These recommendations assume a Mac with 16 GB of memory, which leaves roughly 11 GB usable for a model after macOS takes its share. That makes 8 to 9 billion parameters the sweet spot, with some room for 12B. Start with **Qwen3 8B**: it fits comfortably and handles connectors ("tool use") well.

1. Click the magnifying-glass (search) icon in LM Studio.
2. Type `qwen3 8b` and pick the result from the official Qwen or LM Studio community listing.
3. Choose a version marked **Q4** (for example Q4_K_M). An "MLX" version is built for Apple chips and is a good choice too.
4. Click Download and wait. It's about 5 GB.
5. When it finishes, open the chat icon and choose the model at the top to load it.

Send a test message such as "Explain what OSINT means in two sentences." A reply within a few seconds means it's working.

| Model | Size on disk (approx.) | When to try it |
| --- | --- | --- |
| Qwen3 8B (Q4) | ~5 GB | Best starting point |
| Qwen 3.5 9B (Q4) | ~6 GB | Slightly stronger reasoning, still comfortable |
| Gemma 3 12B (Q4) | ~8 GB | Higher quality, but uses most of your memory |
| Phi-4 Mini | ~2.5 GB | Very fast and light if your computer feels strained |

Sizes are approximate and model releases change quickly; if a newer Qwen or Gemma appears in the search list, it's reasonable to try it.

**How to tell a model is too heavy:** if other apps stutter, responses take more than a minute to begin, or the Mac gets noticeably warm, switch to a smaller one.

## Step 3: Create your Research Library folder

This folder is the most important part of the setup, because it's where your research lives regardless of which app or model you use. Everything else can be replaced; this can't.

1. In Finder, open Documents and create a folder called **Research Library**.
2. Inside it, make a folder for each topic you research, and one called `Inbox` for things you haven't sorted yet.
3. Save things there as you collect them: PDFs, articles saved as PDF (in your browser, Print, then Save as PDF), and your own notes.
4. Name each file so it records where it came from and when, for example `2026-09-19 Source Name - Short Title.pdf`.

That naming habit is your provenance record: months later you can tell what something was and when you got it without opening it. Back the folder up with Time Machine or iCloud Drive.

## Step 4: Connect your first source (web search)

Your first connector is the DuckDuckGo search connector, which needs no account or key. It can search the web and read the text of a page, rate-limited to about 30 searches and 20 page reads per minute.

**Part A: install the helper.** This uses Terminal once, to install a helper called `uv` that runs the connector.

1. Open Terminal (Cmd+Space, type `Terminal`, press Return).
2. Paste this line and press Return:

   ```
   curl -LsSf https://astral.sh/uv/install.sh | sh
   ```

3. When it finishes, close Terminal and open it again.
4. Type `which uvx` and press Return.
5. It should print a path such as `/Users/yourname/.local/bin/uvx`. Copy that whole path for Part B.

**Part B: tell LM Studio about the connector.**

1. In LM Studio, open the Program tab in the right-hand sidebar and choose Install, then Edit mcp.json.
2. Paste the following, replacing the `command` path with the one you copied:

   ```json
   {
     "mcpServers": {
       "duckduckgo": {
         "command": "/Users/yourname/.local/bin/uvx",
         "args": ["duckduckgo-mcp-server"]
       }
     }
   }
   ```

3. Save the file. The connector should appear in the Program tab; make sure its toggle is on.

If the file already had entries inside `mcpServers`, keep them and add `duckduckgo` alongside, separated by a comma. LM Studio asks permission the first time the model uses a connector; read what it's asking before you approve.

**A safety rule for all connectors:** only install connectors from sources you trust, since a connector can run programs on your computer. LM Studio's own documentation gives the same warning.

## Step 5: Your first research session

Open a new chat, load the model, and make sure the connector is on. Start each research chat with standing instructions; this does more for reliability than anything else:

```
You are a careful research assistant. For every claim, search first and cite the source URL. Separate what the sources say from your own inference. If sources conflict or you cannot find something, say so plainly instead of guessing. Never state a fact without a source.
```

Then ask a real question, for example: "Search for recent coverage of [your topic] and summarize the three most credible sources, with links."

- **Open the links yourself** for anything that matters. The summary tells you where to look; the source is what you rely on.
- **Ask it to read a specific page** by giving the URL.
- **Work from your library:** drag a PDF from your Research Library into the chat and ask questions about it.
- **Save what's valuable** into the right folder with the naming pattern from Step 3, with the model's summary in a note beside it.

LM Studio reads files you attach to a chat, but it doesn't build a searchable index of your whole library. If you need that, it's the point to add a library tool.

## Adding new sources later

Every new source follows the same routine:

1. **Find a connector.** Search for the source's name plus "MCP server."
2. **Vet it.** Look for a public project page with recent updates and a clear description. Prefer connectors that need no key, or only a free key from the source itself.
3. **Add it** as another entry inside `mcpServers`. Connectors that run with `uvx` need your full path from Step 4; ones that run with `npx` need Node.js installed first.
4. **Test it** with a question only that source could answer.
5. **Note it** in a `Sources I use` file in your Research Library, with the date added and what it's good for.

Community-built connectors exist for many open-source research sources. They vary in quality, so treat these as leads to vet, not recommendations:

| Source | Good for | Example to look at |
| --- | --- | --- |
| SEC EDGAR | Company filings and disclosures | [sec-edgar-mcp](https://github.com/stefanoamorelli/sec-edgar-mcp) |
| OpenAlex | Scholarly papers and citations | [openalex-mcp-server](https://github.com/dgitj/openalex-mcp-server) |
| Wayback Machine | Past versions of web pages | [wayback-machine-mcp](https://github.com/fuushyn/wayback-machine-mcp) |
| Internet Archive | Digitized books and captures | [mcp-archiveorg](https://github.com/smeet666/mcp-archiveorg) |

## Habits that keep this reliable

- **Treat answers as leads, not facts.** Anything you'll rely on gets checked against the original source.
- **Insist on citations.** An answer it can't source doesn't go in your notes.
- **Keep the folder as the source of truth.** If LM Studio, a model, or a connector breaks, your library is unaffected.
- **Know what leaves your computer.** The model runs locally, but searches and page fetches go out over the internet like any browsing.
- **Stay within each source's terms.** Respect site rules and rate limits, and be thoughtful about compiling information on private individuals, even when each piece is public.
- **Update deliberately.** If something breaks after an update, check the connector first.

## If something goes wrong

| Problem | What to try |
| --- | --- |
| The model never uses search | Check the connector's toggle in the Program tab and that the model supports tools (Qwen3 does). Then ask directly: "Use your search tool to look this up." |
| The connector won't start | Check mcp.json for a missing comma or bracket, and that the `command` path matches what `which uvx` printed. |
| Responses are very slow | Close heavy apps, plug in the charger, and switch to a smaller model. |
| Answers look wrong or invented | Ask for sources, open them yourself, and re-add the standing instructions. Try a larger model if it persists. |
| Unsure a connector is safe | Don't install it until you've vetted it. |

To start fresh, delete LM Studio from Applications; your Research Library stays untouched.

## Sources

Consulted September 19, 2026:

- [LM Studio: Use MCP Servers](https://lmstudio.ai/docs/app/mcp), for the Program tab, mcp.json steps, and the safety warning
- [Best LLM for 16GB Mac (Will It Run AI)](https://willitrunai.com/blog/best-llm-for-16gb-mac), for usable memory and model recommendations
- [DuckDuckGo MCP Server guide (MCP.Directory)](https://mcp.directory/blog/duckduckgo-mcp-complete-guide-2026), for the connector's install command, tools, and rate limits

Model sizes are approximate and from general knowledge, not from those pages.
