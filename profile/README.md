# Wallet8 - Digital Identity Wallet

**Open-Source Secure Post-Quantum-Safe Digital Wallet Platform That Heals and Scales Itself.**

**Wallet8 is [OpenID Certified](https://sentin8.dev/certification)** - certified by the OpenID Foundation for OpenID4VCI and OpenID4VP, including the High Assurance Interoperability Profile (HAIP).

Wallet8 is the wallet vertical of the Sentin8 platform - the missing WALLET role
of the digital-credential stack: the credential vault, the OID4VCI issuance
client, the OID4VP presentation client, and the reference wallet app. Every
vertical (bank8's "Bank8 Wallet", a hotel's guest wallet, a health plan's member
card) embeds the same hardened SDKs and ships its own branded wallet instead of
building verifiable-credential protocols from scratch.

**Standards-first is architectural:** the wallet is built from the
normative texts of the OIDF/IETF/ISO specifications, section by
section. Every SDK protocol function cites its spec section; when a
conformance-suite finding and a specification disagree, the
discrepancy is resolved against the specification's text. Wallet8
implements
**OpenID for Verifiable Credential Issuance 1.0** (Final, 16 Sep 2025) and
**OpenID for Verifiable Presentations 1.0** (Final, 9 Jul 2025) section by
section - Credential Offers by value/reference, both grant flows (Authorization
Code with PAR+PKCE, Pre-Authorized with the End-User's Transaction Code), the
§5.9 request-object verification with the full Client Identifier Prefix resolver
(`x509_san_dns`, `x509_hash`, `verifier_attestation`, `redirect_uri`,
`openid_federation`, DIDs), DCQL credential/claim selection (the exact §6.4.1/
§6.4.2 preference rules), vp_token assembly for `mso_mdoc` (the ISO/IEC
18013-5 DeviceResponse with the OpenID4VP handover transcripts) and
`dc+sd-jwt` (IETF SD-JWT VC over RFC 9901 with the Key Binding JWT), the §8.3
encrypted responses (ECDH-ES over A128GCM/A256GCM), the Digital Credentials API
channel (App. A: origin-bounded, promise-based), the §8.5 error vocabulary,
DPoP proofs (RFC 9449), key attestations, and the IETF Token Status List
(draft-14) revocation layer - targeting the **OIDF wallet-role certification**
(`oid4vci-1_0-wallet(-haip)`, `oid4vp-1final-wallet(-haip)`) and the
**OpenID4VC High Assurance Interoperability Profile 1.0**, the same protocol
layer the **EU Digital Identity Wallet (eIDAS 2.0)** framework builds on.

## Infrastructure independence

**Sentin8** and Wallet8 redefine infrastructure independence by delivering
massive scale directly onto bare-metal, private cloud, and all major
public cloud environments - including **AWS, GCP, and Azure**. Engineered
to bypass the complexity and overhead of Kubernetes or heavy service
meshes, Sentin8 features native auto-scaling and self-healing capabilities
built directly into its core with **minimal to no external dependencies**.
It is purpose-built for ultra-low latency and maximum efficiency, proven
in testing to power real-time services with a blistering throughput of
5,000,000 transactions per second (TPS). With Sentin8, you get
cloud-native resilience and elite performance, stripped of the operational
tax.

Architected for ultimate versatility, Sentin8 is being designed and tested
to power infrastructure across the entire business spectrum, from
**Tier 1 global enterprises to fast-growing startups**. Whether you are
managing massive legacy workloads at the highest enterprise scale or
launching a lean, high-velocity application from a garage, Sentin8
provides the exact same battle-tested reliability and performance to
match your operational needs.

> **A note on our history.** If you found your way here looking for
> **PRIME** - `authPrime`, `exchangePRIME`, and family - you're in the
> right place. Those projects are this one: our open-source
> work is now rebranded to **Sentin8** (this vertical is **Wallet8**) to
> avoid confusion with other brand names and products. At the same
> time we left our self-hosted **Apache SVN** behind and moved
> everything to GitHub. The lineage goes back further than you might
> expect - here's a fun LinkedIn read from 2017, when the platform was
> still called PRIME and still lived on SVN:
> [Advanced Hybrid Securities Exchange](https://www.linkedin.com/pulse/advanced-hybrid-securities-exchange-hamid-jahanzad/).

## Components

| Component | What it does |
|---|---|
| **Wallet8:SDK** (`wallet8-sdk`, the `wallet8` facade crate) | The Rust protocol drivers: the OID4VCI issuance client, the OID4VP presentation client (verification, DCQL, the encrypted/dc_api responses, the status layer), the holder-service contracts. Consumed by any vertical's backend. |
| **Wallet8:Holder** (`wallet8-holder`) | The holder backend service: the AES-256-GCM sealed credential vault, device-key registration (RFC 7638 thumbprints), the authorization-code orchestration (§3.4), presentation resolve/deliver with the §5.9 verification, the presentation history, the Token Status List sync. |
| **Wallet8:Flutter** (`wallet8-flutter`) | The Flutter wallet SDK: the credential store, the OID4VCI/OID4VP drivers (the session-based consent flow, the deferred polling, the authorize round), the status sync - what every branded wallet embeds. |
| **Wallet8:Wallet** (`wallet8-wallet`) | The reference Flutter web wallet: the certification surfaces (the vault, the scan intake, the consent preview, the error screen, the outcome landing), the deep-link routes, the DC API shim. |
| **Wallet8:E2E** (`wallet8-e2e`) | The Playwright harness: the happy-path flows + the OIDF negative modules (a bad-signature request, a missing nonce, an unsatisfiable DCQL query - each lands a visible error screen, never an empty vp_token). |

## The credential formats

| Format | Standard | What Wallet8 holds |
|---|---|---|
| `mso_mdoc` | ISO/IEC 18013-5/-7 (+ 23220) | The IssuerSigned vault rows; the DeviceResponse assembly over the B.2.6.1/B.2.6.2 SessionTranscripts. The wire engine is the PLATFORM's `sentin8::mdoc` - one engine, three roles (auth issuer, auth verifier, wallets). |
| `dc+sd-jwt` | IETF SD-JWT VC (RFC 9901 substrate) | Selective disclosure via the salted-hash disclosures; the SD-JWT + Key Binding JWT presentation (`nonce`, `aud`, `sd_hash`); the `status.status_list` revocation surface. |
| W3C VCDM | `jwt_vc_json` / `ldp_vc` | The DCQL format registry is the extension point - a vertical adds a format arm, not a protocol change. |

## Platform Dependencies

Wallet8 depends on these Sentin8 platform services for cross-cutting concerns:

| Platform Service | What Wallet8 gets from it | Dependency direction |
|---|---|---|
| **Sentin8:Auth** | The issuer + verifier roles the wallet talks to; the single identity plane (one token issuer, one JWKS) | wallet8 → Auth (issuance + presentation rounds) |
| **Sentin8:BFF** | Proxy, delegation gateway, client session custody. Flutter apps never call services directly. | wallet8 ← BFF (user tokens proxied to the holder) |
| **Sentin8:SDK** (`sentin8::mdoc`) | The shared mdoc wire engine (the MSO, the IssuerSigned, the DeviceResponse, the session transcripts) | wallet8 → SDK (the wire forms) |
| **Sentin8:Shell** | The service boot: registration, the PQ-TLS listener, the config, the pool | wallet8 → Shell (the holder boot) |
| **Sentin8:Bus** | Durable event transport (the business events: `credential:stored`, `credential:revoked`, `presentation:recorded`) | wallet8 → Bus (event emission) |
| **Sentin8:Stat** | Business metrics aggregation and telemetry | wallet8 → Stat (event ingestion) |
| **Sentin8:Sys** | Organization/environment lifecycle, service enablement | wallet8 → Sys (env-init events) |

## Key Invariants

- **Nothing unsealed leaves the vault** - credential material persists
  AES-256-GCM sealed; the API surface exposes secret references; a
  presentation is the only unseal, and only under consent.
- **The device key never leaves the device** - only the public JWK registers;
  the issuance proofs and Key Binding JWTs prove possession of the private
  half in place.
- **No consent, no disclosure** - the consent screen shows exactly the
  requested claims BEFORE any release; the decline answers
  `access_denied` (§15.1 + §8.5).
- **Never proceed on an unverified request** - the JAR signature and the
  Client Identifier Prefix rules verify BEFORE the shape gate (§15.8's
  phishing defense); an unsigned request never reaches the DCQL or the
  consent screen.
- **No partial presentations** - an unsatisfiable required DCQL query/set
  fails the whole request; never an empty or partial vp_token (§6.4.2).
- **A revoked credential stops presenting** - the Token Status List
  reference decodes, the TTL-cached list fetch validates the `sub`/`ttl`
  ladder, and a non-VALID entry flips the vault row (TSL draft-14).
- **Spec-first, always** - every SDK function cites its normative
  section; a deviance from the specification is resolved against the
  specification's own text.

---

**Architecture & Design:** wallet8 and its underlying concepts were designed by
[Hamid Jahanzad](https://www.linkedin.com/in/hamidjahanzad/).
