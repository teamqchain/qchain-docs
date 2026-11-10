<p align="center">
  <img src="assets/qchain_logo.jpeg" width="220" alt="QChain logo" />
</p>

<h1 align="center">qchain-docs</h1>

<p align="center">
  <b>The documentation of QChain: reports, the setup guide and the preliminary benchmarks.</b><br/>
  Everything about the project that isn't code: what was built and why, how to set it up, and the first post-quantum measurements.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Docs-Markdown-000000" alt="Markdown" />
  <img src="https://img.shields.io/badge/University%20of%20Sharjah-Capstone-0A66C2" alt="Capstone" />
  <img src="https://img.shields.io/badge/PQC-ML--DSA--44%2F65%2F87-6E40C9" alt="ML-DSA" />
  <img src="https://img.shields.io/badge/License-Proprietary-red" alt="License" />
</p>

---

## 🌐 Live

| What | Link |
|------|------|
| **Project website** | **https://teamqchain.github.io** |
| **QPortal** (issuer, verifier and IT-admin web portal) | **https://qportal.vercel.app** |
| **QChain API** | **https://qchain.tail4fff4b.ts.net/api/health** |

> The live system is a demo on a university VM: it may be offline at times. See
> [qchain-backend](https://github.com/teamqchain/qchain-backend) and
> [qchain-portal](https://github.com/teamqchain/qchain-portal).

---

## Overview

**QChain** is a post-quantum credential system, built as a University of Sharjah capstone project. Each credential
is signed with **ML-DSA-44 (CRYSTALS-Dilithium)** and anchored on a permissioned **Hyperledger Fabric** network whose
own identities are ML-DSA-44; the credential's attributes live only on **IPFS**, encrypted end-to-end to the
holder's **ML-KEM-768** key.

This repository holds the project's documents:

| Part | What it is |
|------|------------|
| `docs/setup.md` | the whole-system setup guide: prerequisites, the Fabric network, the database, IPFS, the backend, the apps and CI/CD |
| `docs/junior/` | the Junior-year deliverables (Spring 2026) as Markdown: team charter, literature review, progress reports 1 and 2, final report |
| `docs/senior/` | the Senior-year progress reports (Fall 2026) as Markdown |
| `algo-benchmarking/` | the first ML-DSA benchmarks, which came before the detailed study in [qchain-kpi](https://github.com/teamqchain/qchain-kpi) |
| `assets/` | the QChain logo and the benchmark graphs |
| `docs/archive/` | the README of the former single `QChain-PQC-blockchain` repository (changelog and roadmap up to 9 November 2026) |

---

## Architecture

```
                         QChain, by repository
   ┌─────────────────────────────────────────────────────────────────────┐
   │  qchain-portal (web, Vercel)      qchain-wallet (mobile)            │
   │            └──── both use qchain-shared (certificate) ────┘          │
   │                                │ HTTPS                              │
   │                                ▼                                    │
   │  qchain-backend: Go API · MySQL schema · gateway · CI/CD · tests    │
   │                                │ Fabric Gateway                     │
   │                                ▼                                    │
   │  qchain-network: Hyperledger Fabric, ML-DSA-44 MSP, chaincode       │
   └─────────────────────────────────────────────────────────────────────┘
       qchain-docs (this repository)  ·  qchain-kpi (benchmarks)  ·  teamqchain.github.io (website)
```

---

## Preliminary benchmarks

`algo-benchmarking/` compares **ML-DSA-44, ML-DSA-65 and ML-DSA-87** across two Go implementations:

| Folder | Implementation |
|--------|----------------|
| `algo-benchmarking/go-bindings/` | **liboqs-go**, the C library through Go bindings (the one QChain uses) |
| `algo-benchmarking/go-native/` | a pure-Go implementation (Cloudflare CIRCL) |

Results and graphs: [algo-benchmarking/results.md](algo-benchmarking/results.md) and `assets/`. The detailed,
controlled study across three machines is in [qchain-kpi](https://github.com/teamqchain/qchain-kpi).

---

## Repository structure

```
qchain-docs/
├── docs/
│   ├── setup.md              # whole-system setup guide
│   ├── junior/               # team-charter, literature-review, progress-report-1/2, final-report
│   ├── senior/               # progress-report-1, progress-report-2
│   └── archive/              # README of the former QChain-PQC-blockchain repository
├── algo-benchmarking/
│   ├── go-bindings/          # liboqs-go benchmark (Dockerfile, main.go, tests)
│   ├── go-native/            # pure-Go benchmark
│   ├── results.md            # numbers and graphs
│   └── sample_transcript.pdf # the sample document that was signed
├── assets/                   # qchain_logo.jpeg, benchmark graphs
└── LICENSE
```

---

## Getting started

### Read

Everything is Markdown and renders on GitHub. Start with `docs/setup.md` to stand the system up, or the reports in
`docs/junior/` and `docs/senior/` for the design and its history.

### Run the preliminary benchmarks

```bash
git clone https://github.com/teamqchain/qchain-docs.git
cd qchain-docs/algo-benchmarking
```

Then follow [algo-benchmarking/README.md](algo-benchmarking/README.md) (the liboqs-go benchmark runs in Docker).

---

## Recent updates

- **Separate repositories (9 November 2026).** The project's single repository, `QChain-PQC-blockchain`, was split into
  `qchain-portal`, `qchain-wallet`, `qchain-shared`, `qchain-backend`, `qchain-network` and this one, each with its
  own history. See [Where things moved](#where-things-moved).
- **Post-quantum Fabric MSP, live since 7 November 2026.** The network runs on ML-DSA-44: peers, orderer, admins and
  application users, including peer and orderer TLS certificates. goleveldb replaces CouchDB, the chaincode runs as a
  service, and the backend signs Gateway transactions with ML-DSA on Go 1.27. The end-to-end API test passes 123/123.
- **Chain and IPFS as the source of truth.** The ledger stores a signed commitment (metadata, CID, salted field
  hashes), never the plaintext credential, which lives only on IPFS, encrypted to the holder. Chaincode v2.0.
- **QWallet is mobile-only**, because its post-quantum library cannot target the web.
- **CI/CD.** Pushing to the backend's `main` auto-deploys it through a self-hosted runner, with automatic rollback.

The full history up to 9 November 2026 is in [the archived README](docs/archive/QChain-PQC-blockchain-README.md#recent-updates).

---

## Roadmap

**Phase 1: authenticity, integrity and confidentiality (complete).** ML-DSA-44 signatures over a signed commitment, a
commitment-on-chain / data-off-chain model with the credential body encrypted end-to-end to the holder's ML-KEM-768
key, salted per-field hashes for selective disclosure, the full issuance, presentation, verification and revocation
lifecycle, the two apps and a publicly reachable demo.

**Phase 2: post-quantum Fabric MSP (complete).** The MSP identities and the peer and orderer TLS certificates use
ML-DSA-44, pure post-quantum and not hybrid. The only classical certificates left are the two CA servers' own TLS
certificates (see [qchain-network](https://github.com/teamqchain/qchain-network)).

**Next:** a multi-organisation network (each organisation running its own peers and keys), issuer signing on the
university's side, and real authentication for the API.

---

## Where things moved

The reports and the [archived README](docs/archive/QChain-PQC-blockchain-README.md) refer to paths in the former
`QChain-PQC-blockchain` repository. Where they are now:

| Former path | Now |
|-------------|-----|
| `UI_WebApp/` (earlier `WebApp_UI/`) | [qchain-portal](https://github.com/teamqchain/qchain-portal) |
| `UI_App/` (earlier `App_UI/`) | [qchain-wallet](https://github.com/teamqchain/qchain-wallet) |
| `shared/` | [qchain-shared](https://github.com/teamqchain/qchain-shared) (private) |
| `offchain/*.go` | [qchain-backend](https://github.com/teamqchain/qchain-backend), in `api/`, `db/`, `ledger/`, `pqc/` and `config/`; `server.go` is now `main.go` |
| `offchain/cmd/`, `offchain/Dockerfile`, `offchain/docker-*.sh` | qchain-backend: `cmd/` and `docker/` |
| `web-gateway/` | qchain-backend: `gateway/` (API only; QPortal is hosted on Vercel) |
| `tests/e2e_api_test.sh` | qchain-backend: `tests/` |
| `.github/workflows/` | qchain-backend: `.github/workflows/` |
| `qchain-network/scripts/schema.sql` | qchain-backend: `db/schema.sql` |
| `qchain-network/scripts/setup-demo.sh`, `start-demo.sh`, `setup-ipfs-service.sh` | qchain-backend: `scripts/` |
| `qchain-network/scripts/setup-tailscale-funnel.sh` | qchain-backend: `gateway/` |
| the rest of `qchain-network/` | [qchain-network](https://github.com/teamqchain/qchain-network) |
| `docs/`, `assets/`, `algo-benchmarking/` | this repository (`docs/home.html`, `docs/index.html` and `docs/assets/` are gone: the website is [teamqchain.github.io](https://github.com/teamqchain/teamqchain.github.io)) |
| the root `README.md` | `docs/archive/QChain-PQC-blockchain-README.md` |

Each repository kept the history of its files, including under their earlier names.

---

## Related repositories

| Repository | What it holds |
|------------|---------------|
| [qchain-portal](https://github.com/teamqchain/qchain-portal) | QPortal, the web portal for issuers, verifiers and IT admins |
| [qchain-wallet](https://github.com/teamqchain/qchain-wallet) | QWallet, the holder's mobile app |
| [qchain-shared](https://github.com/teamqchain/qchain-shared) | the shared Flutter package: certificate template and viewer |
| [qchain-backend](https://github.com/teamqchain/qchain-backend) | the Go REST API, MySQL schema, gateway, CI/CD and end-to-end tests |
| [qchain-network](https://github.com/teamqchain/qchain-network) | the Hyperledger Fabric network with ML-DSA-44 identities, and the chaincode |
| [qchain-docs](https://github.com/teamqchain/qchain-docs) | this repository |
| [qchain-kpi](https://github.com/teamqchain/qchain-kpi) | the post-quantum performance benchmarks |
| [teamqchain.github.io](https://github.com/teamqchain/teamqchain.github.io) | the project website |

Some of these repositories are private.

---

## Team

A University of Sharjah capstone project, supervised by **Dr. Manar Abu Talib** and **Dr. Sohail Abbas**.

| Member | Role |
|--------|------|
| Mohammed Bin Ali Maqqavi ([@M0hd-Maqqavi](https://github.com/M0hd-Maqqavi)) | Project Manager |
| Mohammed Abdul Haris ([@mohammed-ah-14](https://github.com/mohammed-ah-14)) | Technical Lead |
| Mohammed Nihal ([@nihvp](https://github.com/nihvp)) | Quality Assurance Lead |
| Mohammed Obeid ([@MohammedObeid88](https://github.com/MohammedObeid88)) | Blockchain Specialist |

---

## License

**Proprietary — all rights reserved.** © 2026 QChain. This content is proprietary and confidential; unauthorized
copying, modification, distribution, or use is prohibited without prior written permission. See [LICENSE](LICENSE).

---

## Disclaimer

This is an academic research project. The reports describe the system as it was when each was written; the code
repositories above describe it as it is now. The preliminary benchmarks are exploratory; for controlled measurements
see qchain-kpi.
