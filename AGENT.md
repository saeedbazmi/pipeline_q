# `AGENTS.md`

```markdown
# AutoBuilder Platform — Agent Instructions

> This file defines the architecture, workflow, conventions, and execution rules
> for any AI agent (Cursor, Claude Code, Cline, Factory Droid, Copilot) working
> on the **AutoBuilder** platform — an end-to-end autonomous software factory.

---

## 1. Project Identity

- **Name**: AutoBuilder
- **Vision**: Unified platform that turns a natural-language idea into a
  production-ready, monetizable web product through a multi-agent SOP pipeline.
- **Audience**: Solo founders, non-technical builders, internal tool teams.
- **Differentiators**:
  - Research-first gate (market + ICP validation before any code)
  - Race Mode (parallel multi-model generation with user selection)
  - 7 specialized agents coordinated by LangGraph
  - Native backend + Git sync + Auto-Wiki + AIO/GEO readiness

---

## 2. High-Level Architecture

```
UI Layer  →  Orchestration Layer  →  Agent Layer  →  Integration Layer  →  Data Layer
```

| Layer | Responsibility | Primary Tech |
|---|---|---|
| **UI** | Chat, Visual Editor, Monaco Code Editor, Preview | Next.js 15, Tailwind, Zustand |
| **Orchestration** | Task Graph, State Machine, Race Controller | LangGraph, Redis (BullMQ) |
| **Agents** | 7 specialized workers (Research → Growth) | LiteLLM, MCP Protocol |
| **Integration** | LLMs, Git, Cloud, Payments, External APIs | tRPC, Supabase, Stripe |
| **Data** | Projects, Generated Code, Assets, Analytics | PostgreSQL, R2, ClickHouse |

---

## 3. Tech Stack (Canonical)

```yaml
frontend:
  framework: Next.js 15 (App Router)
  ui: Tailwind CSS + shadcn/ui
  editor: Monaco Editor
  state: Zustand
  api-client: tRPC

backend:
  runtime: Node.js 20 LTS (Fastify)
  db: PostgreSQL 16
  cache: Redis 7
  auth: Supabase Auth
  queue: BullMQ
  storage: Cloudflare R2

agents:
  orchestration: LangGraph
  llm-router: LiteLLM
  protocol: MCP (@modelcontextprotocol/sdk)
  task-queue: BullMQ

devops:
  ci: GitHub Actions
  container: Docker
  deploy: Vercel (FE) + Fly.io (BE)
  monitor: Sentry + OpenTelemetry

analytics:
  store: ClickHouse
  tracking: PostHog
```

---

## 4. Agent Roster

Each agent is a **stateless worker** that receives a Task, produces an Output,
and reports back to the Orchestrator.

| Agent | Role | Inputs | Outputs |
|---|---|---|---|
| **ResearchAgent** | Market, ICP, competitor analysis | Prompt, Industry | Market Report, ICP Brief |
| **ArchitectAgent** | System design, schema, API spec | Product Brief | Architecture Doc, Task Graph |
| **UIAgent** | React components, styling, layout | Arch Doc, Design prefs | `/app` + `/components` tree |
| **BackendAgent** | API, DB, Auth, Stripe | API Spec, Schema | `/api`, migrations, handlers |
| **DevOpsAgent** | Docker, CI/CD, deploy scripts | Codebase | `Dockerfile`, `.github/workflows` |
| **QAAgent** | Unit, E2E, SAST, DAST, a11y | Built app | Test report, coverage |
| **GrowthAgent** | SEO, AIO/GEO, Auto-Wiki, analytics | Live URL | SEO audit, wiki, dashboards |

### Orchestration Rules
- Agents NEVER call each other directly. All hand-offs go through LangGraph.
- Every agent must be idempotent and retry-safe (max 3 retries, exponential backoff).
- Every agent MUST write its output to the project state in the DB before signaling completion.

---

## 5. Unified Pipeline (7 Stages)

Every user request flows through this pipeline. **Do NOT skip stages.**

```
[PROMPT]
   ↓
STAGE 1 — Research & Validation     (ResearchAgent)
   ↓
STAGE 2 — Architecture & Planning   (ArchitectAgent)
   ↓
STAGE 3 — Multi-Model Generation    (Race Controller)
   ↓
STAGE 4 — Iterative Development     (UIAgent + BackendAgent)
   ↓
STAGE 5 — Validation & QA           (QAAgent)
   ↓
STAGE 6 — Deployment                (DevOpsAgent)
   ↓
STAGE 7 — Growth & Monitoring       (GrowthAgent)
   ↓
[LIVE PRODUCT]
```

### Race Mode Rules
- Triggered at Stage 3 for architectural decisions and UI scaffolds.
- Run **4 models in parallel** (default: `gpt-4o`, `claude-opus-4`, `gemini-2-pro`, `llama-3.3-70b`).
- User selects winner; continuation is seeded ONLY from the chosen output.
- Store all 4 outputs for audit trail.

---

## 6. Directory Conventions

```
autobuilder/
├── apps/
│   ├── web/              # Next.js frontend
│   └── api/              # Fastify backend
├── packages/
│   ├── agents/           # All 7 agent implementations
│   ├── orchestration/    # LangGraph graphs + state
│   ├── llm-router/       # LiteLLM wrapper
│   ├── mcp-server/       # MCP protocol tools
│   └── shared/           # Types, utils, constants
├── infra/
│   ├── docker/
│   ├── terraform/
│   └── k8s/
├── docs/                 # Auto-generated wiki lives here
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── AGENTS.md             # ← this file
├── package.json
└── turbo.json
```

### Naming
- Files: `kebab-case.ts`
- Components: `PascalCase.tsx`
- Agent classes: `PascalCaseAgent`
- Env vars: `SCREAMING_SNAKE_CASE`

---

## 7. Code Conventions

- **Language**: TypeScript strict mode everywhere. No `any`.
- **Formatting**: Prettier + ESLint (config in `/packages/shared`).
- **Testing**: Vitest for unit, Playwright for E2E. **Minimum 80% coverage**.
- **Types**: All agent I/O defined in `/packages/shared/types/agents.ts`.
- **Errors**: Wrap in typed `AgentError` classes with `code`, `stage`, `retryable`.
- **Logging**: Use Pino. Every agent call logs `{ agentId, taskId, duration, tokens }`.
- **Secrets**: NEVER hardcoded. Always via `env` or HashiCorp Vault.

### Commit Convention
```
<type>(<agent|scope>): <description>

feat(ui-agent): add responsive layout generator
fix(orchestrator): resume task graph after retry
docs(wiki): auto-generated from Stage 7
```

---

## 8. Environment Variables (Required)

```bash
# Core
DATABASE_URL=
REDIS_URL=
NEXTAUTH_SECRET=

# LLMs (at least one)
OPENAI_API_KEY=
ANTHROPIC_API_KEY=
GOOGLE_AI_API_KEY=
GROQ_API_KEY=

# Integrations
SUPABASE_URL=
SUPABASE_SERVICE_KEY=
GITHUB_TOKEN=
STRIPE_SECRET_KEY=
CLOUDFLARE_R2_ACCESS_KEY=
CLOUDFLARE_R2_SECRET_KEY=

# Observability
SENTRY_DSN=
OTEL_EXPORTER_OTLP_ENDPOINT=
```

---

## 9. Commands

```bash
# Setup
pnpm install
pnpm db:migrate
pnpm db:seed

# Dev
pnpm dev              # runs all apps + workers
pnpm dev:web
pnpm dev:api
pnpm dev:workers      # agent workers

# Agents
pnpm agents:test      # run agent unit tests
pnpm agents:trace     # visualize LangGraph execution

# Quality
pnpm test
pnpm test:e2e
pnpm lint
pnpm typecheck

# Deploy
pnpm deploy:staging
pnpm deploy:prod
```

---

## 10. Quality Gates (MUST PASS before merge)

- [ ] `pnpm typecheck` clean
- [ ] `pnpm lint` clean
- [ ] Unit tests ≥ 80% coverage
- [ ] E2E: happy path for full 7-stage pipeline passes
- [ ] SAST (Semgrep): 0 high/critical findings
- [ ] No secrets in diff (gitleaks)
- [ ] Every new agent has a matching spec in `/packages/agents/specs/`
- [ ] `AGENTS.md` updated if architecture/agent roster changes

---

## 11. Error Handling Protocol

1. Catch → classify as `retryable` / `fatal` / `needs-human`.
2. If `retryable` and `retryCount < 3`: exponential backoff (2s, 4s, 8s) → retry.
3. If `fatal`: persist partial state, notify user, emit `stage_failed` event.
4. If `needs-human`: create Linear issue via MCP, pause pipeline.
5. ALWAYS log to Sentry with `agentId`, `taskId`, `stack`.

---

## 12. Execution Checklist for Any Agent Task

Before writing code, verify:
- [ ] Environment variables loaded
- [ ] DB + Redis reachable
- [ ] Target LLM key valid
- [ ] Task is defined in `TaskGraph` with clear input/output types
- [ ] Idempotency key exists (use `taskId`)
- [ ] Output will be persisted before signaling completion
- [ ] No direct agent-to-agent calls (use Orchestrator)

After writing code:
- [ ] Added/updated types in `/packages/shared/types`
- [ ] Added unit test
- [ ] Updated this file if behavior/architecture changed
- [ ] PR passes all Quality Gates (§10)

---

## 13. Anti-Patterns (DO NOT)

- ❌ Hardcode API keys or secrets
- ❌ Let agents import each other directly
- ❌ Skip Research/Architecture stages to "save time"
- ❌ Store LLM outputs without token/metadata logging
- ❌ Merge code without running `pnpm test:e2e`
- ❌ Mutate shared state without a transaction
- ❌ Use `any` type anywhere
- ❌ Deploy without passing Quality Gates

---

## 14. Success Metrics

| Metric | Target |
|---|---|
| Prompt → Live App | < 10 minutes |
| Pipeline success rate | > 95% |
| Agent response time | < 30s |
| Test coverage | > 80% |
| Security score | A+ (0 vulns) |
| User satisfaction | > 4.5 / 5 |

---

## 15. Agent Quick-Start Prompt Template

When starting any new task, agents should internally frame it as:

```
Given:   {input from TaskGraph}
Produce: {output defined in agent spec}
Using:   {tools listed in agent spec}
Via:     {LLM from LiteLLM router, fallback allowed}
Store:   result in DB under projectId + taskId
Signal:  Orchestrator via BullMQ completion queue
Log:     tokens, duration, errors to OTEL + Sentry
```

---

**Last updated**: 2026-09-21
**Maintainer**: Platform Team
**Review cadence**: Every PR that touches `/packages/agents` or `/packages/orchestration`
```