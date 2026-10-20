# Human Contribution Matrix
## Microservices Governance Framework (MGF)

This matrix classifies every substantive file in the repository by its origin.
The 10 `.gitkeep` marker files are excluded (zero-byte placeholders).

## Classification key

| Code | Meaning |
|------|---------|
| **H** | Human original — created directly by the author, without AI drafting assistance |
| **HA** | Human-directed with AI assistance — human decisions, structure and content choices; AI used for drafting, translation or formatting under explicit direction |
| **EXT** | External standard or specification — the file uses or references a third-party standard; the *application and expression* within the file is H or HA as noted |

> All HA files reflect the 70/30 split declared in `AI_USAGE.md`: the architecture,
> structure, section naming and governance rules are human decisions; the prose
> and formatting were produced with AI assistance under human direction.

---

## Root files

| File | Origin | Human decision | AI role | Notes |
|------|--------|----------------|---------|-------|
| `README.md` | HA | Section architecture, dependency graph, build-flow diagram | Prose, Mermaid formatting | The build-flow graph (20 sections + dependencies) is a human design decision |
| `LICENSE` | H | MIT selection, copyright line | None | Standard MIT text with author's name |
| `CONTRIBUTING.md` | HA | Content scope and structure | Prose drafting | |
| `00-sdd-guide.md` | HA | Fill-in order, weekly schedule, priority rules | Prose drafting | |
| `.env.example` | HA | Variable selection and grouping | Formatting | |

---

## `_stacks/`

| File | Origin | Human decision | AI role | Notes |
|------|--------|----------------|---------|-------|
| `_stacks/README.md` | HA | Stack selection (Go, Java, Node, Python) | Prose drafting | |
| `_stacks/go.md` | HA | Folder layout, toolchain rules | Code examples, prose | Go-specific rules under common governance |
| `_stacks/java-spring.md` | HA | Folder layout, toolchain rules | Code examples, prose | |
| `_stacks/node-typescript.md` | HA | Folder layout, toolchain rules | Code examples, prose | |
| `_stacks/python-fastapi.md` | HA | Folder layout, toolchain rules | Code examples, prose | |

---

## `.github/`

| File | Origin | Human decision | AI role | Notes |
|------|--------|----------------|---------|-------|
| `.github/pull_request_template.md` | HA | Required fields and checklist | Prose drafting | |

---

## `00-governance/`

| File | Origin | Human decision | AI role | Notes |
|------|--------|----------------|---------|-------|
| `00-governance/README.md` | HA | Governance model structure | Prose drafting | |
| `00-governance/agile-conventions.md` | HA | Which agile conventions apply and how | Prose drafting | Agile = third-party practices; rules = human |
| `00-governance/definition-of-done.md` | HA | DoD criteria selection and ordering | Prose drafting | |
| `00-governance/definition-of-ready.md` | HA | DoR criteria selection and ordering | Prose drafting | |
| `00-governance/documentation-rules.md` | HA | Mandatory vs. optional docs; update triggers | Prose drafting | |
| `00-governance/git-conventions.md` | HA + EXT | Extended rules on top of Conventional Commits | Prose drafting | See THIRD_PARTY_NOTICES §7 |
| `00-governance/microservices-documentation.md` | HA | What must be documented per service | Prose drafting | |
| `00-governance/security-policy.md` | HA | Security principles selection and integration | Prose drafting | |
| `00-governance/security-rules.md` | HA | Specific security rules for the framework | Prose drafting | |

---

## `01-context/`

| File | Origin | Human decision | AI role | Notes |
|------|--------|----------------|---------|-------|
| `01-context/README.md` | HA | Section purpose and expected content | Prose drafting | |
| `01-context/glossary.md` | HA | Terms selected and defined | Prose drafting | |
| `01-context/overview.md` | HA | Structure of the system overview | Prose drafting | |
| `01-context/scope.md` | HA | Scope boundaries and inclusions/exclusions | Prose drafting | |

---

## `02-domain/`

| File | Origin | Human decision | AI role | Notes |
|------|--------|----------------|---------|-------|
| `02-domain/README.md` | HA | DDD elements prescribed for the framework | Prose drafting | DDD = EXT; prescription = H |
| `02-domain/domain-events.md` | HA + EXT | Event catalogue structure | Prose drafting | See THIRD_PARTY_NOTICES §1 |
| `02-domain/domain-map.md` | HA + EXT | How to map bounded contexts in this framework | Prose drafting | |
| `02-domain/entities-and-rules.md` | HA + EXT | Entity/VO structure prescribed by the author | Prose drafting | |

---

## `03-product/`

| File | Origin | Human decision | AI role | Notes |
|------|--------|----------------|---------|-------|
| `03-product/README.md` | HA | Product section scope | Prose drafting | |
| `03-product/problem-framing.md` | HA | Framing structure and required fields | Prose drafting | |
| `03-product/vision.md` | HA | Vision document structure | Prose drafting | |

---

## `04-requirements/`

| File | Origin | Human decision | AI role | Notes |
|------|--------|----------------|---------|-------|
| `04-requirements/README.md` | HA | Requirements section scope | Prose drafting | |
| `04-requirements/_template-hu.md` | H | All fields and structure | None | Original template — no AI in structure or fields |
| `04-requirements/non-functional.md` | HA | NFR categories and quality attributes selected | Prose drafting | |
| `04-requirements/traceability-matrix.md` | HA | Matrix structure and traceability rules | Prose drafting | |
| `04-requirements/user-stories.md` | HA | Story format and acceptance criteria rules | Prose drafting | |

---

## `05-architecture/`

| File | Origin | Human decision | AI role | Notes |
|------|--------|----------------|---------|-------|
| `05-architecture/README.md` | HA | Architecture section scope | Prose drafting | |
| `05-architecture/hexagonal-architecture.md` | HA + EXT | Multi-stack governance rules; written guide | Prose, code examples | See THIRD_PARTY_NOTICES §3 |
| `05-architecture/overview.md` | HA | Overview structure; layer dependencies | Prose drafting | |
| `05-architecture/pattern-guide.md` | HA + EXT | Pattern selection; when-to-use/when-not structure | Prose, code examples | See THIRD_PARTY_NOTICES §4 |
| `05-architecture/decisions/README.md` | HA | ADR practice prescription | Prose drafting | |
| `05-architecture/decisions/_template-adr.md` | H | All fields and structure | None | Original template |
| `05-architecture/decisions/records/ADR-001-idioma-documentacion.md` | H | Decision itself; evaluated alternatives; consequences | None | Real architectural decision of the author |

---

## `06-data/`

| File | Origin | Human decision | AI role | Notes |
|------|--------|----------------|---------|-------|
| `06-data/README.md` | HA | Data section scope | Prose drafting | |
| `06-data/models.md` | HA | Model documentation structure | Prose drafting | |

---

## `07-api/`

| File | Origin | Human decision | AI role | Notes |
|------|--------|----------------|---------|-------|
| `07-api/README.md` | HA | API section scope and contract rules | Prose drafting | |
| `07-api/contracts/openapi/_shared.yaml` | H | Reusable components structure | None | Original expression; see THIRD_PARTY_NOTICES §6 |
| `07-api/contracts/openapi/_template-service.yaml` | H | Template structure and fields | None | Original template |
| `07-api/contracts/openapi/api-gateway.yaml` | HA | Endpoint design and contract structure | YAML formatting | Original contract |
| `07-api/contracts/openapi/auth-service.yaml` | HA | Endpoint design and contract structure | YAML formatting | Original contract |

---

## `08-uml/`

| File | Origin | Human decision | AI role | Notes |
|------|--------|----------------|---------|-------|
| `08-uml/README.md` | HA | UML section scope and diagram rules | Prose drafting | |
| `08-uml/diagram-index.md` | HA | Index structure and required diagrams | Prose drafting | |
| `08-uml/diagrams/source/c4-container-example.md` | HA + EXT | C4 example for the framework context | Mermaid formatting | See THIRD_PARTY_NOTICES §2 |

---

## `09-microservices/`

| File | Origin | Human decision | AI role | Notes |
|------|--------|----------------|---------|-------|
| `09-microservices/README.md` | HA | Microservices section scope | Prose drafting | |
| `09-microservices/service-catalog.md` | HA | Catalogue structure and fields | Prose drafting | |
| `09-microservices/_template/README.md` | H | Template structure | None | Original template |
| `09-microservices/_template/service/README.md` | H | Service README template fields | None | Original template |
| `09-microservices/_template/service/data-model.md` | H | Data model template fields | None | Original template |
| `09-microservices/_template/service/decisions.md` | H | Decisions template fields | None | Original template |
| `09-microservices/_template/service/events.md` | H | Events template fields | None | Original template |
| `09-microservices/_template/service/runbook.md` | H | Runbook template fields | None | Original template |
| `09-microservices/services/01-api-gateway/README.md` | HA | Service design | Prose drafting | Implemented example |
| `09-microservices/services/01-api-gateway/data-model.md` | HA | Data model design | Prose drafting | |
| `09-microservices/services/01-api-gateway/decisions.md` | HA | Architectural decisions | Prose drafting | |
| `09-microservices/services/01-api-gateway/events.md` | HA | Event catalogue | Prose drafting | |
| `09-microservices/services/01-api-gateway/runbook.md` | HA | Operational procedures | Prose drafting | |
| `09-microservices/services/02-auth-service/README.md` | HA | Service design | Prose drafting | Implemented example |
| `09-microservices/services/02-auth-service/data-model.md` | HA | Data model design | Prose drafting | |
| `09-microservices/services/02-auth-service/decisions.md` | HA | Architectural decisions | Prose drafting | |
| `09-microservices/services/02-auth-service/events.md` | HA | Event catalogue | Prose drafting | |
| `09-microservices/services/02-auth-service/runbook.md` | HA | Operational procedures | Prose drafting | |

---

## `10-devops/`

| File | Origin | Human decision | AI role | Notes |
|------|--------|----------------|---------|-------|
| `10-devops/README.md` | HA | DevOps section scope | Prose drafting | |
| `10-devops/environments.md` | HA | Environment rules and promotion criteria | Prose drafting | |
| `10-devops/local-setup.md` | HA | Local setup procedure design | Prose drafting | |

---

## `11-quality/`

| File | Origin | Human decision | AI role | Notes |
|------|--------|----------------|---------|-------|
| `11-quality/README.md` | HA | Quality section scope | Prose drafting | |
| `11-quality/tdd-guide.md` | HA + EXT | TDD applied to microservices + quality gate rules | Prose drafting | See THIRD_PARTY_NOTICES §5 |
| `11-quality/testing-strategy.md` | HA | Test pyramid, coverage rules, tool selection | Prose drafting | |

---

## `12-ux-ui/`

| File | Origin | Human decision | AI role | Notes |
|------|--------|----------------|---------|-------|
| `12-ux-ui/README.md` | HA | UX/UI section scope | Prose drafting | |
| `12-ux-ui/design-system.md` | HA | Design system structure | Prose drafting | |
| `12-ux-ui/navigation-map.md` | HA | Navigation map structure | Prose drafting | |

---

## `13-operations/`

| File | Origin | Human decision | AI role | Notes |
|------|--------|----------------|---------|-------|
| `13-operations/README.md` | HA | Operations section scope | Prose drafting | |
| `13-operations/incident-management.md` | HA | Incident lifecycle and severity rules | Prose drafting | |
| `13-operations/observability.md` | HA | Observability pillars and tooling rules | Prose drafting | |

---

## `14-training/`

| File | Origin | Human decision | AI role | Notes |
|------|--------|----------------|---------|-------|
| `14-training/README.md` | HA | Training section scope | Prose drafting | |
| `14-training/technical-onboarding.md` | HA | Onboarding steps and required reading | Prose drafting | |

---

## `15-project-control/`

| File | Origin | Human decision | AI role | Notes |
|------|--------|----------------|---------|-------|
| `15-project-control/README.md` | HA | Project control section scope | Prose drafting | |
| `15-project-control/risks.md` | HA | Risk categories and response strategies | Prose drafting | |

---

## `99-archive/`

| File | Origin | Human decision | AI role | Notes |
|------|--------|----------------|---------|-------|
| `99-archive/README.md` | HA | Archive section scope | Prose drafting | |

---

## `docs/legal/` (this directory — created on chore/dnda-registration-readiness)

| File | Origin | Human decision | AI role | Notes |
|------|--------|----------------|---------|-------|
| `docs/legal/AUTHORSHIP.md` | H | All content and declarations | None | Derived from informe-autoria-mgf.md |
| `docs/legal/AI_USAGE.md` | H | All content and declarations | None | Derived from informe-autoria-mgf.md |
| `docs/legal/THIRD_PARTY_NOTICES.md` | H | All attributions and original contributions | None | Derived from informe-autoria-mgf.md |
| `docs/legal/HUMAN_CONTRIBUTION_MATRIX.md` | H | Classification of all 88 files | None | This document |
| `docs/legal/VERSION_HISTORY.md` | H | All version records | None | |

---

## Summary

| Code | File count | % of substantive files |
|------|-----------|------------------------|
| H (human original) | 20 | 23% |
| HA (human-directed + AI execution) | 68 | 77% |
| Total substantive | 88 | 100% |

*The 10 `.gitkeep` marker files are excluded from this count (zero-byte, no protectable content).*

---

*Prepared on 26 August 2026 as part of the DNDA registration process.*
*Technical-legal instrument. Registrability is determined by the DNDA.*
