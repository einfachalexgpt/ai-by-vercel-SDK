# **AI SDK**

![hero illustration](./assets/hero.gif)

Das [AI SDK](https://sdk.vercel.ai/docs) ist ein TypeScript-Toolkit, das dabei hilft, KI-gestützte Anwendungen mit beliebten Frameworks wie Next.js, React, Svelte, Vue und Laufzeiten wie Node.js zu entwickeln.

Um mehr über die Verwendung des AI SDK zu erfahren, sieh dir unsere [API-Referenz](https://sdk.vercel.ai/docs/reference) und [Dokumentation](https://sdk.vercel.ai/docs) an.

## Installation

Du benötigst Node.js 18+ und pnpm auf deinem lokalen Entwicklungsrechner.

```shell
npm install ai
```

## Nutzung

### **AI SDK Core**

Das [AI SDK Core](https://sdk.vercel.ai/docs/ai-sdk-core/overview)-Modul bietet eine einheitliche API zur Interaktion mit Modellanbietern wie [OpenAI](https://sdk.vercel.ai/providers/ai-sdk-providers/openai), [Anthropic](https://sdk.vercel.ai/providers/ai-sdk-providers/anthropic), [Google](https://sdk.vercel.ai/providers/ai-sdk-providers/google-generative-ai) und mehr.

Zunächst installierst du den Modellanbieter deiner Wahl:

```shell
npm install @ai-sdk/openai
```

##### Beispiel: **@/index.ts (Node.js Runtime)**

```ts
import { generateText } from 'ai';
import { openai } from '@ai-sdk/openai'; // Achte darauf, dass die Umgebungsvariable OPENAI_API_KEY gesetzt ist

async function main() {
  const { text } = await generateText({
    model: openai('gpt-4-turbo'),
    system: 'Du bist ein freundlicher Assistent!',
    prompt: 'Warum ist der Himmel blau?',
  });

  console.log(text);
}

main();
```

### **AI SDK UI**

Das [AI SDK UI](https://sdk.vercel.ai/docs/ai-sdk-ui/overview)-Modul bietet eine Reihe von Hooks, die beim Erstellen von Chatbots und generativen Benutzeroberflächen helfen. Diese Hooks sind framework-unabhängig und können in Next.js, React, Svelte, Vue und SolidJS verwendet werden.

##### Beispiel: **@/app/page.tsx (Next.js App Router)**

```tsx
'use client';

import { useChat } from 'ai/react';

export default function Page() {
  const { messages, input, handleSubmit, handleInputChange, isLoading } =
    useChat();

  return (
    <div>
      {messages.map(message => (
        <div key={message.id}>
          <div>{message.role}</div>
          <div>{message.content}</div>
        </div>
      ))}

      <form onSubmit={handleSubmit}>
        <input
          value={input}
          placeholder="Nachricht senden..."
          onChange={handleInputChange}
          disabled={isLoading}
        />
      </form>
    </div>
  );
}
```

##### Beispiel: **@/app/api/chat/route.ts (Next.js App Router)**

```ts
import { CoreMessage, streamText } from 'ai';
import { openai } from '@ai-sdk/openai';

export async function POST(req: Request) {
  const { messages }: { messages: CoreMessage[] } = await req.json();

  const result = await streamText({
    model: openai('gpt-4'),
    system: 'Du bist ein hilfsbereiter Assistent.',
    messages,
  });

  return result.toDataStreamResponse();
}
```

### **AI SDK RSC**

Das [AI SDK RSC](https://sdk.vercel.ai/docs/ai-sdk-rsc/overview)-Modul bietet eine alternative API, die ebenfalls beim Erstellen von Chatbots und generativen Benutzeroberflächen für Frameworks hilft, die [React Server Components](https://nextjs.org/docs/app/building-your-application/rendering/server-components) (RSC) unterstützen.

Diese API nutzt die Vorteile des [Streamings](https://nextjs.org/docs/app/building-your-application/rendering/server-components#streaming) und der [Server Actions](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations), um die Verwaltung von Zuständen zwischen Server und Client zu verbessern.

##### Beispiel: **@/app/actions.tsx (Next.js App Router)**

```tsx
import { streamUI } from 'ai/rsc';
import { z } from 'zod';

async function submitMessage() {
  'use server';

  const stream = await streamUI({
    model: openai('gpt-4-turbo'),
    messages: [
      { role: 'system', content: 'Du bist ein freundlicher Bot!' },
      { role: 'user', content: input },
    ],
    text: ({ content, done }) => {
      return <div>{content}</div>;
    },
    tools: {
      deploy: {
        description: 'Repository auf Vercel deployen',
        parameters: z.object({
          repositoryName: z
            .string()
            .describe('Name des Repositories, z.B.: vercel/ai-chatbot'),
        }),
        generate: async function* ({ repositoryName }) {
          yield <div>Cloning repository {repositoryName}...</div>;
          await new Promise(resolve => setTimeout(resolve, 3000));
          yield <div>Building repository {repositoryName}...</div>;
          await new Promise(resolve => setTimeout(resolve, 2000));
          return <div>{repositoryName} deployed!</div>;
        },
      },
    },
  });

  return {
    ui: stream.value,
  };
}
```

##### Beispiel: **@/app/ai.ts (Next.js App Router)**

```tsx
import { createAI } from 'ai/rsc';
import { submitMessage } from '@/app/actions';

export const AI = createAI({
  initialAIState: {},
  initialUIState: {},
  actions: {
    submitMessage,
  },
});
```

##### Beispiel: **@/app/layout.tsx (Next.js App Router)**

```tsx
import { ReactNode } from 'react';
import { AI } from '@/app/ai';

export default function Layout({ children }: { children: ReactNode }) {
  <AI>{children}</AI>;
}
```

##### Beispiel: **@/app/page.tsx (Next.js App Router)**

```tsx
'use client';

import { useActions } from 'ai/rsc';
import { ReactNode, useState } from 'react';

export default function Page() {
  const [input, setInput] = useState('');
  const [messages, setMessages] = useState<ReactNode[]>([]);
  const { submitMessage } = useActions();

  return (
    <div>
      <input
        value={input}
        onChange={event => {
          setInput(event.target.value);
        }}
      />
      <button
        onClick={async () => {
          const { ui } = await submitMessage(input);
          setMessages(currentMessages => [...currentMessages, ui]);
        }}
      >
        Senden
      </button>
    </div>
  );
}
```

## **Templates**

Wir haben [Templates](https://vercel.com/templates?type=ai) erstellt, die AI SDK-Integrationen für verschiedene Anwendungsfälle, Anbieter und Frameworks enthalten. Diese Templates helfen dir, deine AI-Anwendung schnell zu starten.

## **Community**

Die AI SDK-Community findest du auf [GitHub Discussions](https://github.com/vercel/ai/discussions), wo du Fragen stellen, Ideen äußern und deine Projekte mit anderen teilen kannst.

## **Contributing**

Beiträge zum AI SDK sind willkommen und sehr geschätzt. Bevor du loslegst, sieh dir bitte unsere [Beitragsrichtlinien](https://github.com/vercel/ai/blob/main/CONTRIBUTING.md) an, um einen reibungslosen Ablauf sicherzustellen.

## **Autoren**

Diese Bibliothek wurde von [Vercel](https://vercel.com) und Mitgliedern des [Next.js](https://nextjs.org)-Teams erstellt, mit Beiträgen der [Open-Source-Community](https://github.com/vercel/ai/graphs/contributors).
