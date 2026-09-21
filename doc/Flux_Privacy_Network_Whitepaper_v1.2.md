# Flux Privacy Network (FPN)
## An Open Privacy Protocol and Flux-Native Network

**Whitepaper v1.2 — Engineering Community Draft**  
**September 2026**

> Community draft for technical review, contribution and beta-test planning. This document describes a proposed architecture. Features, privacy properties and timelines are not guarantees until implemented and independently tested.


## Executive Summary

Flux Privacy Network (FPN) is a proposed open-source privacy protocol and network designed around user ownership, modular privacy transports, distributed relays and minimized trust. FPN is Flux-native at the network and economic layers, while the underlying FPN protocol and gateway software are intended to remain deployable on standard Docker-compatible infrastructure. This distinction is deliberate: the public FPN Network can benefit from Flux's decentralized cloud, discovery and FLUX economy without making the protocol itself dependent on a single hosting environment.

FPN is not defined as a VPN. It is a modular privacy relay protocol capable of supporting several user-facing modes: private DNS, application-level proxying, full-device encrypted tunneling and optional multi-hop routing. WireGuard is the preferred MVP tunnel transport because it is mature and maps well to userspace operation on Flux. MASQUE/HTTP/3, CONNECT-UDP and CONNECT-IP are treated as additional transport capabilities rather than replacements that must exist in the first release.

The initial product has two core experiences. A user can install the FPN application and connect immediately to a small official FPN gateway network, or deploy a private MyFlux Gateway and connect only through infrastructure they control. MyFlux Gateway is a first-class feature, not an advanced afterthought: it provides a concrete answer to the trust problem of conventional VPNs — users who do not want to trust the public network can own their exit.

The public network is designed to evolve from an official beta fleet into a mixed official/community network. Independent operators can eventually provide gateway, entry, exit, proxy and DNS capacity. Multi-hop Privacy+ routing introduces explicit administrative-diversity constraints so that entry and exit should not be controlled by the same operator and, where practical, should differ by Flux node, hosting provider, ASN and jurisdiction.

FPN separates three planes. The data plane carries encrypted user traffic. The control plane handles discovery, signed node descriptors, health, deployment and software lifecycle. The economic plane uses the Flux blockchain for settlement, service credits and future operator compensation. User traffic never needs to traverse or be recorded on-chain.

The Minimum Viable Product is intentionally narrow: desktop client, userspace WireGuard gateway, official gateways, MyFlux Gateway, a generic Docker deployment, resilient discovery, local routing decisions and a free public beta. Before that beta, FPN runs an explicit research-and-benchmark phase. Userspace WireGuard performance on Flux, discovery failure modes, CumulusVPN reuse/licensing and portable Docker behavior are engineering gates rather than assumptions.

Paid public access is deliberately postponed until an unlinkable entitlement design is implemented and independently reviewed. The current candidate is an architecture derived from IETF Privacy Pass (RFC 9576/9577/9578), with FLUX payment verification separated from token redemption at gateways. Payments must not make a public blockchain transaction a durable tunnel or session identifier.

The long-term architectural test remains survivability: if the original project team disappears, independent users should still be able to build the clients, deploy gateways, discover compatible infrastructure and operate the protocol without a mandatory FPN-operated API, account database or MCP service.

## 1. Vision and Design Goals

The Internet privacy market is fragmented. VPNs protect an entire device but usually require trust in a centralized provider. Encrypted DNS protects name resolution but does not hide destination IP addresses. Proxies can protect individual applications but frequently depend on centralized accounts and infrastructure. Multi-hop systems improve separation of knowledge but add complexity and performance costs.

FPN proposes a common open protocol and software stack in which these functions are selectable layers rather than separate products.

A key architectural distinction is:

**FPN Protocol** — infrastructure-agnostic protocol, client logic, node descriptors and interoperable service interfaces.

**FPN Network** — the default public network, designed to be Flux-native and progressively community-operated.

**MyFlux Gateway** — the recommended self-hosted/private deployment experience, optimized for Flux but based on portable containerized software.

**FPN Economy** — FLUX-based settlement, credits and future operator compensation.

This prevents the protocol from becoming synonymous with one tunnel technology or one hosting environment while preserving Flux as the project's primary decentralized infrastructure.

The primary goals are:

• Open source by default. Client, gateway, relay, DNS, proxy, protocol and deployment components should be auditable and reproducibly buildable.
• Accountless operation. No email address, phone number or conventional username should be required for core network access.
• User ownership. A user must be able to deploy and use a private gateway without joining the public network.
• Public accessibility. Users who do not want to deploy infrastructure must have a one-click default network.
• Decentralized discovery. A permanent central API must not be required for normal operation.
• Separation of knowledge. Where multi-hop mode is used, source identity, destination knowledge, DNS knowledge and payment identity should be separated as far as practical.
• Minimal data retention. Components should retain only the state required to provide the service.
• Protocol conservatism. FPN should prefer mature cryptographic protocols and established implementations rather than inventing new cryptography.
• Progressive decentralization. The MVP may use a limited official fleet, but the architecture must support independent operators without a redesign.
• Simple user experience. Advanced infrastructure must remain invisible to users who only want a Connect button.
• AI-operable infrastructure. Deployment and maintenance should be automatable through Flux's official MCP tooling and optional FPN-specific MCP interfaces, without putting AI agents in the user traffic path.

## 2. System Model: Three Planes

FPN separates the system into three planes.

DATA PLANE
The data plane carries user traffic. It includes WireGuard, userspace networking, MASQUE/HTTP/3 where supported, DNS relays/resolvers, entry relays and exit gateways. User traffic never needs to be written to the blockchain.

CONTROL PLANE
The control plane discovers services, publishes signed node descriptors, checks health, deploys infrastructure and coordinates software versions. Flux network APIs, direct Flux node queries, signed bootstrap directories, the official Flux MCP server and optional FluxTools/FPN MCP services can participate here.

ECONOMIC PLANE
The Flux blockchain is used for economic settlement and verifiable state. A payment transaction can purchase service credit. Operator rewards can ultimately be derived from measured, privacy-preserving service contribution. The economic plane must not become a browsing-history database.

This separation is fundamental. A blockchain is appropriate for durable public settlement; it is not appropriate for transporting private Internet packets.

## 2A. Transport Abstraction

FPN MUST NOT define WireGuard as the protocol itself. The client and node architecture should expose a transport abstraction so that capabilities can evolve without redesigning discovery, identity, routing or economics.

Initial transport families:

• **WireGuard** — required for the MVP full-device tunnel. Server-side operation on Flux uses a userspace implementation and userspace network stack.
• **MASQUE / HTTP/3** — planned for application proxying and future tunnel use.
• **CONNECT-UDP** — planned for UDP proxying over HTTP.
• **CONNECT-IP** — a standards-based path for IP tunneling over HTTP and a future candidate for full-tunnel operation.
• **Compatibility proxy interfaces** — SOCKS5 or HTTP CONNECT MAY be exposed when useful to applications and developer tooling.

Conceptually:

`FPN Client → Privacy Engine → Transport Provider → FPN Node`

A node advertises supported transports in a signed capability descriptor. Clients negotiate a mutually supported transport and can continue using WireGuard even after additional transports are introduced.

## 3. User Modes

FPN supports multiple operating modes from the same client.

3.1 Public FPN Network
The default consumer mode. The client discovers available gateways and connects to an official or community node according to local policy, latency, country, health and capacity. The user does not need to understand Flux or deploy a server.

3.2 MyFlux Gateway
The user deploys a private FPN gateway on Flux or on compatible Docker/self-hosted infrastructure. Flux is the recommended and best-integrated deployment path, but the gateway protocol must not require Flux in order to function. The gateway is not automatically published into the public community pool. The client pairs with it and uses it as the user's private exit. The user pays the underlying infrastructure cost rather than a public-network subscription.

3.3 Community Operator
An independent operator deploys one or more public services and elects to advertise them to the network. Possible roles include VPN gateway, entry relay, exit relay, proxy, ODoH relay and ODoH resolver. Operator participation must be permissionless or minimally permissioned in the long term, while preserving anti-Sybil and abuse controls.

3.4 Hybrid Mode
A user may combine personal and public infrastructure. Examples include a personal entry plus a public exit, a public entry plus a personal exit, or two personal gateways in different regions.

3.5 Official-Only Policy
Users who prefer a curated experience may restrict routing to nodes operated by the FPN project.

3.6 Community-Only Policy
Advanced users may choose to avoid official infrastructure and use only independent operators.

3.7 My-Gateways-Only Policy
Users can restrict the client to infrastructure they personally control.

## 4. Privacy Modes

4.1 DNS Shield
DNS Shield is implemented as a DNS Privacy Engine rather than as an ODoH-only feature. Depending on the selected mode and platform, it may use DNS through the encrypted tunnel, DoH, DoT or ODoH.

For a private MyFlux Gateway, DNS through the user's own encrypted tunnel may be sufficient and avoids unnecessary relay complexity. For public single-hop operation, ODoH can provide useful separation between the client address and the resolver. In an ODoH topology, a relay can know where a request came from without being able to read the DNS question, while the resolver can process the DNS question without directly receiving the original client address.

DNS Shield must not be marketed as equivalent to a VPN. Encrypted DNS does not by itself hide destination IP addresses. ECH is treated as a complementary ecosystem capability rather than a required FPN protocol dependency.

4.2 Proxy Mode
Proxy Mode protects selected applications instead of the entire device. The long-term preferred transport family is MASQUE over HTTP/3/QUIC, including CONNECT, CONNECT-UDP and potentially CONNECT-IP. SOCKS5 or conventional HTTP CONNECT may be offered as compatibility interfaces where useful.

Proxy Mode is especially relevant to browsers, developer tools, automation and autonomous agents that need a privacy-preserving egress path without taking over the device's complete routing table.

4.3 VPN Mode
VPN Mode protects device traffic through an encrypted tunnel. The MVP uses WireGuard because it is mature, fast, widely supported and compatible with an accountless public-key model. Because Flux application containers do not provide the normal privileged kernel networking capabilities expected by many VPN servers, the server-side design uses a userspace data plane, following the architectural approach demonstrated by CumulusVPN: wireguard-go plus a userspace network stack such as gVisor netstack.

4.4 Privacy+ Multi-Hop
Privacy+ creates at least two independently operated hops. The entry can observe the client network address but should not learn the final destination. The exit can observe the final destination but should see the entry as its source. DNS resolution can use an independent ODoH path. The client should avoid selecting entry and exit services controlled by the same operator.

Multi-hop improves separation of knowledge but does not magically provide perfect anonymity. Timing correlation, traffic analysis, compromised endpoints and colluding operators remain part of the threat model.

## 5. Reference Architecture

A typical public VPN connection is:

Client → encrypted tunnel → FPN Gateway → Internet

A private connection is:

Client → MyFlux Gateway → Internet

A Privacy+ connection is:

Client → Entry Relay → Exit Gateway → Internet

DNS Shield can run independently:

Client → ODoH Relay → ODoH Resolver → Authoritative DNS

A combined high-privacy path can therefore use independent components:

Client
  ├─ DNS → ODoH Relay A → Resolver B
  └─ Traffic → Entry C → Exit D → Internet

The desired property is not that every component knows nothing. The desired property is that no single infrastructure component must possess the complete mapping of user identity, source address, DNS request, destination and payment identity.

## 6. Flux Integration

Flux is the preferred distributed compute environment for the public FPN Network, but it is treated as an infrastructure provider rather than as the FPN protocol itself. The implementation should expose a provider boundary such as `FluxProvider`, `DockerProvider` and future providers.

FPN should support both prebuilt Docker deployment and Flux Deploy with Git/Orbit workflows. The MVP must actually demonstrate a generic Docker gateway outside Flux; portability is an acceptance test, not merely a documentation claim.

Flux also imposes real decentralization limits that FPN must state plainly. Current Flux application discovery commonly relies on public Flux APIs, and Flux governance can affect application publication. FPN therefore does not claim that every dependency of the initial public network is decentralized. Instead, it defines failure behavior and progressively removes single-source dependencies.

The official Flux Cloud MCP server is particularly useful for infrastructure automation. It can build and validate Flux application specifications, quote deployments, deploy and pay in FLUX, wait for applications to start, retrieve logs and statistics, restart or redeploy applications and inspect network information. This makes it possible to create a high-level FPN deployment workflow without making an AI agent part of the privacy-sensitive data path.

A future FPN MCP server can expose domain-specific operations such as deploy_gateway, deploy_odoh_relay, list_nodes, test_gateway, estimate_operator_cost and get_node_health. Internally it can use the official Flux MCP and other Flux tooling. Privileged actions must require explicit confirmation and separate credentials.

FPN should not rely on a single hosted MCP server for network survival. MCP is an operational convenience, not a protocol dependency.

## 7. CumulusVPN Positioning and Reuse Policy

CumulusVPN is treated as an **upstream/reference implementation for the Flux-native VPN layer**, not as a product FPN intends to duplicate. Its public repository currently describes a pre-launch system with userspace WireGuard, local client keys, Flux API discovery, per-key enforcement and on-chain FLUX payment verification. It also documents the key Flux container constraints that make a userspace data plane necessary.

FPN is a broader protocol effort: transport abstraction, MyFlux and generic Docker ownership, DNS privacy, proxy transports, Operator ID, quality/trust separation, unlinkable entitlement and later independently operated multi-hop.

Before implementation work is copied or adapted, `CUMULUS_AUDIT.md` MUST classify relevant components as:

• **REUSE** — suitable for direct use under verified license terms.
• **ADAPT** — useful but requires architectural modification.
• **REIMPLEMENT** — concept is useful but code reuse is unsuitable.
• **NOT REQUIRED** — outside the FPN design.

The audit must record license evidence per component and dependency. Public visibility is not permission to copy. Until that audit is complete, FPN may use CumulusVPN as engineering evidence and architectural reference but should avoid claiming code inheritance.

Public positioning should be explicit: **CumulusVPN demonstrates a Flux-native decentralized VPN architecture; FPN generalizes the problem into an open privacy relay protocol with self-owned and public-network modes.**

## 8. Identity, Keys and Accountless Access

The FPN client generates identity material locally. Private keys should never be required to leave the device for ordinary network use.

A long-term design should avoid using one permanent public key simultaneously as payment identity, device identity and tunnel identity. Instead, FPN should derive or generate independent identities for different purposes:

• Payment identity: used to fund or prove purchase of service.
• Device identity: local device management and pairing.
• Tunnel identity: short-lived or rotating keys used by WireGuard/MASQUE sessions.
• Operator identity: signs node descriptors and operator claims.
• Session credential: authorizes temporary access without exposing the original payment transaction.

This separation reduces linkability across layers.

## 8A. Operator Identity and Administrative Diversity

Every public FPN operator should have an explicit cryptographic Operator ID independent from individual node identities. One operator may run multiple services and locations, but clients must be able to recognize common administrative ownership.

A signed node descriptor therefore includes at minimum:

• protocol version;
• node ID;
• operator ID;
• service roles;
• supported transports;
• DNS capabilities;
• software version;
• region and country claim;
• endpoint information;
• capacity class;
• pricing/credit policy when applicable;
• signature.

Privacy+ routing applies an **Administrative Diversity Constraint**. Entry and exit MUST NOT share the same Operator ID. Clients SHOULD additionally prefer diversity of Flux node, infrastructure provider, ASN and — when requested by the user — jurisdiction.

This is stronger than simply choosing two different IP addresses. The purpose of multi-hop is to reduce concentration of knowledge across independently controlled infrastructure.

## 9. Payments, FLUX and Unlinkable Entitlement

The public MVP is a free beta. This removes payment correlation and wallet friction while FPN validates tunneling, discovery, failover and self-hosting.

MyFlux Gateway owner mode does not require FPN service credits. A user who deploys and pays for their own infrastructure can use it directly.

For future paid access, FPN separates **payment** from **network authorization**:

1. The user makes a FLUX payment or otherwise acquires service value.
2. A payment verifier confirms settlement.
3. An issuer grants privacy-preserving service tokens.
4. The client later redeems a token at a gateway.
5. The gateway validates entitlement without needing the original FLUX transaction identifier.
6. Tunnel/session keys remain separate from payment identity.

The current standards-based candidate is the IETF Privacy Pass architecture. RFC 9576 defines privacy-preserving authorization with unlinkable issuance/redemption semantics; RFC 9577 defines the HTTP authentication scheme; RFC 9578 defines issuance protocols including publicly verifiable Blind RSA tokens and privately verifiable OPRF-based tokens.

This is a **candidate architecture, not a completed security design**. FPN must define credit denomination, double-spend behavior, issuer trust, key rotation, metadata, refund/revocation policy and denial-of-service controls. An independent cryptographic review is required before production paid access.

**Production gate:** the public FPN Network MUST NOT introduce normal paid access by directly binding a public FLUX transaction to a long-lived WireGuard key, device identity or browsing session. If unlinkable entitlement is not ready, the public beta remains free.

FLUX is transparent by design in this context; FPN must not market blockchain payment itself as private. The privacy property comes from separating settlement from later authorization.

## 10. Discovery, Bootstrap and Failure Independence

A privacy network still needs a bootstrap process. In the initial Flux environment, application-location discovery commonly uses Flux public APIs. FPN therefore avoids calling the MVP discovery path fully decentralized. The objective is **multi-source resilient discovery with independently testable failure behavior**.

**Core invariant:** loss of an FPN-operated API or website MUST NOT prevent an already-installed client from using MyFlux Gateway or from reaching known healthy public nodes. Loss of the primary Flux application-location API MUST have a tested degraded path.

The Discovery Engine merges:

1. local cache of recently validated nodes;
2. Flux application-location data when available;
3. a bundled signed bootstrap snapshot shipped with client releases;
4. optional independently hosted/community mirrors;
5. manually imported or paired private gateways;
6. direct health/capability probes to candidate nodes;
7. future signed decentralized registry mechanisms.

A bootstrap snapshot is not trusted merely because it is bundled. Node descriptors remain signed and the client independently probes endpoints before routing.

The minimum chaos test is defined in `MVP_ACCEPTANCE_TESTS.md`: with the primary Flux API unavailable and FPN web/API services unavailable, an installed client must still start, load cache/bootstrap data, use a paired private gateway and connect to reachable previously known or bundled public gateways.

The initial network may still have governance dependencies inherited from Flux. FPN documents those dependencies instead of hiding them behind a blanket decentralization claim.

## 11. Node Selection, Quality and Trust

Routing decisions occur locally on the client. FPN deliberately separates **measured quality** from **trust classification**.

A Node Quality Score can use observable measurements such as availability, handshake success rate, latency, packet loss, throughput, software/protocol compatibility and recent health. Independent probes and clients can contribute privacy-preserving observations.

Trust classification is separate. A node may be an official node, a community node with established history, an unknown community operator or a private node owned by the user. A high-performance node is not automatically a highly trusted node.

The client combines the user's policy with both dimensions. Candidate selection can consider geography, quality, current load, supported transport, operator diversity and trust policy.

Privacy+ adds mandatory operator diversity and should additionally prefer different underlying infrastructure where that information can be measured or credibly established. The scoring algorithm should be transparent and inspectable rather than an opaque server-side recommendation.

## 12. Operator Architecture

An operator should be able to contribute capacity with minimal friction.

Deployment path A: Docker
The operator runs a published, signed container image or builds the image reproducibly from source.

Deployment path B: Deploy on Flux
The operator uses a Flux application specification or Orbit/Deploy with Git workflow.

Deployment path C: AI-assisted deployment
An FPN MCP workflow prepares the specification, estimates cost, requests explicit approval and uses the official Flux MCP to deploy.

Operator configuration should be minimal: role, region, country claim, bandwidth/capacity class, operator public key, payout address and optional policy settings.

A single binary/container can eventually support multiple roles, but security-sensitive roles should be separable so operators can expose only the capabilities they intend.

Community contribution is role-specific. Entry relays, DNS relays and non-exit services should be deployable without automatically accepting public Internet exit liability. A **Community Exit** is a distinct operator role with explicit abuse contact, jurisdiction declaration, outbound policy, bandwidth policy and acknowledgement of operator responsibilities.

## 13. Official and Community Networks

FPN launches with an official gateway fleet so that the first user can install the client and connect immediately. This fleet establishes known-good reference behavior and provides a baseline for testing.

The long-term public network combines:

• Official nodes operated by the project.
• Community nodes operated independently.
• Personal nodes that remain private unless explicitly published.

Client policy options should include Official + Community, Official Only, Community Only and My Gateways Only.

Official infrastructure is a bootstrap and quality mechanism, not a permanent architectural requirement.

## 14. Reputation, Anti-Sybil and Quality

An open operator network creates a Sybil problem: an attacker can claim many fake or malicious nodes. FPN therefore needs a reputation model that does not rely solely on operator self-description.

Initial inputs should prioritize observable proof of service: signed operator identity, Flux/application identity where applicable, uptime history, successful protocol probes, measured latency, packet loss, measured throughput, software/protocol version and independent probe observations. Stake, bonds and hardware/software attestation remain research options rather than MVP trust substitutes.

No single metric should be treated as proof of honesty. Reputation should help routing rather than create an opaque centralized permission system.

The project should publish the scoring algorithm and distinguish measured facts from operator claims.

## 15. Client Applications

Desktop
The recommended initial client is Tauri with a Rust core for Windows, macOS and Linux. It provides a small footprint and creates a path toward a dedicated Rust networking daemon later. The UI should expose one primary Connect action and hide protocol complexity by default.

Android
Use Android VpnService for full-device tunneling and native WireGuard-compatible components where appropriate. DNS-only and application-proxy features can be exposed independently where Android permits.

iOS
Use Network Extension / NEPacketTunnelProvider for VPN functionality and established WireGuard components. Apple's entitlements and App Store VPN policies must be treated as release requirements, not afterthoughts.

Web
A web application can provide onboarding, payment, node information, MyFlux deployment guidance and QR/config workflows. A browser page cannot replace native full-device tunneling.

Shared UX
All platforms should present the same conceptual controls: connection status, location, mode, network policy, My Gateways, DNS Shield, Privacy+ and advanced settings.

## 16. User Experience

The default user journey should require almost no networking knowledge:

Install → Open → Connect.

The client generates local keys, discovers healthy nodes and chooses a suitable default.

An advanced user can open My Gateways and select Deploy Gateway. The workflow should show region, expected infrastructure resources, estimated Flux cost and a Deploy action. After deployment, the application discovers the new gateway, performs health checks and offers Connect.

A community operator can choose Contribute to Network, select a service role, review expected resource requirements and policies, deploy, pass validation and publish a signed node descriptor.

The product must never force ordinary users to understand WireGuard configuration files, Docker ports, Flux application specifications or blockchain memos.

## 17. Privacy and Logging Policy

FPN should be designed for data minimization rather than relying on a marketing promise.

Gateways should not intentionally store browsing history, DNS history, persistent source-to-destination mappings or payment-to-browsing mappings.

Operational metrics may include aggregate bandwidth, active-session counts, CPU, memory, failure rates and health data, provided they cannot reasonably reconstruct individual activity.

Temporary networking state will necessarily exist in memory while forwarding traffic. The whitepaper must distinguish ephemeral processing from persistent logging.

The phrase 'no logs' should only be used after implementation, configuration, observability systems and third-party dependencies have been audited. Debug builds must not silently become production logging configurations.

## 18. Threat Model

FPN is designed to reduce trust, not eliminate all risk.

Threats include malicious gateways, malicious exits, colluding entry/exit operators, compromised clients, traffic-correlation attacks, DNS leakage, WebRTC/application leaks, route hijacking, malicious software updates, Sybil operators, blockchain payment correlation, compromised signing keys, denial of service, abusive users and legal pressure on exit operators.

Out of scope for strong guarantees are compromised end-user devices, destination services that identify users through application-level accounts, global passive adversaries capable of observing both sides of a connection at scale, and user behavior that directly reveals identity.

Privacy+ improves separation of knowledge but should never be described as equivalent to an anonymity network without extensive analysis and evidence.

## 19. Security Engineering

Production readiness requires a security program, not only secure code.

Required practices include:

• reproducible builds where practical;
• signed releases and update manifests;
• SBOM generation;
• dependency and container scanning;
• fuzz testing for parsers and network protocol surfaces;
• least-privilege service separation;
• memory-safe languages for new security-sensitive components where practical;
• external penetration testing;
• independent cryptographic review of anonymous-credit mechanisms;
• public vulnerability disclosure policy;
• security advisories and emergency update procedure;
• secret isolation between deployment, payment and signing roles;
• deterministic configuration validation;
• protocol conformance tests.

MCP and AI tooling must never receive unrestricted production secrets by default. Read-only, operator, deployment and core-administration capabilities should use separate credentials and explicit confirmation gates.

## 20. Abuse, Exit Liability and Governance

Public Internet exit infrastructure creates operational and legal responsibilities. Complaints normally identify the exit address, not the original user. FPN must not imply that operating a Community Exit is equivalent to operating a non-exit relay or is legally risk-free.

Operator roles are separated:

• **Entry Relay** — accepts FPN client/relay traffic but does not provide final public Internet egress.
• **DNS Relay/Resolver** — performs the declared DNS role.
• **Proxy Relay** — provides only explicitly enabled proxy capabilities.
• **Community Exit** — provides final public Internet egress and therefore carries the highest abuse/legal exposure.
• **MyFlux Gateway** — private user-controlled exit, not publicly advertised by default.

A Community Exit descriptor should expose jurisdiction, abuse contact, operator policy version, supported/blocked outbound classes, bandwidth policy and service limits without exposing user activity.

Possible network protections include rate limits, denial-of-service controls, restrictions on high-abuse outbound ports, spam controls, per-service resource ceilings and rapid quarantine of demonstrably malfunctioning nodes. These mechanisms must be designed to avoid creating browsing surveillance.

Before community exits are enabled, the project should publish an operator policy template and obtain jurisdiction-specific legal review for official infrastructure. FPN can provide technical and operational guidance; it cannot guarantee an operator's legal status.

Governance should separate protocol decisions, software maintainership, network policy and commercial operations. The open protocol must remain usable even if one company or foundation ceases to exist.

## 21. Proposed Repository Structure

A practical initial implementation can use a monorepo:

flux-privacy-network/
  apps/
    desktop/
    ios/
    android/
    web/
  services/
    gateway/
    entry/
    exit/
    proxy/
    odoh-relay/
    odoh-resolver/
    entitlement/
    directory/
  crates-or-libs/
    protocol/
    discovery/
    payments/
    routing/
    credentials/
  mcp/
    fpn-mcp/
  deploy/
    docker/
    flux/
    orbit/
  specs/
  docs/
  threat-model/
  tests/
  audits/
  CUMULUS_AUDIT.md
  MVP_ACCEPTANCE_TESTS.md
  SURVIVABILITY_MATRIX.md

As interfaces stabilize, components can be split into independent repositories without changing the protocol.

## 22. Minimum Viable Product (MVP)

The MVP proves one proposition:

> **Can anyone install FPN and either connect immediately to the official beta network or deploy and use their own private gateway — on Flux or generic Docker — without depending on an FPN account or runtime API?**

### 22.1 MVP Product

The MVP contains only two primary connection choices:

**FPN Network** — one-click connection to a small official beta gateway fleet.

**MyFlux Gateway / Private Gateway** — guided deployment and pairing of a private gateway owned by the user. Flux is the preferred path, but the same gateway must be demonstrated on generic Docker.

### 22.2 MVP Gateway

• Go-based service.
• Userspace WireGuard.
• Userspace TCP/IP stack compatible with Flux container restrictions.
• Minimal enrollment/pairing API.
• Health and capability endpoints.
• DNS through the encrypted tunnel.
• No intentional browsing-history logging.
• Docker image and reproducible build instructions.
• Flux application specification.
• Deploy with Git/Orbit instructions.
• Portable Docker operation outside Flux for protocol independence.

### 22.3 MVP Desktop Client

• Windows, macOS and Linux.
• Tauri/Rust architecture.
• Local key generation.
• Connect/disconnect.
• Official gateway selection.
• Resilient Discovery Engine: Flux data when available, signed bootstrap, cache, private pairing and direct probes.
• MyFlux Gateway pairing.
• Basic reconnect and failover.
• Visible connection state and endpoint information.
• Local routing decisions.

### 22.4 MVP Network

• A small official test fleet only after the Phase 0 performance gate passes.
• Signed bootstrap directory.
• Discovery through multiple Flux sources.
• Local health probing.
• No community operator marketplace.
• No mandatory FPN-operated runtime API.

### 22.5 MVP Economics

The initial public beta is free. This intentionally removes wallet/payment friction while networking, discovery and deployment are being validated.

MyFlux Gateway requires no FPN service credits; the user pays only the infrastructure they deploy.

Paid FLUX access is introduced only after the core tunnel workflow is stable **and** unlinkable entitlement has passed security review.

### 22.6 MVP Non-Goals

No anonymity claim.
No community exits.
No multi-hop.
No anonymous credentials.
No MASQUE requirement.
No iOS/Android requirement.
No operator reward marketplace.
No complex Sybil system.
No mandatory MCP.
No mandatory centralized backend.

### 22.7 MVP Success Criteria

A new tester can:

1. obtain or build the desktop client;
2. connect to an official gateway;
3. change gateway and survive a gateway failure;
4. deploy a private MyFlux Gateway from the open repository;
5. pair the client with that gateway;
6. route traffic and DNS through the private gateway;
7. reproduce the gateway build from source;
8. continue using the core system if optional FPN web/API services are unavailable;
9. continue using paired/cached/bootstrap paths when the primary Flux discovery API is unavailable;
10. run the same gateway on generic Docker outside Flux.

## 23. Development Roadmap

Security, testing, documentation and legal work run continuously. The roadmap now begins with an explicit engineering-validation generation before public beta.

### FPN 0.0 — Research, Audit and Benchmark

Objective: invalidate the riskiest assumptions before product expansion.

Deliverables:
• CumulusVPN REUSE / ADAPT / REIMPLEMENT / NOT REQUIRED audit with license evidence;
• userspace WireGuard + netstack benchmark on representative Flux deployments;
• equivalent generic Docker benchmark;
• handshake, throughput, latency overhead, packet loss, CPU, memory, concurrency, NAT and reconnect measurements;
• discovery chaos tests with primary Flux API and FPN services unavailable;
• portable gateway proof outside Flux;
• threat-model update;
• preliminary exit/operator legal work;
• explicit Go/No-Go performance gate for the official beta fleet.

If userspace WireGuard does not meet the agreed gate, alternatives such as different userspace stacks, SoftEther as an interim compatibility path, MASQUE or CONNECT-IP are benchmarked before scaling the fleet.

### FPN 0.1 — Own or Connect

Desktop + userspace WireGuard + official gateways + MyFlux Gateway + generic Docker gateway + free beta.

Deliverables:
• desktop application;
• minimal official multi-region beta fleet after benchmark approval;
• Flux deployment specification;
• generic Docker deployment;
• resilient discovery and signed bootstrap;
• private gateway pairing;
• local routing and failover;
• DNS through tunnel;
• signed releases and baseline security pipeline;
• no public paid access.

### FPN 0.2 — Everywhere

Mobile + operational resilience + developer interfaces.

Deliverables:
• Android client;
• iOS client;
• kill-switch behavior where platform APIs permit;
• DNS Privacy Engine with tunnel DNS/DoH/DoT foundations;
• CLI and initial SDK;
• improved failover and network-change handling;
• privacy-safe diagnostics;
• repeatable benchmark publication.

### FPN 0.3 — Economy and Community

Community infrastructure + Operator ID + quality/trust separation + unlinkable entitlement + FLUX payment.

Deliverables:
• Operator ID;
• signed Network Capability Descriptors;
• community non-exit roles first;
• community exit policy and onboarding after legal/security review;
• independent network probes;
• Quality Score separated from Trust classification;
• Privacy Pass-derived entitlement prototype;
• independent cryptographic review;
• FLUX payment → token issuance → unlinkable gateway redemption;
• production paid access only after the entitlement gate passes;
• operator compensation experiments;
• proof-of-service-focused anti-Sybil work.

### FPN 0.4 — Proxy Privacy

DNS separation + application proxying + agent egress.

Deliverables:
• ODoH relay/resolver roles;
• standalone DNS Shield;
• application proxy mode;
• MASQUE/HTTP/3 production evaluation;
• CONNECT-UDP where appropriate;
• temporary egress API for software/agents;
• continued transport abstraction and conformance testing.

### FPN 1.0 — Privacy+

Independently operated multi-hop + infrastructure diversity + mature economics.

Deliverables:
• Privacy+ Entry → Exit routing;
• mandatory different Operator IDs;
• measured/preferred provider and ASN diversity where available;
• optional jurisdiction-diversity policy;
• nested encryption and circuit rotation;
• separated DNS path options;
• mature operator economics;
• transparent anti-gaming methodology;
• protocol conformance suite;
• independent security audits;
• third-party client/node interoperability;
• progressive reduction of bootstrap and project-operated dependencies.

### Parallel Transport Track

WireGuard remains supported. MASQUE, CONNECT-UDP and CONNECT-IP are integrated behind the transport abstraction only when performance, platform support, implementation maturity and security evidence justify them.

## 24. FPN MCP and Autonomous Operations

FPN can expose an optional MCP interface for infrastructure operations.

Candidate read-only tools:
fpn_get_network_status
fpn_list_nodes
fpn_get_node
fpn_test_latency
fpn_get_node_health
fpn_estimate_cost

Candidate operator tools:
fpn_build_gateway_spec
fpn_deploy_gateway
fpn_deploy_odoh
fpn_update_node
fpn_restart_node
fpn_publish_descriptor

Candidate administrative tools must be isolated and should never be exposed to ordinary users.

An AI agent can therefore help deploy or maintain infrastructure, but the agent must not receive packet contents, DNS histories or user browsing metadata. FPN remains functional without MCP. **MCP MUST NEVER be a runtime dependency of the privacy network.** If MCP services or AI providers are unavailable, established tunnels, discovery and ordinary client operation continue to work.

## 24A. Developer, CLI and Agent Interfaces

FPN should expose developer interfaces early enough that the network can be used by software as well as by graphical clients.

Proposed interfaces include:

• a documented local control API;
• an `fpn` CLI;
• a reusable SDK;
• temporary proxy/tunnel endpoints for applications and autonomous agents;
• an optional FPN MCP for infrastructure deployment and operations.

Illustrative CLI:

`fpn connect --country CH`
`fpn connect --mode proxy`
`fpn connect --gateway my`
`fpn deploy gateway --region eu`
`fpn status`

An application or agent could request a short-lived route with policy such as country, transport, privacy level and duration, then receive a temporary MASQUE/SOCKS/HTTP CONNECT endpoint or a local tunnel binding.

Agent traffic remains ordinary data-plane traffic. AI/MCP systems are never required to inspect packet contents, DNS history or browsing metadata.

## 25. API and Protocol Surface

Initial gateway API candidates:

GET /v1/info
Returns protocol version, role, capabilities, software version, health summary and public operator identity.

POST /v1/enroll
Accepts the minimum material needed to establish a tunnel credential. The MVP may use a WireGuard public key. Later versions should support short-lived entitlement credentials.

GET /v1/health
Machine-readable health information without user-specific data.

GET /v1/capabilities
Supported transports, DNS features, proxy features and protocol versions.

A Network Capability Descriptor should be a first-class protocol object. A conceptual descriptor contains:

```json
{
  "protocol": "fpn/1",
  "node_id": "node-public-id",
  "operator_id": "operator-public-id",
  "roles": ["gateway", "exit", "proxy"],
  "transports": ["wireguard"],
  "dns": ["tunnel", "odoh"],
  "region": "EU",
  "country": "FR",
  "capacity": {"bandwidth_mbps": 1000},
  "version": "0.1.0"
}
```

The descriptor is cryptographically signed by the operator/node identity. Future releases can add `masque`, `connect-udp` and `connect-ip` capabilities without redefining the network.

Node descriptors should be versioned and signed. Protocol evolution must be backward-compatible where practical and must provide explicit minimum/maximum supported versions rather than relying on implicit behavior.

## 26. Observability Without Surveillance

FPN needs enough observability to operate a reliable network without collecting browsing histories.

Acceptable aggregate metrics may include:
• node uptime;
• CPU/RAM usage;
• total bytes transferred;
• aggregate active sessions;
• connection failure rate;
• handshake success rate;
• median latency;
• software version;
• protocol error counts.

Sensitive metrics to avoid include:
• destination-domain logs;
• per-user destination lists;
• persistent source IP logs;
• source-to-destination correlation tables;
• payment-to-session correlation logs.

Debugging features that temporarily increase logging must be explicit, time-bounded and unsuitable for public production nodes by default.

## 27. Performance and Go/No-Go Gates

FPN does not publish aspirational throughput claims before measurement. Phase 0 establishes a repeatable benchmark harness and records at least:

• WireGuard handshake latency;
• download and upload throughput;
• latency overhead versus direct connection;
• packet loss;
• CPU cost per throughput level;
• memory per active session;
• maximum stable concurrent sessions per resource class;
• UDP behavior;
• NAT behavior;
• reconnect time;
• gateway failover time;
• performance on Flux versus equivalent generic Docker infrastructure.

Before an official beta fleet is expanded, maintainers approve explicit numeric thresholds in `MVP_ACCEPTANCE_TESTS.md`. Thresholds may change as evidence improves, but results and methodology should be public.

A failed performance gate is not a failed project. It is a transport decision point. The team benchmarks alternative userspace implementations or transports before committing the public network to an underperforming stack.

## 28. Open Questions

Key research questions include:

1. What parts of CumulusVPN can legally and technically be reused?
2. What is the most efficient userspace WireGuard/netstack combination on Flux?
3. How should MyFlux Gateway pairing work without a central account?
4. Which discovery sources provide sufficient independence from a single Flux API endpoint?
5. Which Privacy Pass token variant and deployment model best fits prepaid FPN entitlement, and what metadata/denomination model preserves unlinkability?
6. How should Privacy Pass issuer availability, key rotation and compromise recovery work without becoming a permanent central runtime dependency?
7. How should operator rewards be measured without per-user surveillance?
8. How should FPN resist Sybil gateways?
9. How should operator geography be verified or represented as a claim?
10. How should clients detect malicious or misconfigured DNS/proxy nodes?
11. How should entry and exit ownership diversity be measured?
12. What are the real performance costs of userspace WireGuard on Flux?
13. When does MASQUE provide enough benefit to justify production complexity?
14. How should iOS payment and VPN rules shape mobile distribution?
15. What abuse controls are compatible with the privacy model?
16. Which jurisdictions are practical for official exit infrastructure?
17. What information belongs on-chain and what must remain off-chain?
18. How can software updates remain secure if the original organization disappears?
19. What emergency mechanism can revoke a critically vulnerable node/software version without becoming a permanent centralized kill switch?
20. What evidence would justify stronger anonymity claims in the future?

## 29. Community Participation

FPN is intended to be developed in public. Contributions are needed from networking engineers, Go/Rust developers, iOS and Android engineers, cryptographers, Flux operators, security researchers, UX designers, privacy researchers, DevOps engineers, legal specialists and beta testers.

Early contributors can help by reviewing the threat model, auditing CumulusVPN and related implementations, benchmarking userspace networking on Flux, building reproducible containers, testing discovery under failure, implementing desktop clients, reviewing mobile constraints, designing ODoH deployment, evaluating MASQUE libraries and stress-testing payment correlation assumptions.

Beta testers should be given clear expectations: early releases are experimental privacy software and should not be relied upon for high-risk anonymity use cases.

## 30. Licensing and Open-Source Policy

The project should adopt explicit licenses from its first commit.

A permissive license such as Apache-2.0 or MIT simplifies reuse, while copyleft licenses can require improvements to remain open. The final choice should reflect project goals and dependency compatibility.

Every imported dependency or copied component must have its license recorded. CumulusVPN and other public repositories must be audited for actual license terms before code reuse.

Documentation, protocol specifications and branding should have separate, explicit licensing where appropriate. The software license should not automatically grant trademark rights.

## 31. Principles for Claims and Marketing

Privacy software must avoid claims that exceed its architecture.

The public README and product onboarding should state plainly: **FPN is not an anonymity network.** Privacy+ may reduce knowledge concentration, but FPN 0.x must not be presented as Tor-like anonymity.

FPN should not claim:
• perfect anonymity;
• invisibility from all observers;
• immunity from traffic analysis;
• zero metadata under all circumstances;
• legal protection for exit operators.

FPN can accurately describe:
• accountless architecture when implemented;
• local key generation;
• open-source components;
• encrypted tunnels;
• ODoH separation properties;
• independently operated multi-hop routing;
• absence of intentionally retained browsing-history logs when verified;
• ability to self-host MyFlux Gateway;
• use of the Flux blockchain for payment verification.

Security and privacy claims should be versioned and tied to audited releases.

## 32. Long-Term End State

The long-term FPN network should allow four independent actions:

A user can connect without understanding Flux.
A user can own their complete exit infrastructure.
An operator can contribute capacity without asking a central company for permission.
A third party can build a compatible client or service from the public protocol.

The strongest measure of decentralization is survivability. `SURVIVABILITY_MATRIX.md` defines this as a set of concrete dependency failures rather than a slogan. The project must test behavior when the FPN website/API, primary Flux discovery API, MCP services, official gateways, payment verifier/issuer or original organization are unavailable. Some failures may degrade new enrollment or payment while existing/private connectivity continues; those limits must be documented honestly.

FPN therefore aims to become an open privacy protocol with a Flux-native public network: not merely a VPN hosted on Flux, but a modular relay system built around user ownership, distributed infrastructure, transport choice and minimized trust.

## 32A. Revision v1.2 — Engineering Decisions

This revision incorporates external technical review and resolves the main tensions identified in v1.1:

1. Adds FPN 0.0 Research/Audit/Benchmark before public beta.
2. Treats userspace WireGuard performance on Flux as the primary MVP technical risk and introduces a Go/No-Go gate.
3. Defines CumulusVPN as an upstream/reference implementation for the Flux-native VPN layer and requires a component/license audit before reuse.
4. Replaces broad “decentralized discovery” language with multi-source resilient discovery and explicit degraded-mode tests.
5. Requires generic Docker operation to be demonstrated in the MVP, proving protocol portability outside Flux.
6. Makes the public MVP free and prohibits normal production paid access until unlinkable entitlement is ready.
7. Selects IETF Privacy Pass as the current candidate architecture for FLUX-payment-to-unlinkable-entitlement separation, subject to independent cryptographic review.
8. Moves unlinkable entitlement into FPN 0.3 alongside FLUX payments rather than leaving it for a later privacy phase.
9. Separates community non-exit roles from Community Exit and strengthens abuse/legal operator requirements.
10. Keeps Privacy+ out of launch claims until Operator ID and meaningful administrative/infrastructure diversity exist.
11. Adds explicit failure testing for Flux API, FPN services, MCP, official gateways and entitlement infrastructure.
12. States plainly that FPN is not an anonymity network.
13. Adds `CUMULUS_AUDIT.md`, `MVP_ACCEPTANCE_TESTS.md` and `SURVIVABILITY_MATRIX.md` as first-class engineering documents.

## 33. References and Technical Starting Points

RunOnFlux CumulusVPN:
https://github.com/RunOnFlux/cumulusvpn

Flux v9 whitepaper announcement / source documents:
https://runonflux.com/flux-v9-whitepaper/

Privacy Pass Architecture — RFC 9576:
https://www.rfc-editor.org/rfc/rfc9576

Privacy Pass HTTP Authentication Scheme — RFC 9577:
https://www.rfc-editor.org/rfc/rfc9577

Privacy Pass Issuance Protocols — RFC 9578:
https://www.rfc-editor.org/rfc/rfc9578

Flux Cloud official MCP documentation:
https://docs.runonflux.com/fluxcloud/ai-agents-mcp/

Flux Deploy with Git / Orbit:
https://docs.runonflux.com/fluxcloud/register-new-app/deploy-with-git/introduction/

Flux GitHub organization:
https://github.com/RunOnFlux

ODoH — RFC 9230:
https://www.rfc-editor.org/rfc/rfc9230

MASQUE working group / standards:
https://datatracker.ietf.org/wg/masque/about/

CONNECT-IP — RFC 9484:
https://www.rfc-editor.org/rfc/rfc9484

Encrypted Client Hello:
https://datatracker.ietf.org/doc/rfc9849/

WireGuard:
https://www.wireguard.com/

gVisor:
https://gvisor.dev/

Tauri:
https://tauri.app/

This list is a technical starting point, not an endorsement of every implementation or a statement that every referenced codebase can be copied. License and security review remain mandatory.
