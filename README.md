# MCP-RAG-Server
In the era of rapidly evolving AI applications, the need for systems that can combine static domain knowledge with real-time web information has never been greater. Traditional retrieval-augmented generation (RAG) models often rely on pre-indexed data, limiting their responsiveness to new developments. The MCP-RAG Server bridges this gap by integrating semantic vector search (via Qdrant) with real-time web search capabilities (via Scrapeless), offering a production-ready foundation for intelligent question answering systems. Whether you're an enterprise building internal knowledge agents or a developer experimenting with LLM integration, this guide will walk you through the complete setup and usage of MCP-RAG—ensuring you're equipped to deploy a modern, responsive AI knowledge system.

## What is MCP-RAG Server?

The MCP-RAG Server is a TypeScript-based system that combines vector search capabilities with real-time web search to create an enhanced AI knowledge system. It provides three main tools:

1. Machine Learning FAQ Retrieval - Semantic search through your vector database
2. Document Addition - Expand your knowledge base with new information
3. Web Search - Fetch current information from the internet

This system solves critical AI limitations: outdated knowledge, lack of domain expertise, and inefficient information retrieval.

## Scrapeless Introduction: RAG's web intelligent enhancement engine

[Scrapeless Google Search Scraping API](https://www.scrapeless.com/en/product/scraping-api?utm_source=official&utm_medium=blog&utm_campaign=mcprag) is a powerful web scraping API that provides stable access to search engine results without the risk of being blocked by traditional crawlers.

**Why Scrapeless is Essential for RAG Systems**

Traditional RAG systems are limited by their static knowledge bases. Scrapeless transforms the MCP-RAG Server by:

1. Real-time information retrieval: Access to the latest information from the web
2. Knowledge base enhancement: Continuously update your vector database with current data
3. Complementary search: Fill gaps when internal knowledge is insufficient
4. Diverse perspectives: Search from different geographic regions and languages

## How does Scrapeless work in MCP-RAG?

Scrapeless integrates with MCP-RAG Server through a TypeScript encapsulation class ScrapelessClient to achieve the following capabilities:

```
export class ScrapelessClient {
  private api: AxiosInstance;
  
  constructor(config: ScrapelessConfig) {
    this.api = axios.create({
      baseURL: config.baseURL,
      headers: {
        "Content-Type": "application/json",
        "x-api-token": config.token,
      },
    });
  }
  
  async searchWeb(params: WebSearchParams) {
    try {
      const response = await this.api.post("/api/v1/scraper/request", {
        actor: "scraper.google.search",
        input: {
          q: params.query,
          gl: params.country || "us",
          hl: params.language || "en",
          google_domain: params.domain || "google.com"
        }
      });
      
      return {
        query: params.query,
        results: response.data
      };
    } catch (error) {
      // Error handling...
    }
  }
}

```
### Advanced Features Supported

| Feature                  | Description                                                                 |
|--------------------------|-----------------------------------------------------------------------------|
| Google Search Integration | Uses the `scraper.google.search` actor to fetch search results             |
| Geo-Targeting             | Controls country/region using the `gl` parameter                            |
| Multilingual Support      | Returns results in different languages using the `hl` parameter             |
| Search Engine Domain Switching | Supports multiple domains like `google.de`, `google.fr`, etc.         |
| Proxy Auto-Management     | Enables proxy rotation by default to avoid IP blocking                     |

## Intelligent Question Answering System Deployment Guide (Based on Vector Search + Real-time Web Search)

### Step 1: Initialize project structure and install dependencies

**Clone and set up the project:**
```
git clone git@github.com:scrapeless-ai/mcp-rag-server.git
cd mcp-rag-server

```
**Analyze the project structure:**
```
mcp-rag-server/
├── src/
│   ├── config.ts
│   ├── index.ts
│   ├── server.ts
│   ├── qdrant-client.ts
│   └── scrapeless-client.ts
├── package.json
├── tsconfig.json
└── .env

```
**Install dependencies:**
```
npm install

```

💡Problem Solved:
Ensure that the TypeScript project environment is ready, necessary dependencies (such as @modelcontextprotocol/sdk, axios, zod, etc.) have been integrated, and the type definitions required for development have also been automatically configured.

### Step 2: Environment Configuration

```



Create `.env` file:
QDRANT_URL=http://localhost:6333
QDRANT_API_KEY=
QDRANT_COLLECTION=ml_faq_collection
SCRAPELESS_KEY=your_scrapeless_api_key
SCRAPELESS_BASE_URL=https://api.scrapeless.com

Understanding the configuration (from config.ts):
const QDRANT_URL = process.env.QDRANT_URL?.trim() || "http://localhost:6333";
const QDRANT_API_KEY = process.env.QDRANT_API_KEY?.trim() || "";
const QDRANT_COLLECTION = process.env.QDRANT_COLLECTION?.trim() || "ml_faq_collection";
const SCRAPELESS_KEY = process.env.SCRAPELESS_KEY?.trim();
const SCRAPELESS_BASE_URL = process.env.SCRAPELESS_BASE_URL?.trim() || "https://api.scrapeless.com";
```


**💡Problem solved:**

Provide correct connection parameters for external dependencies (Qdrant vector database and Scrapeless real-time search). Default values and trim() processing are built into the configuration code to prevent variable format errors. If the Scrapeless Key is missing, a warning will be issued.

### Step 3: Set up Qdrant Vector Database

**Start Qdrant with Docker:**
```
# Pull Qdrant image
docker pull qdrant/qdrant

# Run Qdrant container with data persistence
docker run -d \
  --name qdrant-server \
  -p 6333:6333 \
  -p 6334:6334 \
  -v $(pwd)/qdrant_storage:/qdrant/storage \
  qdrant/qdrant

```
**Create a FAQ vector collection:**
```
curl -X PUT 'http://localhost:6333/collections/ml_faq_collection' \
  -H 'Content-Type: application/json' \
  --data-raw '{
    "vectors": {
      "size": 1536,
      "distance": "Cosine"
    }
  }'

```
**💡Problem solved:**

Configure semantic vector retrieval storage, use 1536 dimensions and cosine similarity, and be compatible with embedding generator output and QdrantClient calls.

### Step 4: Integrate Scrapeless real-time web search

**Obtaining the Scrapeless API Key:**

1. Visit [Scrapeless](https://app.scrapeless.com/passport/login?utm_source=official&utm_medium=blog&utm_campaign=mcprag) and create an account
2. Retrieve your API Token from the dashboard

![Retrieve your API Token](https://assets.scrapeless.com/prod/posts/mcp-rag-server-with-scrapeless/8c967ead40663ba1ba4d552fc94e8303.png)


3. Add it to the .env file under SCRAPELESS_KEY

**Test Scrapeless Connection:**
```
# Test API connection (optional)
curl -X POST 'https://api.scrapeless.com/api/v1/scraper/request' \
  -H 'Content-Type: application/json' \
  -H 'x-api-token: YOUR_API_KEY' \
  -d '{"actor": "scraper.google.search", "input": {"q": "test query"}}'

```
Problem Solved: This step ensures your Scrapeless API is properly configured. The system includes validation to check if the API key is set, preventing runtime errors during web searches.

### Step 5: Build the TypeScript Project

Compile TypeScript to JavaScript:

```
npm run build

```
What happens during build (from package.json):

```language
{
  "scripts": {
    "build": "tsc && chmod 755 build/index.js",
    "start": "node build/index.js"
  }
}
```


Verify build output:
```language
ls build/
# Should show: index.js, server.js, config.js, qdrant-client.js, scrapeless-client.js

```

**Problem Solved: **

This step compiles TypeScript to JavaScript and ensures the main entry point is executable. The build process outputs ES modules (as specified by `"type": "module"` in package.json) compatible with Node.js.

### Step 6: Start the MCP Server

```language
Run the server:
npm start
```


What happens during startup (from index.ts):
```
async function main() {
  try {
    console.log("Starting MCP Agentic RAG Server...");
    const transport = new StdioServerTransport();
    await server.connect(transport);
    console.log("MCP Server is running on port 8080");
  } catch (error) {
    console.error("Fatal error in main():", error);
    process.exit(1);
  }
}

```
**Problem Solved:** 

This step launches the MCP server using STDIO transport for communication. The server initializes with proper error handling and logging.

By following the six steps above, you will have built an AI-powered question-answering system with:
- Semantic QA capabilities (powered by the Qdrant vector database)
- Real-time web augmentation (via Scrapeless API integration)
- LLM-ready infrastructure (based on the MCP protocol standard)
This forms a solid foundation for enterprises or developers to quickly deploy a production-ready RAG (Retrieval-Augmented Generation) system.

## Detailed explanation of core components
### QdrantClient: vector processing engine
QdrantClient provides embedding generation and vector database interaction functions. The example uses a simple deterministic embedding method for demonstration:

```
private generateEmbedding(text: string): number[] {
  const seed = [...text].reduce((sum, char) => sum + char.charCodeAt(0), 0) % 10000;
  const vector: number[] = [];
  
  let value = seed;
  for (let i = 0; i < 1536; i++) {
    value = (value * 48271) % 2147483647;
    vector.push((value / 2147483647) * 2 - 1);
  }
  
  return vector;
}

```

Key features:
- Simple deterministic embedding generation
- Upsert operations for adding documents
- Semantic search with configurable scoring threshold
- Proper error handling and fallback responses

### ScrapelessClient: Web search engine interface
ScrapelessClient accesses the Scrapeless API to implement web search and supports advanced search parameters:
```
async searchWeb(params: WebSearchParams) {
  try {
    if (!this.api.defaults.headers.common["x-api-token"]) {
      throw new Error("Scrapeless API key is not set");
    }
    
    const response = await this.api.post("/api/v1/scraper/request", {
      actor: "scraper.google.search",
      input: {
        q: params.query,
        gl: params.country || "us",
        hl: params.language || "en",
        google_domain: params.domain || "google.com"
      }
    });
    
    return {
      query: params.query,
      results: response.data
    };
  } catch (error) {
    // Error handling...
  }
}

```
Key features:
- Google search integration via Scrapeless
- Configurable country, language, and domain
- Comprehensive error handling
- API key validation

### MCP Server Tools

The server.ts file defines three main tools:

1. machine-learning-faq-retrieval:
- Searches the vector database for ML concepts
- Uses semantic similarity matching
- Returns formatted results with scores

2. add-document-to-faq:
- Adds new documents to the knowledge base
- Supports metadata (category, source, tags)
- Proper error handling with detailed responses

3. scrapeless-web-search:
- Performs web searches via Scrapeless API
- Configurable search parameters
- Real-time information retrieval

## Usage Guide: Using the system with Scrapeless
### Basic Usage Examples

Search the knowledge base:
```
Use machine-learning-faq-retrieval to find information about neural networks

```
Add new information:
```
Use add-document-to-faq to add this:
Text: "Random forests are ensemble learning methods..."
Category: "Ensemble Methods"
Tags: ["random forests", "ensemble learning"]

```
Search the web with Scrapeless:
```language
Use scrapeless-web-search to find recent developments in AI
```


Advanced Scrapeless usage:
```language
Use scrapeless-web-search with:
Query: "latest PyTorch features"
Country: "uk"
Language: "en"
Domain: "google.co.uk"
```
### Advanced Workflows with Scrapeless Integration

Knowledge base enhancement:
```
1. Use scrapeless-web-search to find "latest transformer models 2024"
2. Use add-document-to-faq to add relevant findings
3. Use machine-learning-faq-retrieval to verify the information is searchable

```
Information verification:
```
1. Use machine-learning-faq-retrieval to check existing knowledge
2. Use scrapeless-web-search to find current information
3. Compare and update knowledge base accordingly

```

Multilingual knowledge building:
```
1. Use scrapeless-web-search with country="de" and language="de" to find German AI research
2. Use add-document-to-faq to add translated summaries
3. Build a multilingual knowledge base

```
Integration with Claude Desktop

The project includes a sample configuration for Claude Desktop integration:
```
{
  "mcpServers": {
    "MCP-RAG-app": {
      "command": "node",
      "args": ["your-path/to/build/index.js"],
      "host": "127.0.0.1",
      "port": 8080,
      "timeout": 30000,
      "env": {
        "QDRANT_URL": "http://localhost:6333",
        "QDRANT_API_KEY": "",
        "QDRANT_COLLECTION": "ml_faq_collection",
        "SCRAPELESS_KEY": "SCRAPELESS_KEY"
      }
    }
  }
}

```
## Common Issues and Solutions

1. Build errors:
- Ensure Node.js version >= 18
- Check TypeScript compilation: npx tsc --noEmit

2. Runtime errors:
- Verify Qdrant is running: curl http://localhost:6333/health
- Check environment variables are set correctly
- Ensure Scrapeless API key is valid

3. Scrapeless-specific issues:
- Verify API key is correctly set in environment
- Check API quotas and limits in Scrapeless dashboard
- Ensure correct API endpoint configuration

4. Connection issues:
- Verify ports are available (6333 for Qdrant)
- Check firewall settings
- Ensure Docker containers are running

## Benefits of the Combined System

The integration of Scrapeless with Qdrant creates a powerful hybrid system:

1. Static + Dynamic Knowledge: Combine your curated knowledge base with real-time web data
2. Intelligent Search: Use semantic search for internal data and keyword search for web content
3. Continuous Enhancement: Automatically update your knowledge base with fresh information
4. Global Perspective: Access information from different regions and languages
5. Reliability: Scrapeless ensures consistent web access without blocking issues

## Conclusion

MCP-RAG Server and Scrapeless implement a highly scalable, real-time updated intelligent question-answering system. The core values include:

- Semantic understanding: understanding context through vector similarity
- Real-time information access: access Scrapeless to obtain the latest web content
- Standard protocol integration: using MCP protocol, it is easy to connect to platforms such as Claude
- Flexible configuration: customizable knowledge base collection and search tools
- Future-oriented intelligent platform: support dynamic knowledge enhancement, multi-language support and web intelligent crawling.
  
The addition of Scrapeless makes the system no longer just a static knowledge base, but an AI knowledge engine with global vision and continuous learning capabilities.
