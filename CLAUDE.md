# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run setup        # First-time setup: install deps, generate Prisma client, run migrations
npm run dev          # Start dev server with Turbopack on port 3000
npm run build        # Production build
npm run lint         # ESLint via Next.js
npm run test         # Run all tests with Vitest
npm run db:reset     # Reset SQLite database to initial state
```

Run a single test file:
```bash
npx vitest run src/lib/__tests__/file-system.test.ts
```

The `ANTHROPIC_API_KEY` env var in `.env` is optional. Without it, the app uses a `MockLanguageModel` that returns static component samples (Counter, Form, Card).

## Architecture

UIGen is a Next.js 15 / React 19 / TypeScript app where users describe React components in plain English and receive live-rendered code via an AI chat interface.

### Request Flow

```
User chat input
  → ChatContext (wraps Vercel AI SDK's useChat)
  → POST /api/chat/route.ts
  → streamText() with Claude (or MockLanguageModel)
  → AI tool calls: str_replace_editor / file_manager
  → VirtualFileSystem updated (in-memory, never disk)
  → FileSystemContext broadcasts changes
  → CodeEditor (Monaco) + PreviewFrame (iframe) re-render
  → Project saved to SQLite via Prisma (authenticated users only)
```

### Key Abstractions

**Virtual File System** (`src/lib/file-system.ts`): All generated files live in memory as a JSON-serializable structure. The client serializes the VFS and sends it with each chat request; the server deserializes, applies AI tool edits, and returns the updated state. Nothing is written to disk.

**AI Provider** (`src/lib/provider.ts`): Checks for `ANTHROPIC_API_KEY` at startup. Present → uses `claude-haiku-4-5`. Absent → `MockLanguageModel`. Both implement the `LanguageModelV1` interface from the Vercel AI SDK.

**AI Tools** (`src/lib/tools/`): Two tools exposed to the model:
- `str_replace_editor` — create files or apply targeted string replacements within files
- `file_manager` — delete or rename files

**Preview Execution** (`src/lib/transform/jsx-transformer.ts`): Compiles JSX with Babel Standalone in the browser, builds an import map for dependencies, and renders in a sandboxed iframe (`src/components/preview/PreviewFrame.tsx`).

**Auth**: JWT sessions in HTTP-only cookies. `src/middleware.ts` protects `/api/projects` and `/api/filesystem`. Server actions for sign-up/sign-in are in `src/actions/index.ts`.

### Directory Map

| Path | Purpose |
|------|---------|
| `src/app/` | Next.js App Router: pages, API routes (`/api/chat`, `/api/projects`, `/api/filesystem`) |
| `src/app/main-content.tsx` | Three-panel shell: chat (left), editor + preview toggle (right) |
| `src/components/chat/` | `ChatInterface`, message rendering, input handling |
| `src/components/editor/` | Monaco-based `CodeEditor`, `FileTree` |
| `src/components/preview/` | `PreviewFrame` (iframe sandbox) |
| `src/lib/contexts/` | `FileSystemContext`, `ChatContext` |
| `src/lib/tools/` | AI tool definitions (`str_replace_editor`, `file_manager`) |
| `src/lib/transform/` | JSX → executable code pipeline |
| `src/actions/` | Server actions: auth, project CRUD |
| `prisma/` | Schema (`User`, `Project`) + SQLite migrations |

### Database Schema

- **User**: `id`, `email`, `hashedPassword`, `createdAt`
- **Project**: `id`, `userId`, `name`, `messages` (JSON), `files` (JSON VFS snapshot), `updatedAt`

### Testing

Tests use Vitest + `@testing-library/react` with a jsdom environment. Test files live alongside source in `__tests__/` subdirectories. The AI SDK and complex browser APIs are mocked at test boundaries.
