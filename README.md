# Firecrawl with SearXNG Integration

A self-hosted version of Firecrawl using SearXNG as the search backend.

## Quick Start

1. Clone the repository:
   ```bash
   git clone git@github.com:Ozamatash/localfirecrawl.git
   cd localfirecrawl
   ```

2. Set up environment variables:
   - Copy `.env.example` to `.env`
   - Generate a secure secret key for SearXNG:
     ```bash
     openssl rand -hex 32  # Copy the output to SEARXNG_SECRET_KEY in .env
     ```
   - Update other values in `.env` as needed

3. Build and start the services:
   ```bash
   # Build the Docker images
   docker-compose build

   # Start the services
   docker-compose up -d
   ```

## Services

- SearXNG: http://localhost:8081
- Firecrawl API: http://localhost:3002
- Playwright Service: http://localhost:3000

Test the search endpoint:
```bash
curl -X POST -H "Content-Type: application/json" -d '{"query": "test", "limit": 10}' http://localhost:3002/v1/search
```

## Key Environment Variables

- `OPENAI_API_KEY` (Optional): For LLM-dependent features
- `SEARXNG_ENGINES`: Comma-separated list of search engines (e.g., google,brave,duckduckgo,wikipedia)
- `SEARXNG_CATEGORIES`: Categories to search in (e.g., general)
- `SEARXNG_SECRET_KEY`: Required secure key for SearXNG (generate with `openssl rand -hex 32`)
- `PORT`: API service port (default: 3002)
