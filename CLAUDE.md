# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Initial setup (install deps, generate Prisma client, run migrations)
npm run setup

# Development server (Turbopack)
npm run dev

# Run tests
npm test

# Run a single test file
npx vitest path/to/test.test.ts

# Run tests in watch mode
npx vitest --watch

# Lint
npm run lint

# Build for production
npm run build

# Reset database
npm run db:reset
```

## Architecture

UIGen is an AI-powered React component generator with live preview. Users describe components in a chat interface, and the AI generates code that renders in real-time.

### Core Flow

1. **Chat API** (`src/app/api/chat/route.ts`) - Receives user messages with the current virtual file system state, streams AI responses using Vercel AI SDK with tool calls
2. **AI Tools** - Two tools available to the AI:
   - `str_replace_editor` (`src/lib/tools/str-replace.ts`) - Create files, string replace, insert text
   - `file_manager` (`src/lib/tools/file-manager.ts`) - Rename and delete files
3. **Virtual File System** (`src/lib/file-system.ts`) - In-memory file system that holds all generated code, serialized/deserialized for persistence
4. **JSX Transformer** (`src/lib/transform/jsx-transformer.ts`) - Transforms JSX/TSX to ES modules using Babel standalone, creates blob URLs for import maps
5. **Preview Frame** (`src/components/preview/PreviewFrame.tsx`) - Renders generated components in an iframe with Tailwind CDN, dynamic import maps for local files and esm.sh for npm packages

### Context Providers

- `FileSystemProvider` (`src/lib/contexts/file-system-context.tsx`) - Manages virtual FS state, handles tool call side effects, triggers UI refreshes
- `ChatProvider` (`src/lib/contexts/chat-context.tsx`) - Wraps `useChat` from AI SDK, passes file system state to API

### Data Layer

- SQLite database via Prisma (`prisma/schema.prisma`)
- `User` model for authentication
- `Project` model stores messages and file system data as JSON strings
- Prisma client generated to `src/generated/prisma/`

### Mock Provider

When `ANTHROPIC_API_KEY` is not set, `src/lib/provider.ts` uses a `MockLanguageModel` that generates static demo components (counter, form, card) to allow testing without API costs.

## Key Patterns

- Entry point for generated apps is always `/App.jsx`
- Import alias `@/` maps to virtual FS root
- Third-party packages resolved via esm.sh CDN
- CSS files in virtual FS are collected and injected as `<style>` tags
- Preview uses iframe sandbox with `allow-scripts allow-same-origin allow-forms`
