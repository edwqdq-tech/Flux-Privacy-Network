# FPN MVP Acceptance Tests

**Status:** Engineering gate document  
**Applies to:** FPN 0.0 → FPN 0.1

The MVP is accepted only when the core product works under normal conditions and under defined dependency failures. Numeric performance thresholds are intentionally marked TBD until baseline measurements exist; maintainers must fill and approve them before scaling the official beta fleet.

## A. Build and Supply Chain

| ID | Test | Acceptance |
|---|---|---|
| BUILD-001 | Build gateway from clean checkout | Reproducible documented build succeeds |
| BUILD-002 | Build desktop client | Windows/macOS/Linux build path documented and succeeds |
| BUILD-003 | Verify signed release | Client can verify release/update signature |
| BUILD-004 | Generate SBOM | SBOM produced for release artifacts |

## B. Gateway Portability

| ID | Test | Acceptance |
|---|---|---|
| PORT-001 | Deploy gateway on Flux | Healthy gateway reachable and tunnel establishes |
| PORT-002 | Deploy same gateway on generic Docker host | Healthy gateway reachable without Flux-specific runtime dependency |
| PORT-003 | Pair private gateway | No FPN account required |
| PORT-004 | DNS through private tunnel | DNS resolution works without external FPN DNS service |

## C. Tunnel Functionality

| ID | Test | Acceptance |
|---|---|---|
| TUN-001 | WireGuard handshake | Success |
| TUN-002 | IPv4 web traffic | Success |
| TUN-003 | UDP application traffic | Success |
| TUN-004 | DNS leak test | No unintended resolver path in configured full-tunnel mode |
| TUN-005 | Gateway reconnect | Automatic recovery |
| TUN-006 | Gateway failover | Client selects another eligible official gateway |
| TUN-007 | Network transition | Client recovers after local network change where platform permits |

## D. Performance Gate

Measure on representative Flux and equivalent generic Docker infrastructure.

| Metric | Baseline | Go threshold |
|---|---:|---:|
| P50 handshake latency | TBD | TBD |
| P95 handshake latency | TBD | TBD |
| Download throughput | TBD | TBD |
| Upload throughput | TBD | TBD |
| Added latency | TBD | TBD |
| Packet loss | TBD | TBD |
| CPU at 100 Mbps | TBD | TBD |
| RAM per active session | TBD | TBD |
| Stable concurrent sessions | TBD | TBD |
| Reconnect time | TBD | TBD |
| Failover time | TBD | TBD |

**Rule:** official fleet expansion requires maintainers to publish methodology, hardware/resource class and approved numeric thresholds. If the gate fails, benchmark alternative userspace stacks/transports before scaling.

## E. Discovery Chaos Tests

### DISC-001 — Primary Flux API unavailable
Conditions:
- primary Flux application-location API blocked/unreachable;
- FPN website unavailable;
- FPN-operated API unavailable.

Expected:
- installed client starts;
- bundled signed bootstrap loads;
- local cache loads;
- candidate signatures are checked;
- candidates are directly probed;
- paired private gateway remains usable;
- reachable cached/bootstrap public gateway can be selected.

### DISC-002 — FPN services unavailable, Flux available
Expected:
- normal public discovery continues from Flux source(s);
- MyFlux/private pairing data remains local;
- no login dependency blocks connection.

### DISC-003 — Stale bootstrap
Expected:
- unhealthy/stale nodes are rejected by direct probes;
- stale snapshot cannot force routing to a dead endpoint.

### DISC-004 — Conflicting discovery data
Expected:
- signed descriptor/version rules are deterministic;
- client does not silently trust a central recommendation.

## F. Privacy Baseline

| ID | Test | Acceptance |
|---|---|---|
| PRIV-001 | Local key generation | Private tunnel key never sent as private material |
| PRIV-002 | Production logs | No destination-domain or browsing-history logging |
| PRIV-003 | Metrics | Aggregate only; no persistent source→destination mapping |
| PRIV-004 | Payment | Not applicable in free FPN 0.1 beta |
| PRIV-005 | Marketing | UI/docs do not claim anonymity |

## G. Survivability

| ID | Failure | Minimum FPN 0.1 behavior |
|---|---|---|
| SURV-001 | FPN website down | Existing client connects |
| SURV-002 | FPN API down | Existing client connects |
| SURV-003 | MCP down | Networking unaffected |
| SURV-004 | Primary Flux discovery API down | Degraded discovery works |
| SURV-005 | Official gateway down | Failover or clear degraded state |
| SURV-006 | Project disappears | Source/build/deploy docs remain sufficient for private gateway operation |

## H. Release Decision

FPN 0.1 beta is **GO** only if:
- BUILD, PORT, TUN and privacy-baseline tests pass;
- DISC-001 passes;
- numeric performance gate is approved and passed;
- critical/high security findings have remediation decisions;
- public claims match implemented behavior.

Otherwise status is **NO-GO / RESEARCH CONTINUES**.
