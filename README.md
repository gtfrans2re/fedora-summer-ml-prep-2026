# fedora-summer-ml-prep-2026
Pre-internship preparation repository for my 2026 Fedora ML/AI internship

## Exploring the RamaLama RAG

### Install RamaLama

- To install RamaLama on your Fedora system, run this in a command-line shell

```bash
sudo dnf install ramalama
```
- To install differently or on another system, please check the official installation guide:

[Install][https://github.com/containers/ramalama?tab=readme-ov-file#install]

### Display RamaLama version

```bash
ramalama version
```

### About RamaLama RAG

ramalama-rag is a command-line tool based on RamaLama that takes in documents as input and converts them into a RAG database and package as a container image. Basically two containers together, namely the llama.cpp container serves the AI models, while a lightweight RAG container runs the document processing pipeline.

## Document Processing & Retrieval Pipeline Architecture

### Pipeline Overview

```text
                 ┌──────────────────────────┐
                 │   Input Documents        │
                 │--------------------------│
                 │ .txt / .md / .html       │
                 │ PDFs                     │
                 │ Images                   │
                 └────────────┬─────────────┘
                              │
             ┌────────────────┴────────────────┐
             │                                 │
             ▼                                 ▼
┌─────────────────────────┐      ┌─────────────────────────┐
│ Direct Text Extraction  │      │ Granite Docling VLM     │
│-------------------------│      │ served via llama.cpp    │
│ Parse raw text files    │      │ Page-by-page OCR/VLM    │
└────────────┬────────────┘      └────────────┬────────────┘
             │                                │
             └────────────────┬───────────────┘
                              ▼
                 ┌──────────────────────────┐
                 │ Unified Document Content │
                 └────────────┬─────────────┘
                              ▼
                 ┌──────────────────────────┐
                 │ Section-Based Chunking   │
                 │--------------------------│
                 │ Chunk by headings/       │
                 │ semantic sections        │
                 └────────────┬─────────────┘
                              ▼
                 ┌──────────────────────────┐
                 │ Embedding Generation     │
                 │--------------------------│
                 │ EmbeddingGemma           │
                 │ served via llama.cpp     │
                 └────────────┬─────────────┘
                              ▼
                 ┌──────────────────────────┐
                 │ Vector Storage           │
                 │--------------------------│
                 │ Qdrant On-Disk           │
                 │ Collection               │
                 └────────────┬─────────────┘
                              ▼
                 ┌──────────────────────────┐
                 │ OCI Packaging            │
                 │--------------------------│
                 │ FROM scratch OCI Image   │
                 │ containing Qdrant DB     │
                 └──────────────────────────┘
```

### Convert a single PDF

```bash
ramalama rag ./data/2305.14325v1.pdf ./ramalama-rag-output
```

### Output after successful conversion

.
├── data
│   └── 2305.14325v1.pdf
├── LICENSE
├── ramalama-rag-output
│   ├── collection
│   │   └── rag
│   │       └── storage.sqlite
│   └── meta.json
└── README.md