---
name: ai-assistant-implementation
description: Use when building, enhancing, or integrating an AI assistant, chatbot, or LLM-powered feature on this AWS stack - provides Amazon Bedrock Converse API patterns for Claude Sonnet in eu-north-1 using the EC2 IAM role, including conversation state, structured output, UI rendering, and retry handling.
---

# AI Assistant Implementation

## Overview

Build AI assistant features with the Amazon Bedrock Converse API, Claude Sonnet, region
`eu-north-1`, and credentials from the EC2 instance IAM role. Never hardcode AWS credentials or
pass them explicitly.

Model: `eu.anthropic.claude-sonnet-4-6`

## When to Use

Use this skill for Bedrock-backed chatbots, support bots, summarization, classification, Q&A, or
multi-step language-model workflows. Do not use it for purely rule-based automation or non-AWS LLM
providers.

## Core Workflow

1. Validate and bound user input before calling Bedrock.
2. Assemble context in the system prompt or retrieved data, not by trusting raw user text.
3. Call Converse with `modelId`, `messages`, optional `system`, and `inferenceConfig`.
4. Check `response.output?.message?.content?.[0]?.text` and fail clearly if it is empty.
5. Parse structured output defensively; models can return invalid JSON or markdown fences.
6. Execute side effects only after model output is validated.
7. Render assistant UI safely; sanitize markdown/HTML paths.

## Reference Files

Load only the reference needed for the task:

| Need | Read |
| --- | --- |
| Single-turn, multi-turn, input validation, JSON parsing examples | [implementation.md](references/implementation.md) |
| Retry helper and Bedrock Converse error categories | [error-handling.md](references/error-handling.md) |
| Markdown/plain-text rendering and sanitization examples | [ui-rendering.md](references/ui-rendering.md) |
| Official AWS SDK, Converse, model, and IAM links | [sdk.md](references/sdk.md) |

## Quick Reference

| Parameter | Value | Notes |
| --- | --- | --- |
| `region` | `eu-north-1` | Always this region |
| `modelId` | `eu.anthropic.claude-sonnet-4-6` | Claude Sonnet via Bedrock |
| `maxTokens` | 2000 default | Output cap, not input budget |
| `temperature` | 0.0-0.3 factual, 0.5-0.7 general, 0.7-1.0 creative | Bedrock Claude uses `0-1` |
| `topP` | 0.9 | Leave unless tuning deliberately |
| Auth | EC2 IAM role | Requires `bedrock:InvokeModel` |
| System prompt | `system: [{ text }]` | Keep persona, scope, and format constraints here |

## Common Mistakes

| Mistake | Fix |
| --- | --- |
| Hardcoding AWS credentials | Use the EC2 IAM role |
| Wrong region or missing `eu.` prefix | Use `eu-north-1` and `eu.anthropic.claude-sonnet-4-6` |
| Mutating conversation history before success | Commit user and assistant turns only after a valid response |
| High temperature for JSON/code | Use `temperature: 0` or low values |
| Assuming valid JSON | Strip fences and catch `JSON.parse` failures |
| Unbounded history | Trim or summarize long conversations |
| Retrying access, validation, or missing-resource errors | Fix IAM, input shape, model ID, or ARN |
| Rendering model output as unsanitized HTML | Sanitize markdown/HTML or render as plain text |
