# DPP-Industrial

> A tokenless verifiable-credential infrastructure for the EU Digital Product Passport in industrial measurement equipment.

![Status](https://img.shields.io/badge/status-pre--MVP-yellow)
![Stack](https://img.shields.io/badge/stack-Base%20L2%20%C2%B7%20EAS%20%C2%B7%20W3C%20VC%202.0-1E3A5F)
![License](https://img.shields.io/badge/license-TBD-lightgrey)

---

## Status

**This project is in pre-implementation phase (MVP 0 prep).** The architecture, data model, and protocol specification are complete; the production codebase is not yet written. The reference documents define the design:

- [Academic Whitepaper](./docs/Whitepaper.pdf) — 21 pages, formal architecture description with related work and references

---

## What it is

DPP-Industrial provides production infrastructure for European manufacturers to comply with the EU Ecodesign for Sustainable Products Regulation (ESPR, EU 2024/1781) and the EU Machinery Regulation 2023/1230. The system issues, manages, and verifies cryptographically anchored lifecycle records — manufacture, conformity, calibration, repair, ownership transfer, decommissioning — for industrial measurement equipment and other accredited industrial assets.

Key design choices:

- **Tokenless.** No fungible or transferable on-chain token. The only on-chain primitive is ERC-5192 (soulbound), used as a per-asset identity anchor. GDPR- and MiCA-aligned.
- **Off-chain payloads, on-chain hashes.** No personal or commercially sensitive data on chain. Cryptographic deletion satisfies GDPR Art. 17 obligations.
- **Issuer = real-world authority.** Manufacturers, notified bodies, and DAkkS-accredited calibration laboratories sign attestations with their existing institutional identity (`did:web`). The platform is infrastructure, not an issuer.
- **EU-Registry-compatible, not a substitute.** Every asset is registered in the EU Central DPP Registry (legal obligation) and additionally verifiably attested on chain (added value).
- **Selective disclosure by default.** SD-JWT and Noir zero-knowledge proofs for verifier interactions; full disclosure is opt-in.

---

## Architecture at a glance

Nine layers, of which seven are general-purpose verifiable-credential infrastructure and two are DPP-specific:

| # | Layer | Technology |
|---|---|---|
| 9 | Application | Issuer Portal · Owner Mobile App · Verifier Console |
| 8 | API Gateway | REST + GraphQL · OIDC4VCI/VP · Webhooks |
| 7 | Data Carrier *(DPP-specific)* | GS1 Digital Link · QR · NFC NTAG 424 DNA |
| 6 | EU Registry Sync *(DPP-specific)* | EU Central DPP Registry connector |
| 5 | ZK / Privacy | Noir circuits · Mopro mobile prover · SD-JWT |
| 4 | Credentials | W3C VC 2.0 (final 2025) · EBSI/EUDI compatible |
| 3 | Attestations / Identity | EAS on Base · ERC-5192 SBT · did:web / did:pkh |
| 2 | Storage | Filebase (hot S3) · Arweave + Bundlr (permanent) |
| 1 | Chain | Base L2 (OP-Stack) · Coinbase Smart Wallet |

Full architecture description: [Section 4 of the technical architecture document](./docs/Tech_Architecture.pdf).

---

## Quick start

> The commands below describe the planned developer workflow. Most are not yet operational. This section will be filled in during MVP 0.

```bash
# Prerequisites: Node.js 20+, Foundry, Docker, a Base Sepolia RPC endpoint
git clone https://github.com/daavidsc/dpp-industrial.git
cd dpp-industrial
pnpm install

# Run the local dev stack (Hardhat + local services + mock NFC)
pnpm dev

# Run tests
pnpm test

# Deploy the EAS schemas to Base Sepolia
pnpm deploy:schemas --network base-sepolia

# Mint a test asset and issue a genesis attestation
pnpm cli mint-asset --serial TEST-001
```

---

## Project structure

```
dpp-industrial/
├── apps/
│   ├── issuer-portal/         # Next.js web app for manufacturers and labs
│   ├── owner-mobile/          # React Native app (iOS + Android)
│   └── verifier-console/      # Verifier web app + companion mobile
├── packages/
│   ├── attestation-service/   # EAS adapter, schema registry, refUID chaining
│   ├── storage-service/       # Filebase + Arweave dual-write, encryption
│   ├── did-service/           # did:web resolution + did:pkh derivation
│   ├── registry-sync/         # EU Central DPP Registry connector
│   ├── zk-circuits/           # Noir circuits + Mopro mobile prover bindings
│   └── shared/                # Schemas, types, crypto utilities
├── contracts/
│   ├── src/                   # EAS resolvers, ERC-5192 SBT, MultiSig
│   └── test/                  # Foundry test suite
├── docs/
│   ├── Tech_Architecture.pdf  # Implementation specification
│   ├── Whitepaper.pdf         # Academic description
│   └── adr/                   # Architecture decision records
└── infra/
    └── terraform/             # AWS / Azure deployment definitions
```

---

## Roadmap

Five MVPs across thirty months. Each MVP isolates one hypothesis and one primary risk.

| MVP | Phase | Duration | Hypothesis |
|---|---|---|---|
| 0 | Local Proof | months 0–2 | Stack works end-to-end on Hardhat |
| 1 | Testnet Pilot | months 2–6 | A real calibration lab can integrate without friction |
| 2 | Production Soft-Launch | months 6–12 | System handles production load; first paying customer |
| 3 | Compliance Layer | months 12–20 | ESPR-compliant, EUDI-compatible, notified-body partner |
| 4 | Marketplace & Scale | months 20–30 | Service marketplace with positive unit economics |

Detailed scope and exit criteria for each MVP are in [Section 9 of the technical architecture document](./docs/Tech_Architecture.pdf).

---

## Documentation

| Document | Purpose | Audience |
|---|---|---|
| [Whitepaper](./docs/Whitepaper.pdf) | Formal architecture description with references | Technical reviewers, academic readers, due-diligence |
| [Technical Architecture](./docs/Tech_Architecture.pdf) | Layer-by-layer implementation specification | Engineers, CTO candidates |
| [ADRs](./docs/adr/) | Architecture Decision Records | Maintainers, contributors |
| [Threat Model](./docs/threat-model.md) | Adversarial analysis and mitigations | Security reviewers |

---

## Standards and dependencies

Built on production-grade open standards:

- [W3C Verifiable Credentials Data Model 2.0](https://www.w3.org/TR/vc-data-model-2.0/)
- [W3C Decentralized Identifiers 1.0](https://www.w3.org/TR/did-core/)
- [EIP-5192 Minimal Soulbound NFTs](https://eips.ethereum.org/EIPS/eip-5192)
- [Ethereum Attestation Service](https://docs.attest.org)
- [Base L2 Documentation](https://docs.base.org)
- [GS1 Digital Link (ISO/IEC 18975)](https://www.gs1.org/standards/gs1-digital-link)
- [Noir (Aztec)](https://noir-lang.org)
- [SD-JWT (IETF draft)](https://datatracker.ietf.org/doc/draft-ietf-oauth-selective-disclosure-jwt/)
- [OpenID4VCI / OpenID4VP](https://openid.net/openid4vc/)

Regulatory framework:

- [EU 2024/1781 (ESPR)](https://eur-lex.europa.eu/eli/reg/2024/1781/oj)
- [EU 2023/1230 (Machinery Regulation)](https://eur-lex.europa.eu/eli/reg/2023/1230/oj)
- [EU 2016/679 (GDPR)](https://eur-lex.europa.eu/eli/reg/2016/679/oj)
- [EU 2023/1114 (MiCA)](https://eur-lex.europa.eu/eli/reg/2023/1114/oj)
- [CIRPASS-2 Consortium](https://cirpassproject.eu/)

---

## Contributing

This project is in pre-implementation phase and not yet open to external contributions. If you are interested in joining as a co-founder, early engineer, or pilot partner, see the contact section below.

When the codebase opens for contribution, we will follow:

- Trunk-based development with short-lived feature branches
- Conventional Commits for commit messages
- ADRs for any change to the public API surface or to security-relevant decisions
- All smart contracts changes require two reviewers; one must be the security lead
- Test coverage minimum 80% for new code; 100% for cryptographic primitives

---

## Security

If you discover a security vulnerability, please **do not** open a public issue. Email the security team at `security@<domain>.tbd` with a description and proof of concept. We commit to acknowledging within 48 hours.

External security audit by an established Web3 security firm (Trail of Bits, OpenZeppelin, or ChainSecurity DACH) is scheduled before MVP 2 mainnet deployment.

---

## License

License is not yet determined. The likely choice is a dual model: Apache 2.0 for the SDK and client libraries (to encourage ecosystem adoption) and a source-available license (e.g., Business Source License 1.1) for the platform-operator code. This will be finalized before the first public commit.

---

## Contact

For pilot partnerships, investment inquiries, or technical co-founder discussions:

**David Schmitt** 
[LinkedIn](https://de.linkedin.com/in/david-schmitt-b5364a18b) 

---

*This README is part of the DPP-Industrial public-facing repository. Detailed technical materials are available under NDA upon request.*
