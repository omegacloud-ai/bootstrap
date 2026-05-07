# Bedrock Converse Implementation

Use these examples when implementing Amazon Bedrock Converse calls with Claude Sonnet in
`eu-north-1`.

## Setup

```bash
pnpm install @aws-sdk/client-bedrock-runtime
```

The EC2 instance must have an IAM policy granting `bedrock:InvokeModel` on the target model ARN.
Do not pass credentials in code; the SDK reads the EC2 instance role automatically.

## Single-Turn Call

```typescript
import {
  BedrockRuntimeClient,
  ConverseCommand,
} from "@aws-sdk/client-bedrock-runtime";

const client = new BedrockRuntimeClient({ region: "eu-north-1" });
const MODEL_ID = "eu.anthropic.claude-sonnet-4-6";

async function callClaude(
  prompt: string,
  systemPrompt?: string,
): Promise<string> {
  const input = {
    modelId: MODEL_ID,
    messages: [
      {
        role: "user" as const,
        content: [{ text: prompt }],
      },
    ],
    inferenceConfig: {
      maxTokens: 2000,
      temperature: 0.7,
      topP: 0.9,
    },
    ...(systemPrompt && { system: [{ text: systemPrompt }] }),
  };

  const response = await client.send(new ConverseCommand(input));
  const text = response.output?.message?.content?.[0]?.text;
  if (!text) throw new Error("Empty response from Bedrock");
  return text;
}
```

## Multi-Turn Conversation

```typescript
import {
  BedrockRuntimeClient,
  ConverseCommand,
  Message,
} from "@aws-sdk/client-bedrock-runtime";

const client = new BedrockRuntimeClient({ region: "eu-north-1" });
const MODEL_ID = "eu.anthropic.claude-sonnet-4-6";

class Conversation {
  private history: Message[] = [];
  private systemPrompt?: string;

  constructor(systemPrompt?: string) {
    this.systemPrompt = systemPrompt;
  }

  async send(userMessage: string): Promise<string> {
    const userTurn: Message = {
      role: "user",
      content: [{ text: userMessage }],
    };
    const nextHistory = [...this.history, userTurn];

    const input = {
      modelId: MODEL_ID,
      messages: nextHistory,
      inferenceConfig: { maxTokens: 2000, temperature: 0.7, topP: 0.9 },
      ...(this.systemPrompt && { system: [{ text: this.systemPrompt }] }),
    };

    const response = await client.send(new ConverseCommand(input));
    const assistantText = response.output?.message?.content?.[0]?.text;
    if (!assistantText) throw new Error("Empty response from Bedrock");

    const assistantTurn: Message = {
      role: "assistant",
      content: [{ text: assistantText }],
    };
    this.history = [...nextHistory, assistantTurn];

    return assistantText;
  }

  reset() {
    this.history = [];
  }
}
```

## Structured JSON Output

```typescript
function validatePrompt(input: string): string {
  const trimmed = input.trim();
  if (!trimmed) throw new Error("Prompt must not be empty");
  if (trimmed.length > 10_000)
    throw new Error("Prompt exceeds 10 000 character limit");
  return trimmed;
}

async function callClaudeJson<T>(
  prompt: string,
  systemPrompt: string,
): Promise<T> {
  const raw = await callClaude(validatePrompt(prompt), systemPrompt);
  const cleaned = raw
    .replace(/^```(?:json)?\n?/, "")
    .replace(/\n?```$/, "")
    .trim();

  try {
    return JSON.parse(cleaned) as T;
  } catch {
    throw new Error(`Model returned invalid JSON: ${cleaned.slice(0, 200)}`);
  }
}
```
