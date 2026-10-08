# CatalogueIQ

## A RAG-Powered Product Intelligence Assistant

**Project:** CatalogueIQ  
**Domain:** E-commerce, Product Catalogue and Recommendations  
**Fictional Company:** ShopSmart India  
**Project Status:** Capstone deliverable  
**Proficiency Level:** Level 2, Working Knowledge

---

## 1. Project Overview

CatalogueIQ is a Retrieval-Augmented Generation (RAG) assistant built over ShopSmart India's product and policy knowledge base. It helps shoppers, sellers, and support agents obtain accurate, grounded, and traceable answers from product catalogue records, return policies, seller guides, category documentation, buyer FAQs, and product review summaries.

Unlike a basic chatbot, CatalogueIQ retrieves relevant evidence before generating an answer. Every response is expected to remain within the retrieved context and include citations such as a product ID, source file, or document section.

The project uses **LanceDB** as its vector database. LanceDB stores document embeddings together with useful metadata, enabling semantic retrieval, filtering, full-text search, and hybrid retrieval without requiring a separate database server for the local capstone setup.

---

## 2. Problem Statement

A shopper searching for the best wireless earbuds under a budget may need information distributed across multiple sources:

- Product specifications and price
- Review themes and ratings
- Seller identity and verification details
- Return and refund rules
- Warranty information
- Buyer protection FAQs

CatalogueIQ provides one conversational interface that connects these sources and answers questions such as:

- What is the battery life of the boAt Airdopes 141?
- Which wireless earbuds under ₹3,000 support ANC and provide at least 20 hours of battery life?
- Can a fashion item purchased during a sale be returned?
- What images are required when a seller lists an Electronics product?
- What options does a shopper have if a third-party smartwatch stops working after 45 days?

---

## 3. Supported Query Types

CatalogueIQ is designed to support five core query types:

1. **Product factual lookup**  
   Retrieves an exact product and its required specification.

2. **Policy and eligibility**  
   Retrieves and combines relevant policy clauses and exceptions.

3. **Comparative recommendation**  
   Retrieves multiple eligible products, applies constraints, and compares their attributes.

4. **Seller policy lookup**  
   Locates a specific onboarding, listing, compliance, or image requirement.

5. **Multi-hop reasoning**  
   Connects evidence from several sources, such as returns, warranty, seller accountability, and buyer protection.

---

## 4. Key Features

### Must-Have Features

- Ingestion from CSV, Markdown, and HTML
- Row-level chunking for structured product records
- Section-aware and recursive chunking for unstructured documents
- Two chunking strategies with a comparison report
- Configurable OpenAI or SentenceTransformer embeddings
- LanceDB vector storage and retrieval
- Query expansion with e-commerce synonyms
- Source-aware context formatting and citations
- Grounded generation with an explicit no-hallucination instruction
- Conversation memory for at least five turns
- RAGAS evaluation for faithfulness, answer relevancy, and context precision

### Should-Have Features

- Metadata filters for category, price, rating, source type, and persona
- Persistent embeddings and incremental ingestion
- Streamlit interface for Shopper, Seller, and Support Agent personas
- Retrieved-evidence panel alongside the generated answer

### Stretch Features

- Hybrid search using vector retrieval and BM25 full-text search
- SQLite-backed deterministic filtering for price and rating constraints
- Session-level brand and product preferences
- Hinglish-aware query expansion
- Retrieval tracing and latency reporting

---


## 6. Why LanceDB?

CatalogueIQ uses LanceDB instead of Chroma or FAISS for the primary implementation.

- It can run locally and persist data on disk.
- It stores vectors, text, and metadata in the same table.
- It supports similarity search and metadata filtering.
- It supports full-text and hybrid retrieval for exact product-name queries.
- It provides a practical path from a small local catalogue to a larger deployment.
- It simplifies inspection because retrieved rows can include the original text and all metadata.

For this project, cosine distance is recommended for unnormalized text embeddings. The same distance strategy must be used consistently during indexing and retrieval.

---

## 7. Repository Structure

```text
CatalogueIQ/
├── app.py
├── README.md
├── requirements.txt
├── .env.example
├── config/
│   ├── settings.yaml
│   └── personas.yaml
├── data/
│   ├── raw/
│   │   ├── product_catalog.csv
│   │   ├── product_review_summaries.csv
│   │   ├── returns_refunds_policy.md
│   │   ├── seller_onboarding.md
│   │   ├── category_electronics.md
│   │   ├── category_fashion.md
│   │   └── buyer_faq.html
│   └── processed/
│       └── ingestion_manifest.json
├── catalogueiq/
│   ├── __init__.py
│   ├── loaders.py
│   ├── chunking.py
│   ├── embeddings.py
│   ├── lancedb_store.py
│   ├── query_expansion.py
│   ├── filters.py
│   ├── retriever.py
│   ├── prompts.py
│   ├── memory.py
│   ├── rag_chain.py
│   └── citations.py
├── scripts/
│   ├── generate_sample_data.py
│   ├── ingest.py
│   ├── inspect_retrieval.py
│   └── run_evaluation.py
├── tests/
│   ├── test_loaders.py
│   ├── test_chunking.py
│   ├── test_query_expansion.py
│   ├── test_retrieval.py
│   └── test_grounding.py
├── evaluation/
│   ├── test_set.json
│   ├── baseline_results.json
│   ├── improved_results.json
│   └── Evaluation_Report.md
├── demo/
│   ├── Demo_Script.md
│   └── Panel_QA.md
└── storage/
    └── lancedb/
```

---

## 8. Dataset Requirements

The knowledge base should contain:

- 500 to 1,000 product records across Electronics, Fashion, Home & Kitchen, Beauty, and Books
- At least 200 products across at least three categories for the minimum submission
- A complete returns and refunds policy
- A buyer FAQ in HTML
- At least two category-specific attribute guides
- A seller onboarding guide
- Review summaries for selected products

### Recommended Product Fields

```text
product_id
name
brand
category
subcategory
price
rating
seller_id
seller_verified
specifications
warranty
stock_status
returnable
```

A product record should be converted into readable text while retaining typed metadata for filters.

```text
Product ID: P101
Name: boAt Airdopes 141
Brand: boAt
Category: Electronics
Price: ₹1,499
Battery life: 42 hours
ANC: Yes
Rating: 4.3
Seller ID: S1001
Seller verified: Yes
Warranty: 1 year
```

---

## 9. Chunking Strategy

### Strategy A: Product Row Chunking

Each CSV product row becomes one complete chunk. Product attributes are not split across multiple chunks.

**Why this is used:**

- A product is a natural business entity.
- Price, rating, specifications, seller, and warranty remain together.
- Product citations can use a stable product ID.
- Multiple product chunks can be retrieved for comparison.

### Strategy B: Section-Aware Document Chunking

Markdown headings and HTML sections are preserved where possible. Long sections are divided using a recursive splitter with overlap.

Suggested starting values:

```text
Chunk size: 800 characters
Chunk overlap: 120 characters
```

These values are starting points and must be validated through retrieval inspection and RAGAS evaluation.

### Alternative to Compare

Run a fixed-size chunking experiment against the section-aware strategy. Record:

- Number of generated chunks
- Whether headings remain attached to rules
- Whether exceptions are separated from their conditions
- Retrieval quality for policy and multi-hop questions
- Context precision before and after the selected strategy

---

## 10. LanceDB Schema

A single `knowledge_chunks` table can support the initial implementation.

```text
id                string
text              string
vector            fixed-size float vector
source             string
source_type        string
section            string
chunk_type         string
product_id         string or null
product_name       string or null
brand              string or null
category           string or null
price              float or null
rating             float or null
seller_id          string or null
seller_verified    boolean or null
persona_scope      string
content_hash       string
```

Recommended chunk types:

```text
product
review
policy
faq
seller_guide
category_guide
```

`content_hash` is used to detect unchanged chunks and avoid unnecessary re-embedding.

---

## 11. Query Processing and Retrieval

### Query Expansion

Every user query is rewritten into at least two retrieval variants. Expansion can combine deterministic e-commerce synonyms with an LLM-based rewrite.

Example:

```text
Original: best earphones under 2000
Variant 1: best wireless earbuds below ₹2,000
Variant 2: top-rated TWS earphones under ₹2,000
```

Suggested synonym groups:

```text
earphones -> earbuds, TWS, in-ear headphones
mobile -> smartphone, phone, handset
fridge -> refrigerator
washing machine -> washer
return -> replacement, refund, return eligibility
original -> authentic, genuine
```

### Constraint Extraction

Before vector search, the query analyser should identify structured constraints such as:

```text
category = Electronics
subcategory = Wireless Earbuds
price <= 3000
battery_life_hours >= 20
anc = true
rating >= 4.0
```

Use deterministic metadata or SQL filtering for numeric constraints. Use vector or hybrid search to rank semantically relevant candidates.

### Retrieval Depth

Recommended starting values:

```text
Factual lookup: top_k = 4
Policy lookup: top_k = 6
Comparative recommendation: top_k = 10 to 15
Multi-hop question: top_k = 8 to 12
```

A comparative query must retrieve multiple product chunks. Returning only one product is considered a retrieval failure even if that product is relevant.

### Hybrid Search

Hybrid retrieval is recommended for:

- Exact product names
- Product IDs
- Brand and model numbers
- Policy phrases
- Semantic queries with colloquial terms

The final ranking may combine vector similarity, BM25 relevance, metadata eligibility, and an optional reranker.

---

## 12. Citation Format

Retrieved context must be formatted before generation.

```text
[Source: product_catalog.csv | Product ID: P101]
Name: boAt Airdopes 141
Price: ₹1,499
Battery life: 42 hours
ANC: Yes

[Source: returns_refunds_policy.md | Section: Electronics Returns]
Eligible Electronics products may be returned within the stated return window,
subject to condition and category exceptions.
```

Expected answer style:

```text
The boAt Airdopes 141 provides up to 42 hours of battery life and supports ANC.
[Source: product_catalog.csv | Product ID: P101]
```

If the retrieved context is insufficient, return:

```text
I could not find enough information in the ShopSmart India knowledge base to answer that confidently.
```

---

## 13. Persona Behaviour

### Shopper

- Focus on product discovery, comparison, delivery, returns, warranty, and authenticity.
- Explain recommendations using retrieved evidence.
- Do not recommend products that fail mandatory constraints.

### Seller

- Focus on onboarding, product listing, images, pricing, compliance, and seller responsibilities.
- Prefer seller and category-guide sources.
- Do not invent marketplace requirements.

### Support Agent

- Combine product, policy, FAQ, seller, warranty, and buyer-protection evidence.
- Clearly separate confirmed policy from missing information.
- Provide traceable citations for operational answers.

---

## 14. Conversation Memory

The application must support at least five connected turns.

```text
User: Show wireless earbuds under ₹2,000.
User: Only include models with ANC.
User: Which one has the best battery life?
User: Compare the top three.
User: Exclude boAt and recommend again.
```

Memory should retain conversation constraints and preferences, but retrieved evidence must still be refreshed for each turn. Memory must not be treated as a trusted product or policy source.

---

## 15. Installation

### Prerequisites

- Python 3.11 or later
- Git
- An OpenAI API key when using OpenAI embeddings or generation

### Clone and Enter the Project

```bash
git clone <your-repository-url>
cd CatalogueIQ
```

### Create a Virtual Environment

```bash
python -m venv .venv
```

Linux or macOS:

```bash
source .venv/bin/activate
```

Windows PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

### Install Dependencies

```bash
pip install -r requirements.txt
```

Suggested dependencies:

```text
lancedb
pyarrow
pandas
sentence-transformers
openai
python-dotenv
streamlit
beautifulsoup4
markdown
pydantic
pyyaml
ragas
datasets
pytest
rank-bm25
```

Pin tested versions in `requirements.txt` before final submission.

---

## 16. Environment Configuration

Copy the example environment file:

```bash
cp .env.example .env
```

Example `.env`:

```env
LLM_PROVIDER=openai
OPENAI_API_KEY=replace_with_your_key
CHAT_MODEL=your_supported_chat_model
EMBEDDING_PROVIDER=sentence_transformers
EMBEDDING_MODEL=all-MiniLM-L6-v2
LANCEDB_URI=./storage/lancedb
LANCEDB_TABLE=knowledge_chunks
RETRIEVAL_TOP_K=8
ENABLE_HYBRID_SEARCH=true
```

Never commit `.env` or API keys to source control.

---

## 17. Running the Project

### Step 1: Generate or Validate the Dataset

```bash
python scripts/generate_sample_data.py
```

### Step 2: Ingest and Index the Knowledge Base

```bash
python scripts/ingest.py
```

Expected ingestion summary:

```text
Products loaded: <count>
Policy documents loaded: <count>
FAQ entries loaded: <count>
Total chunks: <count>
New embeddings created: <count>
Cached chunks reused: <count>
LanceDB table: knowledge_chunks
```

### Step 3: Inspect Retrieval Before Generation

```bash
python scripts/inspect_retrieval.py --query "best earbuds under 2000 rupees"
```

Confirm that:

- Multiple eligible products are returned.
- Price filters are respected.
- Product IDs and source metadata are present.
- Policy questions retrieve policy sections rather than unrelated products.

### Step 4: Start the Streamlit App

```bash
streamlit run app.py
```

### Step 5: Run Tests

```bash
pytest -q
```

### Step 6: Run RAGAS Evaluation

```bash
python scripts/run_evaluation.py
```


## 21. Future Enhancements

- Incremental catalogue updates using content hashes
- Scheduled policy re-indexing
- Multilingual English, Hindi, and Hinglish queries
- Cross-encoder reranking
- Product image search
- Feedback-based retrieval evaluation
- Role-based access for internal support data
- Observability for latency, token usage, retrieval quality, and citation coverage
- Cloud or object-storage-backed LanceDB deployment

---

## 25. References

- LanceDB documentation: https://docs.lancedb.com/
- LanceDB Python API: https://lancedb.github.io/lancedb/python/python/
- RAGAS documentation: https://docs.ragas.io/
- Streamlit documentation: https://docs.streamlit.io/
- Sentence Transformers documentation: https://www.sbert.net/

---

## License

This project is intended for educational and capstone demonstration purposes. Add an appropriate license before public distribution.
