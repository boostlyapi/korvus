# korvus

Retrieval-augmented generation needs document preparation, embeddings, search, and context for text generation. korvus brings those steps into Postgres through a search SDK, with the RAG pipeline expressed as a single database query.

## Features

- Split document text and generate embeddings through a configured pipeline.
- Insert or update documents in a collection with `upsert_documents`.
- Retrieve relevant text with vector search and rerank results.
- Combine retrieval and text generation in a database query.
- Access the SDK from Rust, Python, JavaScript, or C.

## Installation

**A standard Postgres installation is not enough.** The database must have the PostgresML `pgml` extension and `pgvector` installed. See the upstream [database setup guide](https://postgresml.org/docs/open-source/pgml/developers/quick-start-with-docker) for setup instructions.

From an existing Rust project, add the SDK dependency:

```bash
cargo add korvus
```

Set the database connection string:

```bash
export KORVUS_DATABASE_URL="postgresql://postgres:postgres@localhost:5432/postgresml"
```

Replace the credentials, host, port, and database name with your database settings.

The usage example below uses the Python binding. Install it separately:

```bash
pip install korvus
```

## Usage

korvus is an SDK. The following example runs a small ingestion script rather than assuming a `korvus` command-line executable exists.

Save this as `ingest.py`. It configures text splitting and semantic search, then inserts a document supplied as a command-line argument. Reusing the document ID updates that document.

```python
import argparse
import asyncio
from korvus import Collection, Pipeline


async def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("text", help="Document text to insert")
    parser.add_argument("--id", default="1", help="Document ID")
    args = parser.parse_args()

    collection = Collection("korvus-demo-v0")
    pipeline = Pipeline(
        "v1",
        {
            "text": {
                "splitter": {"model": "recursive_character"},
                "semantic_search": {"model": "Alibaba-NLP/gte-base-en-v1.5"},
            }
        },
    )

    await collection.add_pipeline(pipeline)
    await collection.upsert_documents([
        {"id": args.id, "text": args.text},
    ])
    print(f"Upserted document {args.id}")


asyncio.run(main())
```

Run it against the configured database:

```bash
python ingest.py --id 1 "Korvus puts the RAG pipeline inside Postgres."
```

This example prepares documents for semantic retrieval; it does not generate an answer. Refer to the [upstream SDK documentation](https://postgresml.org/docs/open-source/korvus/) for search and RAG calls and language-specific examples.

## License

MIT. This project is based on [postgresml/korvus](https://github.com/postgresml/korvus), which is licensed under the MIT License. Retain the upstream copyright and permission notices in copies or substantial portions of the original software.
