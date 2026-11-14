Product Architecture
--------------------
Detailed AI & System Architecture

To handle unstructured PDF crash sheets, dynamic OEM brochures, insurance matrices, and geospatial dealer lookups across 40 countries, the system requires a Hybrid Agentic RAG + Text-to-SQL Architecture.

Below three primary layers has been identified,
-----------------------------------------------
1. Data Ingestion & ETL Layer (Multi-Source Pipelines)
Hybrid Storage Layer
Agentic Orchestration Layer (LangGraph / LlamaIndex)

- Frontend-to-Backend Integration (safercar.html)
- Anti-Hallucination & Safety Guardrails

Data Ingestion & ETL Layer (Multi-Source Pipelines)
---------------------------------------------------
NCAP & Regulatory Scrapers: Automated Playwright/Scrapy workers and PDF parsers (LlamaParse / Unstructured.io) scheduled via Apache Airflow to extract structured tables (Adult Occupant, Child Occupant, Pedestrian Safety, Safety Assist scores, bodyshell integrity) from all 11 NCAP/IIHS/NHTSA portals.
OEM & Feature Scrapers: Headless browser agents targeting OEM websites (e.g., auto.mahindra.com) to ingest trim-level specifications, engine metrics, and ADAS feature lists. An LLM-based standardization step maps brand-specific marketing terms into a unified taxonomy (e.g., Level 2 ADAS -> Autonomous Emergency Braking, Lane Keep Assist, Adaptive Cruise Control).
Insurance Engine Ingestion: API connectors and web scrapers targeting insurers (e.g., icicilombard.com) to extract base rates, IDV (Insured Declared Value) depreciation rules, zero-depreciation add-on costs, and safety-feature discounts.
Geospatial Dealer Ingestion: Crawlers extracting dealer addresses, contact numbers, and WhatsApp business endpoints (e.g., mandovi.in), geocoded via Google Maps API into latitude/longitude coordinates.

Hybrid Storage Layer
--------------------
Relational Database (PostgreSQL + PostGIS): Stores structured vehicle trims, exact numeric crash test scores, insurance premium rate tables, country availability mappings, and geospatial dealer coordinates (PostGIS for nearest-neighbor ST_Distance queries).
Vector Database (Pinecone / Qdrant): Stores chunked, embedded unstructured text from detailed NCAP technical crash commentary, IIHS test notes, and policy terms using dense embeddings (text-embedding-3-large) paired with sparse BM25 vectors for exact model-year matching.
Semantic Cache (Redis): Caches frequent queries (e.g., "Honda Civic 2024 vs Toyota Corolla") to reduce LLM latency and API token costs by 40–60%.

Agentic Orchestration Layer (LangGraph / LlamaIndex)
----------------------------------------------------
When a user submits a query, an Intent Router & Orchestrator Agent decomposes the prompt and invokes specialized deterministic tools in parallel:
Geo-Resolver Middleware: Detects the user's IP/country (across the 40 target countries) and filters the database strictly to car variants and NCAP bodies relevant to that region.
Safety & Spec SQL Tool: Executes deterministic SQL queries to fetch side-by-side numeric crash ratings, ADAS matrices, and general specifications.
RAG Retrieval Tool: Queries the Vector DB with metadata filters (country, make, model, year) to retrieve qualitative crash test findings.
Insurance Calculator Tool: Computes estimated policy premiums based on vehicle ex-showroom price, safety rating discounts, and country-specific coverage tiers.
Dealer Locator & WhatsApp Generator Tool: Queries PostGIS using the user's location to find the closest authorized dealership and constructs an RFC-compliant [https://wa.me/](https://wa.me/) deep link pre-populated with the user's selected car variant.

Frontend-to-Backend Integration (safercar.html)
-----------------------------------------------
The built safercar.html serves as the presentation layer. Below is how each UI component hooks into the backend microservices (built on FastAPI): 
Hero Search Box (.search-box input & button):
Frontend Action: Captures natural language input (e.g., "Honda Civic 2024 vs Toyota Corolla") and triggers an asynchronous POST /api/v1/chat/query request via WebSockets or Server-Sent Events (SSE), passing the query string and browser Geolocation/IP header. 
Backend Action: Streams back a structured JSON payload containing markdown prose alongside structured UI widget schemas (comparison tables, radar charts for safety scores, insurance cards, and dealer CTA buttons) that dynamically render below the .hero section. 
Navigation & Feature Cards (#features, #compare, #insurance, #dealers): 
Global Crash Ratings Card: Connects to GET /api/v1/safety/{country}/{make}/{model}, returning localized NCAP star ratings and test year.
Crash Rating & ADAS / General Feature Comparison Cards (#compare): Connects to POST /api/v1/compare, accepting an array of vehicle_ids and returning a normalized JSON matrix categorizing structural integrity, active ADAS features, and powertrain specs. 
Insurance & Cost Analysis Card (#insurance): Connects to POST /api/v1/insurance/quotes, querying partner tables (like ICICI Lombard) and returning tiered policy cards (Third-Party, Comprehensive, Zero-Dep) with estimated annual premiums. 
Instant Dealer Connect Card (#dealers): Connects to GET

Anti-Hallucination & Safety Guardrails
--------------------------------------
Because automotive safety and insurance pricing carry high consumer trust and liability implications, the system enforces a Zero-Hallucination Defence-in-Depth Architecture:
Deterministic Tool-Use Over Free Generation: The LLM is strictly prohibited from using its pre-trained weights to state crash ratings, ADAS availability, or insurance prices. All numeric values and feature booleans (Available / Not Available) are fetched via SQL and injected directly into UI templates; the LLM only synthesizes the surrounding explanatory summary.
Strict Grounding & System Prompt Constraints: Inference runs at Temperature = 0.0. The system prompt enforces a hard fallback rule: "If a specific vehicle trim, crash test score, or regional variant is not explicitly present in the retrieved database context, state 'Official crash test data is not yet available for the [Country] specification of this model.' Never extrapolate Euro NCAP ratings to Bharat NCAP or Latin NCAP models."
Variant & Country Disambiguation Guardrail: Automakers frequently sell cars with identical names but different structural steel grades and airbag counts across regions. If a user asks a broad query (e.g., "How safe is the Hyundai Creta?"), the input guardrail intercepts the query, verifies the user's country via IP, and forces variant/year confirmation before displaying ratings.
Input & Output Guardrails (NeMo Guardrails / Guardrails AI):
Input Rail: Blocks prompt injections, competitor scraping attempts, and off-topic queries unrelated to automobiles, safety, insurance, or dealerships.
Output Rail (Hallucination Detection): Runs an automated NLI (Natural Language Inference) faithfulness check comparing every generated claim against the retrieved JSON/RAG chunks before streaming to the user, appending clickable source URLs (e.g., direct links to the bncap.in or euroncap.com PDF report).









