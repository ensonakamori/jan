# Integration Guide

**Creating extensions, adding providers, and integrating external services**

**Documented:** November 18, 2025

---

## Extension System Overview

Jan AI uses a **plugin architecture** with two types of extensions:

1. **Native Extensions** (TypeScript) - Full Tauri access
2. **Web Extensions** (TypeScript) - Browser-only

---

## Creating a Native Extension

### Step 1: Setup Extension Directory

```bash
cd extensions
mkdir my-extension
cd my-extension
npm init -y
```

### Step 2: Install Dependencies

```json
{
  "name": "@janhq/my-extension",
  "dependencies": {
    "@janhq/core": "workspace:*"
  },
  "devDependencies": {
    "typescript": "^5.9.2"
  }
}
```

### Step 3: Implement Extension Interface

```typescript
// src/index.ts
import { Extension } from '@janhq/core'

export default class MyExtension implements Extension {
  async onLoad() {
    console.log('MyExtension loaded')
    // Initialize extension
    await this.registerCapabilities()
  }

  async onUnload() {
    console.log('MyExtension unloading')
    // Cleanup resources
  }

  private async registerCapabilities() {
    // Register models, providers, tools, etc.
  }
}
```

### Step 4: Build Configuration

```json
// package.json
{
  "scripts": {
    "build": "tsc && rollup -c",
    "build:publish": "yarn build && yarn pack"
  }
}
```

### Step 5: Build and Test

```bash
yarn build:publish
# Creates my-extension-v1.0.0.tgz
```

### Step 6: Install Extension

Copy `.tgz` to `~/jan/extensions/` or `pre-install/`

---

## Creating a Provider Integration

### Overview

Providers enable Jan to use external LLM APIs (OpenAI, Anthropic, etc.)

### Step 1: Define Provider Interface

```typescript
// extensions/my-provider/src/types.ts
import { Model, InferenceRequest } from '@janhq/core'

export interface MyProviderConfig {
  apiKey: string
  baseUrl: string
}

export interface ProviderExtension {
  loadModel(model: Model): Promise<void>
  inference(request: InferenceRequest): Promise<Response>
  unloadModel(modelId: string): Promise<void>
}
```

### Step 2: Implement Provider

```typescript
// extensions/my-provider/src/index.ts
export default class MyProviderExtension implements Extension {
  private config: MyProviderConfig

  async onLoad() {
    // Load API key from settings
    this.config = await this.loadConfig()
  }

  async loadModel(model: Model): Promise<void> {
    // Provider-specific model loading
    console.log(`Loading model: ${model.id}`)
  }

  async inference(request: InferenceRequest): Promise<Response> {
    const response = await fetch(`${this.config.baseUrl}/completions`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${this.config.apiKey}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        model: request.model,
        messages: request.messages,
        temperature: request.temperature,
        stream: request.stream,
      }),
    })

    return response
  }

  private async loadConfig(): Promise<MyProviderConfig> {
    // Load from settings
    return {
      apiKey: process.env.MY_PROVIDER_API_KEY || '',
      baseUrl: 'https://api.myprovider.com',
    }
  }
}
```

### Step 3: Add Settings UI

```typescript
// web-app/src/routes/settings/providers/my-provider.tsx
import { createFileRoute } from '@tanstack/react-router'

export const Route = createFileRoute('/settings/providers/my-provider')({
  component: MyProviderSettings,
})

function MyProviderSettings() {
  const [apiKey, setApiKey] = useState('')

  const handleSave = async () => {
    await providerService.saveConfig({
      provider: 'my-provider',
      apiKey,
    })
    toast.success('Settings saved')
  }

  return (
    <div>
      <h2>My Provider Settings</h2>
      <input
        type="password"
        value={apiKey}
        onChange={(e) => setApiKey(e.target.value)}
        placeholder="API Key"
      />
      <button onClick={handleSave}>Save</button>
    </div>
  )
}
```

---

## MCP (Model Context Protocol) Integration

### What is MCP?

MCP allows AI models to interact with external tools and data sources.

### Adding an MCP Server

**File:** Settings → MCP Servers

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allowed"],
      "env": {}
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "your-token"
      }
    }
  }
}
```

### Using MCP Tools in Chat

Once configured, MCP tools are available to the AI:

```
User: "Read the contents of file.txt"
AI: [Uses filesystem MCP server to read file]
AI: "The file contains: ..."
```

### Creating a Custom MCP Server

```typescript
// my-mcp-server/index.ts
import { Server } from '@modelcontextprotocol/sdk/server/index.js'
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js'

const server = new Server(
  {
    name: 'my-custom-server',
    version: '1.0.0',
  },
  {
    capabilities: {
      tools: {},
    },
  }
)

// Register tools
server.setRequestHandler('tools/list', async () => ({
  tools: [
    {
      name: 'my_tool',
      description: 'Does something useful',
      inputSchema: {
        type: 'object',
        properties: {
          query: { type: 'string' },
        },
      },
    },
  ],
}))

server.setRequestHandler('tools/call', async (request) => {
  if (request.params.name === 'my_tool') {
    const result = await doSomething(request.params.arguments.query)
    return {
      content: [{ type: 'text', text: JSON.stringify(result) }],
    }
  }
})

const transport = new StdioServerTransport()
await server.connect(transport)
```

**Usage in Jan:**
```json
{
  "mcpServers": {
    "my-custom": {
      "command": "node",
      "args": ["/path/to/my-mcp-server/index.js"]
    }
  }
}
```

---

## Integrating External APIs

### HTTP API Integration

```typescript
// services/external/myService.ts
export class MyExternalService {
  private baseUrl = 'https://api.example.com'
  private apiKey: string

  constructor(apiKey: string) {
    this.apiKey = apiKey
  }

  async fetchData(query: string): Promise<Data> {
    const response = await fetch(`${this.baseUrl}/search`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${this.apiKey}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ query }),
    })

    if (!response.ok) {
      throw new Error(`API error: ${response.statusText}`)
    }

    return await response.json()
  }
}
```

### Tauri Integration (For Native APIs)

```rust
// src-tauri/src/core/external/commands.rs
use reqwest;

#[tauri::command]
pub async fn fetch_external_data(query: String) -> Result<String, String> {
    let client = reqwest::Client::new();
    let response = client
        .post("https://api.example.com/search")
        .json(&serde_json::json!({ "query": query }))
        .send()
        .await
        .map_err(|e| e.to_string())?;

    let data = response.text().await.map_err(|e| e.to_string())?;
    Ok(data)
}
```

---

## Adding a New Model Format

### Step 1: Create Model Loader Extension

```typescript
// extensions/my-model-format/src/index.ts
export default class MyModelFormatExtension implements Extension {
  async onLoad() {
    // Register model format handler
    this.registerModelFormat({
      format: 'myformat',
      extensions: ['.mymodel'],
      loader: this.loadModel.bind(this),
    })
  }

  private async loadModel(path: string): Promise<Model> {
    // Parse model file
    const modelData = await readModelFile(path)

    return {
      id: modelData.id,
      name: modelData.name,
      format: 'myformat',
      parameters: modelData.config,
    }
  }
}
```

### Step 2: Add Tauri Plugin for Inference

```rust
// src-tauri/plugins/tauri-plugin-myformat/src/lib.rs
use tauri::plugin::{Builder, TauriPlugin};

#[tauri::command]
async fn generate_myformat(
    model_id: String,
    prompt: String,
) -> Result<String, String> {
    // Model inference logic
    let response = run_inference(&model_id, &prompt)?;
    Ok(response)
}

pub fn init<R: tauri::Runtime>() -> TauriPlugin<R> {
    Builder::new("myformat")
        .invoke_handler(tauri::generate_handler![generate_myformat])
        .build()
}
```

---

## RAG (Retrieval-Augmented Generation) Integration

### Adding Document Source

```typescript
// extensions/rag-extension/src/sources/mySource.ts
export class MyDocumentSource implements DocumentSource {
  async fetchDocuments(query: string): Promise<Document[]> {
    // Fetch documents from your source
    const docs = await this.querySource(query)

    return docs.map((doc) => ({
      id: doc.id,
      content: doc.text,
      metadata: {
        source: 'mySource',
        url: doc.url,
      },
    }))
  }

  private async querySource(query: string) {
    // Your source-specific logic
  }
}
```

### Registering with RAG Extension

```typescript
// In your extension's onLoad()
await ragExtension.registerSource(new MyDocumentSource())
```

---

## OAuth Integration

### Step 1: Configure OAuth Provider

```typescript
// services/auth/oauth.ts
export class OAuthService {
  private clientId: string
  private redirectUri: string

  async initiateLogin(provider: 'google' | 'github') {
    const authUrl = this.getAuthUrl(provider)

    // Open browser for auth
    await open(authUrl)
  }

  async handleCallback(code: string): Promise<AuthToken> {
    const response = await fetch('https://oauth.example.com/token', {
      method: 'POST',
      body: JSON.stringify({
        code,
        client_id: this.clientId,
        redirect_uri: this.redirectUri,
      }),
    })

    return await response.json()
  }
}
```

### Step 2: Deep Link Handler (Tauri)

```rust
// src-tauri/src/lib.rs
#[cfg(feature = "deep-link")]
{
    app_builder = app_builder.plugin(tauri_plugin_deep_link::init());
}
```

```typescript
// Handle callback URL
await listen('deep-link://oauth-callback', (event) => {
  const code = extractCodeFromUrl(event.payload)
  await oauthService.handleCallback(code)
})
```

---

## Testing Extensions

### Unit Tests

```typescript
// extensions/my-extension/src/__tests__/index.test.ts
import { describe, it, expect } from 'vitest'
import MyExtension from '../index'

describe('MyExtension', () => {
  it('loads successfully', async () => {
    const ext = new MyExtension()
    await expect(ext.onLoad()).resolves.not.toThrow()
  })

  it('provides expected capabilities', () => {
    const ext = new MyExtension()
    expect(ext.getCapabilities()).toContain('my-feature')
  })
})
```

### Integration Tests

```typescript
// Test with actual Tauri commands
vi.mock('@tauri-apps/api/core', () => ({
  invoke: vi.fn().mockResolvedValue({ success: true }),
}))

it('calls Tauri command correctly', async () => {
  const ext = new MyExtension()
  await ext.doSomething()

  expect(invoke).toHaveBeenCalledWith('my_command', {
    param: 'value',
  })
})
```

---

## Publishing Extensions

### Step 1: Package Extension

```bash
cd extensions/my-extension
yarn build:publish
# Creates my-extension-v1.0.0.tgz
```

### Step 2: Distribution Options

**Option A: Pre-bundled**
- Copy `.tgz` to `pre-install/`
- Ships with app

**Option B: User Installation**
- User downloads `.tgz`
- Installs via Settings → Extensions

**Option C: Extension Registry** (Future)
- Publish to Jan extension marketplace
- Users install from UI

---

## Next Steps

- **Extension Examples:** Check [extensions/](../../extensions/)
- **API Reference:** [API_DOCUMENTATION.md](./API_DOCUMENTATION.md)
- **Testing:** [TESTING_GUIDE.md](./TESTING_GUIDE.md)

**Questions?** Return to the [Learning Hub](./README.md)
