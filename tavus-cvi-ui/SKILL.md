---
name: tavus-cvi-ui
description: Integrate Tavus CVI into React apps using @tavus/cvi-ui components. Use when embedding conversations in web apps, creating conversations from a web app (through a server route that keeps the API key off the client), customizing the video UI, using React hooks for CVI events, or building custom conversation interfaces with Vite/Next.js.
---

# Tavus CVI React UI Components

Pre-built React components for embedding CVI conversations.

## Rule: Keep the API Key on the Server

- Never put the Tavus API key in client code or in a client-prefixed env var (`VITE_`, `NEXT_PUBLIC_`, `REACT_APP_`, `EXPO_PUBLIC_`, or similar). Bundlers copy those variables into the JavaScript every visitor downloads, so anyone can read the key.
- Never call `https://tavusapi.com` or send an `x-api-key` header from browser code.
- The browser calls the app's own route, `POST /api/tavus`. Only that route reads `TAVUS_API_KEY` (no prefix) and calls the Tavus API. `npx @tavus/cvi-ui@latest add tavus-api` generates it (`add tavus-api-vite-ssr` on Vite).
- If existing code reads the key in a component or calls `tavusapi.com` from the browser, move that call into the server route.

## Quick Setup (Vite)

```bash
npm create vite@latest my-app -- --template react-ts
cd my-app
npm install
npx @tavus/cvi-ui@latest init
npx @tavus/cvi-ui@latest add conversation
npx @tavus/cvi-ui@latest add tavus-api-vite-ssr
```

Creates:
```
src/components/cvi/components/
├── cvi-provider/
└── conversation/
src/components/cvi/lib/
├── tavus-api-vite-ssr.ts   # server handler: reads TAVUS_API_KEY, calls the Tavus API
└── tavus-client.ts         # browser client: calls POST /api/tavus
```

Create `cvi-components.json` in project root:
```json
{
  "tsx": true
}
```

On Next.js, Remix or TanStack Start, run `npx @tavus/cvi-ui@latest add tavus-api` instead: it installs the route for your framework (see [Server Route](#server-route)). On plain Vite, `add tavus-api` exits with an error because Vite has no server, so use `add tavus-api-vite-ssr` and mount it as below.

### Mount the Server Route (Vite)

```bash
npm install hono @hono/node-server
npm install -D tsx concurrently @types/node
```

`server.ts` in project root:
```ts
import { serve } from "@hono/node-server";
import { Hono } from "hono";
import { handleTavusRequest } from "./src/components/cvi/lib/tavus-api-vite-ssr";

const app = new Hono();
app.post("/api/tavus", (c) => handleTavusRequest(c.req.raw));

serve({ fetch: app.fetch, port: 3001 });
```

`vite.config.ts`, so the dev server forwards `/api` to it:
```ts
import react from "@vitejs/plugin-react";
import { defineConfig } from "vite";

export default defineConfig({
  plugins: [react()],
  server: { proxy: { "/api": "http://localhost:3001" } },
});
```

`package.json`, so `npm run dev` starts both:
```json
"dev": "concurrently -k -n api,web \"tsx watch --env-file=.env server.ts\" \"vite\""
```

The handler reads `process.env`, so add `"node"` to `compilerOptions.types` in `tsconfig.app.json` (`"types": ["vite/client", "node"]`), or `npm run build` fails type-checking.

## Environment Variables

`.env` in project root:
```
TAVUS_API_KEY=your_api_key
```

- Server-only: no `VITE_` prefix, read only by the server route.
- Add `.env` to `.gitignore`. The Vite template does not ignore it.
- Face and PAL IDs are not secrets. Pass them to `createTavusConversation()`, or set them in the server route if callers must not change them.

## Basic Implementation

```tsx
import { useState } from "react";
import { CVIProvider } from "./components/cvi/components/cvi-provider";
import { Conversation } from "./components/cvi/components/conversation";
import {
  createTavusConversation,
  endTavusConversation,
} from "./components/cvi/lib/tavus-client";

export default function App() {
  const [call, setCall] = useState<{ id: string; url: string } | null>(null);

  const startConversation = async () => {
    // POST /api/tavus on your own server. The API key never reaches the browser.
    const { conversation_id, conversation_url } = await createTavusConversation({
      replica_id: "rfe12d8b9597",
      persona_id: "pdced222244b",
    });
    setCall({ id: conversation_id, url: conversation_url });
  };

  const leave = () => {
    if (call) void endTavusConversation(call.id);
    setCall(null);
  };

  return (
    <CVIProvider>
      {!call ? (
        <button onClick={startConversation}>Start</button>
      ) : (
        <Conversation 
          conversationUrl={call.url} 
          onLeave={leave} 
        />
      )}
    </CVIProvider>
  );
}
```

## Component: CVIProvider

Wraps your app, provides CVI context:

```tsx
import { CVIProvider } from "./components/cvi/components/cvi-provider";

function App() {
  return (
    <CVIProvider>
      {/* Your app */}
    </CVIProvider>
  );
}
```

## Component: Conversation

The main video conversation UI:

```tsx
<Conversation
  conversationUrl={url}
  onLeave={() => setUrl(null)}
/>
```

**Required**: Parent container must have defined dimensions.

```tsx
<div style={{ width: "100%", height: "600px" }}>
  <Conversation conversationUrl={url} onLeave={handleLeave} />
</div>
```

## Hooks

### useAppMessage

Listen for CVI events:

```tsx
import { useAppMessage } from "./components/cvi/hooks/use-app-message";

function MyComponent() {
  useAppMessage((event) => {
    if (event.event_type === "conversation.utterance") {
      console.log("Said:", event.properties.content);
    }
  });
  
  return <div>...</div>;
}
```

### useSendAppMessage

Send interactions to CVI:

```tsx
import { useSendAppMessage } from "./components/cvi/hooks/use-send-app-message";

function Controls() {
  const send = useSendAppMessage();
  
  const interrupt = () => {
    send({
      message_type: "conversation",
      event_type: "conversation.interrupt",
      conversation_id: "xxx"
    });
  };
  
  return <button onClick={interrupt}>Stop</button>;
}
```

## Alternative: iframe Embedding

For non-React or quick integration:

```html
<iframe
  src="CONVERSATION_URL"
  allow="camera; microphone; display-capture"
  style="width: 100%; height: 600px; border: none;"
></iframe>
```

## Server Route

`npx @tavus/cvi-ui@latest add tavus-api` detects the framework and installs a route that keeps the key on the server:

| Framework | Route file |
|-----------|------------|
| Next.js (App Router) | `app/api/tavus/route.ts` |
| Next.js (Pages Router) | `pages/api/tavus.ts` |
| Remix | `app/routes/api.tavus.ts` |
| TanStack Start | `app/routes/api/tavus.ts` |
| Vite with a server (`add tavus-api-vite-ssr`) | `lib/tavus-api-vite-ssr.ts` exporting `handleTavusRequest`, which you mount at `POST /api/tavus` |

Each also installs `lib/tavus-client.ts` with `createTavusConversation(params)` and `endTavusConversation(conversationId)` for the browser.

The route takes `POST /api/tavus` with either `{ "action": "create", "params": { ... } }`, forwarded to `POST https://tavusapi.com/v2/conversations`, or `{ "action": "end", "conversationId": "..." }`. It adds `x-api-key` from `process.env.TAVUS_API_KEY`. If you write the route yourself for another backend, keep that contract and return only what the browser needs (`conversation_id`, `conversation_url`).

Before production: anyone who can reach `/api/tavus` can create conversations on your account. Put your own authentication or rate limiting in front of it, and set values that must not change (such as the PAL ID) in the route instead of trusting `params` from the browser.

## Styling the Conversation

Wrap in styled container:

```tsx
<div style={{
  width: "100%",
  maxWidth: "900px",
  margin: "0 auto",
  borderRadius: "12px",
  overflow: "hidden",
  boxShadow: "0 4px 20px rgba(0,0,0,0.15)",
}}>
  <Conversation conversationUrl={url} onLeave={onLeave} />
</div>
```

## HairCheck Component

Pre-call device check:

```bash
npx @tavus/cvi-ui@latest add haircheck
```

```tsx
import { HairCheck } from "./components/cvi/components/haircheck";

<HairCheck onComplete={() => setShowConversation(true)} />
```

## Example Projects

- [CVI UI Conversation](https://github.com/Tavus-Engineering/tavus-examples/tree/main/examples/cvi-ui-conversation)
- [CVI UI Haircheck + Conversation](https://github.com/Tavus-Engineering/tavus-examples/tree/main/examples/cvi-ui-haircheck-conversation)
- [Santa Demo](https://github.com/Tavus-Engineering/santa-template)
