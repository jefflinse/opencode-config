---
name: ts-project-init
description: Standard TypeScript/Node.js project scaffolding with directory layout, package.json, tsconfig, build tooling, linting, CI pipeline, and Dockerfile
---

## What This Skill Does

Guides scaffolding a new TypeScript/Node.js project with a production-ready structure. Covers directory layout, TypeScript configuration, build tooling, linting, testing setup, containerization, and CI pipeline.

## When To Use

Use this when starting a new TypeScript or Node.js project from scratch — a REST API, CLI tool, React app, Next.js app, or shared library. Do not use this to restructure an existing project.

## Workflow

### Phase 1: Gather Requirements
**Agent**: planner

Collect the information needed to scaffold correctly:
1. Project type: REST API (Express/Fastify/Hono), Next.js app, React SPA, CLI tool, or library
2. Package manager: npm, pnpm, yarn, or bun
3. Runtime: Node.js (which version?), Bun, or Deno
4. Framework and key dependencies
5. Testing framework: Jest or Vitest
6. Whether the organization has existing projects with conventions to match
7. CI platform: GitHub Actions, GitLab CI, or other

**Gate**: Requirements must be confirmed before scaffolding begins.

### Phase 2: Scaffold Project Structure
**Agent**: ts-builder

Create the directory layout and core files based on requirements:

#### REST API Layout
```
.
├── src/
│   ├── index.ts              # Entry point
│   ├── config/               # Configuration loading
│   ├── routes/               # Route handlers
│   ├── middleware/            # Express/Fastify middleware
│   ├── services/             # Business logic
│   ├── repositories/         # Data access
│   ├── types/                # Shared type definitions
│   └── utils/                # Shared utilities
├── tests/
│   ├── unit/
│   └── integration/
├── .env.example
├── .gitignore
├── package.json
├── tsconfig.json
└── README.md
```

#### Next.js App Layout
```
.
├── src/
│   ├── app/                  # App Router pages and layouts
│   ├── components/           # Shared React components
│   ├── lib/                  # Utility functions and shared logic
│   └── types/                # Shared type definitions
├── public/                   # Static assets
├── tests/
├── .env.example
├── .gitignore
├── next.config.ts
├── package.json
├── tsconfig.json
└── README.md
```

**Important**: Research current versions of all dependencies and framework configs before writing files. Do not use versions from memory.

**Acceptance criteria**: `npm run build` (or equivalent) succeeds, `npm run lint` passes, TypeScript compiles with zero errors.

### Phase 3: TypeScript Configuration
**Agent**: ts-builder

Create `tsconfig.json` with strict mode enabled:
```json
{
  "compilerOptions": {
    "strict": true,
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "outDir": "dist"
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

Adjust based on project type (Next.js has its own tsconfig needs, libraries need different module settings).

### Phase 4: Build Tooling
**Agent**: ci-ops

1. **Linting**: ESLint with TypeScript plugin, Prettier for formatting
2. **Dockerfile**: Multi-stage build (build stage with dev deps, production stage with runtime only)
3. **docker-compose.yml**: For local development dependencies (database, cache, etc.)
4. **Scripts in package.json**: `build`, `dev`, `test`, `lint`, `format`, `typecheck`

**Important**: Research current ESLint flat config format, Prettier config, and Docker base image tags before writing.

### Phase 5: CI Pipeline
**Agent**: ci-ops

Create GitHub Actions workflow (`.github/workflows/ci.yml`):
1. Install dependencies (with package manager cache)
2. Type check: `npx tsc --noEmit`
3. Lint: `npm run lint`
4. Test: `npm test`
5. Build: `npm run build`
6. Pin all action versions to specific SHAs

**Important**: Research current SHA-pinned versions of all GitHub Actions before writing.

### Phase 6: Documentation
**Agent**: docs-writer

Create `README.md` with:
1. One-line project description
2. Prerequisites (Node.js version, package manager)
3. Quick-start setup instructions
4. Available scripts and what they do
5. Project structure overview
6. Environment variables

### Phase 7: Review
**Agent**: code-reviewer

1. Verify consistency across all generated files
2. TypeScript compiles with zero errors
3. All scripts work (`build`, `test`, `lint`)
4. No hardcoded secrets or placeholder values
5. `.gitignore` covers all generated artifacts

**Gate**: Present review findings to the user.

## Principles

- **Strict TypeScript from day one**: Enable `strict: true`. It's much harder to add later.
- **Match the ecosystem**: Use the conventions of the chosen framework (Next.js, Express, etc.)
- **Research current versions**: Framework configs, ESLint config format, and Docker images change frequently. Always verify.
- **Minimal but complete**: Include everything needed to develop, test, lint, build, and deploy. Nothing more.
