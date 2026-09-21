# CumulusVPN Audit Plan for FPN

**Status:** Pre-implementation engineering document  
**FPN version:** v1.2  
**Purpose:** Decide what FPN should REUSE, ADAPT, REIMPLEMENT or NOT REQUIRE from CumulusVPN before code inheritance.

## 1. Public Positioning

CumulusVPN is treated as an upstream/reference implementation for the Flux-native VPN layer. FPN is not positioned as a competing clone. FPN generalizes the architecture into an open privacy relay protocol with private self-owned gateways, generic Docker portability, transport abstraction, DNS/proxy modes, Operator ID, unlinkable entitlement and later independently operated multi-hop.

## 2. Audit Rule

No source code is copied into FPN until the applicable license and dependency obligations are verified. Public GitHub visibility is not sufficient evidence of reuse permission.

Each component receives one classification:

- **REUSE** — direct use is technically appropriate and license-compatible.
- **ADAPT** — reuse is appropriate after architectural changes.
- **REIMPLEMENT** — retain the idea/protocol behavior but implement independently.
- **NOT REQUIRED** — not needed by FPN.

## 3. Component Matrix

| Component | Initial FPN view | Questions to resolve |
|---|---|---|
| Gateway userspace WireGuard | ADAPT candidate | Performance, API boundaries, license, netstack coupling |
| gVisor/netstack integration | ADAPT/REUSE candidate | Throughput, maintenance, dependency terms |
| Enrollment API | REIMPLEMENT candidate | FPN pairing and future short-lived entitlement differ |
| WireGuard key identity | REIMPLEMENT | FPN separates tunnel, device and payment identity |
| Flux app discovery | ADAPT | Must add cache/bootstrap/direct-probe failure paths |
| FLUX chain scanner | ADAPT candidate | Settlement useful; direct TX→WG identity is not target design |
| OP_RETURN payment binding | NOT TARGET DESIGN | Avoid durable payment/session linkage |
| Per-key rate limiter | ADAPT candidate | Useful for service policy, not identity architecture |
| Desktop client | AUDIT | UX/stack comparison with FPN Tauri/Rust plan |
| Mobile clients | AUDIT | Native tunnel path maturity and store constraints |
| Web onboarding | AUDIT | Useful patterns; FPN should avoid mandatory web dependency |
| Flux deployment specs | ADAPT candidate | FPN needs public + private + generic Docker paths |
| Abuse/legal docs | REUSE AS REFERENCE | Validate jurisdiction and project-specific assumptions |
| CI/reproducible build | AUDIT | Prefer reuse of proven patterns if license-compatible |

## 4. Required Evidence per Component

For every REUSE or ADAPT decision record:

1. repository path and commit hash;
2. copyright holder;
3. license file or explicit license statement;
4. dependency licenses;
5. security-sensitive dependencies;
6. tests and current implementation maturity;
7. FPN architectural changes required;
8. performance evidence;
9. maintenance ownership;
10. final decision and reviewer.

## 5. Technical Questions

The audit must answer:

- Does the gateway actually pass full-device traffic end-to-end on current Flux?
- Which tunnel paths are implemented versus scaffolded?
- What are the measured CPU/RAM/throughput characteristics?
- How is UDP ingress handled across Flux placements?
- What assumptions are made about api.runonflux.io availability?
- How does payment scanning recover from chain/API failure?
- Which identity fields become persistent correlators?
- Are any logs or metrics privacy-sensitive?
- Can the gateway be cleanly separated from Cumulus-specific entitlement?
- Can FPN preserve compatibility with standard WireGuard clients during the MVP?

## 6. Decision Gate

FPN 0.1 implementation does not depend on completing every Cumulus feature. It does require a written decision for the gateway, netstack, discovery and payment components. Unknown licensing defaults to REIMPLEMENT or defer.

## 7. Initial Architectural Arbitration

FPN currently expects:

- userspace WireGuard concept: retain;
- Flux container constraints: treat as validated engineering input;
- direct WireGuard-key-as-payment-identity: do not retain as long-term architecture;
- Flux API discovery: retain only as one discovery source;
- generic Docker gateway: implement as a first-class MVP proof;
- public paid tier: defer until unlinkable entitlement exists.
