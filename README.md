# Microservices Governance Framework

> Generic documentation template · v1.0 · MIT License
> Built by CORHUILA — Distributed Systems / Software Design Program

This repository is a **documentation scaffold** for any microservices project.
It contains no project-specific information. Its purpose is to teach how to document
a software system professionally — showing what goes in each section, why it matters,
and how every document relates to the others.

---

## How to use this framework

1. **Fork** (or copy the structure) into your project repository.
2. **Pick your stack** → read `_stacks/[your-language].md` for folder structure and stack-specific commands.
3. **Follow the fill-in order** → see [`00-sdd-guide.md`](./00-sdd-guide.md) for the weekly schedule and how to split work across team members.
4. **Fill in each file** starting from sections marked as priority (⭐).
5. **Read each `README.md`** before filling in documents in that section — they explain what is expected.
6. **Do not delete instructions** until a document is fully filled in.
7. Once a document is complete and reviewed, remove the `> [!NOTE] INSTRUCTIONS` block.

---

## Build flow — section dependencies

<svg viewBox="0 0 780 420" xmlns="http://www.w3.org/2000/svg" style="max-width:100%;font-family:monospace;font-size:13px">
  <!-- background -->
  <rect width="780" height="420" fill="#0d1117" rx="12"/>

  <!-- Phase labels -->
  <text x="20" y="30" fill="#58a6ff" font-size="11" font-weight="bold">DISCOVERY</text>
  <text x="20" y="110" fill="#58a6ff" font-size="11" font-weight="bold">DESIGN</text>
  <text x="20" y="210" fill="#58a6ff" font-size="11" font-weight="bold">DETAIL</text>
  <text x="20" y="310" fill="#58a6ff" font-size="11" font-weight="bold">IMPL &amp; OPS</text>

  <!-- Governance band -->
  <rect x="100" y="10" width="660" height="390" fill="none" stroke="#30363d" stroke-width="1" rx="8" stroke-dasharray="6,3"/>
  <text x="110" y="395" fill="#484f58" font-size="10">00-governance — applies to the entire project</text>

  <!-- Row 1: Discovery -->
  <rect x="110" y="22" width="120" height="50" rx="6" fill="#1c2029" stroke="#388bfd" stroke-width="1.5"/>
  <text x="170" y="44" fill="#e6edf3" text-anchor="middle" font-weight="bold">01-context</text>
  <text x="170" y="60" fill="#8b949e" text-anchor="middle" font-size="10">vision · scope · glossary</text>

  <rect x="270" y="22" width="120" height="50" rx="6" fill="#1c2029" stroke="#388bfd" stroke-width="1.5"/>
  <text x="330" y="44" fill="#e6edf3" text-anchor="middle" font-weight="bold">02-domain</text>
  <text x="330" y="60" fill="#8b949e" text-anchor="middle" font-size="10">DDD · entities · events</text>

  <rect x="430" y="22" width="120" height="50" rx="6" fill="#1c2029" stroke="#388bfd" stroke-width="1.5"/>
  <text x="490" y="44" fill="#e6edf3" text-anchor="middle" font-weight="bold">03-product</text>
  <text x="490" y="60" fill="#8b949e" text-anchor="middle" font-size="10">PRD · vision · backlog</text>

  <rect x="590" y="22" width="120" height="50" rx="6" fill="#1c2029" stroke="#388bfd" stroke-width="1.5"/>
  <text x="650" y="44" fill="#e6edf3" text-anchor="middle" font-weight="bold">04-requirements</text>
  <text x="650" y="60" fill="#8b949e" text-anchor="middle" font-size="10">user stories · NFRs</text>

  <!-- Arrows row 1 -->
  <line x1="230" y1="47" x2="270" y2="47" stroke="#388bfd" stroke-width="1.5" marker-end="url(#arr)"/>
  <line x1="390" y1="47" x2="430" y2="47" stroke="#388bfd" stroke-width="1.5" marker-end="url(#arr)"/>
  <line x1="550" y1="47" x2="590" y2="47" stroke="#388bfd" stroke-width="1.5" marker-end="url(#arr)"/>

  <!-- Vertical arrow to arch -->
  <line x1="380" y1="72" x2="380" y2="105" stroke="#388bfd" stroke-width="1.5" marker-end="url(#arr)"/>

  <!-- Row 2: Architecture -->
  <rect x="270" y="110" width="120" height="50" rx="6" fill="#1c2029" stroke="#3fb950" stroke-width="1.5"/>
  <text x="330" y="132" fill="#e6edf3" text-anchor="middle" font-weight="bold">05-architecture</text>
  <text x="330" y="148" fill="#8b949e" text-anchor="middle" font-size="10">ADRs · hexagonal · C4</text>

  <!-- From arch to data and api -->
  <line x1="270" y1="135" x2="190" y2="185" stroke="#3fb950" stroke-width="1.5" marker-end="url(#arr)"/>
  <line x1="390" y1="135" x2="490" y2="185" stroke="#3fb950" stroke-width="1.5" marker-end="url(#arr)"/>

  <!-- Row 2b: data + api -->
  <rect x="110" y="190" width="120" height="50" rx="6" fill="#1c2029" stroke="#3fb950" stroke-width="1.5"/>
  <text x="170" y="212" fill="#e6edf3" text-anchor="middle" font-weight="bold">06-data</text>
  <text x="170" y="228" fill="#8b949e" text-anchor="middle" font-size="10">models · migrations</text>

  <rect x="430" y="190" width="120" height="50" rx="6" fill="#1c2029" stroke="#3fb950" stroke-width="1.5"/>
  <text x="490" y="212" fill="#e6edf3" text-anchor="middle" font-weight="bold">07-api</text>
  <text x="490" y="228" fill="#8b949e" text-anchor="middle" font-size="10">OpenAPI · contracts</text>

  <!-- From data+api to microservices -->
  <line x1="230" y1="215" x2="310" y2="285" stroke="#3fb950" stroke-width="1.5" marker-end="url(#arr)"/>
  <line x1="430" y1="215" x2="375" y2="285" stroke="#3fb950" stroke-width="1.5" marker-end="url(#arr)"/>

  <!-- Row 3: microservices -->
  <rect x="270" y="290" width="140" height="50" rx="6" fill="#1c2029" stroke="#d2a8ff" stroke-width="1.5"/>
  <text x="340" y="312" fill="#e6edf3" text-anchor="middle" font-weight="bold">09-microservices</text>
  <text x="340" y="328" fill="#8b949e" text-anchor="middle" font-size="10">catalog · runbooks</text>

  <!-- From microservices to 08, 10, 12 -->
  <line x1="270" y1="315" x2="210" y2="315" stroke="#d2a8ff" stroke-width="1.5" marker-end="url(#arr)"/>
  <line x1="340" y1="340" x2="180" y2="365" stroke="#d2a8ff" stroke-width="1.5" marker-end="url(#arr)"/>
  <line x1="410" y1="315" x2="470" y2="315" stroke="#d2a8ff" stroke-width="1.5" marker-end="url(#arr)"/>
  <line x1="340" y1="340" x2="490" y2="365" stroke="#d2a8ff" stroke-width="1.5" marker-end="url(#arr)"/>

  <!-- Row 3b: 08-uml, 10-devops, 12-ux-ui -->
  <rect x="110" y="295" width="95" height="40" rx="6" fill="#1c2029" stroke="#d2a8ff" stroke-width="1"/>
  <text x="157" y="313" fill="#e6edf3" text-anchor="middle" font-size="11">08-uml</text>
  <text x="157" y="327" fill="#8b949e" text-anchor="middle" font-size="9">diagrams</text>

  <rect x="470" y="295" width="95" height="40" rx="6" fill="#1c2029" stroke="#d2a8ff" stroke-width="1"/>
  <text x="517" y="313" fill="#e6edf3" text-anchor="middle" font-size="11">10-devops</text>
  <text x="517" y="327" fill="#8b949e" text-anchor="middle" font-size="9">CI/CD · envs</text>

  <!-- Row 4: quality, ops, training -->
  <rect x="110" y="355" width="90" height="40" rx="6" fill="#1c2029" stroke="#ffa657" stroke-width="1"/>
  <text x="155" y="373" fill="#e6edf3" text-anchor="middle" font-size="11">11-quality</text>
  <text x="155" y="387" fill="#8b949e" text-anchor="middle" font-size="9">TDD · metrics</text>

  <rect x="430" y="355" width="90" height="40" rx="6" fill="#1c2029" stroke="#ffa657" stroke-width="1"/>
  <text x="475" y="373" fill="#e6edf3" text-anchor="middle" font-size="11">13-ops</text>
  <text x="475" y="387" fill="#8b949e" text-anchor="middle" font-size="9">observability</text>

  <rect x="580" y="355" width="100" height="40" rx="6" fill="#1c2029" stroke="#ffa657" stroke-width="1"/>
  <text x="630" y="373" fill="#e6edf3" text-anchor="middle" font-size="11">14-training</text>
  <text x="630" y="387" fill="#8b949e" text-anchor="middle" font-size="9">onboarding</text>

  <!-- Arrow marker -->
  <defs>
    <marker id="arr" markerWidth="8" markerHeight="8" refX="6" refY="3" orient="auto">
      <path d="M0,0 L0,6 L8,3 z" fill="#484f58"/>
    </marker>
  </defs>
</svg>

---

## Section index

| # | Folder | Purpose | Phase |
|---|--------|---------|-------|
| 00 | [00-governance](./00-governance/README.md) | Team rules: Git, naming, DoD/DoR, security | ⭐ First |
| 01 | [01-context](./01-context/README.md) | Why the system exists: vision, scope, glossary | ⭐ First |
| 02 | [02-domain](./02-domain/README.md) | The business problem: entities, rules, domain events | ⭐ First |
| 03 | [03-product](./03-product/README.md) | What to build: PRD, product vision, initial backlog | ⭐ First |
| 04 | [04-requirements](./04-requirements/README.md) | What the system must do: functional & non-functional | ⭐ First |
| 05 | [05-architecture](./05-architecture/README.md) | How the system is organized: ADRs, deployment, patterns | 🔵 Design |
| 06 | [06-data](./06-data/README.md) | How data is stored: models, dictionary, migrations | 🔵 Design |
| 07 | [07-api](./07-api/README.md) | Service contracts: OpenAPI, authentication, REST guidelines | 🔵 Design |
| 08 | [08-uml](./08-uml/README.md) | Diagrams: class, sequence, component, ER | 🔵 Design |
| 09 | [09-microservices](./09-microservices/README.md) | Each microservice documented individually | 🟢 Impl |
| 10 | [10-devops](./10-devops/README.md) | CI/CD, environments, local setup, release process | 🟢 Impl |
| 11 | [11-quality](./11-quality/README.md) | Testing strategy, code review, metrics | 🟢 Impl |
| 12 | [12-ux-ui](./12-ux-ui/README.md) | Interface design: design system, flows, wireframes | 🔵 Design |
| 13 | [13-operations](./13-operations/README.md) | Production operations: observability, incidents, SLA/SLO | 🟠 Ops |
| 14 | [14-training](./14-training/README.md) | User manuals, admin guides, technical onboarding | 🟠 Ops |
| 15 | [15-project-control](./15-project-control/README.md) | Risks, dependencies, open questions, tech backlog | 🔵 Design |
| 99 | [99-archive](./99-archive/README.md) | Deprecated decisions and documents | — |

---

## Stack guides

Choose the guide that matches your project's primary language:

| Stack | File |
|-------|------|
| Java + Spring Boot | [`_stacks/java-spring.md`](./_stacks/java-spring.md) |
| Go | [`_stacks/go.md`](./_stacks/go.md) |
| Node.js + TypeScript | [`_stacks/node-typescript.md`](./_stacks/node-typescript.md) |
| Python + FastAPI | [`_stacks/python-fastapi.md`](./_stacks/python-fastapi.md) |

---

## The golden rule of documentation

> **A document nobody reads is a document that does not exist.**
>
> Before creating a document, ask: who will read it? when? what decision does it help them make?
> If you cannot answer all three questions, do not create it yet.

---

## Key cross-section dependencies

| If you change... | You must also review... |
|-----------------|------------------------|
| Scope (01-context) | PRD (03), requirements (04), architecture overview (05) |
| A domain entity (02) | Data models (06), API contracts (07), UML diagrams (08) |
| A functional requirement (04) | Acceptance criteria, test cases (11), user stories (03) |
| Architecture (05) | ADRs (05/decisions), each affected microservice (09) |
| A data model (06) | Owner service API contract (07, 09), ER diagrams (08) |
| An API contract (07) | Owner microservice (09), contract consumers (09) |
| A microservice (09) | dependency-map (09), event-catalog (09), data-ownership-matrix (09) |
| CI/CD pipeline (10) | Release checklist (10), environments (10) |

---

## Naming conventions

- **Content files:** `kebab-case.md`
- **Templates:** `_template-name.md` (leading `_` so they sort first)
- **ADRs:** `ADR-NNN-short-title.md` (sequential numbering)
- **OpenAPI contracts:** `service-name.yaml`

---

## Methodologies included

| Methodology | Primary document | Section |
|-------------|-----------------|---------|
| **SDD** (Software Design Documentation) | [`00-sdd-guide.md`](./00-sdd-guide.md) | Root |
| **DDD** (Domain-Driven Design) | [`02-domain/domain-map.md`](./02-domain/domain-map.md) | 02-domain |
| DDD — Entities, VOs, Aggregates | [`02-domain/entities-and-rules.md`](./02-domain/entities-and-rules.md) | 02-domain |
| DDD — Domain events | [`02-domain/domain-events.md`](./02-domain/domain-events.md) | 02-domain |
| **Hexagonal Architecture** | [`05-architecture/hexagonal-architecture.md`](./05-architecture/hexagonal-architecture.md) | 05-architecture |
| **Patterns** (GoF + Microservices) | [`05-architecture/pattern-guide.md`](./05-architecture/pattern-guide.md) | 05-architecture |
| **TDD** (Test-Driven Development) | [`11-quality/tdd-guide.md`](./11-quality/tdd-guide.md) | 11-quality |
| TDD — Strategy and pyramid | [`11-quality/testing-strategy.md`](./11-quality/testing-strategy.md) | 11-quality |

---

## Reference resources

- [adr.github.io](https://adr.github.io/) — Architecture Decision Records
- [12factor.net](https://12factor.net/) — Twelve-factor app best practices
- [OpenAPI Specification](https://swagger.io/specification/) — REST contract standard
- [C4 Model](https://c4model.com/) — Architecture diagrams (System, Container, Component, Code)
- [Domain-Driven Design Reference](https://www.domainlanguage.com/ddd/reference/) — Eric Evans
- [Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture/) — Alistair Cockburn
- [Test Driven Development](https://www.amazon.com/Test-Driven-Development-Kent-Beck/dp/0321146530) — Kent Beck

---

## License

MIT © 2026 Jesús Ariel González Bonilla — CORHUILA
