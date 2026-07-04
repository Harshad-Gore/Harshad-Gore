# Spring AI RAG Orchestrator

## System

A full-stack document-grounded question-answering platform. Users create isolated notebooks, ingest PDF, DOCX, text, or web sources, and query an assistant whose responses are bounded by retrieved notebook context.

```mermaid
flowchart LR
    UI[React client] -->|JWT / REST| API[Spring Boot API]
    API --> Auth[Spring Security]
    API --> Ingest[Ingestion pipeline]
    API --> Query[Retrieval pipeline]
    Ingest --> Files[(S3-compatible storage)]
    Ingest --> Parse[PDFBox / Apache POI]
    Parse --> Embed[Spring AI embeddings]
    Embed --> Vector[(PostgreSQL + pgvector)]
    Query --> Vector
    Query --> Model[OpenAI-compatible model]
    Model -->|answer + citations| UI
```

## Engineering decisions

- Notebook ownership scopes documents, chunks, and conversations.
- Original files and searchable content use separate storage systems.
- Flyway controls schema evolution; JPA validates application mappings.
- Email verification precedes authenticated access.
- RAG responses retain source references instead of returning unsupported prose alone.

## Verified baseline

Validated locally on 4 July 2026.

| Check | Result |
|---|---:|
| Spring production package | Passed without starting the application context |
| React production build | Passed |
| Render runtime configuration | Left untouched by profile verification |

The profile workflow deliberately verifies the deployable artifact rather than bootstrapping the application's external database, mail, storage, and model integrations. DB-backed integration tests remain repository-level work and are not represented as passing CI evidence here.

## Security boundary

- Signed JWT authentication through HTTP-only cookies or bearer headers.
- Explicit CORS origins and stateless server sessions.
- User ownership checks for notebook resources.
- Untrusted document parsing isolated from raw object storage.
- Provider, storage, mail, database, and token secrets supplied through environment variables.

## Open engineering work

- Add containerized PostgreSQL tests for pgvector-specific behavior.
- Align the isolated Flyway fixture and authentication expectations before making backend tests a required profile gate.
- Introduce retrieval evaluation for relevance, citation precision, and faithfulness.
- Split the large frontend bundle and enforce a size budget.
- Finish the full frontend lint backlog before making it a required gate.
