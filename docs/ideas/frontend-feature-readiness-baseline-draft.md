# Draft: Frontend feature-readiness baseline

**Status:** Accepted baseline design. It becomes a versioned reference inside `skills/create-feature-checklist/` only after the complete skill design is approved.

## Purpose

Provide a lean fallback catalog for frontend feature readiness. It describes observable outcomes and activation rules. Repository evidence supplies the actual proof method, tool, threshold, environment, command, owner, and approval path.

The baseline does not grant acceptance. `check-feature-readiness` can only evaluate supplied or already-observed evidence against the resulting project checklist.

## Design rule

Use outcome-oriented concern clusters. Do not copy the frontend-concepts image as a checklist taxonomy.

A concept can be one of three things:

- **A readiness concern:** An observable user or system outcome.
- **Routing vocabulary:** A feature characteristic that activates one or more concerns.
- **Evidence context:** A possible proof source. It cannot prove readiness by itself.

## Always-considered concerns

Every frontend feature considers these concerns. The project checklist adds detailed entries only when the feature exposes the relevant surface.

| ID | Concern | Observable outcome |
| --- | --- | --- |
| FE-BASE-01 | Behavior and state integrity | Users can complete the intended flow. Loading, empty, error, partial, interrupted, and completed states remain coherent when they exist. Retry, navigation, refresh, and repeated action do not silently corrupt state. |
| FE-BASE-02 | Inclusive interaction | Applicable users can perceive, understand, and operate the feature. Controls, focus, feedback, errors, and micro-interactions make status, consequences, and action outcomes understandable without one sensory cue or pointer-only interaction. Motion respects user preferences when present. |
| FE-BASE-03 | Adaptive presentation and compatibility | The feature remains usable across repository-supported layouts, browsers, devices, zoom levels, and input modes. Presentation does not hide meaning or block an action. |
| FE-BASE-04 | Trust and data protection | The interface does not expose restricted data or actions. Untrusted content cannot alter trusted behavior. Sensitive data and privacy choices remain protected in visible output, client state, diagnostics, and third-party boundaries. |
| FE-BASE-05 | Performance and resilience | The feature remains usable within repository-defined expectations. Slow, failed, or obsolete work has a bounded outcome. Recovery does not require an unsafe or unexplained action. |
| FE-BASE-06 | Change safety and supportability | The feature has enough observable failure context and compatibility behavior for repository-defined diagnosis, withdrawal, recovery, and rollback. |

## Conditional concerns

The authoring skill activates these only when repository and feature evidence show the stated characteristic.

| Activation characteristic | Concern | Observable outcome |
| --- | --- | --- |
| Users enter data or cause a mutation. | Input, validation, and mutation safety | Input survives recoverable failures. Validation identifies the affected input. Duplicate submission and irreversible actions have proportionate safeguards. |
| The UI reads or changes remote, concurrent, streamed, cached, or optimistic data. | Data freshness and concurrency | Late responses do not replace newer state. Retry, reconnect, cancellation, and reconciliation preserve intent. Conflicts do not silently lose user work. |
| The feature retains local data or must tolerate unreliable connectivity. | Persistence and synchronization | Reloads and upgrades preserve valid state. Storage failure is visible and recoverable. Offline work and synchronization conflicts have a defined outcome. |
| The feature supports multiple languages, regions, scripts, or directions. | Localization and internationalization | Text, formats, pluralization, sorting, layout expansion, and direction preserve meaning and operability. Locale changes do not leave stale mixed content. |
| The feature uses media, canvas, graphics, workers, or specialized computation. | Capability-heavy experience | Loading and capability loss have a bounded outcome. Resources stop when no longer needed. A usable alternative exists where the project requires one. |
| The feature requests browser or device access. | Permissions and device access | The request is contextual. Denial, revocation, interruption, and unsupported states remain usable. Access stops when the feature no longer needs it. |
| The feature creates public routes or navigable application state. | Navigation and discoverability | Refresh, deep links, history navigation, and shared URLs preserve intended state. Public content has required titles and discovery metadata. |
| The feature changes shared components, tokens, remote UI, or independently deployed units. | Shared and distributed UI contracts | Compatible behavior and semantics reach consumers. One unit failure does not corrupt unrelated UI. Version skew has a defined outcome. |
| The feature handles rich text, user content, embeds, or third-party scripts. | Rich and external content | Content remains safe to parse and render. Editing preserves user work where relevant. External failure does not break the primary path or disclose unexpected data. |

## Image concept triage

### Use as routing vocabulary

- Responsive design, forms, component design, state management, browser rendering, rendering strategies, media optimization, fonts, security, performance, privacy and permissions, offline-first, internationalization, and design systems.
- Real-time apps, local-first systems, rich text editors, server-driven UI, microfrontends, media and WebRTC, WebGL/WebGPU, and WebAssembly.

### Use as evidence or operating context

- Testing, deployment, browser DevTools, build systems, frontend tooling, and networking.

### Do not use as standalone readiness concerns

- Web fundamentals, HTML, CSS, JavaScript, TypeScript, browser internals, and scalable CSS.

These terms may explain implementation or help activate a concern. Their presence does not prove a feature is ready.

## Important concerns absent or underemphasized in the image

- Requirement-to-behavior traceability for the intended user path.
- Explicit loading, empty, denied, stale, partial, interrupted, and recovery states.
- Visible authorization behavior. Hiding a control is not authorization.
- Race conditions, duplicate effects, stale overwrites, and lost work.
- Refresh, deep-link, history, and version/cache transition behavior.
- Failure diagnosis that preserves useful context without exposing sensitive data.
- Third-party, analytics, embed, SDK, and font boundaries.
- Resource lifecycle for subscriptions, timers, workers, object URLs, media streams, and graphics resources.

## Evidence rule

A project checklist entry derived from this baseline must name an observable condition and its local evidence source. Only evidence that directly supports that condition can support `pass`.

Missing, inaccessible, stale, ambiguous, or unrelated evidence remains `unverified` or `blocked`. A successful test or deployment alone does not establish readiness.
