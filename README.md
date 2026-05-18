# fedora-summer-ml-prep-2026

Pre-internship preparation repository for my 2026 Fedora ML/AI internship.

---

## Exploring RamaLama RAG

### Install RamaLama

To install RamaLama on a Fedora system, run:

```bash
sudo dnf install ramalama
```

To install RamaLama on another Linux distribution or using alternative methods, check the official installation guide:

[Install RamaLama](https://github.com/containers/ramalama?tab=readme-ov-file#install)

---

### Display RamaLama Version

```bash
ramalama version
```

![RamaLama Version Output](screenshots/ramalama_version.png)

---

### About RamaLama RAG

`ramalama-rag` is a command-line tool built on top of RamaLama that converts documents into a Retrieval-Augmented Generation (RAG) database and packages it as a container image.

The architecture relies on two cooperating containers:

- A `llama.cpp` container responsible for serving AI models.
- A lightweight RAG container responsible for document ingestion, processing, chunking, embedding generation, and vector database creation.

---

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
                 │ Chunk by headings and    │
                 │ semantic sections        │
                 └────────────┬─────────────┘
                              ▼
                 ┌──────────────────────────┐
                 │ Embedding Generation     │
                 │--------------------------│
                 │ EmbeddingGemma           │
                 │ served via llama.cpp    │
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

---

### Convert a Single PDF

```bash
ramalama rag ./data/2305.14325v1.pdf ./ramalama-rag-output
```

![RamaLama PDF Conversion](screenshots/ramalama_rag_pfd.png)

![RamaLama Version Output](screenshots/ramalama_rag_pdf_output.png)

---

### Output After Successful Conversion

```text
.
├── data
│   └── 2305.14325v1.pdf
├── LICENSE
├── ramalama-rag-output
│   ├── collection
│   │   └── rag
│   │       └── storage.sqlite
│   └── meta.json
└── README.md
```
