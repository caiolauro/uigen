# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Setup and Installation
```bash
npm run setup  # Installs dependencies, generates Prisma client, and runs migrations
```

### Development
```bash
npm run dev           # Start development server with turbopack
npm run dev:daemon    # Start dev server in background, logs to logs.txt
```

### Testing
```bash
npm test              # Run test suite with Vitest
```

### Building and Production
```bash
npm run build         # Build production version
npm run start         # Start production server
npm run lint          # Run ESLint
```

### Database Operations
```bash
npm run db:reset      # Reset database (force reset all migrations)
```

## Architecture Overview

UIGen is an AI-powered React component generator with live preview. The application uses a sophisticated virtual file system architecture that allows for real-time code generation and preview without writing files to disk.

### Core Architecture

**Virtual File System**: The heart of the application is a virtual file system (`src/lib/file-system.ts`) that maintains component files in memory. This allows for:
- Real-time code generation and editing
- Live preview updates
- File persistence for authenticated users
- No disk I/O during development

**Dual Context Architecture**: Two main React contexts manage the application state:
- `FileSystemProvider` (`src/lib/contexts/file-system-context.tsx`): Manages virtual files, handles tool calls for file operations
- `ChatProvider` (`src/lib/contexts/chat-context.tsx`): Manages AI chat interactions using Vercel AI SDK

**AI Integration**: Uses Anthropic Claude via custom tools:
- `str_replace_editor` tool: Creates, edits, and modifies files
- `file_manager` tool: Handles file operations (rename, delete)
- Real-time streaming responses with tool execution

### Key Components

**Main Layout** (`src/app/main-content.tsx`):
- Resizable panel layout with chat on left, preview/code on right
- Switches between Preview and Code views
- Manages tab state and user authentication

**Chat Interface** (`src/components/chat/`):
- `ChatInterface`: Main chat container with auto-scroll
- `MessageList`: Renders messages with markdown support and tool execution indicators
- `MessageInput`: Handles user input with loading states

**Preview System** (`src/components/preview/PreviewFrame.tsx`):
- Transforms JSX using Babel standalone
- Creates import maps for module resolution
- Generates blob URLs for in-browser execution
- Handles CSS imports and styling

**Code Editor** (`src/components/editor/`):
- Monaco Editor integration
- File tree navigation
- Syntax highlighting for JSX/TSX

### File Generation Pattern

All generated components follow this pattern:
- Root file must be `/App.jsx` with default export
- Use `@/` import alias for local files (e.g., `import Button from '@/components/Button'`)
- Style with Tailwind CSS classes only
- No HTML files - JSX components only

### Authentication & Persistence

**Anonymous Mode**: Users can work without authentication. Work is tracked in localStorage and can be recovered when signing up.

**Authenticated Mode**: Projects are persisted to SQLite database via Prisma:
- `User` model: Basic user authentication
- `Project` model: Stores serialized messages and virtual file system state

### Development Environment

**Stack**:
- Next.js 15 with App Router
- React 19 with automatic JSX runtime
- TypeScript with strict mode
- Tailwind CSS v4
- Prisma with SQLite
- Vitest for testing

**Special Configuration**:
- Uses `node-compat.cjs` for Node.js compatibility
- Turbopack for fast development builds
- Custom Prisma output directory: `src/generated/prisma`

### API Architecture

**Chat Endpoint** (`src/app/api/chat/route.ts`):
- Streams responses using Vercel AI SDK
- Integrates with virtual file system
- Handles tool execution
- Persists chat history for authenticated users
- Falls back to mock responses when no Anthropic API key is provided

### Testing Strategy

Uses Vitest with React Testing Library:
- Component testing for UI elements
- Context testing for state management
- File system operations testing
- JSX transformation testing