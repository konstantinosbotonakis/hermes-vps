# 10, Bonus, RAG Memory with ChromaDB

This is an optional advanced guide.

Do not do this during your first Hermes setup.

First make the normal Hermes Agent setup work with Telegram, automations, and a hosted model provider.

Then come back to this if you want a local searchable memory store for larger notes, project facts, research snippets, customer context, or reusable knowledge.

## What RAG memory is

RAG means Retrieval-Augmented Generation.

For memory, the idea is simple:

- Save useful facts or notes into a database.
- Convert each note into a vector using an embedding model.
- Search by meaning, not just exact keywords.
- Give the most relevant notes back to Hermes when they are useful.

This is different from a normal markdown memory file.

A markdown memory file is easy to read and edit, but it becomes awkward when it gets large.

A RAG memory store can hold many more entries and retrieve the closest matches for a question.

## How this fits with Hermes memory

This does not replace Hermes built-in memory.

Hermes already has built-in memory files:

- `MEMORY.md` for important agent notes.
- `USER.md` for important user profile facts.

Those should stay small and focused. They are for facts that should always be available.

Use this ChromaDB RAG memory as a companion store for deeper recall.

Good use cases:

- Project notes.
- Longer research summaries.
- Past decisions.
- Customer or product context.
- Reusable snippets that do not need to be in every prompt.

Hermes also has official external memory providers. You can check them with:

```bash
hermes memory setup
hermes memory status
```

Use this DIY ChromaDB guide if you want a local semantic memory store that Hermes can query through terminal/tool use, or that you may later wrap as a plugin or MCP server.

## When this is worth trying

Try this if:

- Your normal Hermes setup already works.
- You are comfortable running Python scripts.
- You want local storage instead of a cloud memory provider.
- You have more memory notes than should fit in `MEMORY.md` or `USER.md`.
- You want semantic search over notes, not just manual markdown files.

Do not try this if:

- You are still setting up Hermes for the first time.
- You only need a few permanent preferences.
- You do not want to maintain a Python environment.
- You expect this to automatically improve every answer without retrieval steps.

## Requirements

Use a VPS or local machine with:

- Ubuntu 24.04 LTS or similar.
- Python 3.10 or newer.
- 2 GB RAM minimum.
- 4 GB RAM preferred.
- A working Hermes Agent setup.

The embedding model runs locally on CPU. No GPU is required.

The first add or search command may take longer because the embedding model has to download.

## Step 1, Install Python tools

Connect to your VPS as the `hermes` user.

Then run:

```bash
sudo apt update
sudo apt install -y python3 python3-pip python3-venv
```

## Step 2, Create the memory folder

Run:

```bash
mkdir -p ~/hermes_memory/chromadb
cd ~/hermes_memory
```

## Step 3, Create a Python environment

Run:

```bash
python3 -m venv .venv
. .venv/bin/activate
pip install -U pip
pip install chromadb sentence-transformers torch numpy
```

Keep this virtual environment for your memory scripts.

When you come back later, activate it with:

```bash
cd ~/hermes_memory
. .venv/bin/activate
```

## Step 4, Create the memory helper script

Create a small command-line helper:

```bash
cat > ~/hermes_memory/rag_memory.py <<'PY'
#!/usr/bin/env python3
from __future__ import annotations

import argparse
import hashlib
import json
import shutil
from datetime import datetime, timezone
from pathlib import Path

import chromadb
from sentence_transformers import SentenceTransformer

BASE_DIR = Path.home() / "hermes_memory"
DB_DIR = BASE_DIR / "chromadb"
COLLECTION_NAME = "memory_store"
MODEL_NAME = "all-MiniLM-L6-v2"

BASE_DIR.mkdir(parents=True, exist_ok=True)
DB_DIR.mkdir(parents=True, exist_ok=True)

client = chromadb.PersistentClient(path=str(DB_DIR))
collection = client.get_or_create_collection(
    name=COLLECTION_NAME,
    metadata={"description": "Local Hermes RAG memory store"},
)

_embedding_model: SentenceTransformer | None = None


def get_embedding_model() -> SentenceTransformer:
    global _embedding_model
    if _embedding_model is None:
        _embedding_model = SentenceTransformer(MODEL_NAME)
    return _embedding_model


def now_iso() -> str:
    return datetime.now(timezone.utc).isoformat()


def make_id(text: str, memory_type: str, source: str) -> str:
    digest = hashlib.sha256(f"{memory_type}\n{source}\n{text}".encode("utf-8")).hexdigest()
    return f"mem_{digest[:16]}"


def embed(text: str) -> list[float]:
    return get_embedding_model().encode([text], normalize_embeddings=True)[0].tolist()


def add_memory(args: argparse.Namespace) -> None:
    memory_id = args.id or make_id(args.text, args.memory_type, args.source)
    metadata = {
        "type": args.memory_type,
        "source": args.source,
        "created_at": now_iso(),
    }

    collection.upsert(
        ids=[memory_id],
        documents=[args.text],
        embeddings=[embed(args.text)],
        metadatas=[metadata],
    )

    print(json.dumps({"id": memory_id, "count": collection.count()}, indent=2))


def search_memory(args: argparse.Namespace) -> None:
    results = collection.query(
        query_embeddings=[embed(args.query)],
        n_results=args.limit,
        include=["documents", "metadatas", "distances"],
    )

    items = []
    documents = results.get("documents", [[]])[0]
    metadatas = results.get("metadatas", [[]])[0]
    distances = results.get("distances", [[]])[0]
    ids = results.get("ids", [[]])[0]

    for index, document in enumerate(documents):
        items.append(
            {
                "id": ids[index],
                "distance": distances[index],
                "metadata": metadatas[index],
                "text": document,
            }
        )

    print(json.dumps(items, indent=2))


def count_memories() -> None:
    print(json.dumps({"count": collection.count()}, indent=2))


def backup_memory() -> None:
    timestamp = datetime.now(timezone.utc).strftime("%Y%m%d_%H%M%S")
    backup_dir = BASE_DIR / f"chromadb_backup_{timestamp}"
    shutil.copytree(DB_DIR, backup_dir)
    print(json.dumps({"backup": str(backup_dir)}, indent=2))


def clear_memory(args: argparse.Namespace) -> None:
    if not args.yes:
        raise SystemExit("Refusing to clear memory without --yes")

    try:
        client.delete_collection(COLLECTION_NAME)
    except Exception as exc:
        raise SystemExit(f"Could not delete collection: {exc}") from exc

    print(json.dumps({"cleared": COLLECTION_NAME}, indent=2))


def main() -> None:
    parser = argparse.ArgumentParser(description="Local ChromaDB RAG memory helper")
    subcommands = parser.add_subparsers(dest="command", required=True)

    add_parser = subcommands.add_parser("add", help="Add or update one memory")
    add_parser.add_argument("--text", required=True)
    add_parser.add_argument("--type", dest="memory_type", default="note")
    add_parser.add_argument("--source", default="manual")
    add_parser.add_argument("--id", default=None)

    search_parser = subcommands.add_parser("search", help="Search memories by meaning")
    search_parser.add_argument("--query", required=True)
    search_parser.add_argument("--limit", type=int, default=5)

    subcommands.add_parser("count", help="Count stored memories")
    subcommands.add_parser("backup", help="Copy the ChromaDB folder")

    clear_parser = subcommands.add_parser("clear", help="Delete all memories")
    clear_parser.add_argument("--yes", action="store_true")

    args = parser.parse_args()

    if args.command == "add":
        add_memory(args)
    elif args.command == "search":
        search_memory(args)
    elif args.command == "count":
        count_memories()
    elif args.command == "backup":
        backup_memory()
    elif args.command == "clear":
        clear_memory(args)


if __name__ == "__main__":
    main()
PY

chmod +x ~/hermes_memory/rag_memory.py
```

Why the script uses `Path.home()`:

- It expands your home directory safely.
- It avoids storing data in a literal folder named `~`.
- It keeps the database at `~/hermes_memory/chromadb`.

## Step 5, Add test memories

Run:

```bash
cd ~/hermes_memory
. .venv/bin/activate

python rag_memory.py add \
  --text "The user prefers concise responses." \
  --type preference \
  --source manual

python rag_memory.py add \
  --text "The user works in the Europe/London timezone." \
  --type profile \
  --source manual
```

The script prints the memory ID and total count after each write.

The IDs are stable. If you add the same text with the same type and source again, it updates the existing memory instead of creating a duplicate.

## Step 6, Search memory

Run:

```bash
python rag_memory.py search \
  --query "What communication style does the user prefer?" \
  --limit 3
```

You should see the concise response preference in the results.

Then try:

```bash
python rag_memory.py search \
  --query "What timezone should automations use?" \
  --limit 3
```

You should see the London timezone memory.

## Step 7, Count memories

Run:

```bash
python rag_memory.py count
```

This prints the total number of stored memories.

## Step 8, Use it from Hermes

Hermes can use terminal tools, so the simplest path is to ask Hermes to query the helper script when it needs deeper memory.

Example prompt:

```text
Search my local RAG memory before answering this. Use ~/hermes_memory/rag_memory.py and query for: "OpenCart deployment notes". Then answer using only relevant memories plus the current context.
```

Another example:

```text
Save this in local RAG memory as a project note: "The OpenCart Greece staging server uses PHP 8.2 and Redis object cache."
```

For a cleaner long-term setup, wrap this script as a Hermes plugin or MCP tool. That is more work, but it gives Hermes a dedicated memory search/write tool instead of relying on shell commands.

## Step 9, Back up memory

Run:

```bash
cd ~/hermes_memory
. .venv/bin/activate
python rag_memory.py backup
```

This creates a timestamped copy such as:

```text
~/hermes_memory/chromadb_backup_20260522_120000
```

You can also back it up manually:

```bash
cp -r ~/hermes_memory/chromadb ~/hermes_memory/chromadb_backup_$(date +%Y%m%d)
```

## Step 10, Clear memory

Only do this if you really want to delete the collection.

Run:

```bash
cd ~/hermes_memory
. .venv/bin/activate
python rag_memory.py clear --yes
```

Without `--yes`, the script refuses to clear memory.

## Verification checklist

Check that the helper works:

```bash
cd ~/hermes_memory
. .venv/bin/activate
python rag_memory.py count
python rag_memory.py search --query "user preferences" --limit 3
```

Check that the database exists:

```bash
ls -lah ~/hermes_memory/chromadb
```

You should see ChromaDB files, including a SQLite database file.

## Troubleshooting

If Python says the packages are missing, activate the virtual environment:

```bash
cd ~/hermes_memory
. .venv/bin/activate
```

If the first add or search command is slow, wait for the embedding model download to finish.

If search returns nothing useful:

- Check that `python rag_memory.py count` is greater than zero.
- Search with a simpler query.
- Store clearer memories with specific nouns and project names.

If the VPS runs out of memory:

- Use fewer simultaneous Python processes.
- Keep the embedding model small.
- Use a VPS with 4 GB RAM.

If Hermes forgets to use this store:

- Keep only critical facts in `MEMORY.md` and `USER.md`.
- Add a short reminder there that deeper notes live in `~/hermes_memory/rag_memory.py`.
- Ask Hermes explicitly to search local RAG memory for tasks where old context matters.

## Practical recommendation

Use Hermes built-in memory for a small number of always-important facts.

Use this ChromaDB RAG store for larger searchable notes.

Do not dump everything into memory. Save facts, summaries, decisions, and reusable context that you are likely to search for later.

## Sources

- Hermes Persistent Memory: https://hermes-agent.nousresearch.com/docs/user-guide/features/memory/
- Hermes Memory Providers: https://hermes-agent.nousresearch.com/docs/user-guide/features/memory-providers/
- Chroma PersistentClient docs: https://cookbook.chromadb.dev/core/clients/
- Chroma core API overview: https://cookbook.chromadb.dev/core/
