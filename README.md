# chromem-go

[![Go Reference](https://pkg.go.dev/badge/github.com/philippgille/chromem-go.svg)](https://pkg.go.dev/github.com/philippgille/chromem-go)

Embeddable in-memory vector database for Go with Chroma-like interface and zero third-party dependencies.

It's *not* a library to connect to ChromaDB. It's an in-memory database on its own.

Being embeddable enables you to add retrieval augmented generation (RAG) features into your Go app *without having to run a separate database*. Like when using SQLite instead of PostgreSQL/MySQL/etc.

The focus is not scale or number of features, but simplicity.

## Contents

1. [Interface](#interface)
2. [Features](#features)
3. [Usage](#usage)
4. [Motivation](#motivation)
5. [Related projects](#related-projects)

## Interface

Our inspiration, the [Chroma](https://www.trychroma.com/) interface, is the following (taken from their [README](https://github.com/chroma-core/chroma/blob/0.4.21/README.md)).

```python
import chromadb
# setup Chroma in-memory, for easy prototyping. Can add persistence easily!
client = chromadb.Client()

# Create collection. get_collection, get_or_create_collection, delete_collection also available!
collection = client.create_collection("all-my-documents")

# Add docs to the collection. Can also update and delete. Row-based API coming soon!
collection.add(
    documents=["This is document1", "This is document2"], # we handle tokenization, embedding, and indexing automatically. You can skip that and add your own embeddings as well
    metadatas=[{"source": "notion"}, {"source": "google-docs"}], # filter on these!
    ids=["doc1", "doc2"], # unique for each doc
)

# Query/search 2 most similar results. You can also .get by id
results = collection.query(
    query_texts=["This is a query document"],
    n_results=2,
    # where={"metadata_field": "is_equal_to_this"}, # optional filter
    # where_document={"$contains":"search_string"}  # optional filter
)
```

Our Go library exposes the same interface:

```go
package main

import "github.com/philippgille/chromem-go"

func main() {
    // Set up chromem-go in-memory, for easy prototyping. Persistence will be added in the future.
    // We call it DB instead of client because there's no client-server separation. The DB is embedded.
    db := chromem.NewDB()

    // Create collection. GetCollection, GetOrCreateCollection, DeleteCollection also available!
    collection := db.CreateCollection("all-my-documents", nil, nil)

    // Add docs to the collection. Update and delete will be added in the future.
    // Row-based API will be added when Chroma adds it!
    _ = collection.Add(ctx,
        []string{"doc1", "doc2"}, // unique ID for each doc
        nil, // We handle embedding automatically. You can skip that and add your own embeddings as well.
        []map[string]string{{"source": "notion"}, {"source": "google-docs"}}, // Filter on these!
        []string{"This is document1", "This is document2"},
    )

    // Query/search 2 most similar results. Getting by ID will be added in the future.
    results, _ := collection.Query(ctx,
        "This is a query document",
        2,
        map[string]string{"metadata_field": "is_equal_to_this"}, // optional filter
        map[string]string{"$contains": "search_string"},         // optional filter
    )
}
```

Initially `chromem-go` started with just these methods, but we added more over time. We intentionally don't want to cover 100% of Chroma's API surface though.  
Instead, we will add some alternative methods that are more Go-idiomatic.

See the Godoc for details: <https://pkg.go.dev/github.com/philippgille/chromem-go>

## Features

- [X] Zero dependencies on third party libraries
- [X] Embeddable (like SQLite, i.e. no client-server model, no separate DB to maintain)
- [X] Multi-threaded processing (when adding and querying documents), making use of Go's native concurrency features
- Embedding creators:
  - [X] [OpenAI](https://platform.openai.com/docs/guides/embeddings/embedding-models) (default)
  - [X] [Mistral](https://docs.mistral.ai/platform/endpoints/#embedding-models)
  - [X] [Jina](https://jina.ai/embeddings)
  - [X] [mixedbread.ai](https://www.mixedbread.ai/)
  - [X] [LocalAI](https://github.com/mudler/LocalAI)
  - [X] Bring your own
  - You can also pass existing embeddings when adding documents to a collection.
  - [ ] [ollama](https://ollama.ai/)
    - (As of 2024-02-10 their OpenAI compatible API doesn't support embeddings yet, but they have a custom API which does)
- Similarity search:
  - [X] Exact nearest neighbor search using cosine similarity
  - [ ] Approximate nearest neighbor search with index
    - [ ] Hierarchical Navigable Small World (HNSW)
    - [ ] Inverted file flat (IVFFlat)
- Filters:
  - [X] Document filters: `$contains`, `$not_contains`
  - [X] Metadata filters: Exact matches
  - [ ] Operators (`$and`, `$or` etc.)
- Storage:
  - [X] In-memory
  - [ ] Persistent (file)
  - [ ] Persistent (others (S3, PostgreSQL, ...))
- Data types:
  - [X] Documents (text)
  - [ ] Images
  - [ ] Videos

## Usage

For a full, working example, using the vector database for retrieval augmented generation (RAG), see [example/main.go](example/main.go)

## Motivation

In December 2023, when I wanted to play around with retrieval augmented generation (RAG) in a Go program, I looked for a vector database that could be embedded in the Go program, just like you would embed SQLite in order to not require any separate DB setup and maintenance. I was surprised when I didn't find any, given the abundance of embedded key-value stores in the Go ecosystem.

At the time most of the popular vector databases like Pinecone, Qdrant, Milvus, Weaviate and others were not embeddable at all. ChromaDB was, but only in Python.

Then I found [@eliben](https://github.com/eliben)'s [blog post](https://eli.thegreenplace.net/2023/retrieval-augmented-generation-in-go/) and [example code](https://github.com/eliben/code-for-blog/tree/eda87b87dad9ed8bd45d1c8d6395efba3741ed39/2023/go-rag-openai) which showed that with very little Go code you could create a very basic PoC of a vector database.

That's when I decided to build my own vector database, embeddable in Go, inspired by the ChromaDB interface. ChromaDB stood out for being embeddable (in Python), and by showing its core API in 4 commands on their README and on the landing page of their website.

## Related projects

- Shoutout to [@eliben](https://github.com/eliben) whose [blog post](https://eli.thegreenplace.net/2023/retrieval-augmented-generation-in-go/) and [example code](https://github.com/eliben/code-for-blog/tree/eda87b87dad9ed8bd45d1c8d6395efba3741ed39/2023/go-rag-openai) inspired me to start this project!
- [Chroma](https://github.com/chroma-core/chroma): Looking at Pinecone, Qdrant, Milvus, Weaviate and others, Chroma stood out by showing its core API in 4 commands on their README and on the landing page of their website. It was also the only one which could be embedded (in Python).
- The big, full-fledged client-server-based vector databases for maximum scale and performance:
  - [Pinecone](https://www.pinecone.io/): Closed source
  - [Qdrant](https://github.com/qdrant/qdrant): Written in Rust
  - [Milvus](https://github.com/milvus-io/milvus): Written in Go, but not embeddable as of December 2023
  - [Weaviate](https://github.com/weaviate/weaviate): Written in Go, but not embeddable as of December 2023
