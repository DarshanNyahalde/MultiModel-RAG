# NovaCore Multimodal RAG

A notebook-based demonstration of **multimodal Retrieval-Augmented Generation (RAG)** using LangChain, Pinecone, Hugging Face embeddings, and Groq-hosted language and vision models. It answers questions about a PDF report by retrieving relevant text, tables, and visual evidence such as charts and diagrams.

> **Demo data:** `NovaCore Systems Ltd. – FY2026 Business Performance & Operations Report` is explicitly fictional and is provided for testing and demonstration. Its company names, metrics, people, and events are not real-world business facts.

## Project files

| File | Purpose |
|---|---|
| `Build_Multimodal_RAG_NovaCore.ipynb` | Main notebook: document extraction, visual summarization, embedding, indexing, retrieval, and question answering. |
| `NovaCore_Multimodal_Company_Report_2026.pdf` | Sample PDF used as the RAG knowledge source. |
| `Multimodal_RAG_LangChain_Pinecone_With_Examples.pptx.pdf` | PDF version of the presentation explaining the multimodal RAG concept and pipeline. |

## What the project does

Traditional text-only RAG can miss information that appears only in charts, images, and diagrams. This project converts multiple content types into searchable LangChain documents while retaining references to original visuals.

- **Text extraction:** extracts page text from the PDF with PyMuPDF.
- **Table extraction:** attempts to extract tables and represents them as Markdown so row/column relationships are easier to preserve.
- **Visual extraction:** extracts embedded PDF images, including charts and diagrams.
- **Visual summarization:** sends each extracted visual to a vision model and creates a concise factual text summary.
- **Embeddings:** encodes text, table content, and visual summaries with `sentence-transformers/all-MiniLM-L6-v2`.
- **Vector storage and retrieval:** stores documents in a Pinecone index and retrieves the top five relevant documents.
- **Answer generation:** uses a text RAG chain when no visual evidence is retrieved; when retrieved documents include available visual paths, sends the question, retrieved context, and up to three images to a vision model.
- **Source display:** prints retrieved page numbers and content modalities; optionally displays retrieved images.

## Architecture

```text
Input PDF
   |
   +--> Page text --------------------+
   |                                  |
   +--> Tables --> Markdown ----------+--> LangChain Documents
   |                                  |           |
   +--> Images / charts / diagrams    |           +--> Text embeddings
                |                     |                    |
                v                     |                    v
          Groq vision model           |              Pinecone index
                |                     |                    |
                v                     |               Top-5 retrieval
          Text summaries -------------+                    |
                                                         Query
                                                           |
                                            +--------------+--------------+
                                            |                             |
                                     No visual retrieved           Visual retrieved
                                            |                             |
                                        Text LLM                 Vision LLM + original
                                                                    visual(s)
                                            |                             |
                                            +--------------+--------------+
                                                           |
                                                     Grounded answer
```

The original visual files are retained locally, and their paths are stored in document metadata. The vision model can therefore inspect retrieved original images when visual evidence is available.

## Technology stack

- Python and Jupyter Notebook
- LangChain and LangChain Core
- LangChain Groq integration
- LangChain Hugging Face embeddings integration
- LangChain Pinecone integration and Pinecone
- Hugging Face Sentence Transformers
- PyMuPDF (`fitz`) for PDF text, table, and image extraction
- Pillow for image processing
- pandas and tabulate for table handling and display

Models configured in the notebook:

| Purpose | Configured model |
|---|---|
| Text answer generation | `openai/gpt-oss-20b` through Groq |
| Visual summarization and visual answers | `qwen/qwen3.8-27b` through Groq |
| Text embeddings | `sentence-transformers/all-MiniLM-L6-v2` |

Model availability depends on the provider account and the models currently supported by the API.

## Requirements

- Python environment that can run Jupyter notebooks
- Internet access for downloading packages, calling Groq, and connecting to Pinecone
- A Groq API key
- A Pinecone API key
- The sample PDF report in the location configured by the notebook

## Setup and run

### 1. Get the project files

Keep the notebook and sample report together in the same working directory, or update `PDF_PATH` in the notebook to point to the report.

### 2. Open the notebook

Open `Build_Multimodal_RAG_NovaCore.ipynb` in Jupyter Notebook, JupyterLab, or Google Colab. Run the cells **in order**, starting with package installation.

### 3. Install dependencies

The first notebook cell installs the main dependencies:

```bash
pip install -U \
  langchain \
  langchain-core \
  langchain-groq \
  langchain-huggingface \
  langchain-pinecone \
  pinecone \
  sentence-transformers \
  pymupdf \
  groq \
  pandas \
  tabulate \
  pillow
```

The notebook also includes a Pillow reinstall cell for its execution environment. If you use a different environment and encounter image-library conflicts, troubleshoot that dependency separately before continuing.

### 4. Configure API keys

Set the following environment variables before running the API setup cell:

```bash
GROQ_API_KEY=your_groq_api_key
PINECONE_API_KEY=your_pinecone_api_key
```

For a local terminal, you can set them in your shell before starting Jupyter. In Google Colab, use Secrets or another secure environment-variable method. **Do not commit API keys to GitHub or place real keys in the notebook.**

The notebook contains placeholder fallback strings. Replace the placeholders or, preferably, set the environment variables securely. Confirm that your Groq account supports the configured text and vision models.

### 5. Check the input PDF path

The notebook initially looks for:

```python
PDF_PATH = Path("NovaCore_Multimodal_Company_Report_2026.pdf")
```

It also includes a fallback path for `/mnt/data`. Adjust this setting if your report is stored elsewhere.

### 6. Run the complete pipeline

Run the cells in order to:

1. Initialize the Groq clients and model settings.
2. Extract PDF text, tables, and embedded visuals.
3. Generate text summaries for visuals.
4. Create embeddings with Hugging Face.
5. Create or connect to the Pinecone index.
6. Add the extracted documents to Pinecone.
7. Create a retriever and the text RAG chain.
8. Run the demonstration questions.

The notebook configures the Pinecone index as `novacore-multimodal-rag` and the namespace as `fy2026-demo`. The index is configured for cosine similarity in the AWS `us-east-1` region when it needs to be created.

## Example questions

The notebook includes examples such as:

- What does NovaCore Systems do, and where is it headquartered?
- Which region had the highest year-over-year revenue growth?
- According to the revenue graph, which quarter had the highest revenue?
- According to the supply-chain diagram, what is the critical quality-control point?
- How did average support resolution time change from January to August?
- What percentage of electricity at the Penang facility came from solar?
- Summarize FY2026 performance using revenue, regional growth, support resolution, and solar share.

These are demonstration questions based on the fictional report, not verified facts about a real company.

## Configuration reference

| Setting | Value in notebook | Meaning |
|---|---|---|
| `TEXT_MODEL` | `openai/gpt-oss-20b` | Text answer model through Groq |
| `VISION_MODEL` | `qwen/qwen3.8-27b` | Vision model through Groq |
| `EMBEDDING_MODEL` | `sentence-transformers/all-MiniLM-L6-v2` | Embedding model |
| `PINECONE_INDEX_NAME` | `novacore-multimodal-rag` | Pinecone index name |
| `PINECONE_NAMESPACE` | `fy2026-demo` | Namespace used for this demo |
| Retriever `k` | `5` | Number of documents retrieved for a query |
| Vision images | Up to `3` | Maximum original images passed to the visual answer helper |

## Important data and cost warning

**The notebook clears the configured Pinecone namespace before inserting documents.** In particular, it calls `index.delete(delete_all=True, namespace=PINECONE_NAMESPACE)`. Use a dedicated demo namespace and make sure it contains no data you need to keep before running that cell. Do not point this code at a production namespace.

Groq API calls may incur usage charges or be subject to account limits. PDF image extraction and visual summarization make API calls for extracted visuals, so review the document and expected usage before processing large PDFs.

## Troubleshooting

- **Missing PDF:** verify `PDF_PATH` and ensure the sample PDF is available in the notebook's working directory.
- **Authentication errors:** check that `GROQ_API_KEY` and `PINECONE_API_KEY` are set correctly and have not expired.
- **Model errors:** confirm the configured model IDs are available to your Groq account and support the requested input type.
- **Pinecone dimension errors:** the index dimension must match the embedding model's output dimension. If you change embedding models, use an appropriately configured index.
- **No visual evidence in an answer:** check whether images were extracted, visual summaries were successfully generated, the relevant visual summary was retrieved, and the saved image path still exists.
- **Table extraction warnings:** PyMuPDF table extraction may not work equally well for every PDF layout. Review extracted documents when table content is important.
- **Unexpected or incomplete answers:** retrieval is based on semantic similarity and the top-five results. Try a more specific question and inspect the retrieved sources and page metadata.

## Limitations

- This is a demonstration notebook, not a production-ready hosted application.
- PDF extraction quality depends on document structure; scanned pages may require OCR, which is not implemented as a separate OCR pipeline here.
- The notebook extracts embedded PDF images. Visuals that are not embedded as images or are rendered in unusual ways may not be captured as expected.
- Visual summaries can omit details or misread charts; verify important outputs against the source PDF.
- The answer is only as reliable as the retrieved context, extracted visuals, and model output. The prompt asks the model to say when information is missing, but this is not a guarantee against hallucinations.
- The local `novacore_extracted_images` directory must remain available for image paths stored in retrieved metadata to work.

## Suggested repository structure

```text
NovaCore-Multimodal-RAG/
├── README.md
├── Build_Multimodal_RAG_NovaCore.ipynb
├── NovaCore_Multimodal_Company_Report_2026.pdf
├── Multimodal_RAG_LangChain_Pinecone_With_Examples.pptx.pdf
└── novacore_extracted_images/       # created during notebook execution
```

## Future improvements

- Add a user interface (for example, Streamlit) for asking questions.
- Add OCR for scanned PDFs and improve extraction of complex tables.
- Add automated evaluation questions and retrieval/answer quality metrics.
- Add persistent file management and safer namespace controls.
- Add logging, retries, and structured error handling for API and extraction failures.
- Add tests and a reproducible dependency lock file.

## License and attribution

No license is specified in the supplied project files. Add a license before distributing or reusing the project publicly. The NovaCore company report is identified in the source document as fictional demonstration data.
