# Flux Privacy Network

**An open privacy protocol and Flux-native network.**

> Own your gateway. Choose your network. Minimize trust.

Flux Privacy Network (FPN) is an open-source privacy network designed around a simple idea:

**privacy infrastructure should be understandable, portable and, when desired, owned by the user.**

FPN combines private gateways, encrypted tunnels, privacy-preserving DNS, application proxies and eventually multi-hop routing through independently operated infrastructure.

FPN uses the Flux ecosystem as its preferred decentralized infrastructure layer while keeping the protocol portable to standard Docker infrastructure.

---

## Why FPN?

Most VPN services ask users to move trust from their Internet provider to a VPN company.

FPN takes a different approach.

Users can:

* connect through the public FPN Network;
* deploy and own a private **MyFlux Gateway**;
* use community-operated infrastructure;
* restrict routing to specific operator classes;
* eventually use independently operated entry and exit relays through **Privacy+**.

The long-term goal is not to create another centralized VPN provider.

The goal is to create an **open privacy protocol and interoperable network** that can continue operating independently of its original developers.

---

## Core principles

FPN is designed around several architectural principles.

### Accountless by design

The core network should not require a traditional user account.

No mandatory email address, username or centralized user database should be necessary to establish a privacy connection.

### Own your exit

Users can deploy their own private gateway through Flux or compatible Docker infrastructure.

A MyFlux Gateway is controlled by the user and does not need to participate in the public FPN gateway marketplace.

### Multi-source discovery

FPN must not depend on a single discovery API.

Clients are designed to progressively support:

* Flux application-location sources;
* signed bootstrap manifests;
* validated local caches;
* community mirrors;
* direct gateway probes;
* private paired gateways;
* future signed network registries.

### Local routing decisions

Gateway selection should happen locally whenever possible.

Clients may evaluate:

* latency;
* availability;
* capacity;
* transport compatibility;
* geography;
* software version;
* quality measurements;
* operator identity;
* infrastructure diversity;
* user policy.

### Quality is not trust

FPN intentionally separates service quality from operator trust.

A gateway can provide excellent performance without automatically becoming more trustworthy.

### Minimal knowledge

The architecture attempts to prevent any single component from unnecessarily learning all of:

* user identity;
* source IP;
* destination;
* DNS activity;
* payment identity;
* session identity.

### Protocol portability

Flux is FPN's preferred infrastructure ecosystem.

It is not the identity of the protocol itself.

A compatible FPN gateway should also be deployable using standard Docker infrastructure.

---

# FPN modes

## FPN Network

The simplest user experience.

Install the client, select a location or automatic routing policy, and connect.

The client discovers compatible gateways and selects an appropriate route locally.

Early public infrastructure may contain FPN-operated gateways while community participation is progressively introduced.

---

## MyFlux Gateway

**Your gateway. Your infrastructure. Your exit.**

Users can deploy their own private FPN gateway.

Conceptually:

```
Device
   │
   │ encrypted tunnel
   ▼
MyFlux Gateway
   │
   ▼
Internet
```

The gateway can run on Flux infrastructure or compatible Docker infrastructure.

MyFlux Gateway is one of FPN's primary MVP use cases.

---

## Community Network

Independent operators will eventually be able to contribute infrastructure to FPN.

Possible roles include:

* Entry Relay
* Exit Gateway
* DNS Relay
* DNS Resolver
* Proxy Relay

Community Exit operation carries additional legal and abuse-management responsibilities and is therefore treated separately from lower-risk network roles.

---

## Privacy+

Privacy+ is FPN's planned multi-hop privacy architecture.

```
Device
   │
   ▼
Entry Relay
Operator A
   │
   ▼
Exit Relay
Operator B
   │
   ▼
Internet
```

Entry and exit operators must be administratively independent.

Future routing policies may additionally consider:

* operator diversity;
* hosting-provider diversity;
* ASN diversity;
* jurisdiction;
* infrastructure characteristics.

**Privacy+ is not an MVP launch claim.**

It will only be enabled when FPN can verify the required diversity properties.

---

# Privacy modes

FPN is being designed as a modular privacy stack rather than a single VPN tunnel.

### VPN

Full-device encrypted connectivity.

Initial implementation:

**userspace WireGuard**

The userspace implementation is subject to explicit performance benchmarking before public deployment.

### DNS Shield

Privacy-oriented DNS modes may include:

* DNS through the encrypted tunnel;
* DNS over HTTPS;
* DNS over TLS;
* Oblivious DNS over HTTPS.

Encrypted Client Hello may provide complementary hostname privacy where supported.

### FPN Proxy

Application-level privacy routing.

Future transports may include:

* HTTP CONNECT;
* CONNECT-UDP;
* MASQUE;
* CONNECT-IP.

### Privacy+

Multi-hop routing using administratively independent operators.

---

# Architecture

FPN separates the system into three planes.

## Data plane

Carries user traffic.

Examples:

* WireGuard
* QUIC
* MASQUE
* HTTP CONNECT
* ODoH

## Control plane

Handles infrastructure operations.

Examples:

* deployment;
* discovery;
* health checks;
* capability descriptors;
* monitoring;
* node administration.

FPN may integrate with the official Flux MCP and advanced FluxOS tooling.

**MCP is never part of the user traffic path and must never become a runtime dependency for ordinary connections.**

## Economic plane

Handles future network economics.

The FLUX blockchain can provide:

* infrastructure payments;
* operator economics;
* verifiable transactions.

Blockchain transactions do **not** carry VPN traffic.

---

# Payments and privacy

Public FPN beta access is planned to remain free until a privacy-preserving entitlement system is ready.

FPN will not intentionally bind a normal public FLUX transaction directly to a persistent tunnel identity for production paid access.

The intended architecture is:

```
FLUX payment
      │
      ▼
Payment verifier
      │
      ▼
Privacy-preserving entitlement
      │
      ▼
Rotating service credential
      │
      ▼
FPN Gateway
```

The current standards-based candidate is the IETF Privacy Pass architecture.

Any production implementation must undergo independent cryptographic review.

MyFlux Gateway owner mode does not require FPN service credits.

---

# Discovery resilience

FPN treats discovery as a multi-source resilience problem.

An installed client should not become unusable simply because the FPN website or a primary discovery service becomes unavailable.

The target degraded-mode architecture includes:

```
Local cache
    +
Signed bootstrap
    +
Flux discovery
    +
Community mirrors
    +
Direct probes
    +
Private pairing
```

One of the project's required acceptance tests intentionally disables both the primary Flux application-location source and FPN-operated web infrastructure.

---

# FPN is not an anonymity network

FPN is designed to improve privacy and reduce unnecessary trust.

It does **not** claim to provide perfect anonymity.

FPN does not promise:

* invisibility;
* immunity from traffic analysis;
* zero metadata under all conditions;
* protection against every global adversary;
* perfect unlinkability.

Security and privacy claims must correspond to properties that can actually be demonstrated and audited.

---

# Current development status

FPN is currently in the **research, architecture and benchmarking phase**.

The architecture is intentionally being validated before significant production code is inherited or written.

## FPN 0.0 — Research, Audit & Benchmark

Current priorities:

* audit CumulusVPN components and licensing;
* benchmark userspace WireGuard on Flux;
* validate generic Docker portability;
* implement discovery abstraction;
* test discovery failure modes;
* establish security baselines;
* define protocol boundaries.

## FPN 0.1 — Own or Connect

Initial usable release:

* desktop client;
* FPN Network;
* MyFlux Gateway;
* WireGuard tunnel;
* local gateway selection;
* signed bootstrap;
* Flux deployment;
* generic Docker deployment.

## FPN 0.2 — Everywhere

Focus:

* mobile clients;
* DNS privacy improvements;
* reconnect and failover;
* routing policies;
* broader infrastructure support.

## FPN 0.3 — Economy & Community

Focus:

* community operators;
* operator identities;
* proof-of-service;
* privacy-preserving entitlement;
* FLUX-based network economics.

## FPN 0.4 — Proxy Privacy

Focus:

* application proxies;
* MASQUE;
* CONNECT-UDP;
* advanced DNS privacy.

## FPN 1.0 — Privacy+

Focus:

* independent entry and exit relays;
* operator diversity;
* infrastructure diversity;
* multi-hop routing.

---

# CumulusVPN

RunOnFlux's CumulusVPN project is an important upstream/reference implementation for FPN's Flux-native VPN layer.

FPN does not assume that CumulusVPN source code can automatically be reused.

Each relevant component must be audited for:

* licensing;
* dependencies;
* maturity;
* security;
* performance;
* architectural compatibility.

Components are classified as:

**REUSE · ADAPT · REIMPLEMENT · NOT REQUIRED**

See:

`CUMULUS_AUDIT.md`

---

# Security

Security is a release requirement, not a post-launch feature.

The project intends to progressively require:

* reproducible builds;
* signed releases;
* dependency scanning;
* container scanning;
* SBOM generation;
* fuzz testing;
* protocol conformance testing;
* least-privilege services;
* secret isolation;
* independent penetration testing;
* cryptographic review;
* vulnerability disclosure procedures.

FPN does not invent cryptography where established protocols exist.

Security issues should be reported according to:

`SECURITY.md`

---

# Survivability

A core design question for every FPN dependency is:

> If the original FPN organization disappears tomorrow, what continues to work?

FPN is being designed so that failure of the original organization does not necessarily mean failure of the protocol.

See:

`SURVIVABILITY_MATRIX.md`

---

# Repository structure

The intended monorepo structure is:

```
flux-privacy-network/
│
├── apps/
│   ├── desktop/
│   ├── ios/
│   ├── android/
│   └── web/
│
├── services/
│   ├── gateway/
│   ├── entry/
│   ├── exit/
│   ├── proxy/
│   ├── odoh-relay/
│   ├── odoh-resolver/
│   └── entitlement/
│
├── crates/
│   ├── protocol/
│   ├── crypto/
│   ├── discovery/
│   ├── payments/
│   └── routing/
│
├── mcp/
│   └── fpn-mcp/
│
├── deploy/
│   ├── docker/
│   ├── flux/
│   └── orbit/
│
├── specs/
│   ├── gateway.json
│   ├── relay.json
│   ├── exit.json
│   └── dns.json
│
├── docs/
├── threat-model/
│
├── README.md
├── WHITEPAPER.md
├── ROADMAP.md
├── SECURITY.md
├── CONTRIBUTING.md
├── CUMULUS_AUDIT.md
├── MVP_ACCEPTANCE_TESTS.md
└── SURVIVABILITY_MATRIX.md
```

---

# Documentation

The current architecture reference is:

**Flux Privacy Network Whitepaper v1.2**

Supporting engineering documents:

* `WHITEPAPER.md`
* `CUMULUS_AUDIT.md`
* `MVP_ACCEPTANCE_TESTS.md`
* `SURVIVABILITY_MATRIX.md`
* `ROADMAP.md`
* `SECURITY.md`

---

# Contributing

FPN is intended to become a community-developed open protocol.

Useful contribution areas include:

* networking;
* WireGuard;
* QUIC;
* MASQUE;
* DNS privacy;
* Rust;
* Go;
* mobile VPN APIs;
* Flux infrastructure;
* Docker;
* security engineering;
* cryptography review;
* protocol design;
* distributed systems;
* UX;
* documentation.

Please read `CONTRIBUTING.md` before submitting major architectural changes.

---

# License

**License selection is intentionally pending final architectural and dependency review.**

Possible licensing models under consideration include permissive licensing for protocols/client libraries and stronger copyleft for selected network services.

No third-party source code should be copied into FPN until its license and compatibility have been explicitly verified.

---

# Project philosophy

FPN should pass a simple test:

**Can users understand who they are trusting?**

And a harder one:

**Can the network survive without us?**

If both answers eventually become yes, FPN is moving in the right direction.

---

**Flux Privacy Network**

*Own your gateway. Choose your network. Minimize trust.*
