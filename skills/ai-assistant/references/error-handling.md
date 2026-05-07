# Bedrock Converse Error Handling

Handle documented Bedrock Converse errors by category. Do not retry permission, validation, or
missing-resource errors. Retry transient service, capacity, timeout, and model-readiness errors
with backoff.

```typescript
import { setTimeout } from "timers/promises";

const RETRYABLE_BEDROCK_ERRORS = new Set([
  "ThrottlingException",
  "ServiceUnavailableException",
  "InternalServerException",
  "ModelNotReadyException",
  "ModelTimeoutException",
]);

function isRetryableBedrockError(err: any): boolean {
  if (RETRYABLE_BEDROCK_ERRORS.has(err.name)) return true;
  if (err.name !== "ModelErrorException") return false;

  // ModelErrorException wraps provider-side failures. Retry only transient statuses.
  return [408, 429, 500, 503].includes(err.originalStatusCode);
}

async function callWithRetry(
  fn: () => Promise<string>,
  retries = 3,
): Promise<string> {
  for (let i = 0; i < retries; i++) {
    try {
      return await fn();
    } catch (err: any) {
      switch (err.name) {
        case "ValidationException":
          throw new Error(
            `Bedrock validation error: ${err.message} - check modelId and input shape`,
          );
        case "AccessDeniedException":
          throw new Error(
            "Bedrock access denied - verify the EC2 IAM role has bedrock:InvokeModel",
          );
        case "ResourceNotFoundException":
          throw new Error(
            `Bedrock resource not found: ${err.message} - check modelId or ARN`,
          );
      }

      if (isRetryableBedrockError(err) && i < retries - 1) {
        console.warn(`Retrying Bedrock call after ${err.name}: ${err.message}`);
        await setTimeout(2 ** i * 1000);
        continue;
      }

      throw err;
    }
  }
  throw new Error("Max retries exceeded");
}
```

Documented Converse errors include `AccessDeniedException`, `InternalServerException`,
`ModelErrorException`, `ModelNotReadyException`, `ModelTimeoutException`,
`ResourceNotFoundException`, `ServiceUnavailableException`, `ThrottlingException`, and
`ValidationException`.
