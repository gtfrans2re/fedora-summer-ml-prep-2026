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

### Convert a directory file content

As RamaLama RAG converts pdf, txt, and html files into a RAG image, we can simply run it on a directory and it will process its content and save the output.

```bash
ramalama rag ./data myrag:latest
```

* note that the data directory contains three files of PDF, TXT, and HTML format.

RamaLama-RAG successfully converted the PDF and the HTML files in the data directory, but got stuck in converting that of the txt file, which currently contains an ics calendar data content but saved as a txt file. It threw an this : `Error: File format not allowed: schedule.txt`.

![RamaLama Directory Output](screenshots/ramalama_dir_ragging.png)

The hypothesis above was further invalidated by run it a simple 1-line text file and it failed again.
However, looking closely at the verbose output, we notice that txt type files are not allowed even though it is mentioned on the RamaLama RAG documentation page here: [https://github.com/containers/ramalama/blob/main/docs/ramalama-rag.1.md](https://github.com/containers/ramalama/blob/main/docs/ramalama-rag.1.md) that it is a supported format.

![RamaLama Simple TXT Output](screenshots/ramalama_simple_txt_test.png)

### Runnning the Hugging Face GPT-OSSS with RamaLama

To run the model locally, we need to do the following:

```bash
ramalama run gpt-oss
```

![RamaLama RUN HF:GPT-OSS](screenshots/ramalama_run_gpt_oss.png)

You can also supply a full model URL if you prefer to reference a specific location directly (for example, `hf://unsloth/gpt-oss-20b-GGUF`).

More about this on: [https://www.redhat.com/en/blog/run-containerized-ai-models-locally-ramalama](https://www.redhat.com/en/blog/run-containerized-ai-models-locally-ramalama)

Moreover, `ramalama serve` makes it possible to expose a local model through a REST endpoint, an interactive GUI that allows you to type in a prompt and get the model's answer to the prompt after reasoning with:

```bash
ramalama serve gpt-oss --port 8000
```

![RamaLama Serve GUI Prompting](screenshots/ramalama_serve_gpt_oss_ask.png)

![RamaLama Serve GUI Model Answering](screenshots/ramalama_serve_gpt_oss_response.png)

