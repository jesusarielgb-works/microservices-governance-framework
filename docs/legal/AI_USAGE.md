# AI Usage Declaration
## Microservices Governance Framework (MGF)

| Field               | Value                                      |
|---------------------|--------------------------------------------|
| **AI tool**         | Claude Sonnet 4.6 (Anthropic)              |
| **Declared split**  | 70% human contribution / 30% AI execution |
| **AI role**         | Execution tool under human direction       |
| **Human role**      | Author — all creative and architectural decisions |
| **Reference commit**| `2aeb921dba2f555a61c38b28fa30dfbf8f48fb6c` |

---

## Purpose of this document

This file declares the nature and scope of AI assistance used in producing the
MGF. It is a transparency instrument for the DNDA registration process and for
any reviewer of the work's provenance.

The declared authorship of this work is **individual and human**. AI was used
exclusively as an execution tool. This document delimits precisely what the tool
did and what the author decided.

---

## What the AI tool did (30% — execution)

The following activities were carried out with AI assistance. In each case the
tool operated under explicit human direction and produced output that the author
reviewed, corrected and approved.

### Commit `76af293` — Initial content generation
- Generated initial drafts of the majority of the Markdown files (11,116 lines
  added) following the structure, sections and rules defined by the author.
- The structure of 20 sections, their names, their dependency order and the
  governance rules were human decisions communicated as instructions to the tool.

### Commit `88fa9c9` — Translation to English + initial diagrams
- Translated the entire content from Spanish to English (6,722 additions,
  5,383 deletions across 84 files).
- The decision to use English as the single language for all artefacts was a
  documented human decision (ADR-001).

### Commit `01d04fe` — Diagram replacement (SVG → Mermaid)
- Replaced inline SVG diagrams with Mermaid syntax for compatibility with
  GitHub and VS Code.
- The decision to use Mermaid was a human decision based on toolchain
  compatibility criteria.

### General across all commits
- Text formatting and Markdown structure inside files.
- Generation of examples and code snippets in the stack guides, under the
  human-defined structure per technology.

---

## What the author decided (70% — all creative direction)

The following elements are exclusively human decisions. They constitute the
original contribution of the work and the basis of its registrability:

| Decision | Description |
|----------|-------------|
| Architecture of 20 sections | Selection, naming and ordering of all sections |
| Inter-section dependency rules | Which section depends on which, and in what direction |
| Integrated governance model | How Git conventions, DoD, DoR and security integrate |
| Multi-stack abstraction | One governance source for Go, Java, Node/TS, Python |
| ADR as prescribed practice | Decision to institutionalise ADRs with template and example |
| OpenAPI contract design | Structure of `_shared`, `_template-service`, `api-gateway`, `auth-service` |
| Microservice template structure | Fields and organisation of the service template |
| English-only language rule | ADR-001: single language across all artefacts |
| Section dependency graph | The build-flow diagram in README |
| Selection of included practices | DDD, C4, hexagonal, GoF, TDD, OpenAPI — and how each is applied |
| Prompt direction | All instructions given to the AI tool were authored by the human |

---

## What the AI tool did NOT do

- The AI did not decide the structure of the framework.
- The AI did not select which practices to include.
- The AI did not define the dependency rules between sections.
- The AI did not design the governance model.
- The AI did not write the OpenAPI contracts from scratch without structural
  direction from the author.
- The AI is not a co-author of the work. It is a tool, comparable to a text
  editor or a compiler: it executes; the author decides.

---

## On prompts not being preserved

The prompts used with Claude Sonnet 4.6 to direct the generation and translation
of content were not systematically archived. Their absence is acknowledged as a
gap in the evidence of creative direction.

This gap is partially offset by:
- The documented existence of a prior version of the framework in Spanish,
  authored by the same person, before the first GitHub commit.
- The structural consistency of the framework: 20 sections with an explicit
  dependency graph reflect a design decision that precedes any specific prompt.
- The author's ability to describe, justify and reproduce the architectural
  decisions that define the work.

---

## Transparency note

The git history of this repository transparently records AI co-authorship via
`Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>` in the relevant
commits. This history is preserved intact. No rebase, filter-branch or any other
history-rewriting operation has been applied to remove or obscure AI
participation.

---

*Prepared on 26 August 2026 as part of the DNDA registration process.*
*Technical-legal instrument. Registrability is determined by the DNDA.*
