# Firecrawl with SearXNG統合環境 (日本語版)

SearXNGを検索バックエンドとして使用するFirecrawlのセルフホスト版です。日本語検索と深度リサーチに最適化されています。

## 🚀 クイックスタート

### 前提条件
- Docker及びDocker Composeがインストールされていること
- 8GB以上のRAM推奨（Playwright Chromiumビルドのため）

### 1. リポジトリのクローン
```bash
git clone git@github.com:Ozamatash/localfirecrawl.git
cd localfirecrawl
```

### 2. 環境変数の設定
`.env.example`をコピーして`.env`を作成し、以下の値を設定：

```bash
# LMStudio API設定（推奨）
OPENAI_API_BASE=http://133.53.17.85:1234/v1
OPENAI_API_KEY=lm-studio
OPENAI_MODEL_NAME=glm-4.5-air #Think modelです

# SearXNGシークレットキー（必須）
SEARXNG_SECRET_KEY=生成されたキー

# ポート設定
PORT=3002
```

**SearXNGシークレットキーの生成：**
```bash
# Node.js使用
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"

# またはOpenSSL使用
openssl rand -hex 32
```

### 3. SearXNG設定の最適化（日本語対応）
[`settings.yml`](settings.yml)で以下の重要設定を確認：

```yaml
general:
  default_lang: "ja"  # 日本語をデフォルトに
  
ui:
  default_locale: "ja"
  
search:
  languages: ["ja", "ja-JP", "en-US", "en", "auto"]
  
engines:
  # 日本語検索に強いBingエンジンが含まれていることを確認
  - name: bing
    engine: bing
    categories: general
```

### 4. ポート設定の変更
[`docker-compose.yml`](docker-compose.yml)で以下のポート設定を確認：

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

### 5. 環境の構築と起動
```bash
# Dockerイメージのビルド（初回約13-15分）
docker-compose build

# サービスの起動
docker-compose up -d

# サービス状態の確認
docker-compose ps
```

## 🌐 サービスエンドポイント

### 主要サービス
- **SearXNG Web UI**: http://localhost:8088
- **SearXNG API**: http://localhost:8088/search
- **Firecrawl API**: http://localhost:3100
- **Playwright Service**: http://localhost:3111

### 動作確認
```bash
# SearXNG検索テスト
curl "http://localhost:8088/search?q=test&format=json"

# Firecrawl APIテスト
curl -X POST "http://localhost:3100/v0/scrape" \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com"}'

# Playwrightサービステスト
curl "http://localhost:3111/health"
```

## 🔍 日本語検索の使用方法

### Webインターフェース使用
1. ブラウザで http://localhost:8088 にアクセス
2. 検索ボックスに日本語を直接入力（例：「人工知能」）
3. 多様な検索エンジンから統合結果を取得

### API経由での日本語検索
日本語クエリにはURLエンコードが必要です：

```bash
# 日本語検索例：「坪田陽一」
# URLエンコード: %E5%9D%AA%E7%94%B0%E9%99%BD%E4%B8%80
curl "http://localhost:8088/search?q=%E5%9D%AA%E7%94%B0%E9%99%BD%E4%B8%80&format=json"

# Python使用例
import urllib.parse
import requests

query = "坪田陽一"
encoded_query = urllib.parse.quote(query)
response = requests.get(f"http://localhost:8088/search?q={encoded_query}&format=json")
```

## 🛠️ Deep Research設定方法

### 1. 検索エンジンの拡張設定
`settings.yml`で学術・専門検索を強化：

```yaml
engines:
  # 学術検索
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
    
  # 開発者リサーチ
  - name: github
    engine: github
    categories: it
```

### 2. カテゴリ別検索の活用
```bash
# 学術論文検索
curl "http://localhost:8088/search?q=machine+learning&categories=science&format=json"

# IT技術検索
curl "http://localhost:8088/search?q=docker&categories=it&format=json"

# 総合検索
curl "http://localhost:8088/search?q=研究&categories=general&format=json"
```

### 3. Firecrawl統合ワークフロー
検索→詳細分析のワークフロー：

```bash
# ステップ1: 関連情報を検索
curl "http://localhost:8088/search?q=%E4%BA%BA%E5%B7%A5%E7%9F%A5%E8%83%BD&format=json" \
  | jq '.results[].url' > urls.txt

# ステップ2: 見つかったURLを詳細スクレイピング
curl -X POST "http://localhost:3100/v0/scrape" \
  -H "Content-Type: application/json" \
  -d '{"url": "見つかったURL", "formats": ["markdown", "html"]}'
```

## ⚙️ 高度な設定

### LMStudio統合
LLM支援機能のための設定：

```bash
# .envファイル
OPENAI_API_BASE=http://133.53.17.85:1234/v1
OPENAI_API_KEY=lm-studio
OPENAI_MODEL_NAME=glm-4-5-air(Think modelです)

# APIテスト
curl -X POST "http://localhost:3100/v0/scrape" \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com", "formats": ["llm-extraction"]}'
```

### プロキシとセキュリティ設定
`settings.yml`でネットワーク設定の調整：

```yaml
server:
  # セキュリティヘッダー
  secret_key: "your-secret-key"
  
outgoing:
  # プロキシ設定（必要に応じて）
  proxies:
    http: "http://proxy:8080"
    https: "https://proxy:8080"
```

### パフォーマンス最適化
```yaml
# 検索結果数の調整
search:
  default_http_headers:
    User-Agent: "SearXNG/1.0"
  
# タイムアウト設定
engines:
  - name: google
    timeout: 10.0
    
# キャッシュ設定
redis:
  url: redis://redis:6379/0
```

## 🧪 実際の研究ワークフロー例

### 例1: 研究者情報の深度調査
```bash
# 1. 研究者名で検索
RESEARCHER="坪田陽一"
ENCODED=$(python3 -c "import urllib.parse; print(urllib.parse.quote('$RESEARCHER'))")
curl "http://localhost:8088/search?q=$ENCODED&format=json" > search_results.json

# 2. 研究機関ページを特定・抽出
jq -r '.results[] | select(.url | contains("researchmap")) | .url' search_results.json | head -1 > target_url.txt

# 3. 詳細情報をスクレイピング
curl -X POST "http://localhost:3100/v0/scrape" \
  -H "Content-Type: application/json" \
  -d "{\"url\": \"$(cat target_url.txt)\"}" > researcher_data.json

# 4. 論文リストも取得
jq -r '.results[] | select(.url | contains("scholar")) | .url' search_results.json | head -5 | \
while read url; do
  curl -X POST "http://localhost:3100/v0/scrape" \
    -H "Content-Type: application/json" \
    -d "{\"url\": \"$url\"}" >> papers_data.json
done
```

### 例2: 技術動向調査
```bash
# 最新のAI技術動向を調査
curl "http://localhost:8088/search?q=GPT-4&categories=science,it&format=json" | \
jq '.results[] | select(.score > 0.8) | {title, url, snippet}' > ai_trends.json

# 重要なページを詳細分析
jq -r '.url' ai_trends.json | head -10 | \
while read url; do
  curl -X POST "http://localhost:3100/v0/scrape" \
    -H "Content-Type: application/json" \
    -d "{\"url\": \"$url\", \"formats\": [\"markdown\"]}" \
    >> tech_analysis.json
done
```

## 📊 モニタリングとメンテナンス

### ログ確認
```bash
# 全サービスのログ
docker-compose logs

# 特定サービスのログ
docker-compose logs searxng
docker-compose logs api
docker-compose logs playwright-service

# リアルタイムログ
docker-compose logs -f searxng
```

### パフォーマンス監視
```bash
# コンテナリソース使用状況
docker stats

# サービス応答時間測定
time curl "http://localhost:8088/search?q=test&format=json" > /dev/null
time curl -X POST "http://localhost:3100/v0/scrape" \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com"}' > /dev/null
```

### 設定変更の適用
```bash
# SearXNG設定変更後
docker-compose restart searxng

# 環境変数変更後
docker-compose down
docker-compose up -d

# 完全な再ビルド
docker-compose down
docker-compose build --no-cache
docker-compose up -d
```

## 🔧 トラブルシューティング

### よくある問題と解決方法

#### 1. SearXNG Forbiddenエラー
```bash
# 言語設定を確認
grep -A 5 "default_lang" settings.yml

# 修正方法: settings.ymlで
# default_lang: "ja"
# languages: ["ja", "ja-JP", "en-US", "en", "auto"]
```

#### 2. 日本語検索で結果が出ない
```bash
# URLエンコードを確認
python3 -c "import urllib.parse; print(urllib.parse.quote('検索語'))"

# Bingエンジンが有効か確認
grep -A 10 "name: bing" settings.yml
```

#### 3. Firecrawl APIが応答しない
```bash
# サービス状態確認
docker-compose ps api

# ログ確認
docker-compose logs api

# 再起動
docker-compose restart api
```

#### 4. メモリ不足
```bash
# 現在のメモリ使用量
docker stats --no-stream

# 不要なコンテナ削除
docker system prune

# スワップファイル作成（Linux）
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

## 📚 API リファレンス

### SearXNG API
```bash
# 基本検索
GET /search?q={query}&format=json

# カテゴリ指定検索
GET /search?q={query}&categories=science&format=json

# 言語指定検索
GET /search?q={query}&lang=ja&format=json

# エンジン指定検索
GET /search?q={query}&engines=google,bing&format=json
```

### Firecrawl API
```bash
# 基本スクレイピング
POST /v0/scrape
{
  "url": "https://example.com"
}

# フォーマット指定
POST /v0/scrape
{
  "url": "https://example.com",
  "formats": ["markdown", "html", "structured"]
}

# バッチスクレイピング
POST /v0/batch-scrape
{
  "urls": ["https://example1.com", "https://example2.com"],
  "formats": ["markdown"]
}
```

## 🤝 貢献とサポート

### バグ報告
問題が発生した場合は、以下の情報と共にIssueを作成してください：
- OS情報とDockerバージョン
- エラーログ（`docker-compose logs`の出力）
- 再現手順

### 機能改善
- 新しい検索エンジンの追加
- 多言語対応の改善
- パフォーマンス最適化

---

**このREADMEは日本語環境での使用に最適化されています。英語版は[README.md](README.md)をご覧ください。**