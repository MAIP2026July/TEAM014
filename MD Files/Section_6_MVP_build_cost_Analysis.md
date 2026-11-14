MVP Build Cost Analysis
-----------------------

User Base: 20,000 active users spread across 40 countries.
Engagement Volume: Assuming ~15 sessions per user/year and ~5 AI interactions per session = 1,500,000 total AI queries/year (~125,000 queries/month or ~4,100 queries/day).
Cache Hit Ratio: 40% via Redis Semantic Caching, leaving 75,000 net LLM API calls/month.
Token Footprint per Call: ~2,500 input tokens (system prompt + retrieved NCAP/OEM/Insurance JSON context) + ~500 output tokens. Total monthly volume: ~187.5M input tokens and ~37.5M output tokens.

Global passenger car sales in Y2025: 71 Millions
Reasonably assumed 20,000 users(0.03%) as a base condition for MVP


Year 1 Cost Breakdown (CapEx + 12-Month OpEx)
======================

1. AI MODELS & GUARDRAILS
-------------------------
Technical Specification:
Hybrid routing (80% Gemini 2.5 Flash / GPT-4o-mini;
20% Claude 3.5 Sonnet / GPT-4o), Embedding API and Guardrails.

Monthly Cost:       $180
Annual Cost:        $2,160


2. CLOUD HOSTING & DATABASES
----------------------------
Technical Specification:
AWS/GCP hosting with:
- 2x ECS/Cloud Run containers
- Managed PostgreSQL + PostGIS
- Redis Cache
- S3 Storage
- Cloudflare CDN

Monthly Cost:       $450
Annual Cost:        $5,400


3. VECTOR DB & SCRAPING INFRASTRUCTURE
--------------------------------------
Technical Specification:
Qdrant Cloud / Pinecone Starter with rotating
residential proxies (BrightData/ScraperAPI) for 40 countries.

Monthly Cost:       $320
Annual Cost:        $3,840


4. THIRD-PARTY APIs
-------------------
Technical Specification:
- Google Maps Geocoding API (Dealer Search)
- WhatsApp Business API / Link Routing
- Domain & SSL (safercar.com)

Monthly Cost:       $100
Annual Cost:        $1,200


5. MVP DEVELOPMENT (CAPEX)
--------------------------
Technical Specification:
4-Month Lean Development Team:
- 1 Full-Stack/AI Engineer
- 1 Data/Scraping Engineer
- 1 Frontend Developer
- 0.5 QA/Project Manager

Development Location: India / Offshore Rates

Cost Type:          One-Time
One-Time Cost:      $28,000


6. ANNUAL MAINTENANCE (OPEX)
----------------------------
Technical Specification:
- Scraper selector fixes when OEMs/NCAPs update UI
- Quarterly data audits
- Server patching
- Part-time DevOps/Data Engineer

Monthly Cost:       $1,000
Annual Cost:        $12,000


MVP Build Cost Summary
----------------------

Initial MVP Build Cost (CapEx): ~$28,000 (or ~$15,000–$18,000 if leveraging an internal engineering team).
Total Year 1 Operating Cost (AI + Cloud + Scraping + Maintenance OpEx): ~$24,600 / year (~$2,050 / month).
Total Year 1 Investment (CapEx + OpEx): ~$52,600 USD (approx. ₹44–45 Lakhs INR).
Unit Economics: Excluding initial build CapEx, serving 20,000 users costs roughly $1.23 per user per year in total infrastructure, AI tokens, and maintenance. 
These expenses are estimated at 24,600 US dollars for the first year.

