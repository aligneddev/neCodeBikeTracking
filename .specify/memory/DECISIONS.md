# Bike Tracking Application — Decision Record

This document preserves the amendment history and decision rationale for the [Bike Tracking Constitution](./constitution.md). Refer to this file when understanding why specific architectural choices were made and how the project governance has evolved.

## Key Decision Rationales

### Why Specification-Driven Development (SDD)?

The Bike Tracking project uses SpecKit to enforce Specification-Driven Development. Each feature is captured in a specification document before coding begins. Benefits:

- **Clarity**: Spec approval removes ambiguity before implementation
- **Testing discipline**: Test plan approved upfront; tests written idempotently
- **Accountability**: User signs off on specifications and acceptance criteria
- **Traceability**: Each commit references a spec; easy to understand why code exists
- **Quality gates**: Code review verifies architecture alignment with constitution

### Why Event Sourcing + CQRS?

Bike Tracking tracks rides, expenses, and savings over time. Event Sourcing enables:

- **Complete audit trail**: Every change (record ride, edit distance, recalculate savings) stored as immutable event
- **Temporal queries**: "Show me rides in March 2024" via event filters  
- **Replay & debugging**: Event replay reconstructs state at any point in time
- **CQRS separation**: Write path (append events) faster than read path (query projections)
- **Future analytics**: Events preserve data for future analyses (e.g., weather correlations, motivation patterns)

Trade-offs: Added complexity (separate event/projection tables, eventual consistency, schema versioning). Justified for a financial/historical application.

### Why F# for Domain Layer?

F# enforces functional programming discipline at compile time, reducing bug categories:

- **Discriminated unions**: Model "ride recorded" vs "ride deleted" as separate types; compiler prevents invalid state transitions
- **Pattern matching**: Exhaustive event handling; compiler warns if new event type not handled
- **Immutability by default**: All data structures immutable unless explicitly marked mutable (rare)
- **Reduced null references**: F# option types (`Some`/`None`) replace nullable C# (no "null reference exception" fears)
- **ROP (Railway Oriented Programming)**: Error handling via `Result<'T, 'E>` instead of exceptions; control flow explicit

Trade-offs: Team requires F# training; C#/F# interop requires value converters. Effort pays off in reduced production bugs (discriminated unions catch invalid states at compile time).

### Why Aspire Orchestration?

Microsoft Aspire enables:

- **Local-to-cloud parity**: Develop locally with Docker containers; same Bicep IaC deploys to Azure
- **Service discovery**: Services find each other automatically in Aspire dashboard (no hardcoded URLs)
- **Secrets management**: Azure Key Vault connection tested locally; same vault used in Azure
- **Team onboarding**: `dotnet run` spins up full stack (API, frontend, DB, functions) with one command
- **Health checks**: Dashboard shows service health in real-time

Trade-offs: Docker/Podman required; slight learning curve. Simplifies DevOps and debugging pipelines.

### Why Three-Layer Validation?

Data integrity is non-negotiable for financial data (savings calculations):

1. **Client-side (Blazor)**: Immediate feedback; better UX responsiveness
2. **Server-side (API)**: Prevents bypass attacks; enforces business rules
3. **Database layer**: Last-line defense; constraints prevent corrupted data from entering event store

If any layer is missing, data corruption risk rises. All three required.

### Why Fluent UI Blazor (latest version)?

Fluent UI provides:
- **Design tokens**: Centralized color, spacing, typography; enforces brand consistency
- **Accessibility**: Built-in WCAG 2.1 AA compliance; keyboard navigation, screen reader support
- **Responsive components**: Mobile-first design; components adapt to breakpoints
- **Microsoft ecosystem**: Native C# integration; no JavaScript bridging complexity

Trade-offs: Tied to Blazor version; updates require testing. Lock version to v4.13.x and higher as new versions are released for stability but also keeping up to date.

---

## Future Potential Amendments

These topics are under consideration for future constitution amendments (not yet ratified):

### Option 1: Projection Lag SLO
**Trigger**: If business needs tighter saga/process manager orchestration  
**Change**: Reduce "eventual consistency within 5 seconds" to "within 1 second" for critical projections  
**Impact**: May require moving from Change Event Streaming to in-process event handlers; architectural complexity or Azure Cosomos DB
**Decision pending performance measurement in production**

### Option 2: API Versioning Strategy
**Trigger**: When breaking Minimal API changes are required (e.g., new command format)  
**Change**: Add API versioning principle via URL paths (/v1/, /v2/) or headers  
**Impact**: Multiple code paths to maintain; dual testing of old+new APIs  
**Decision pending first breaking change scenario**

---

**Last Review**: 2026-03-03  
**Next Review**: 2026-04-03
