# Authorship Declaration
## Microservices Governance Framework (MGF)

| Field                  | Value                                                                |
|------------------------|----------------------------------------------------------------------|
| **Author**             | Jesús Ariel González Bonilla                                         |
| **Contact**            | ariel5253@hotmail.com                                                |
| **GitHub**             | github.com/ariel5253 · github.com/jesusarielgb-works                |
| **Work type**          | Individual authorship                                                |
| **Nature of the work** | Literary work of a technical character — manual and governance system |
| **Version**            | 1.0.0                                                                |
| **Reference commit**   | `2aeb921dba2f555a61c38b28fa30dfbf8f48fb6c`                          |
| **Repository**         | jesusarielgb-works/microservices-governance-framework                |
| **License**            | MIT                                                                  |
| **Year**               | 2026                                                                 |

---

## Purpose of this document

This file is a formal authorship declaration prepared as part of the registration
process before Colombia's Dirección Nacional de Derecho de Autor (DNDA).
It states what the work is, who created it, and what the protected original
contribution consists of.

This is a technical-legal instrument; the final determination of registrability
rests with the DNDA. Validation by a specialist in copyright law is recommended
before filing.

---

## What the MGF is

The *Microservices Governance Framework* is a manual and governance system for
building microservices. It organizes a set of practices, patterns and standards
into a guided path — from understanding the business domain to operating services
in production — through 20 numbered sections with explicit dependency rules
between them.

The framework is not a collection of articles. It is a **construction model**:
a structured sequence of decisions, artefacts and governance rules that a team
can follow to build a software system in an ordered, traceable and reproducible
way.

---

## What is original and protected

The MGF does not claim ownership of the concepts it uses. DDD, C4, hexagonal
architecture, GoF patterns, TDD, OpenAPI and Conventional Commits are the work
of their respective authors or belong to the public domain. They are expressly
attributed in `THIRD_PARTY_NOTICES.md`.

The original contribution — and the object of this work — is **the orchestration**:

1. **Documentary architecture of 20 sections with explicit dependency rules.**
   The numbered structure and its dependency graph (domain → product →
   requirements → architecture → API → quality → operations) is a design
   decision of the author that does not exist in any source individually.

2. **Integrated governance model.**
   Git conventions, Definition of Done, Definition of Ready, security and
   documentation are presented as a unified system, not as separate pieces.
   The integration — which rule governs which artefact, in which order, with
   which acceptance criterion — is the original contribution.

3. **Inter-layer dependency rule.**
   The framework defines which sections can reference which others and in what
   direction. This is an information-architecture choice that imposes intellectual
   order on heterogeneous material.

4. **Multi-stack mapping over a single governance source.**
   Go, Java Spring, Node/TypeScript and Python FastAPI are covered from a single
   governance source, using a stack-specific guide per technology. The
   abstraction "one governance, multiple stacks" and its concrete expression is
   the author's design.

5. **ADR as an institutionalised practice.**
   The framework prescribes Architecture Decision Records as a required habit,
   with template and functional example integrated into the flow.

6. **OpenAPI contracts and reusable template.**
   Four contracts (`_shared`, `_template-service`, `api-gateway`,
   `auth-service`) and a reusable template are original written expressions of
   the author.

7. **Microservice templates and implemented examples.**
   The complete service template (README, data-model, decisions, events, runbook)
   and two implemented examples are original text.

8. **The construction model that unifies everything.**
   The narrative thread that explains how to move from the business domain to
   operations — what is done first, what depends on what, how each phase is
   closed — is the highest-level creation and the one that gives the work its
   unity.

---

## Formal declaration

> I, **Jesús Ariel González Bonilla**, declare that I am the sole author of the
> *Microservices Governance Framework*, a literary work of a technical character
> consisting of a manual and governance system for building microservices.
>
> The work integrates concepts, methods and standards pre-existing from third
> parties — Domain-Driven Design, the C4 model, hexagonal architecture, GoF
> design patterns, Test-Driven Development, OpenAPI and Conventional Commits —
> whose authorship I expressly acknowledge in favour of their respective authors,
> and over which I assert no right whatsoever.
>
> My original contribution, and the object of this work, consists of the
> **selection, coordination, arrangement and expression** of those elements in a
> new structure: a documentary architecture of 20 sections with explicit
> dependency rules, an integrated governance model, original templates and
> contracts, and a construction model that unifies the whole into a guided and
> reproducible journey. This orchestration and its concrete expression are my
> own creation.
>
> In producing the work I used artificial intelligence tools (Claude Sonnet 4.6)
> exclusively as an instrument of execution (drafts, translation and formatting)
> under my intellectual direction, retaining at all times the decisions of
> architecture, structure, selection and content. The authorship of all creative
> decisions is entirely human and my own.

---

*Prepared on 26 August 2026 as part of the DNDA registration process.*
*Technical-legal instrument. Registrability is determined by the DNDA.*
