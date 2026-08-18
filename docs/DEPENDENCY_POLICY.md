# Dependency-First / No-Fallback Policy

This is the canonical Vitemis-wide contract for selecting and integrating external capabilities. It applies to every first-party project, platform target, tool, test surface, and future implementation.

## Mandatory Decision Order

1. Identify whether a user-selected, already-adopted, or license/provenance/security/platform-approved external dependency already provides the required capability.
2. If it does, integrate its official API or official extension point directly.
3. If the exact dependency cannot be integrated, stop that capability and report the blocker. Request a user decision before changing the dependency or scope.

There is no step that authorizes a first-party replacement implementation or a silent fallback.

## Prohibited Implementations

- Reimplementing an external dependency's equivalent core capability.
- Adding a substitute adapter, shim, compatibility layer, wrapper, proxy, facade, protocol translation layer, parallel backend, preview backend, shadow implementation, or temporary fallback.
- Silently switching to a legacy implementation, another provider, another renderer, another parser, a cached result, a mock, a simplified path, or an incomplete but continuing path.
- Using future replacement plans, schedule pressure, build difficulty, signing difficulty, or missing platform support to justify "fallback now, replace later."
- Treating an existing duplicate or fallback implementation as precedent for new work.

## Permitted Local Wiring

Local code is limited to the thinnest lifecycle, type, permission, configuration, and bundle wiring strictly required to call the dependency's official API. That wiring must not:

- reproduce business logic or algorithms owned by the dependency;
- reinterpret or broaden dependency semantics;
- expose a second product implementation;
- survive as an independently selectable backend;
- hide dependency failure or make an unavailable capability appear successful.

## Blocker Behavior

If version, build, signature, license, provenance, platform, security, or official-API constraints prevent direct integration:

- fail explicitly and diagnostically;
- stop the affected capability;
- preserve unrelated product functionality only when it does not impersonate the blocked capability;
- report the exact blocker and the evidence needed to remove it;
- request a user decision.

Do not silently degrade, choose another dependency, or write an internal substitute.

## Existing Compatibility And Safety Boundaries

- Existing noncompliant fallback, adapter, or duplicate paths must not be expanded. Record them as removal candidates for a separately authorized task.
- Security fail-closed behavior is mandatory and is not a functional fallback.
- Explicitly required backward decoding and data migration may remain only as narrow compatibility contracts. They must not become alternate product implementations or accept new writes in a legacy format unless the user explicitly requires it.
- Test doubles may exist only inside tests. They must never enter production selection or act as runtime fallback behavior.

## Documentation And Verification

Before implementing an external capability, project documentation must record:

- the selected dependency and pinned/versioned identity when applicable;
- evidence that its official API supplies the required capability;
- license, provenance, security, platform, and distribution constraints;
- the exact thin local wiring boundary;
- the explicit failure behavior when the dependency is unavailable.

Tests and review must prove:

- dependency absence or incompatibility produces an explicit failure;
- no secondary provider, backend, parser, renderer, cache, mock, legacy path, or internal implementation is invoked;
- local wiring does not duplicate the dependency's core behavior;
- existing migration or fail-closed paths remain within their documented scope.

Only a new explicit user decision naming the exact dependency, exact scope, and exit condition may override this policy.
