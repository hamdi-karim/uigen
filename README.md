# UIGen

AI-powered React component generator with live preview.

## Motivation

Building UI components often requires context-switching between design, code, and preview — slowing down iteration. UIGen lets you describe what you want in plain English and instantly see a live-rendered React component, keeping you in flow. It's built for developers who want to prototype quickly without leaving the browser.

## Prerequisites

- Node.js 18+
- npm

## Quick Start

1. Clone the repo and run `npm run setup` to install dependencies and initialize the database.
2. Optionally add your `ANTHROPIC_API_KEY` to `.env` (the app works without it using a mock model).
3. Run `npm run dev` and open [http://localhost:3000](http://localhost:3000).

## Setup

1. **Optional** Edit `.env` and add your Anthropic API key:

```
ANTHROPIC_API_KEY=your-api-key-here
```

The project will run without an API key. Rather than using a LLM to generate components, static code will be returned instead.

2. Install dependencies and initialize database

```bash
npm run setup
```

This command will:

- Install all dependencies
- Generate Prisma client
- Run database migrations

## Running the Application

### Development

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000)

## Usage

1. Sign up or continue as anonymous user
2. Describe the React component you want to create in the chat
3. View generated components in real-time preview
4. Switch to Code view to see and edit the generated files
5. Continue iterating with the AI to refine your components

## Features

- AI-powered component generation using Claude
- Live preview with hot reload
- Virtual file system (no files written to disk)
- Syntax highlighting and code editor
- Component persistence for registered users
- Export generated code

## Tech Stack

- Next.js 15 with App Router
- React 19
- TypeScript
- Tailwind CSS v4
- Prisma with SQLite
- Anthropic Claude AI
- Vercel AI SDK

## Contributing

Contributions are welcome! To get started:

1. Fork the repository and create a branch for your change.
2. Run `npm run setup` to prepare your local environment.
3. Make your changes and ensure tests pass with `npm run test`.
4. Open a pull request describing what you changed and why.
