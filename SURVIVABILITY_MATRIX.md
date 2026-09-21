# FPN Survivability Matrix

**Purpose:** Convert “if the project disappears, the network should survive” into explicit dependency behavior.

Legend:
- **Works** — expected to continue normally.
- **Degraded** — core use remains possible but some functions are unavailable.
- **Blocked** — function cannot proceed until dependency returns or an alternative exists.

| Dependency failure | Existing MyFlux/private tunnel | Existing public connection | Discover new public nodes | Deploy new private gateway | New paid entitlement | Community operation |
|---|---|---|---|---|---|---|
| FPN website down | Works | Works | Works/Degraded | Works from repo/docs | Works if issuer path independent | Works |
| FPN-operated API down | Works | Works | Works/Degraded | Works | Depends on entitlement architecture | Works |
| FPN MCP down | Works | Works | Works | Manual deployment works | Works | Manual ops work |
| Primary Flux app-location API down | Works | Degraded | Degraded via cache/bootstrap/mirrors | Flux deployment may degrade; Docker works | Settlement may still work | Degraded |
| Bundled bootstrap stale | Works | Works if cache/other source valid | Degraded | Works | Works | Works |
| Official gateways down | Works | Degraded/Blocked for official-only | Community/private may remain | Works | Tokens should remain valid for eligible nodes | Community can remain |
| FLUX explorer/indexer down | Works | Works during free beta | Works | Deployment/payment UX may degrade | Degraded unless direct chain verification available | Works |
| FLUX chain unavailable | Works for already deployed private gateway | Existing free/valid-token access may work | Discovery unaffected | Flux deployment/payment blocked; Docker works | Blocked | Non-payment operations work |
| Privacy Pass issuer down | Works | Existing unspent tokens should work | Works | Works | New token issuance blocked | Gateways can redeem existing valid tokens |
| Payment verifier down | Works | Existing valid entitlement works | Works | Works | New issuance blocked | Works |
| Official signing key compromised | Private paired gateway can be isolated | Emergency trust action required | Bootstrap/update trust degraded | Source build/manual trust possible | Depends on issuer keys | Community descriptors remain independently signed |
| FPN organization disappears | Works if software/deploy artifacts retained | Degraded; community/official residual nodes may continue | Community/bootstrap mirrors required | Works from open source on Docker/Flux | Requires independent issuer/payment continuation | Protocol can continue if governance/artifacts are forkable |
| Flux governance removes an FPN app | Unaffected if private gateway elsewhere | Affected instances disappear | Other nodes remain | Docker/other placements work | Unaffected at protocol layer | Operators can redeploy elsewhere |

## Required Architectural Consequences

1. **No mandatory FPN account database.**
2. **MCP is never a runtime dependency.**
3. **Private gateway pairing state is local/exportable.**
4. **Gateway software runs on generic Docker as well as Flux.**
5. **Client ships with signed bootstrap material and maintains a validated local cache.**
6. **Discovery candidates are directly probed before use.**
7. **Node descriptors are independently signed by node/operator identities.**
8. **Payment identity is separated from tunnel/session identity.**
9. **Paid public access waits for unlinkable entitlement.**
10. **Release/update trust must have documented key-rotation and compromise procedures.**

## Chaos-Test Schedule

For every release candidate:
- FPN API unavailable;
- FPN website unavailable;
- MCP unavailable;
- at least one official gateway unavailable.

For major networking releases:
- primary Flux discovery source unavailable;
- stale bootstrap;
- conflicting descriptors;
- DNS resolver failure;
- packet loss/network transition.

Before paid production:
- issuer unavailable;
- payment verifier unavailable;
- chain/indexer unavailable;
- token replay/double-spend attempts;
- issuer-key rotation and compromise drill.

## Survivability Claim Policy

FPN should not say “fully decentralized” merely because gateways run on Flux. Claims should identify the layer:

- protocol portability;
- public network infrastructure;
- discovery;
- settlement;
- entitlement issuance;
- governance;
- software distribution.

A layer is only described as independent/decentralized when the corresponding failure test demonstrates it.
