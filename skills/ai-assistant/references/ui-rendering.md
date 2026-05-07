# Assistant UI Rendering

Claude commonly returns markdown. If the product should display markdown, render it through a
sanitizing renderer. If plain text is the desired product behavior, use text-only rendering and
never inject model output as unsanitized HTML.

## React Markdown

```bash
pnpm install react-markdown rehype-sanitize
```

```tsx
import ReactMarkdown from "react-markdown";
import rehypeSanitize from "rehype-sanitize";

function AssistantMessage({ content }: { content: string }) {
  return (
    <ReactMarkdown rehypePlugins={[rehypeSanitize]}>{content}</ReactMarkdown>
  );
}
```

## Non-React HTML

```typescript
import { marked } from "marked";
import DOMPurify from "dompurify";

function renderResponse(content: string): string {
  return DOMPurify.sanitize(marked.parse(content) as string);
}
```

Only assign the returned value to `innerHTML` after sanitization.
