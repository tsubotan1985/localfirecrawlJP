# Firecrawl with SearXNG Integration

A self-hosted version of Firecrawl using SearXNG as the search backend, optimized for deep research and multilingual support including Japanese.

## 🚀 Quick Start

### Prerequisites
- Docker and Docker Compose installed
- 8GB+ RAM recommended (for Playwright Chromium build)

### 1. Clone Repository
```bash
git clone https://github.com/tsubotan1985/localfirecrawlJP.git
cd localfirecrawlJP
```

### 2. Environment Setup
Copy `.env.example` to `.env` and configure:

```bash
# LMStudio API Configuration (Recommended)
OPENAI_API_BASE=http://localhost:1234/v1
OPENAI_API_KEY=lm-studio
OPENAI_MODEL_NAME=glm-4.5-air #Think model

# SearXNG Secret Key (Required)
SEARXNG_SECRET_KEY=generated-key

# Port Configuration
PORT=3002
```

**Generate SearXNG Secret Key:**
```bash
# Using Node.js
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"

# Using OpenSSL
openssl rand -hex 32
```

### 3. SearXNG Configuration (Multilingual Support)
Verify important settings in [`settings.yml`](settings.yml):

```yaml
general:
  default_lang: "ja"  # Default to Japanese
  
ui:
  default_locale: "ja"
  
search:
  languages: ["ja", "ja-JP", "en-US", "en", "auto"]
  
engines:
  # Bing engine for strong Japanese search support
  - name: bing
    engine: bing
    categories: general
```

### 4. Port Configuration
Verify port settings in [`docker-compose.yml`](docker-compose.yml):

```yaml
searxng:
  ports:
    - "8088:8080"  # SearXNG Web UI

api:
  ports:
    - "3100:3002"  # Firecrawl API

playwright-service:
  ports:
    - "3111:3000"  # Playwright Service
```

### 5. Build and Launch
```bash
# Build Docker images (initial build ~13-15 minutes)
docker-compose build

# Start services
docker-compose up -d

# Check service status
docker-compose ps
```

## 🌐 Service Endpoints

### Main Services
- **SearXNG Web UI**: http://localhost:8088
- **SearXNG API**: http://localhost:8088/search
- **Firecrawl API**: http://localhost:3100
- **Playwright Service**: http://localhost:3111

### Health Checks
```bash
# SearXNG search test
curl "http://localhost:8088/search?q=test&format=json"

# Firecrawl API test
curl -X POST "http://localhost:3100/v0/scrape" \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com"}'

# Playwright service test
curl "http://localhost:3111/health"
```

## 🔍 Multilingual Search Usage

### Web Interface
1. Access http://localhost:8088 in browser
2. Enter search terms directly in any language (e.g., "artificial intelligence" or "人工知能")
3. Get integrated results from multiple search engines

### API Search with Japanese
Japanese queries require URL encoding:

```bash
# Japanese search example: "坪田陽一"
# URL encoded: %E5%9D%AA%E7%94%B0%E9%99%BD%E4%B8%80
curl "http://localhost:8088/search?q=%E5%9D%AA%E7%94%B0%E9%99%BD%E4%B8%80&format=json"

# Python example
import urllib.parse
import requests

query = "researcher-name"
encoded_query = urllib.parse.quote(query)
response = requests.get(f"http://localhost:8088/search?q={encoded_query}&format=json")
```

## 🛠️ Deep Research Configuration

### 1. Enhanced Search Engine Setup
Configure academic and specialized search in `settings.yml`:

```yaml
engines:
  # Academic Search
  - name: google_scholar
    engine: google_scholar
    categories: science
    
  - name: arxiv
    engine: arxiv
    categories: science
    
  - name: wikipedia_ja
    engine: wikipedia
    base_url: 'https://ja.wikipedia.org/'
    categories: general
    
  # Developer Research
  - name: github
    engine: github
    categories: it
```

### 2. Category-based Search
```bash
# Academic paper search
curl "http://localhost:8088/search?q=machine+learning&categories=science&format=json"

# IT technology search
curl "http://localhost:8088/search?q=docker&categories=it&format=json"

# General search
curl "http://localhost:8088/search?q=research&categories=general&format=json"
```

### 3. Firecrawl Integration Workflow
Search → Detailed Analysis workflow:

```bash
# Step 1: Search for relevant information
curl "http://localhost:8088/search?q=artificial+intelligence&format=json" \
  | jq '.results[].url' > urls.txt

# Step 2: Detailed scraping of found URLs
curl -X POST "http://localhost:3100/v0/scrape" \
  -H "Content-Type: application/json" \
  -d '{"url": "found-url", "formats": ["markdown", "html"]}'
```

## ⚙️ Advanced Configuration

### LMStudio Integration
Configuration for LLM-assisted features:

```bash
# .env file
OPENAI_API_BASE=http://localhost:1234/v1
OPENAI_API_KEY=lm-studio
OPENAI_MODEL_NAME=your-model-name

# API test
curl -X POST "http://localhost:3100/v0/scrape" \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com", "formats": ["llm-extraction"]}'
```

### Proxy and Security Settings
Network configuration adjustments in `settings.yml`:

```yaml
server:
  # Security headers
  secret_key: "your-secret-key"
  
outgoing:
  # Proxy settings (if needed)
  proxies:
    http: "http://proxy:8080"
    https: "https://proxy:8080"
```

### Performance Optimization
```yaml
# Search result count adjustment
search:
  default_http_headers:
    User-Agent: "SearXNG/1.0"
  
# Timeout settings
engines:
  - name: google
    timeout: 10.0
    
# Cache settings
redis:
  url: redis://redis:6379/0
```

## 🧪 Real Research Workflow Examples

### Example 1: Researcher Deep Investigation
```bash
# 1. Search researcher by name
RESEARCHER="researcher-name"
ENCODED=$(python3 -c "import urllib.parse; print(urllib.parse.quote('$RESEARCHER'))")
curl "http://localhost:8088/search?q=$ENCODED&format=json" > search_results.json

# 2. Identify and extract research institution page
jq -r '.results[] | select(.url | contains("researchmap")) | .url' search_results.json | head -1 > target_url.txt

# 3. Detailed information scraping
curl -X POST "http://localhost:3100/v0/scrape" \
  -H "Content-Type: application/json" \
  -d "{\"url\": \"$(cat target_url.txt)\"}" > researcher_data.json

# 4. Get paper list as well
jq -r '.results[] | select(.url | contains("scholar")) | .url' search_results.json | head -5 | \
while read url; do
  curl -X POST "http://localhost:3100/v0/scrape" \
    -H "Content-Type: application/json" \
    -d "{\"url\": \"$url\"}" >> papers_data.json
done
```

### Example 2: Technology Trend Analysis
```bash
# Research latest AI technology trends
curl "http://localhost:8088/search?q=GPT-4&categories=science,it&format=json" | \
jq '.results[] | select(.score > 0.8) | {title, url, snippet}' > ai_trends.json

# Detailed analysis of important pages
jq -r '.url' ai_trends.json | head -10 | \
while read url; do
  curl -X POST "http://localhost:3100/v0/scrape" \
    -H "Content-Type: application/json" \
    -d "{\"url\": \"$url\", \"formats\": [\"markdown\"]}" \
    >> tech_analysis.json
done
```

## 📊 Monitoring and Maintenance

### Log Checking
```bash
# All services logs
docker-compose logs

# Specific service logs
docker-compose logs searxng
docker-compose logs api
docker-compose logs playwright-service

# Real-time logs
docker-compose logs -f searxng
```

### Performance Monitoring
```bash
# Container resource usage
docker stats

# Service response time measurement
time curl "http://localhost:8088/search?q=test&format=json" > /dev/null
time curl -X POST "http://localhost:3100/v0/scrape" \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com"}' > /dev/null
```

### Applying Configuration Changes
```bash
# After SearXNG configuration changes
docker-compose restart searxng

# After environment variable changes
docker-compose down
docker-compose up -d

# Complete rebuild
docker-compose down
docker-compose build --no-cache
docker-compose up -d
```

## 🔧 Troubleshooting

### Common Issues and Solutions

#### 1. SearXNG Forbidden Error
```bash
# Check language settings
grep -A 5 "default_lang" settings.yml

# Fix: In settings.yml
# default_lang: "ja"
# languages: ["ja", "ja-JP", "en-US", "en", "auto"]
```

#### 2. No Japanese Search Results
```bash
# Check URL encoding
python3 -c "import urllib.parse; print(urllib.parse.quote('search-term'))"

# Verify Bing engine is enabled
grep -A 10 "name: bing" settings.yml
```

#### 3. Firecrawl API Not Responding
```bash
# Check service status
docker-compose ps api

# Check logs
docker-compose logs api

# Restart
docker-compose restart api
```

#### 4. Memory Issues
```bash
# Current memory usage
docker stats --no-stream

# Clean up unused containers
docker system prune

# Create swap file (Linux)
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

## 📚 API Reference

### SearXNG API
```bash
# Basic search
GET /search?q={query}&format=json

# Category-specific search
GET /search?q={query}&categories=science&format=json

# Language-specific search
GET /search?q={query}&lang=ja&format=json

# Engine-specific search
GET /search?q={query}&engines=google,bing&format=json
```

### Firecrawl API
```bash
# Basic scraping
POST /v0/scrape
{
  "url": "https://example.com"
}

# Format specification
POST /v0/scrape
{
  "url": "https://example.com",
  "formats": ["markdown", "html", "structured"]
}

# Batch scraping
POST /v0/batch-scrape
{
  "urls": ["https://example1.com", "https://example2.com"],
  "formats": ["markdown"]
}
```

## 🌏 Language Support

### Supported Languages
- **Japanese (ja, ja-JP)**: Full support with optimized search engines
- **English (en, en-US)**: Complete support
- **Auto-detection**: Automatic language detection for mixed queries

### Language-specific Features
- **Japanese Wikipedia**: Direct access to ja.wikipedia.org
- **Bing Engine**: Enhanced Japanese search capabilities
- **Unicode Support**: Full UTF-8 encoding for all Asian languages
- **Mixed Content**: Support for multilingual content in single documents

## 🤝 Contributing and Support

### Bug Reports
When reporting issues, please include:
- OS information and Docker version
- Error logs (`docker-compose logs` output)
- Steps to reproduce

### Feature Improvements
- Adding new search engines
- Enhancing multilingual support
- Performance optimizations

---

**For Japanese documentation, see [README_ja.md](README_ja.md). このREADMEの日本語版は[README_ja.md](README_ja.md)をご覧ください。**
