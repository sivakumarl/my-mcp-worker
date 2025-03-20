# MCP Server with Cloudflare Workers

## Introduction

Model Context Protocol (MCP) is an open standard that enables AI agents and assistants to interact with services. By setting up an MCP server, you can allow AI assistants to access your APIs directly.

Cloudflare Workers, combined with the `workers-mcp` package, provide a powerful and scalable solution for building MCP servers.

## Prerequisites

Before starting, ensure you have:

- A [Cloudflare account](https://dash.cloudflare.com/)
- Node.js installed
- Wrangler CLI installed (`npm install -g wrangler`)

---

## Getting Started

### Step 1: Create a New Cloudflare Worker

First, initialize a new Cloudflare Worker project:

```bash
npx create-cloudflare@latest my-mcp-worker
cd my-mcp-worker
```

Then, authenticate your Cloudflare account:

```bash
wrangler login
```

### Step 2: Configure Wrangler

Update your `wrangler.toml` file with the correct account details:

```toml
name = "my-mcp-worker"
main = "src/index.ts"
compatibility_date = "2025-03-03"
account_id = "your-account-id"
```

---

## Installing MCP Tooling

To enable MCP support, install the `workers-mcp` package:

```bash
npm install workers-mcp
```

Run the setup command to configure MCP:

```bash
npx workers-mcp setup
```

This will:

- Add necessary dependencies
- Set up a local proxy for testing
- Configure the Worker for MCP compliance

---

## Writing MCP Server Code

Update your `src/index.ts` to define your MCP server:

```typescript
import { WorkerEntrypoint } from 'cloudflare:workers';
import { ProxyToSelf } from 'workers-mcp';

export default class MyWorker extends WorkerEntrypoint<Env> {
  /**
   * A friendly greeting from your MCP server.
   * @param name {string} The name of the user.
   * @return {string} A personalized greeting.
   */
  sayHello(name: string) {
    return `Hello from an MCP Worker, ${name}!`;
  }

  /**
   * @ignore
   */
  async fetch(request: Request): Promise<Response> {
    return new ProxyToSelf(this).fetch(request);
  }
}
```

### Key Components:
- **WorkerEntrypoint**: Manages incoming requests and method exposure.
- **ProxyToSelf**: Ensures MCP protocol compliance.
- **sayHello method**: An example MCP function that AI assistants can call.

---

## Adding API Calls

You can extend your MCP server by integrating with external APIs. Here's an example of fetching weather data:

```typescript
export default class WeatherWorker extends WorkerEntrypoint<Env> {
  /**
   * Fetch weather data for a given location.
   * @param location {string} The city or ZIP code.
   * @return {object} Weather details.
   */
  async getWeather(location: string) {
    const response = await fetch(`https://api.weather.example/v1/${location}`);
    const data = await response.json();
    return {
      temperature: data.temp,
      conditions: data.conditions,
      forecast: data.forecast
    };
  }

  async fetch(request: Request): Promise<Response> {
    return new ProxyToSelf(this).fetch(request);
  }
}
```

---

## Deploying the MCP Server

Once your Worker is set up, deploy it to Cloudflare:

```bash
npx wrangler deploy
```

After deployment, your Worker is live and AI assistants can discover and use your MCP tools.

To update your MCP server, redeploy with:

```bash
npm run deploy
```

---

## Testing the MCP Server

To test your MCP setup locally:

```bash
npx workers-mcp proxy
```

This command starts a local proxy allowing MCP clients (like Claude Desktop) to connect.

---

## Security

To secure your MCP server, use Wrangler Secrets:

```bash
npx wrangler secret put MCP_SECRET
```

This adds a shared-secret authentication mechanism to prevent unauthorized access.

---

## Conclusion

Congratulations! You have successfully built and deployed an MCP server using Cloudflare Workers. You can now extend it with more features and expose new tools for AI assistants.

For more details, check the [Cloudflare MCP documentation](https://developers.cloudflare.com/agents/guides/build-mcp-server/).

---
