# 11 Claude Code Skills Every Software Engineer Should Know

Claude Code has evolved from a simple terminal-based coding assistant into a full-fledged agentic platform. With the introduction of **Skills** — self-contained packages of instructions, scripts, and resources that Claude loads dynamically — engineers can now automate entire categories of work that used to eat hours out of every week.

This article walks through the **11 most useful Claude Code skills** for software engineers, covering everything from codebase exploration to security auditing to infrastructure automation. Each section includes what the skill does, how to set it up, and a practical example of it in action.

---

## Table of Contents

1. [Repository Explainer (`/repo-explain`)](#1-repository-explainer-repo-explain)
2. [Automated Code Review (`/code-review`)](#2-automated-code-review-code-review)
3. [Test-Driven Development with Superpowers (`/superpowers:brainstorm`)](#3-test-driven-development-with-superpowers)
4. [Security Auditing with Trail of Bits Skills](#4-security-auditing-with-trail-of-bits-skills)
5. [Smart Commit & PR Automation (`/commit`, `/create-pr`)](#5-smart-commit--pr-automation)
6. [Infrastructure as Code with Terraform Skill](#6-infrastructure-as-code-with-terraform-skill)
7. [Database Schema Design & Migration](#7-database-schema-design--migration)
8. [Documentation Generation & API Docs](#8-documentation-generation--api-docs)
9. [DevOps Pipeline Builder](#9-devops-pipeline-builder)
10. [Dependency Auditing & Code Health](#10-dependency-auditing--code-health)
11. [Blog Writing & Content Generation (`/write-blog`)](#11-blog-writing--content-generation-write-blog)

---

## How Skills Work — A Quick Primer

Before diving in, here's the 30-second version of how Claude Code skills work.

A **skill** is a folder containing a `SKILL.md` file with YAML frontmatter and markdown instructions. Skills can also bundle scripts, reference docs, and asset files. Claude scans available skills at the start of a conversation using only ~100 tokens per skill for metadata. When a skill is relevant, the full instructions load at under 5k tokens, and bundled resources load on demand.

```
.claude/skills/my-skill/
├── SKILL.md           # Required: frontmatter + instructions
├── scripts/           # Optional: executable code
├── references/        # Optional: documentation
└── assets/            # Optional: templates, configs
```

Skills can be **project-scoped** (`.claude/skills/`) and shared via version control, or **personal** (`~/.claude/skills/`) for cross-project utilities. You invoke them explicitly with `/skill-name` or let Claude auto-load them when context matches.

Now, on to the skills.

---

## 1. Repository Explainer (`/repo-explain`)

**What it does:** Given a path to a directory or repository, this skill analyzes the codebase and produces a structured walkthrough — starting from the entrypoint and tracing the flow of control down to the leaf nodes. The output includes an ASCII diagram of the architecture and a step-by-step written explanation of how the system works.

**Why it matters:** Onboarding onto a new codebase is one of the most time-consuming parts of software engineering. Instead of spending days reading files and drawing mental maps, you get a guided tour in seconds.

### Setting It Up

Create the skill at `.claude/skills/repo-explain/SKILL.md`:

```yaml
---
name: repo-explain
description: >
  Explains a repository or directory by tracing the flow of control from
  entrypoint to leaf nodes. Produces an ASCII architecture diagram and a
  step-by-step written walkthrough. Use when user says 'explain this repo',
  'how does this codebase work', or invokes /repo-explain.
user-invocable: true
disable-model-invocation: false
---
```

```markdown
# Repository Explainer

When the user provides a path via `/repo-explain <path>`, perform the following:

## Step 1: Identify the Entrypoint

- Look for `main`, `index`, `app`, `server`, or equivalent entrypoint files.
- If ambiguous, check `package.json` (main/scripts), `Makefile`, `Dockerfile`,
  or framework conventions (e.g., `manage.py` for Django, `cmd/` for Go).

## Step 2: Trace the Flow of Control

Starting from the entrypoint, recursively follow imports/requires/includes to
map the dependency graph. Identify:
- **Initialization**: Config loading, dependency injection, DB connections
- **Routing/Dispatch**: How requests or commands get routed
- **Business Logic**: Core domain operations
- **Data Layer**: Database queries, external API calls, file I/O
- **Output**: Response formatting, logging, event emission

## Step 3: Generate the ASCII Architecture Diagram

Produce a box-and-arrow ASCII diagram showing the flow from entrypoint to leaf
nodes. Use standard box-drawing characters. The diagram should read top-to-bottom
or left-to-right and clearly label each component.

Example format:

```
┌─────────────┐
│  index.ts   │  ← Entrypoint
└──────┬──────┘
       │
       ▼
┌─────────────┐     ┌──────────────┐
│   router.ts │────▶│  middleware/  │
└──────┬──────┘     └──────────────┘
       │
  ┌────┴─────┐
  ▼          ▼
┌──────┐  ┌──────┐
│/users│  │/posts│   ← Route handlers
└──┬───┘  └──┬───┘
   │         │
   ▼         ▼
┌──────────────┐
│  services/   │     ← Business logic
└──────┬───────┘
       │
       ▼
┌──────────────┐
│    db.ts     │     ← Data layer
└──────────────┘
```

## Step 4: Write the Walkthrough

After the diagram, write a structured walkthrough with these sections:

### Starting Point
Explain what happens when the application starts. What gets initialized first?
What config is loaded?

### Request/Command Lifecycle
Trace a typical request or command through the system. What touches what, in
what order?

### Core Domain
Explain the central business logic. What are the key abstractions and data
models?

### Data Flow
How does data enter the system, get transformed, and exit? Include external
dependencies (databases, APIs, message queues).

### Key Design Decisions
Call out notable architectural patterns: dependency injection, event-driven
architecture, middleware chains, plugin systems, etc.

## Formatting Rules

- Use `$ARGUMENTS` as the target path. Default to `.` if no path is provided.
- Always start with the ASCII diagram, then the written walkthrough.
- Use analogies where they help (e.g., "The router acts like a switchboard
  operator, directing incoming requests to the right handler").
- Keep it concrete — reference actual filenames and function names.
```

### Example Usage

```
> /repo-explain ./src

┌──────────────────┐
│    main.ts       │  ← Entrypoint: bootstraps the app
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│    app.ts        │  ← Express app setup, middleware registration
└────────┬─────────┘
         │
    ┌────┴────────────┐
    ▼                 ▼
┌──────────┐    ┌───────────┐
│ routes/  │    │middleware/ │  ← Auth, logging, rate-limiting
└────┬─────┘    └───────────┘
     │
  ┌──┴───┬──────────┐
  ▼      ▼          ▼
┌─────┐┌──────┐┌────────┐
│users││orders││products│     ← Domain route handlers
└──┬──┘└──┬───┘└───┬────┘
   │      │        │
   ▼      ▼        ▼
┌──────────────────────┐
│     services/        │      ← Business logic layer
└──────────┬───────────┘
           │
     ┌─────┴──────┐
     ▼            ▼
┌─────────┐ ┌──────────┐
│   db/   │ │ external/ │     ← Postgres, Redis, Stripe API
└─────────┘ └──────────┘

### Starting Point

The application starts in `main.ts`, which calls `createApp()` from
`app.ts`. This function:

1. Loads environment config from `.env` via `dotenv`
2. Initializes the database connection pool (`db/connection.ts`)
3. Registers global middleware: authentication (`middleware/auth.ts`),
   request logging (`middleware/logger.ts`), and rate limiting
   (`middleware/rateLimit.ts`)
4. Mounts route handlers from `routes/`
5. Starts the HTTP server on the configured port

### Request Lifecycle

A typical API request follows this path:

1. **Ingress** → Express receives the HTTP request
2. **Middleware chain** → `logger` records the request, `rateLimit` checks
   throttle limits, `auth` validates the JWT and attaches `req.user`
3. **Router** → The request is dispatched to the matching route handler
   (e.g., `POST /api/orders` → `routes/orders.ts`)
4. **Controller** → The route handler validates input, then delegates to
   the appropriate service
5. **Service** → `services/orderService.ts` executes the business logic:
   checks inventory, calculates pricing, creates the order record
6. **Data layer** → The service calls `db/orders.ts` to persist to
   Postgres and `external/stripe.ts` to process payment
7. **Response** → The result bubbles back up and is serialized as JSON

### Core Domain

The system is an e-commerce API with three main domain objects:
- **Users** — authentication, profiles, preferences
- **Products** — catalog, inventory, pricing
- **Orders** — cart management, checkout, payment processing

### Key Design Decisions

- **Service layer pattern**: Route handlers are thin — they validate input
  and delegate. All business logic lives in `services/`.
- **Repository pattern**: Database access is abstracted behind `db/`
  modules, making it straightforward to swap Postgres for another store.
- **Middleware composition**: Cross-cutting concerns (auth, logging,
  rate-limiting) are handled via Express middleware, keeping route
  handlers focused on business logic.
```

This skill is especially powerful when combined with Claude's ability to read your actual source files — it doesn't guess at structure, it reads the code and maps it accurately.

---

## 2. Automated Code Review (`/code-review`)

**What it does:** Runs a multi-agent code review on your current branch or pull request. Four specialized agents analyze the diff in parallel — looking at correctness, security, performance, and style — then surface only high-confidence issues (score >= 80).

**Why it matters:** AI-generated code is shipping faster than ever, which means review volume has exploded. This skill catches real logic errors and security issues, not just style nits.

### Built-In Plugin

Claude Code ships with a built-in `/code-review` plugin. No installation needed:

```
> /code-review              # Review locally, output to terminal
> /code-review --comment    # Post results as a PR comment
```

The plugin launches 4 review agents in parallel:

```
┌─────────────────────────────────────────────────────────┐
│                    /code-review                         │
└───────────┬──────────┬──────────┬──────────┬────────────┘
            │          │          │          │
            ▼          ▼          ▼          ▼
      ┌──────────┐┌──────────┐┌────────┐┌────────┐
      │Correctness││ Security ││  Perf  ││ Style  │
      │  Agent   ││  Agent   ││ Agent  ││ Agent  │
      └────┬─────┘└────┬─────┘└───┬────┘└───┬────┘
           │           │          │          │
           ▼           ▼          ▼          ▼
      ┌─────────────────────────────────────────┐
      │  Confidence Scoring (threshold >= 80)   │
      └─────────────────────────┬───────────────┘
                                │
                                ▼
                    ┌───────────────────┐
                    │  Filtered Output  │
                    └───────────────────┘
```

Each agent scores its findings on a confidence scale. Only issues with confidence >= 80 make it to the output, which drastically reduces noise compared to a naive "comment on everything" approach.

### GitHub Actions Integration

For automated reviews on every PR, Anthropic provides `claude-code-action`:

```yaml
# .github/workflows/claude-review.yml
name: Claude Code Review
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

When a PR is opened or updated, Claude analyzes the changes and posts review comments directly on the PR. You can also trigger it with `@claude` in a comment.

### Customizing the Review

The real power comes from customizing what the review looks for. Create a `.claude/skills/code-review-config/SKILL.md` that encodes your team's standards:

```markdown
When reviewing code, always check for:
- All database queries use parameterized statements (no string concatenation)
- Error responses never leak internal stack traces
- New API endpoints have corresponding OpenAPI schema updates
- React components with side effects use proper cleanup in useEffect
- No secrets or API keys in committed code
```

This turns a generic review into one that enforces your team's specific conventions.

---

## 3. Test-Driven Development with Superpowers

**What it does:** The [Superpowers](https://github.com/obra/superpowers) framework by Jesse Vincent provides 20+ battle-tested skills that implement a complete TDD workflow — from brainstorming and design through test-first implementation to debugging and verification.

**Why it matters:** TDD is powerful but demanding. Most engineers skip it under time pressure. Superpowers makes TDD the *path of least resistance* by automating the ceremony while keeping you in control of the design.

### Installation

```
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace
```

### The Workflow

Superpowers provides three core slash commands that map to the stages of development:

```
┌──────────────────────────────────────────────────────────┐
│              Superpowers TDD Workflow                     │
└──────────────────────────────────────────────────────────┘

  /superpowers:brainstorm         /superpowers:write-plan
  ┌───────────────────┐          ┌────────────────────────┐
  │ 1. Explore idea   │          │ 3. Implementation plan │
  │ 2. Ask questions   │───────▶ │ 4. Task breakdown      │
  │ 3. Consider alts   │          │ 5. TDD test ordering   │
  │ 4. Design doc      │          └───────────┬────────────┘
  └───────────────────┘                       │
                                              ▼
                               /superpowers:execute-plan
                               ┌────────────────────────┐
                               │ For each task:         │
                               │  RED:   Write test     │
                               │  GREEN: Implement      │
                               │  REFACTOR: Clean up    │
                               │                        │
                               │ Uses subagents for     │
                               │ parallel task execution│
                               └────────────────────────┘
```

#### Step 1: Brainstorm (`/superpowers:brainstorm`)

This activates *before* any code is written. Claude asks probing questions about your requirements, explores alternative approaches, presents the design in sections for your approval, and saves a design document.

```
> /superpowers:brainstorm Add rate limiting to our API

Claude: Before we start, let me understand the requirements:
1. What granularity? Per-user, per-IP, per-API-key?
2. What are the limits? Requests per minute? Per hour?
3. Should rate limits vary by endpoint or plan tier?
4. What should happen when limits are exceeded? 429 with retry-after?
5. Do you need distributed rate limiting (multi-server)?
...
```

#### Step 2: Plan (`/superpowers:write-plan`)

After brainstorming, Claude produces an implementation plan that emphasizes:
- **Red-Green-Refactor cycle** for every feature
- **YAGNI** (You Aren't Gonna Need It) — no speculative features
- **DRY** — but only when the abstraction is truly earned

#### Step 3: Execute (`/superpowers:execute-plan`)

Claude launches subagents to work through each task. It writes the failing test first, implements just enough to pass, then refactors. This is where the multi-hour autonomous coding sessions happen — Claude can work through a plan without deviation because the design was locked down in the brainstorm phase.

### Key Skills Under the Hood

| Skill | What It Does |
|-------|-------------|
| `tdd` | RED-GREEN-REFACTOR cycle enforcement |
| `async-testing` | Patterns for testing async operations, race conditions |
| `systematic-debugging` | 4-phase root cause analysis: observe, hypothesize, test, verify |
| `verification-before-completion` | Never marks a task done until tests pass |
| `brainstorming` | Structured ideation with alternatives analysis |
| `subagent-driven-development` | Parallel task execution with independent agents |

Because these skills auto-load based on context, you don't have to remember to invoke them. Ask Claude to implement a feature and it automatically brainstorms, plans, then test-drives the implementation.

---

## 4. Security Auditing with Trail of Bits Skills

**What it does:** [Trail of Bits](https://github.com/trailofbits/skills) — one of the top security research firms — has published a comprehensive set of Claude Code skills for security research, vulnerability detection, and audit workflows. The skills integrate with **CodeQL**, **Semgrep**, and SARIF parsing to perform static analysis, variant analysis, and differential code review.

**Why it matters:** Security review is specialized and time-consuming. These skills encode the methodology of professional auditors, making enterprise-grade security analysis accessible to every engineer.

### Installation

```
/plugin marketplace add trailofbits/skills
```

### Available Skills

```
┌───────────────────────────────────────────────────────┐
│           Trail of Bits Security Skills                │
├───────────────────────────────────────────────────────┤
│                                                       │
│  ┌─── Code Auditing ───────────────────────────────┐  │
│  │ • audit-context-building  (architectural review)│  │
│  │ • differential-review     (git history analysis)│  │
│  │ • semgrep-rule-creator    (custom rule writing) │  │
│  │ • sharp-edges             (error-prone API IDs) │  │
│  │ • static-analysis         (CodeQL + Semgrep)    │  │
│  │ • variant-analysis        (cross-codebase vuln) │  │
│  │ • testing-handbook-skills (fuzzers, sanitizers) │  │
│  └─────────────────────────────────────────────────┘  │
│                                                       │
│  ┌─── Verification ────────────────────────────────┐  │
│  │ • constant-time-analysis  (timing side-channels)│  │
│  │ • property-based-testing  (invariant checking)  │  │
│  └─────────────────────────────────────────────────┘  │
│                                                       │
│  ┌─── Smart Contract Security ─────────────────────┐  │
│  │ • building-secure-contracts (6 blockchain scan) │  │
│  │ • entry-point-analyzer     (state-change audit) │  │
│  └─────────────────────────────────────────────────┘  │
│                                                       │
└───────────────────────────────────────────────────────┘
```

### Static Analysis in Practice

The `static-analysis` plugin integrates CodeQL and Semgrep based on the Trail of Bits [Testing Handbook](https://appsec.guide/). When activated, it:

1. **Runs Semgrep** with rules relevant to your language/framework
2. **Runs CodeQL** queries for common vulnerability patterns
3. **Parses SARIF output** and prioritizes findings by severity
4. **Explains each finding** with context on why it matters and how to fix it

```
> Audit the authentication module for security issues

Claude: I'll run a security audit on src/auth/ using the Trail of Bits
static-analysis skill.

[Running Semgrep with auth-related rules...]
[Running CodeQL for injection and auth bypass patterns...]

Found 3 issues:

HIGH: SQL injection in src/auth/login.ts:47
  The `username` parameter is concatenated directly into a SQL query.
  Fix: Use parameterized query with $1 placeholder.

MEDIUM: Timing side-channel in src/auth/verify.ts:23
  String comparison of password hashes uses `===` which is not
  constant-time. An attacker could use timing differences to
  leak hash bytes.
  Fix: Use crypto.timingSafeEqual() for hash comparison.

LOW: Verbose error in src/auth/login.ts:62
  Login failure response includes "user not found" vs "wrong password",
  allowing username enumeration.
  Fix: Return a generic "invalid credentials" message for all failures.
```

### Variant Analysis

The `variant-analysis` skill is particularly powerful — it takes a known vulnerability and searches your entire codebase (and optionally across multiple repos) for similar patterns. This is the same technique security researchers use to find bugs at scale after an initial discovery.

---

## 5. Smart Commit & PR Automation

**What it does:** A collection of skills that automate the git workflow — from writing conventional commit messages to creating branches, formatting code, and submitting pull requests with structured descriptions.

**Why it matters:** Git hygiene is important but tedious. These skills enforce consistent commit messages and PR descriptions without you having to think about formatting.

### The `/commit` Workflow

Claude Code has built-in commit intelligence. When you type `/commit`, it:

1. Analyzes the current diff
2. Reviews recent commit messages to match your project's style
3. Generates a [Conventional Commit](https://www.conventionalcommits.org/) message
4. Commits the changes

```
> /commit

Claude: Analyzing changes...

Staged changes:
  M src/auth/login.ts    (+12, -3)
  M src/auth/verify.ts   (+8, -5)
  A src/auth/__tests__/login.test.ts (+47)

Suggested commit message:
  fix(auth): use parameterized queries and constant-time comparison

  - Replace string concatenation with parameterized SQL queries in login
  - Switch to crypto.timingSafeEqual() for hash comparison in verify
  - Add unit tests for login authentication flow

Proceed? [Y/n]
```

### The `/create-pr` Workflow

The community-built [`/create-pr`](https://github.com/toyamarinyon) skill by toyamarinyon automates the entire PR pipeline:

```
┌────────────────────────────────────────────────────┐
│                  /create-pr                        │
└───────────┬────────────────────────────────────────┘
            │
    ┌───────┴────────┐
    ▼                ▼
┌──────────┐  ┌──────────────┐
│ Create   │  │  Format code │
│ branch   │  │  (Biome/     │
└────┬─────┘  │   Prettier)  │
     │        └──────┬───────┘
     │               │
     ▼               ▼
┌──────────────────────────┐
│     Stage & Commit       │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│     Push & Create PR     │
│  (via gh CLI)            │
└──────────────────────────┘
```

### Structured Commit Hooks

For teams that want *every* commit to follow conventions, you can combine a skill with a Claude Code hook:

```json
// .claude/settings.json
{
  "hooks": {
    "pre-commit": {
      "command": "npx commitlint --edit $1"
    }
  }
}
```

This ensures that even when Claude generates commit messages, they pass your project's commitlint rules.

---

## 6. Infrastructure as Code with Terraform Skill

**What it does:** The [Terraform Skill](https://github.com/antonbabenko/terraform-skill) by Anton Babenko transforms Claude from a general-purpose assistant into a Terraform and OpenTofu specialist. It enforces module patterns, testing strategies, CI/CD workflows, and production-ready infrastructure patterns.

**Why it matters:** Without specific guidance, AI assistants write Terraform that dumps everything into a single `main.tf`, uses default VPCs, and writes `"*"` IAM policies. This skill prevents that.

### Installation

```
npx skills add antonbabenko/terraform-skill
```

### What It Enforces

The skill acts as a senior infrastructure architect sitting between you and Claude. It implements a four-pillar framework:

```
┌────────────────────────────────────────────────────────┐
│              Terraform Skill — Four Pillars             │
├────────────────────────────────────────────────────────┤
│                                                        │
│  1. MODULE PATTERNS                                    │
│     • Proper file organization (variables.tf,          │
│       outputs.tf, main.tf, versions.tf)                │
│     • Input validation with custom conditions          │
│     • Semantic versioning for modules                  │
│                                                        │
│  2. TESTING STRATEGIES                                 │
│     • terraform validate as first gate                 │
│     • terraform plan with review                       │
│     • Integration tests with Terratest                 │
│     • Policy-as-code with OPA/Sentinel                 │
│                                                        │
│  3. CI/CD WORKFLOWS                                    │
│     • Plan on PR, apply on merge                       │
│     • State locking and backend configuration          │
│     • Drift detection schedules                        │
│                                                        │
│  4. PRODUCTION PATTERNS                                │
│     • Least-privilege IAM policies                     │
│     • Network segmentation (no default VPC)            │
│     • Encryption at rest and in transit                │
│     • Tagging standards for cost allocation            │
│                                                        │
└────────────────────────────────────────────────────────┘
```

### Example

```
> Create a Terraform module for an ECS Fargate service

Claude: I'll create a production-ready ECS Fargate module following
Terraform best practices.

modules/ecs-fargate/
├── main.tf          # ECS cluster, service, task definition
├── variables.tf     # Typed inputs with validation
├── outputs.tf       # Service URL, cluster ARN, task role ARN
├── versions.tf      # Required provider versions
├── iam.tf           # Least-privilege task execution role
├── networking.tf    # Security groups, ALB target group
└── README.md        # Auto-generated usage docs

Key decisions:
- Task execution role uses least-privilege (only ECR pull + CloudWatch logs)
- Security group allows inbound only from ALB, not 0.0.0.0/0
- Container insights enabled for observability
- Auto-scaling based on CPU/memory with configurable thresholds
```

Without this skill, Claude would likely create a single `main.tf` with an overly permissive security group and `AdministratorAccess` IAM policy. The skill fixes this class of problem structurally.

---

## 7. Database Schema Design & Migration

**What it does:** A family of skills that help design database schemas, generate migration files, and optimize queries. Implementations exist for PostgreSQL, MySQL, SQLite, and frameworks like Drizzle, Prisma, and Django ORM.

**Why it matters:** Schema design has cascading consequences — a bad decision early on creates technical debt that's expensive to fix. These skills encode best practices for normalization, indexing, and safe migration patterns.

### The SQL Expert Skill

The SQL Expert skill transforms Claude into a database consultant:

```yaml
---
name: sql-expert
description: >
  Expert SQL query writing, optimization, and database schema design.
  Supports PostgreSQL, MySQL, SQLite, and SQL Server. Use when user needs
  help with queries, schema design, migrations, or performance tuning.
---
```

**Capabilities:**
- Writing complex SQL with joins, subqueries, window functions, and CTEs
- Optimizing slow queries (identifying missing indexes, N+1 patterns)
- Designing schemas with proper normalization (1NF through BCNF)
- Generating migration files with rollback support
- Explaining query execution plans

### The Drizzle Migration Skill

For TypeScript projects using [Drizzle ORM](https://orm.drizzle.team/), the Drizzle Migration skill automates the migration workflow:

```
┌──────────────────────────────────────────────────────┐
│           Drizzle Migration Workflow                  │
└──────────────────────────────────────────────────────┘

  1. Modify TypeScript schema files
              │
              ▼
  2. /drizzle-migration
     ┌─────────────────────────────┐
     │  drizzle-kit generate       │ ← Generates SQL from schema diff
     └──────────────┬──────────────┘
                    │
                    ▼
     ┌─────────────────────────────┐
     │  Review generated SQL       │ ← NEVER skip this step
     └──────────────┬──────────────┘
                    │
                    ▼
     ┌─────────────────────────────┐
     │  drizzle-kit migrate        │ ← Apply to database
     └─────────────────────────────┘
```

The key enforcement: **the review step is mandatory**. The skill refuses to skip it because blindly applying generated migrations is how you lose data.

### Atlas CLI Integration

For more advanced schema management, the [Atlas](https://atlasgo.io/guides/ai-tools/claude-code-instructions) skill handles:
- Schema diffing across environments
- Lint checks on migration files (detecting destructive changes)
- Multi-ORM support (GORM, Drizzle, SQLAlchemy, Django, Ent, Sequelize, TypeORM)
- Declarative schema-as-code with versioned migrations

```
> Design a schema for a multi-tenant SaaS application

Claude: Here's a schema design using the shared-database,
shared-schema approach with row-level security:

┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   tenants    │    │    users     │    │  workspaces  │
├──────────────┤    ├──────────────┤    ├──────────────┤
│ id (PK)      │◄──┐│ id (PK)      │    │ id (PK)      │
│ name         │   ││ tenant_id(FK)│◄──┐│ tenant_id(FK)│
│ plan         │   │└──────────────┘   │└──────┬───────┘
│ created_at   │   │                   │       │
└──────────────┘   │                   │       │
                   └───────────────────┘       │
                                               ▼
                                        ┌──────────────┐
                                        │   projects   │
                                        ├──────────────┤
                                        │ id (PK)      │
                                        │ workspace_id │
                                        │ tenant_id(FK)│
                                        └──────────────┘

Row-level security policy on every table:
  CREATE POLICY tenant_isolation ON users
    USING (tenant_id = current_setting('app.current_tenant')::uuid);

Indexes:
  - UNIQUE (tenant_id, email) ON users — email unique per tenant
  - INDEX (tenant_id) ON every table — partition key for RLS
  - INDEX (workspace_id) ON projects — foreign key lookup
```

---

## 8. Documentation Generation & API Docs

**What it does:** Skills that auto-generate documentation from code — including API references, architecture docs, and PR descriptions. Unlike traditional tools like Sphinx or JSDoc that require manual annotations, these skills infer documentation directly from the source.

**Why it matters:** Documentation perpetually lags behind code. These skills make documentation a byproduct of development rather than a separate chore.

### API Doc Generator Skill

```yaml
---
name: api-doc-generator
description: >
  Generates consistent REST API documentation from endpoint definitions.
  Use when user says 'create API docs', 'document this endpoint',
  'generate API reference', or has an OpenAPI spec file.
---
```

The skill reads your route definitions and produces structured documentation:

```
> /api-doc-generator ./src/routes

## POST /api/v1/users

Creates a new user account.

### Request

| Field     | Type   | Required | Description              |
|-----------|--------|----------|--------------------------|
| email     | string | Yes      | Valid email address       |
| password  | string | Yes      | Min 8 chars, 1 uppercase  |
| name      | string | Yes      | Display name              |
| role      | string | No       | Default: "member"         |

### Response (201 Created)

```json
{
  "id": "usr_abc123",
  "email": "user@example.com",
  "name": "Jane Doe",
  "role": "member",
  "created_at": "2026-02-23T10:30:00Z"
}
```

### Error Responses

| Status | Code              | Description                    |
|--------|-------------------|--------------------------------|
| 400    | VALIDATION_ERROR  | Invalid email or weak password |
| 409    | DUPLICATE_EMAIL   | Email already registered       |
| 429    | RATE_LIMITED      | Too many requests              |
```

### PR Documentation Generator

The [Claude Code Automatic PR Documentation Generator](https://github.com/marketplace/actions/claude-code-automatic-pr-documentation-generator) GitHub Action automatically generates comprehensive documentation for merged PRs. It features:

- Smart filtering — only documents PRs that exceed a configurable threshold of changed lines/files
- Customizable tags for categorization
- Runs on your own infrastructure

### Skills as Self-Documenting Runbooks

One of the most compelling use cases: skills themselves serve as living documentation. As [Zack Proser notes](https://zackproser.com/blog/claude-skills-internal-training), the SKILL.md file that Claude executes is the same artifact a human reads. They can't drift apart because they're the same thing. This turns fragile ad-hoc workflows into shareable, reviewable, CI-testable capabilities.

---

## 9. DevOps Pipeline Builder

**What it does:** A collection of skills for building and managing CI/CD pipelines, Docker configurations, Kubernetes manifests, and monitoring setups. Covers GitHub Actions, GitLab CI, Jenkins, Azure Pipelines, and more.

**Why it matters:** CI/CD configuration is boilerplate-heavy but has to be correct — a misconfigured pipeline can deploy broken code or leak secrets. These skills generate production-ready configurations.

### DevOps Engineer Skill

The [DevOps Engineer skill](https://github.com/Jeffallan/claude-skills) from the Fullstack Dev Skills collection gives Claude a senior DevOps persona:

```yaml
---
name: devops-engineer
description: >
  Senior DevOps engineer covering CI/CD pipelines, container management,
  deployment strategies (blue-green, canary, rolling), and infrastructure
  as code across AWS, GCP, and Azure. Use when building pipelines, Docker
  configs, Kubernetes manifests, or cloud infrastructure.
---
```

### What It Covers

```
┌───────────────────────────────────────────────────────────┐
│                 DevOps Pipeline Builder                    │
├───────────────────────────────────────────────────────────┤
│                                                           │
│  CI/CD Pipelines                                          │
│  ├── GitHub Actions workflows                             │
│  ├── GitLab CI/CD configurations                          │
│  ├── Jenkins pipelines (declarative & scripted)           │
│  └── Azure Pipelines YAML                                 │
│                                                           │
│  Containerization                                         │
│  ├── Multi-stage Dockerfiles (minimal image size)         │
│  ├── Docker Compose for local development                 │
│  ├── Container security scanning (Trivy, Snyk)            │
│  └── Registry management (ECR, GCR, ACR)                  │
│                                                           │
│  Kubernetes                                               │
│  ├── Deployment manifests with resource limits            │
│  ├── Horizontal Pod Autoscaler configs                    │
│  ├── NetworkPolicies for pod-to-pod isolation             │
│  ├── RBAC configurations                                  │
│  └── Helm charts with values overrides                    │
│                                                           │
│  Monitoring & Observability                               │
│  ├── Prometheus alert rules (golden signals)              │
│  ├── OpenTelemetry collector configuration                │
│  ├── Grafana dashboard templates                          │
│  └── Incident runbooks with diagnostic steps              │
│                                                           │
│  Deployment Strategies                                    │
│  ├── Blue-green with traffic cutover                      │
│  ├── Canary with progressive rollout                      │
│  ├── Rolling update with health checks                    │
│  └── Rollback procedures with verification                │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

### Example: Multi-Stage Docker + GitHub Actions

```
> Create a CI/CD pipeline for a Node.js API with Docker

Claude: I'll create a production-ready pipeline with multi-stage Docker
build and GitHub Actions.

# Dockerfile (multi-stage)
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

FROM node:20-alpine AS runner
RUN addgroup -g 1001 -S nodejs && adduser -S nextjs -u 1001
WORKDIR /app
COPY --from=builder --chown=nextjs:nodejs /app/dist ./dist
COPY --from=builder --chown=nextjs:nodejs /app/node_modules ./node_modules
USER nextjs
EXPOSE 3000
CMD ["node", "dist/main.js"]

# GitHub Actions pipeline
# → Lint & test on every push
# → Build & push Docker image on main
# → Deploy to staging automatically
# → Deploy to production on manual approval
```

### Kubernetes Security Hardening

The `k8s-security-policies` skill handles Kubernetes-specific security:
- **NetworkPolicies** — default-deny ingress with explicit allow rules
- **Pod Security Standards** — enforcing restricted or baseline profiles
- **RBAC** — least-privilege service account permissions
- **OPA Gatekeeper constraints** — policy enforcement at admission time
- **Service mesh mTLS** — encrypted pod-to-pod communication

---

## 10. Dependency Auditing & Code Health

**What it does:** Skills that scan your project's dependencies for outdated packages, known CVEs, license compliance issues, and code quality problems. Covers multiple languages and package managers.

**Why it matters:** Supply chain attacks are one of the fastest-growing threat vectors. An outdated dependency with a known CVE is a ticking time bomb.

### Comprehensive Audit Skills

The [claude-code-skills](https://github.com/levnikolaevich/claude-code-skills) collection by levnikolaevich includes **28 audit skills** organized into 7 groups:

```
┌───────────────────────────────────────────────────┐
│            Dependency & Code Health Audits         │
├───────────────────────────────────────────────────┤
│                                                   │
│  1. SECURITY VULNERABILITIES                      │
│     • Known CVE scanning across dependencies      │
│     • OWASP Top 10 pattern detection              │
│     • Secret detection in committed code          │
│                                                   │
│  2. BUILD HEALTH                                  │
│     • Broken dependency resolution                │
│     • Circular dependency detection               │
│     • Build reproducibility checks                │
│                                                   │
│  3. CODE PRINCIPLES                               │
│     • DRY violations (duplicated logic)           │
│     • KISS violations (unnecessary complexity)    │
│     • YAGNI violations (speculative features)     │
│                                                   │
│  4. CODE QUALITY                                  │
│     • Cyclomatic complexity hotspots              │
│     • Dead code detection                         │
│     • Code smell identification                   │
│                                                   │
│  5. DEPENDENCIES                                  │
│     • Outdated package detection                  │
│     • License compliance (GPL, MIT, Apache)       │
│     • Upgrade path analysis with breaking changes │
│                                                   │
│  6. TEST SUITE QUALITY                            │
│     • Coverage gap analysis                       │
│     • Flaky test detection                        │
│     • Missing edge case identification            │
│                                                   │
│  7. ARCHITECTURAL PATTERNS                        │
│     • Layer violation detection                   │
│     • Dependency direction analysis               │
│     • Module coupling assessment                  │
│                                                   │
└───────────────────────────────────────────────────┘
```

### Example: Full Project Health Check

```
> Run a full dependency and code health audit

Claude: Running 28 audit checks across 7 categories...

SECURITY (2 issues)
  ✗ HIGH: lodash@4.17.20 — CVE-2021-23337 (prototype pollution)
    Fix: npm install lodash@4.17.21
  ✗ MEDIUM: jsonwebtoken@8.5.1 — CVE-2022-23529 (insecure default)
    Fix: npm install jsonwebtoken@9.0.0

DEPENDENCIES (3 issues)
  ✗ 47 packages are outdated (12 major, 18 minor, 17 patch)
  ✗ License conflict: package `gpl-lib` is GPL-3.0, your project is MIT
  ✗ Unused dependency: `moment` is in package.json but not imported

CODE QUALITY (2 issues)
  ✗ src/utils/parser.ts — Cyclomatic complexity 34 (threshold: 15)
  ✗ src/legacy/adapter.ts — 147 lines of dead code (unreachable branches)

TEST SUITE (1 issue)
  ✗ Coverage gap: src/services/payment.ts has 0% test coverage

ARCHITECTURE (1 issue)
  ✗ Layer violation: src/routes/users.ts imports directly from
    src/db/queries.ts, bypassing the service layer

Summary: 9 issues found (2 high, 3 medium, 4 low)
```

### Multi-Language Support

The dependency scanning skills support multiple ecosystems:

| Language | Package Manager | Lock File |
|----------|----------------|-----------|
| JavaScript/TypeScript | npm, yarn, pnpm | package-lock.json, yarn.lock |
| Python | pip, poetry, pipenv | requirements.txt, poetry.lock |
| Go | go modules | go.sum |
| Rust | cargo | Cargo.lock |
| Java | Maven, Gradle | pom.xml, build.gradle |
| Ruby | Bundler | Gemfile.lock |

---

## 11. Blog Writing & Content Generation (`/write-blog`)

**What it does:** A skill that turns Claude into a technical blog writer. Given a topic, it researches the subject, outlines the post, writes draft content with code examples, and iterates based on your feedback — all while maintaining your voice and targeting your audience.

**Why it matters:** Engineers have valuable knowledge to share but rarely have time to write. The gap between "I should blog about this" and a published post is enormous. This skill closes that gap by handling the heavy lifting — research, structure, formatting — while you focus on the ideas.

### Setting It Up

Create the skill at `.claude/skills/write-blog/SKILL.md`:

```yaml
---
name: write-blog
description: >
  Writes technical blog posts. Handles research, outlining, drafting, and
  revision. Supports markdown output with code examples, diagrams, and
  structured sections. Use when user says 'write a blog post', 'draft an
  article', or invokes /write-blog.
user-invocable: true
disable-model-invocation: false
---
```

```markdown
# Blog Writer

When the user invokes `/write-blog <topic>`, follow this workflow:

## Step 1: Research & Scope

- Search the web for recent content, documentation, and discussions on the topic
- Read any relevant files in the current project that relate to the topic
- Identify the target audience (beginners, intermediate, advanced)
- Determine the angle — what makes this post different from what already exists?

## Step 2: Outline

Present a structured outline for approval before writing. Include:
- **Title** — concise, specific, searchable
- **Hook** — why should the reader care? (1-2 sentences)
- **Sections** — 4-8 major sections with bullet points for each
- **Code examples** — list what examples will be included
- **Audience** — who is this for?
- **Estimated length** — word count target

Wait for user approval before proceeding.

## Step 3: Draft

Write the full post following these rules:
- Lead with the problem, not the solution
- Use concrete examples over abstract explanations
- Include runnable code snippets (not pseudocode)
- Add ASCII diagrams for architecture and workflows
- Keep paragraphs short (3-4 sentences max)
- Use section headers that tell a story when read in sequence
- End with actionable next steps, not a generic conclusion

## Step 4: Revision

After presenting the draft:
- Ask for specific feedback on tone, depth, and accuracy
- Offer to expand, condense, or restructure any section
- Check all code examples for correctness
- Verify all external links are valid

## Formatting Rules

- Output as GitHub-flavored markdown
- Use `$ARGUMENTS` as the topic. If no topic provided, ask for one.
- Include frontmatter (title, date, tags, description) for static site generators
- Use language-tagged fenced code blocks for all code
```

### The Workflow

```
┌───────────────────────────────────────────────────────────┐
│                    /write-blog <topic>                     │
└──────────────────────┬────────────────────────────────────┘
                       │
                       ▼
              ┌─────────────────┐
              │  1. Research     │  ← Web search, codebase scan,
              │                 │    prior art analysis
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │  2. Outline      │  ← Structured outline with
              │                 │    sections, examples, audience
              └────────┬────────┘
                       │
                  ┌────┴────┐
                  ▼         │
            ┌──────────┐    │
            │ User     │    │  ← Approve / revise outline
            │ Approval │────┘
            └────┬─────┘
                 │ ✓
                 ▼
              ┌─────────────────┐
              │  3. Draft        │  ← Full post with code examples,
              │                 │    diagrams, frontmatter
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │  4. Revision     │  ← Tone, accuracy, depth,
              │                 │    link verification
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │  Output .md     │  ← Ready to publish
              └─────────────────┘
```

### Example Usage

```
> /write-blog How we migrated from REST to GraphQL

Claude: I'll research this topic and create an outline.

**Research findings:**
- REST-to-GraphQL migration is a popular topic with many angles
- Most existing posts focus on greenfield GraphQL, not migration
- Key pain points: N+1 queries, schema design, client migration

**Proposed outline:**

# How We Migrated from REST to GraphQL (Without Breaking Everything)

**Audience:** Backend engineers considering or actively doing a migration
**Length:** ~2,500 words

1. **Why we migrated** — the REST pain points that pushed us
2. **The strangler fig pattern** — running REST and GraphQL side-by-side
3. **Schema design decisions** — mapping REST resources to GraphQL types
4. **The N+1 trap** — DataLoader and batching strategies
5. **Client migration** — incremental adoption without a big bang
6. **What we got wrong** — mistakes and lessons learned
7. **Results** — before/after metrics (latency, payload size, DX)

Code examples:
- REST endpoint → equivalent GraphQL resolver
- DataLoader implementation for batching
- Schema stitching for gradual migration

Shall I proceed with this outline, or would you like changes?
```

### Customizing for Your Voice

The skill works best when you add a `references/style-guide.md` that captures your writing voice:

```
.claude/skills/write-blog/
├── SKILL.md
└── references/
    └── style-guide.md    # Your tone, vocabulary, formatting preferences
```

Example style guide:

```markdown
# Writing Style Guide

- Tone: conversational but technical, like explaining to a smart colleague
- Avoid: "leverage", "utilize", "in order to", "it's worth noting"
- Prefer: short sentences, active voice, concrete examples
- Code examples: always runnable, never pseudocode
- Humor: dry, occasional, never forced
- Structure: problem → context → solution → results → next steps
```

Claude loads this reference on demand and adapts its writing to match your voice, so the output reads like *you* wrote it, not an AI.

### Pairing with Other Skills

The blog writing skill composes well with skills already covered:
- **`/repo-explain`** — Generate a "How Our Codebase Works" post from the walkthrough output
- **`/code-review`** — Write a "Lessons from Code Review" post based on common findings
- **API Doc Generator** — Turn API reference docs into a "Getting Started with Our API" tutorial

---

## Putting It All Together

These 11 skills aren't isolated — they compose naturally. A typical development workflow might look like:

```
┌─────────────────────────────────────────────────────────┐
│              Integrated Skill Workflow                   │
└─────────────────────────────────────────────────────────┘

  /repo-explain                      Understand the codebase
       │
       ▼
  /superpowers:brainstorm            Design the feature
       │
       ▼
  /superpowers:execute-plan          TDD implementation
       │
       ▼
  security-audit (auto)              Scan for vulnerabilities
       │
       ▼
  /code-review                       Multi-agent review
       │
       ▼
  dependency-audit (auto)            Check for CVEs
       │
       ▼
  /api-doc-generator                 Update API docs
       │
       ▼
  /commit                            Conventional commit
       │
       ▼
  /create-pr                         Branch, format, push, PR
       │
       ▼
  /write-blog                        Write about what you built
```

Each skill handles its domain while Claude orchestrates the transitions. The result is a development workflow where the tedious parts are automated and the interesting parts — design decisions, code review judgment calls, architectural choices — remain with you.

---

## Getting Started

1. **Start with built-ins:** `/code-review` and `/commit` work out of the box.
2. **Install Superpowers:** The TDD workflow is transformative.
   ```
   /plugin marketplace add obra/superpowers-marketplace
   /plugin install superpowers@superpowers-marketplace
   ```
3. **Add security:** Trail of Bits skills are free and professional-grade.
   ```
   /plugin marketplace add trailofbits/skills
   ```
4. **Build your own:** Start with `/repo-explain` — copy the SKILL.md from this article and customize it for your stack.
5. **Share with your team:** Put skills in `.claude/skills/` and commit them. Everyone who clones the repo gets them automatically.

---

## Resources

- [Claude Code Skills Documentation](https://code.claude.com/docs/en/skills) — Official docs on creating and using skills
- [anthropics/skills](https://github.com/anthropics/skills) — Anthropic's official skills repository
- [awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code) — Curated community list of skills, hooks, and plugins
- [awesome-claude-skills](https://github.com/travisvn/awesome-claude-skills) — Curated skills collection with 5.9k+ GitHub stars
- [obra/superpowers](https://github.com/obra/superpowers) — TDD, debugging, and collaboration skills framework
- [trailofbits/skills](https://github.com/trailofbits/skills) — Security auditing skills with CodeQL and Semgrep
- [antonbabenko/terraform-skill](https://github.com/antonbabenko/terraform-skill) — Terraform and OpenTofu best practices
- [Claude Command Suite](https://github.com/qdhenry/Claude-Command-Suite) — 148+ slash commands and 54 AI agents
- [The Complete Guide to Building Skills for Claude](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf) — Anthropic's comprehensive skill-building guide
- [Claude Code GitHub Actions](https://github.com/anthropics/claude-code-action) — Official GitHub Actions integration
