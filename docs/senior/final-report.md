# QChain — Final Report

**Project:** Post Quantum Cryptography in Blockchain Technology  
**Submitted:** 23rd November 2026  
**Team:** Team 5M's  
**Supervisors:** Dr. Manar Abu Talib (Head), Dr. Sohail Abbas (Co-Supervisor)  
**Committee:** Prof. Ahmed Mohamed Khedr (Chair), Dr. Mian Ahmad Jan (Examiner), Dr. Bouziane Brik

In collaboration with the Dubai Electronic Security Center (DESC).

---

## Team Members

| Name | Role | GitHub |
| --- | --- | --- |
| Mohammed Bin Ali Maqqavi | Project Lead | @M0hd-Maqqavi |
| Mohammed Abdul Haris | Frontend Lead | @mohammed-ah-14 |
| Muhammed Nihal Valiya Puthiyakath | Backend Lead | @nihvp |
| Mohammed Hisham Obeid | Blockchain Specialist | @MohammedObeid88 |

---

## Abstract

The emergence of quantum computing threatens the classical cryptography that secures blockchain systems and digital credentials: RSA and elliptic-curve signatures can be forged, and encrypted data stored today can be decrypted later. This project, QChain, designs, implements and evaluates a quantum-resistant credential system on a permissioned blockchain in which every trust decision rests on NIST-standardised post-quantum cryptography.

QChain runs on Hyperledger Fabric whose own identities, including peers, orderer, users and node TLS certificates, use ML-DSA-44 through modified Fabric, Fabric CA and cryptographic-provider code. Issuers sign a commitment to each credential with ML-DSA-44. The credential's contents are encrypted with ML-KEM-768 to a key held only on the holder's phone and stored on IPFS, while the ledger keeps the signed commitment and salted SHA3-256 hashes that enable selective disclosure.

The system serves issuers, holders and verifiers through the QPortal web portal and the QWallet mobile wallet. It covers the complete credential lifecycle: issuance, encrypted storage, selective-disclosure presentations signed by the holder, verification, suspension, restoration and revocation. It is deployed on the university's server with continuous deployment and automatic rollback.

The implementation passed all end-to-end, unit, smart-contract and tamper tests, and all 58 identity and node certificates of the network were confirmed to be ML-DSA-44. A benchmarking study of 30 algorithms on three machines found that the complete cryptographic lifecycle of a credential costs 0.69 to 1.32 ms, between 0.78 and 1.13 times its classical equivalent, while an ML-DSA-44 credential transaction is 2.8 times the size of an ECDSA one.

The results show that a credential system can be post-quantum at both the credential layer and the ledger's identity layer at a modest computational cost, with storage as the main price, offering a practical path for institutions moving to post-quantum cryptography.

---

## 1. Introduction

### 1.1 Overview

Digital credentials such as university degrees, transcripts and government certificates are issued, stored and verified electronically, and their trustworthiness rests on public-key cryptography. The algorithms that provide it today, RSA and elliptic-curve cryptography, can be broken by Shor's algorithm on a cryptographically relevant quantum computer (CRQC). A 2025 resource estimate shows that RSA-2048 could be factored with fewer than one million noisy qubits in under a week [1], and experts estimate a 28–49 % probability that such a machine exists within ten years [2].

**QChain** is a post-quantum digital credential system for academic and government credentials in the United Arab Emirates. Issuers sign every credential with ML-DSA-44 [3]; the credential's contents are encrypted with ML-KEM-768 [4] to a key that only the holder's phone possesses and are stored on IPFS; the ledger of a permissioned Hyperledger Fabric network holds a signed commitment with salted SHA3-256 hashes of the attributes, which enables selective disclosure; and holders sign every presentation with their own ML-DSA-44 key. Every identity and node TLS certificate of the Fabric network itself is ML-DSA-44. Issuers and verifiers use the QPortal web portal and holders the QWallet mobile wallet, and a benchmarking study measured 30 algorithms and the complete credential lifecycle on three machines.

### 1.2 Project Motivation

Credentials are long-lived: a degree stays valid for the lifetime of its holder, so a signature that can be forged in the next decade undermines every credential issued today. Ledgers add a second exposure, because data written to them is replicated and cannot be erased, so ciphertext stored there can be harvested now and decrypted later; in a permissioned blockchain the identities that endorse and order transactions are a third, since Hyperledger Fabric signs them with ECDSA by default. The transition is also a regulatory obligation: NIST will disallow quantum-vulnerable algorithms from 2035 [5], and the UAE's National Encryption Policy of November 2025 requires government entities to plan their move to post-quantum cryptography [6].

### 1.3 Problem Statement

Existing blockchain credential systems rely on classical signatures at two layers. At the credential layer, an issuer's ECDSA or RSA signature can be forged once its private key is derived from the public key, so counterfeit credentials would verify correctly. At the ledger layer, the identities of the peers, orderers and officials that write to the ledger are classical, so a quantum adversary could impersonate them and the integrity of the ledger itself would be lost.

Confidentiality and privacy are a second problem. Credential data in plaintext on a replicated, immutable ledger exposes more than a verifier needs and conflicts with data-protection principles [7]; data on IPFS is public unless it is encrypted [8]; and data encrypted with classical public-key schemes remains exposed to harvest-now-decrypt-later attacks. A holder must also be able to reveal only some attributes of a credential.

The third problem is cost. Post-quantum signatures and keys are one to two orders of magnitude larger than elliptic-curve ones, and their speed differs widely between algorithms and processors, yet published measurements time the primitives in isolation, mostly on one machine. The project therefore had to build a system that is post-quantum at both layers and measure what this costs on real hardware.

### 1.4 Project Aim and Objectives

The aim of the project is to design, implement, deploy and evaluate a credential system in which every trust decision, at the credential layer and at the ledger's identity layer, rests on NIST-standardised post-quantum cryptography. The objectives are:

1. **Post-quantum authenticity:** every credential is signed by its issuer with ML-DSA-44 over a commitment to all of its data.
2. **Post-quantum confidentiality:** credential contents are encrypted with ML-KEM-768 so that only the holder can read them.
3. **Selective disclosure and holder binding:** a holder reveals chosen attributes in a presentation signed with the holder's own ML-DSA-44 key and bound to one credential.
4. **No plaintext credential data on the ledger:** the ledger stores commitments, salted hashes and signatures; the ledger and IPFS are the sources of truth.
5. **Post-quantum ledger identities:** every Fabric identity and node TLS certificate uses ML-DSA-44, without a classical fallback.
6. **Performance evaluation:** the cost of the post-quantum primitives and of the full credential lifecycle is measured on three machines.
7. **Deployment and automated delivery:** the system runs on the university's server, recovers after restarts and is deployed through a tested pipeline with automatic rollback.

### 1.5 Project Scope

The project covers the following work:

- requirements, architecture review and threat analysis; the post-quantum Hyperledger Fabric network with modified Fabric, Fabric CA and cryptographic-provider code, and the smart contract (chaincode);
- the Go backend with 39 REST endpoints, IPFS storage and the operational database; the QPortal web portal and the QWallet mobile wallet;
- deployment on the university's virtual machine with continuous deployment; unit, smart-contract and end-to-end testing; and a benchmarking study of 30 algorithms and the credential lifecycle on three machines.

### 1.6 Project Software and Hardware Requirements

- **Software:** Hyperledger Fabric and Fabric CA built with ML-DSA-44 support (Fabric 3.1.5 and Fabric CA 1.5.22 lines) with Go 1.27 and Docker Compose; Go and liboqs 0.16.0 for the backend; Flutter with liboqs and PointyCastle for QPortal and QWallet; IPFS (Kubo), MySQL 8, Nginx, Tailscale Funnel, GitHub Actions and Vercel.
- **Hardware:** the university's virtual machine (2 virtual CPUs, 30 GiB of memory, 246 GB of disk) for the deployment; an Apple M1 MacBook Air and an AMD Ryzen 5 7520U laptop, which with the virtual machine are the three measurement machines; an Android or iOS phone for QWallet.

### 1.7 Project Limitations

- All organisations of the Fabric network run on one machine, which generates their keys; a production consortium would place each organisation on its own servers.
- The two Fabric CA servers present ECDSA P-256 TLS certificates; every other identity and TLS certificate in the network is ML-DSA-44.
- ML-DSA-44 is the only signature level used at the identity layer.
- The benchmark results describe three specific machines, one of each kind; the ledger experiments describe the network configuration that was in use while they ran (Section 9.6).
- The deployment holds demonstration data and is reached through an open demonstration portal.

### 1.8 Project Expected Output

- a working post-quantum credential system, with the QPortal web portal and the QWallet mobile wallet, covering issuance, encryption, selective disclosure, presentation, verification, suspension, restoration and revocation;
- a Hyperledger Fabric network whose identities and node TLS certificates are ML-DSA-44;
- a benchmarking dataset and its analysis for 30 algorithms and the credential lifecycle on three machines;
- ten public, documented source-code repositories (listed in the Appendix) and this report.

### 1.9 Project Schedule

The project ran from 24 August to 27 November 2026 in five task groups: requirements, design review and security analysis (24 August to 10 September); the post-quantum security mechanisms (11 to 24 September); deployment hardening, testing and the first progress report (14 to 25 September); the post-quantum ledger identities and the benchmarking study (14 September to 12 November); and evaluation, reporting and demonstration (24 October to 27 November). The team met weekly with the supervisors and tracked progress in the Gantt chart shown in the Appendix (Figure 13-1).

### 1.10 Project, Product and Schedule Risks

- **Integration of ML-DSA into Fabric's identity layer.** Fabric's provider, MSP and CA had to change together; the work took longer than first planned, without moving the final deliverables.
- **Ledger consistency under load.** A shared state database caused read conflicts in burst tests (Section 10.11); each peer now has its own goleveldb database.
- **Measurement noise.** Disturbed measurements were detected and repeated (Section 9.5).
- **Larger artefacts and a single host.** Credentials stay off the ledger behind signed commitments; containers restart automatically and failed deployments roll back.

### 1.11 Success Metrics and Acceptance Criteria

#### 1.11.1 Success Metrics

Table 1-I lists the success metrics, how each was measured and the result achieved.

**Table 1-I. SUCCESS METRICS, MEASUREMENT AND RESULTS**

| Metric | Target | How measured | Result |
| --- | --- | --- | --- |
| Functional correctness | Every endpoint and lifecycle path works | End-to-end suite, Go unit tests, chaincode tests | 123 / 123 assertions; all unit and chaincode tests pass |
| Post-quantum coverage | Signatures, key exchange and ledger identities post-quantum | Certificate inspection on the live network; signature and envelope sizes | 58 / 58 certificates ML-DSA-44; 2,420-byte signatures; ML-KEM-768 envelopes |
| Confidentiality and privacy | No plaintext credential data on the ledger | Inspection of ledger records; selective-disclosure tests | Met |
| Tamper resistance | Every manipulation detected | Tamper and negative tests | All rejected |
| Cryptographic cost | Under 2 ms per credential on every machine | Lifecycle benchmark (five stages) | 0.69 / 0.96 / 1.32 ms (Mac / laptop / VM) |
| Workflow completeness | Issue, present, verify and revoke for all roles | End-to-end groups 6–15 | Complete |
| Reliability | Data survives restarts; failed deployments roll back | Docker restart test; pipeline health check | Met |

#### 1.11.2 Acceptance Criteria

- all lifecycle functions work end to end on the deployed system, and all automated tests pass;
- every credential signature, presentation signature and Fabric identity uses ML-DSA-44, credential contents use ML-KEM-768, and no attribute value is stored on the ledger;
- the benchmark covers every planned experiment on the planned machines, and the documentation and source code are complete.

### 1.12 Project Governance

#### 1.12.1 Team Structure

The team consists of four members supervised by Dr. Manar Abu Talib (head supervisor) and Dr. Sohail Abbas (co-supervisor):

- **Mohammed Bin Ali Maqqavi, Project Lead:** project management, security design and review, continuous deployment, benchmark design and harness;
- **Mohammed Abdul Haris, Frontend Lead:** QWallet and QPortal, on-device keys, interface refinement, benchmark measurements;
- **Muhammed Nihal Valiya Puthiyakath, Backend Lead:** envelope encryption, holder presentations, security testing, benchmark analysis;
- **Mohammed Hisham Obeid, Blockchain Specialist:** the post-quantum Fabric network and its ML-DSA-44 identities, ledger recovery, end-to-end tests.

#### 1.12.2 Document and Version Control

All code and documentation are kept in the team's GitHub organisation, in ten public repositories listed in Table 13-I. Changes reach the main branch through pull requests, the backend is tested and deployed automatically on every push, and each repository documents its own setup. Reports are written in Microsoft Word with automatic captions, cross-references and citations, and the schedule is maintained in the team's Gantt chart.

## 2. Literature Review

The review examines how soon quantum computers threaten today's cryptography and which standards and deadlines apply; how quantum attacks affect blockchains; which post-quantum algorithm families suit a credential system; how credential data can be kept confidential and private; how far post-quantum cryptography has reached Hyperledger Fabric's identity layer; and how post-quantum performance has been measured. It ends with the research gap and a summary of the key works.

### 2.1 The Quantum Threat and Migration Timelines

Figure 2-1 places the main quantum-computing milestones beside the standards and migration deadlines that followed them. Resource estimates for breaking RSA-2048 fell by more than an order of magnitude between 2019 and 2025, to fewer than one million noisy qubits running for under a week [1]; hardware crossed the error-correction threshold that fault tolerance requires, with logical error rates that fall as the code grows [9]; and vendor roadmaps target fault-tolerant machines with hundreds of logical qubits by 2029 [10]. The latest expert survey rates a cryptographically relevant quantum computer within ten years as quite possible and within fifteen years as likely [2]. For data that must stay confidential for decades, the threat is already present: ciphertext recorded today can be decrypted once such a machine exists.

*Figure 2-1. Quantum capability milestones and post-quantum standards and migration deadlines, 2024–2035*

In August 2024 NIST published ML-KEM (FIPS 203), ML-DSA (FIPS 204) and SLH-DSA (FIPS 205) [4, 3, 11]; FN-DSA (FIPS 206), based on Falcon, is in preparation [12], and HQC was selected as an additional key-encapsulation mechanism [13]. Migration deadlines followed quickly, as Table 2-I shows. NIST recommends crypto-agility, the ability to replace algorithms without redesigning systems, as a design principle for the transition [14], and the UAE's National Encryption Policy makes the transition binding for government entities [6].

**Table 2-I. POST-QUANTUM STANDARDS AND MIGRATION MANDATES**

| Document | Issuer and date | Key requirement | Implication for QChain |
| --- | --- | --- | --- |
| FIPS 203 / 204 / 205 [4, 3, 11] | NIST, Aug 2024 | ML-KEM, ML-DSA and SLH-DSA standardised | ML-KEM-768 and ML-DSA-44 adopted |
| NIST IR 8547 [5] | NIST, Nov 2024 | Deprecate quantum-vulnerable algorithms 2030, disallow 2035 | ECDSA identities must be replaced |
| NCSC timeline [15] | UK NCSC, Mar 2025 | Discovery 2028, priority migration 2031, completion 2035 | Confirms the 2035 horizon |
| EU roadmap [16] | NIS Cooperation Group, Jun 2025 | National plans 2026, high-risk systems 2030 | Credential infrastructure is high-risk |
| UAE National Encryption Policy [6] | UAE Government, Nov 2025 | Post-quantum transition plans for government entities | Directly applicable to UAE issuers |
| CSC–ATRC programme [17] | UAE CSC and ATRC, May 2026 | National post-quantum implementation programme | National migration support |

### 2.2 Blockchain Security in the Quantum Era

A recent systematisation of knowledge analyses how post-quantum attackers reshape blockchain security and performance [18]. Shor's algorithm breaks the signatures that authorise transactions and identify participants, whereas Grover's algorithm only halves the effective security of hash functions, so SHA3-256 with a 256-bit output remains adequate. Replacing the signatures has a system-wide cost: post-quantum signatures and keys are one to two orders of magnitude larger than elliptic-curve ones, which enlarges blocks and storage and lengthens verification.

Systematic reviews classify the proposed post-quantum consensus and blockchain designs [19] and survey post-quantum cryptography across blockchain and IoT systems [20], and a further review discusses the open challenges of post-quantum blockchains [21]. Protocol proposals for public blockchains favour hybrid signatures that combine a classical and a post-quantum scheme during the transition [22]. These studies concentrate on public, permissionless chains and on transaction signatures. In a permissioned network such as Hyperledger Fabric the ordering protocol (Raft) uses no proof-of-work and needs no quantum-sensitive cryptography of its own; the quantum exposure lies in the X.509 identities that endorse and order transactions, and in the application data written to the ledger.

### 2.3 Post-Quantum Cryptographic Families

Five families of post-quantum schemes have been studied; three of them reached standardisation.

#### 2.3.1 Lattice-Based Cryptography

Lattice schemes base their security on the hardness of problems such as Module Learning With Errors. ML-KEM and ML-DSA work over module lattices and offer the best overall balance of key size, signature or ciphertext size and speed, which is why they dominate the NIST selections [4, 3]. Their arithmetic is regular and maps well onto vector instructions. FN-DSA (Falcon) works over NTRU lattices and produces the smallest signatures of the family, but its signing relies on floating-point Gaussian sampling that is difficult to implement in constant time [12]. Benchmark studies rank the lattice signatures far ahead of the other families in signing speed, with ML-DSA verification among the fastest of all signature schemes [23, 24].

#### 2.3.2 Hash-Based Cryptography

SLH-DSA (SPHINCS+) relies only on the security of hash functions, the most conservative assumption available, and has public keys of 32 to 64 bytes [11]. Its price is size and speed: signatures range from 7.9 to 49.9 KB, and signing takes tens of milliseconds in the fast variants and hundreds in the small ones. It is therefore suited to rare, long-lived signatures, such as firmware or root certificates, rather than to frequent credential operations.

#### 2.3.3 Code-Based and Multivariate Cryptography

Code-based HQC was selected as a second key-encapsulation mechanism to provide algorithmic diversity should a weakness in lattices appear, at the cost of ciphertexts of 4.4 to 14.4 KB [13]. Multivariate schemes suffered a practical break when Rainbow was shown to be recoverable in a weekend on a laptop [25], and isogeny-based key exchange was broken in the same year; neither is suitable for a long-lived credential system.

Figure 2-2 compares the three standardised signature families, and Table 2-II lists their sizes. Comparative analyses of quantum-resistant blockchains reach the same conclusion for ledger workloads, where verification is performed far more often than signing [26].

*Figure 2-2. Qualitative comparison of the three NIST post-quantum signature families*

**Table 2-II. KEY AND SIGNATURE OR CIPHERTEXT SIZES OF STANDARDISED SCHEMES**

| Scheme | Standard | Family | NIST category | Public key (bytes) | Signature or ciphertext (bytes) |
| --- | --- | --- | --- | --- | --- |
| ML-DSA-44 | FIPS 204 | Lattice | 2 | 1,312 | 2,420 |
| ML-DSA-65 | FIPS 204 | Lattice | 3 | 1,952 | 3,309 |
| ML-DSA-87 | FIPS 204 | Lattice | 5 | 2,592 | 4,627 |
| FN-DSA-512 | FIPS 206 (draft) | Lattice (NTRU) | 1 | 897 | ≈ 666 |
| SLH-DSA-SHA2-128s | FIPS 205 | Hash | 1 | 32 | 7,856 |
| ML-KEM-768 | FIPS 203 | Lattice | 3 | 1,184 | 1,088 |
| HQC-1 | Selected | Code | 1 | 2,241 | 4,433 |
| ECDSA P-256 (classical) | FIPS 186-5 | Elliptic curve | Broken by Shor | 64 | ≈ 64–72 |

On this evidence QChain uses **ML-DSA-44** for signatures and **ML-KEM-768** for key establishment. ML-DSA-44 is final, fast to sign and verify, and its 2.4 KB signature is acceptable for a ledger that stores one signature per credential; ML-KEM-768 provides NIST category 3 security for data that must remain confidential for decades. Both are available in the Open Quantum Safe library, maintained under the Linux Foundation's Post-Quantum Cryptography Alliance [27], which serves the backend and the wallet alike.

### 2.4 Confidentiality and Selective Disclosure

#### 2.4.1 Off-Chain Storage and Encryption

Blockchain credential systems commonly place documents on the InterPlanetary File System (IPFS) and record only their content identifiers (CIDs) on the ledger. IPFS provides no confidentiality: any node that knows or discovers a CID can retrieve the content, so data must be encrypted before it is added [8], and content persists only while at least one node pins it [28]. Academic-record systems built on Fabric and IPFS store certificate files and verification data with classical cryptography and without encryption to the holder [29, 30]. The standard construction for encrypting to a recipient's public key is the KEM-DEM paradigm of Hybrid Public Key Encryption: a key-encapsulation mechanism establishes a shared secret, a key-derivation function expands it and an authenticated cipher encrypts the data [31]; hybrid KEMs such as X-Wing combine ML-KEM-768 with X25519 to hedge against a weakness in either [32]. The European Data Protection Board advises that personal data should not, as a rule, be stored on a blockchain, and discusses off-chain storage with on-chain commitments as a way to reduce it [7].

#### 2.4.2 Verifiable Credentials and Selective Disclosure

The W3C Verifiable Credentials Data Model 2.0 defines the issuer–holder–verifier model that QChain follows [33]. Selective disclosure is achieved in practice with salted digests: SD-JWT replaces each disclosable claim with the hash of a random salt of at least 128 bits, the claim name and its value, and binds presentations to the holder with a key-binding signature [34]; the ISO/IEC 18013-5 mobile driving licence uses the same structure [35]. Both rely on classical signatures. Post-quantum verifiable-credential work has analysed the cryptography of the main credential schemes and evaluated NIST signatures in existing formats [36]; lattice-based anonymous credentials offer stronger privacy through zero-knowledge proofs, at the cost of large proofs and complex implementation [37]. Implementations of post-quantum anonymous credentials [38], hybrid credentials for IoT systems [39] and post-quantum self-sovereign identity [40] concentrate on functionality rather than cost.

### 2.5 Post-Quantum Cryptography in Hyperledger Fabric

Fabric identifies every peer, orderer, client and administrator with an X.509 certificate issued by a Fabric certificate authority and validated by its Membership Service Provider (MSP); signatures are produced by its cryptographic service provider (BCCSP), which supports ECDSA. PQFabric introduced post-quantum signatures into Fabric with a **hybrid** design in which classical and post-quantum signatures coexist during the transition [41], and Abbas et al. modified Fabric's certificate generation to issue hybrid X.509 certificates, measured with Hyperledger Caliper [42]. Kasula et al. integrated **lightweight** post-quantum schemes together with national algorithms [43], and Grebnev et al. identified signature size, performance and certificate formats as the main obstacles to post-quantum migration in Ethereum and Fabric [44].

At the application layer, Ouadrassi et al. signed documents with ML-DSA-65 alongside ECDSA in an external service and stored the signatures on a Fabric 2.5 ledger; they found blocks about 1.43 times larger and no significant slowdown, and listed off-chain storage with on-chain commitments as an open optimisation [45]. Earlier work measured Fabric's throughput and latency with classical cryptography [46] or simulated a post-quantum healthcare application [47]. The building blocks of a purely post-quantum identity layer have since appeared: Fabric v3.0 accepts a new signature algorithm, Ed25519, in its MSP, which shows that the identity layer can be extended [48]; RFC 9881 defines X.509 algorithm identifiers for ML-DSA [49]; and Go 1.27, the language Fabric is written in, adds ML-DSA to its standard library and to its X.509 and TLS packages [50].

### 2.6 Measuring Post-Quantum Performance

#### 2.6.1 Benchmarks of the Primitives

Most performance studies time the primitives in isolation. Commey et al. benchmarked KEMs and signatures on a desktop, a laptop and a Raspberry Pi with liboqs 0.8.0 through Python [51]; Algar-Fernandez et al. compared five signature schemes and three KEMs on two laptops and showed that the integration route changes the measured times [52]; Lucaciu and Popescu compared C, Go and Python with a different library per language [53]; and Oleksiichuk compared the standards across Java versions and libraries [54]. Chhetri et al. measured ML-KEM and ML-DSA on an ARM Cortex-M0+ and reported coefficients of variation of 66–74 % for ML-DSA signing, caused by its rejection sampling [55]. Puch et al. compared native, container and unikernel execution and found container overhead negligible for the primitives [56]; Gillot et al. measured energy [57]; Lopez et al. evaluated KEMs on Raspberry Pi devices [58]; and post-quantum TLS has been evaluated in 5G user equipment [59, 60], for 6G networks [61] and through an evaluation framework built on OpenSSL [62]. In liboqs 0.16.0, the release used by QChain, ML-KEM and ML-DSA come from the formally verified mlkem-native and mldsa-native code [63, 64], so older measurements describe different code.

#### 2.6.2 Blockchains, Credentials and Integration Cost

For blockchains, Schemitt et al. benchmarked 62 signature implementations on x64 and ARM and simulated their effect on a chain [65]; Juaristi et al. estimated the cost of the NIST signatures in Ethereum-based blockchains [66]; Holmes examined post-quantum block sizes and intervals [67]; and Saviozzi et al. integrated ML-DSA into a Substrate-based chain [68]. The closest credential study, by Arakawa, evaluated 15 signature schemes in SD-JWT credentials on one cloud server, without key encapsulation, memory, tail latency, concurrency or a ledger [69]. PQ-FundusChain combines ML-KEM, AES-GCM and ML-DSA with off-chain storage in a medical application [70], similar combinations appear in a session protocol [71] and in multi-file cloud storage [72], and Fabrizio et al. quantified the post-quantum transition of an academic identity federation [73].

How a library is called can matter as much as the algorithm. Kotukh et al. measured a foreign-function overhead below 10 ns per call for OCaml bindings of liboqs and stated, without measuring it, that Go's boundary to C costs about 50 ns [74]. Measurement practice also varies: most studies report means or medians of 10 to 10,000 repetitions on one machine, Shaw pinned the processor and reported bootstrapped confidence intervals [75], and Frost proposed a reproducibility protocol for cross-implementation tests [76]. What a deployed application adds on top of the library, and what a complete credential lifecycle costs on several kinds of machine, had not been measured.

### 2.7 Conclusion and Summary of Reviewed Literature

The review shows that the quantum threat falls within the validity period of today's credentials, that binding migration mandates apply in the UAE, and that the necessary primitives are standardised. It also reveals a gap. Fabric and IPFS credential systems use classical cryptography and unencrypted data; post-quantum credential work does not anchor credentials on a permissioned ledger; post-quantum Fabric designs are hybrid or lightweight and address neither confidentiality nor selective disclosure; and performance studies measure primitives rather than the lifecycle of a deployed system on several machines. No published system combines post-quantum authenticity, holder-held post-quantum confidentiality, salted selective disclosure and holder-signed presentations on a permissioned ledger whose own identities are purely post-quantum, together with a multi-machine measurement of what this costs. Table 2-III summarises the key works.

**Table 2-III. SUMMARY OF THE KEY REVIEWED LITERATURE**

| Ref. No. | Paper Title | Year of Publication | Publisher | Key Insights | Limitations |
| --- | --- | --- | --- | --- | --- |
| [1] | How to factor 2048 bit RSA integers with less than a million noisy qubits | 2025 | arXiv | RSA-2048 breakable with < 1 M noisy qubits in < 1 week | Estimate; hardware not yet built |
| [2] | Quantum Threat Timeline Report 2025 | 2026 | Global Risk Institute | CRQC 28–49 % likely within 10 years | Expert survey |
| [18] | Quantum Disruption: An SOK of How Post-Quantum Attackers Reshape Blockchain Security and Performance | 2025 | arXiv | Quantum attacks on blockchains and post-quantum costs | Survey; no credential system |
| [23] | Performance and Applicability of Post-Quantum Digital Signature Algorithms in Resource-Constrained Environments | 2023 | MDPI Algorithms | Lattice signatures outperform SPHINCS+ | Pre-standard parameter sets |
| [41] | PQFabric: A Permissioned Blockchain Secure from Both Classical and Quantum Attacks | 2021 | IEEE ICBC | First post-quantum signatures in Fabric | Hybrid; pre-standard algorithms |
| [42] | Quantum-Safe Blockchain in Hyperledger Fabric | 2025 | IEEE | Hybrid X.509 certificates for Fabric | Hybrid; identity layer only |
| [43] | Enhancing Hyperledger Fabric Security with Lightweight PQC and National Cryptographic Algorithms | 2025 | IEEE FRUCT | Lightweight post-quantum schemes in Fabric | Non-standard schemes; no data privacy |
| [45] | Toward Quantum-Safe Permissioned Blockchains | 2026 | IEEE | ML-DSA-65 with ECDSA on Fabric 2.5; blocks 1.43× larger | Hybrid; signing outside Fabric |
| [44] | Analysis and challenges of implementing PQC algorithms in Ethereum and Hyperledger Fabric networks | 2025 | Springer JCVHT | Size, performance and certificate obstacles | Analysis only |
| [29] | Verification of Academic Records Using Hyperledger Fabric and IPFS | 2023 | IEEE ICPCSN | Fabric and IPFS academic records | Classical; unencrypted files |
| [36] | Towards Post-Quantum Verifiable Credentials | 2024 | ACM ARES | NIST signatures in credential formats | No ledger or encryption |
| [34] | Selective Disclosure for JSON Web Tokens | 2025 | IETF | Salted digests (≥ 128-bit salts) and key binding | Classical signatures |
| [69] | Post-Quantum Cryptography for Verifiable Credentials | 2025 | TechRxiv | 15 signature schemes in SD-JWT credentials | No KEM, memory, tails or ledger; one server |
| [65] | Assessing the Impact of Post-Quantum Digital Signature Algorithms on Blockchains | 2025 | arXiv | 62 signature implementations on x64 and ARM | Signatures only; no memory or tails |
| [66] | Benchmarking Post-quantum Cryptography in Ethereum-Based Blockchains | 2025 | Springer | Cost of NIST signatures in Ethereum-based chains | Public chains; signatures only |
| [51] | Performance Analysis and Deployment Considerations of PQC for Consumer Electronics | 2025 | arXiv | KEMs and signatures on three platforms | liboqs 0.8.0 through Python |
| [55] | Benchmarking NIST-Standardised ML-KEM and ML-DSA on ARM Cortex-M0+ | 2026 | arXiv | Variance and tail latency of ML-DSA signing | Microcontroller only |
| [56] | Benchmarking Post-Quantum Cryptography in Lightweight Virtualization Environments | 2026 | arXiv | Container overhead negligible for primitives | Embedded ARM; primitives and TLS only |
| [70] | PQ-FundusChain | 2026 | Frontiers | ML-KEM, AES-GCM and ML-DSA with off-chain data | One application; no algorithm comparison |
| [73] | Measuring the Impact of Post-Quantum Cryptography on Complex Applications | 2025 | IEEE | Post-quantum transition of an academic identity federation | Not credential-centric; no lifecycle |

## 3. System Architecture

### 3.1 Architecture Overview

QChain follows the layered architecture shown in Figure 3-1. Issuers, verifiers and IT administrators use QPortal, a web portal served from the Vercel content-delivery network; holders use QWallet on their phones. Both applications reach the backend through one public HTTPS endpoint on the university's virtual machine. The QChain API, a Go service, implements the post-quantum protocols and stores data in three places with separate responsibilities: the Fabric ledger is authoritative for signed credential commitments, status and holder public keys; IPFS holds the only copy of each credential's contents, encrypted to the holder; and MySQL stores operational records that exist nowhere else, such as presentation sessions, verification logs and subscriptions.

```text
  Issuer · Verifier · IT admin                               Holder
              │                                                  │
  QPortal  (Flutter web, Vercel CDN)               QWallet  (Flutter, Android / iOS)
  issue · revoke · verify · staff · audit          ML-KEM-768 + ML-DSA-44 keys on the phone
              │  JSON over HTTPS                                 │  JSON over HTTPS
              └─────────────────────────┬────────────────────────┘
                                        ▼
            HTTPS · Tailscale Funnel ──▶ Nginx gateway (only /api) · university VM
                                        ▼
  QChain API (Go)
  ├── api     lifecycle: issue, revoke, suspend, present, verify (five checks), staff, subscriptions
  ├── pqc     ML-DSA-44 sign / verify · ML-KEM-768 + AES-256-GCM envelopes · salted SHA3-256 hashes
  ├── ledger  Fabric Gateway client with an ML-DSA-44 identity (issuer1)
  └── db      MySQL access (operational data)
        │                               │                               │
        ▼                               ▼                               ▼
  Hyperledger Fabric               IPFS (Kubo)                     MySQL 8
  2 peers (goleveldb) · orderer    encrypted credential            IDs, staff, sessions,
  2 PQC CAs · qchaincode 2.0       envelopes, by CID               logs, subscriptions
  every identity and node TLS                                      (no credential body)
  certificate: ML-DSA-44
```
**Figure 3-1.** QChain system architecture

### 3.2 User Applications

Table 3-I summarises the two applications. The wallet is a native mobile application by design: each holder has exactly one ML-KEM-768 and one ML-DSA-44 key pair, generated and kept in the phone's secure storage, which a browser could not guarantee.

**Table 3-I. CLIENT APPLICATIONS**

| Application | Platform | Users | Main functions | Security role |
| --- | --- | --- | --- | --- |
| QPortal | Flutter web build on the Vercel CDN | Issuer, verifier and IT-administrator staff | Issue, re-date, suspend, restore and revoke credentials; verify by QR code or OTP; dashboards, history, staff, subscriptions, audit | Shows verification results; holds no keys |
| QWallet | Flutter, native Android and iOS | Credential holders | Key generation, retrieval and decryption, selective-disclosure presentations, subscription approvals | Generates and keeps holder keys; decrypts; signs presentations |

### 3.3 Backend Service Modules

The backend is a single Go service organised into the packages listed in Table 3-II. All cryptographic operations sit in one package, so that every protocol has one reference implementation that the unit tests exercise directly.

**Table 3-II. BACKEND PACKAGES**

| Package | Responsibility |
| --- | --- |
| `api` | The 39 REST endpoints and the health check: credential lifecycle, verification, holders, subscriptions, staff, dashboard and the wallet endpoints |
| `pqc` | Canonical commitment, ML-DSA-44 signing and verification, salted SHA3-256 field hashes, ML-KEM-768 envelope sealing, IPFS access |
| `ledger` | Fabric Gateway client with an ML-DSA-44 identity; ledger reads that treat the chain as authoritative |
| `db` | Parameterised MySQL access for the operational tables |
| `config` | Environment configuration, trusted issuer key, time zone |
| `gateway` | Nginx gateway image and the Tailscale Funnel set-up |
| `cmd/mkwallet`, `cmd/keygen` | Writing ML-DSA-44 Fabric identities into the wallet; key generation tools |

### 3.4 Cryptographic Library Stack

Table 3-III lists the primitives and the libraries that implement them. The backend and the wallet use the same Open Quantum Safe implementation of ML-DSA-44 and ML-KEM-768, so signatures and envelopes produced on one side are accepted on the other. Fabric's own identities use the ML-DSA implementation of Go's standard library inside the modified cryptographic provider.

**Table 3-III. CRYPTOGRAPHIC PRIMITIVES AND LIBRARIES**

| Primitive | Standard | Backend (Go) | Wallet (Dart) | Used for |
| --- | --- | --- | --- | --- |
| ML-DSA-44 | FIPS 204 | liboqs-go | liboqs | Issuer commitment signatures; holder presentation signatures |
| ML-DSA-44 (ledger identities) | FIPS 204 | Go `crypto/mldsa` | — | Fabric identities, endorsements, blocks, node TLS; Gateway transactions |
| ML-KEM-768 | FIPS 203 | liboqs-go | liboqs | Envelope key encapsulation to the holder |
| SHA3-256 | FIPS 202 | x/crypto/sha3 | PointyCastle | Commitment, salted field hashes, presentation hash |
| HKDF-SHA3-256 | RFC 5869 | x/crypto/hkdf | PointyCastle | Single-use per-field key-wrapping keys |
| AES-256-GCM | SP 800-38D | crypto/aes, crypto/cipher | PointyCastle | Field encryption and data-key wrapping |

### 3.5 Post-Quantum Hyperledger Fabric Network

The ledger runs on a Hyperledger Fabric network in which every identity is post-quantum. Building it required changes to Fabric itself, to Fabric CA and to Fabric's cryptographic library, and it is one of the project's main contributions.

#### 3.5.1 Network Topology

Figure 3-2 shows the network. Two member organisations each run one peer and one certificate authority: GeneralMSP, the university, whose application identities issuer1 and verifier1 sign the transactions, and GovernmentMSP, the second endorsing organisation. An ordering organisation runs one etcdraft orderer that is managed through the channel participation API. The channel mychannel uses capability V3_0, which ML-DSA identities require, and the default MAJORITY endorsement policy, so with two organisations every write needs both peers. Each peer keeps its own copy of the ledger and its own goleveldb state database; the chaincode qchaincode 2.0 runs as a service that both peers call. All nodes run as containers of one Docker Compose project with a restart policy, listening on the loopback interface only.

```text
                  QChain API · Fabric Gateway client (issuer1, ML-DSA-44)
            ① proposal / ② endorsement │ ③ signed transaction │ ① proposal / ② endorsement
          ┌────────────────────────────┘          │           └────────────────────────────┐
          ▼                                       │                                        ▼
  GeneralMSP · University (issuer)                │               GovernmentMSP · second endorser
  ┌──────────────────────────────────┐            │              ┌──────────────────────────────────┐
  │ peer0.general.uae.com :29051     │            │              │ peer0.government.uae.com :27051  │
  │ ledger copy · goleveldb          │            │              │ ledger copy · goleveldb          │
  │ qchaincode as a service · PQC    │            │              │ qchaincode as a service · PQC    │
  │ ca.general :28054 (PQC Fabric CA)│            │              │ ca.government :27054             │
  │ issuer1, verifier1 (role attr.)  │            │              │ (own TLS cert: ECDSA P-256)      │
  └──────────────────────────────────┘            ▼              └──────────────────────────────────┘
                 ▲      ④ blocks     orderer0.orderer.example.com :27050     ④ blocks      ▲
                 └──────────────────  OrdererMSP · etcdraft · signs blocks  ───────────────┘
                                      (10 tx or 2 s) · channel participation API
  Channel mychannel · capability V3_0 · MAJORITY endorsement (both organisations)
  Identities: cryptogen (ML-DSA-44) for peers, orderer, admins and their TLS; Fabric CA per organisation
  for application users; only the two CA servers' own TLS certificates are ECDSA P-256.
  Built from qchain-fabric (e472b4ea), qchain-fabric-ca (e82caf43), qchain-fabric-lib-go (bd30150), tag pqc.
```
**Figure 3-2.** The post-quantum Hyperledger Fabric network: organisations, nodes, identities and the software built by the team

#### 3.5.2 The ML-DSA-44 Membership Service Provider

Every node and user identity in the network is an X.509 certificate with an ML-DSA-44 key (object identifier 2.16.840.1.101.3.4.3.17). Peers, the orderer and the administrators receive their keys and certificates, including their TLS certificates, from a modified `cryptogen`. Application users receive theirs from a modified Fabric CA in each organisation, which signs with the organisation's root key so that its users chain to the channel's MSP root; issuer1 and verifier1 carry a role attribute that the chaincode checks. Every node's cryptographic provider is set to `PQC`, a new provider whose ML-DSA operations use Go's standard library. Without the build tag `pqc`, a version-3 MSP refuses ML-DSA identities, so the same source code still builds a classical Fabric. Table 3-IV summarises the three modified code bases.

**Table 3-IV. THE MODIFIED FABRIC CODE BASES**

| Repository | Base version | Main changes | Result |
| --- | --- | --- | --- |
| qchain-fabric | Fabric 3.1.5 line | ML-DSA-44 keys in cryptogen; MSP accepts ML-DSA identities (tag pqc); peer and orderer pick up the PQC provider | Peer, orderer and tools images |
| qchain-fabric-ca | Fabric CA 1.5.22 | CA server and client issue and enrol ML-DSA-44 certificates | CA server image |
| qchain-fabric-lib-go | fabric-lib-go | The PQC cryptographic provider: ML-DSA-44/65/87 (Go crypto/mldsa), key store, expanded-key cache | Linked into all of the above |

The backend connects to the network through the Fabric Gateway client. Its signing helper supports only ECDSA and Ed25519 keys, so the backend supplies its own signer for ML-DSA-44 keys: the Gateway hashes each message with SHA-256, and the MSP verifies the ML-DSA signature over that digest, so the signer signs the digest as given. Figure 3-3 shows this signer: an ML-DSA-44 key signs the digest directly, and any other key falls back to the Gateway's own signer. The `mkwallet` tool writes each identity's certificate and ML-DSA-44 private key into the backend's wallet.

```go
// newSign returns the Gateway signing function for privateKey. fabric-gateway's
// identity.NewPrivateKeySign only supports ECDSA and Ed25519, so ML-DSA keys get
// their own signer. The Gateway hashes each message with its default SHA-256
// before calling sign, and the Fabric MSP verifies ML-DSA signatures over that
// SHA-256 digest (msp/identities.go), so the signer must sign the digest as
// given: pure ML-DSA, empty context.
func newSign(privateKey any) (identity.Sign, error) {
	if key, ok := privateKey.(*mldsa.PrivateKey); ok {
		return func(digest []byte) ([]byte, error) {
			return key.Sign(nil, digest, nil)
		}, nil
	}
	return identity.NewPrivateKeySign(privateKey)
}
```
**Figure 3-3.** The backend's ML-DSA-44 signer for Fabric Gateway transactions

#### 3.5.3 Transaction Flow

Figure 3-4 follows one issuance through the network. The client's proposal, each peer's endorsement, the client's transaction and the orderer's block are all signed with ML-DSA-44, and every peer checks every signature against the ML-DSA-44 certificate chain of its MSP before it commits the block. A block closes after ten transactions or after two seconds, whichever comes first.

```text
  QChain API        peer0.general     peer0.government    qchaincode        orderer0
      │ ① proposal (signed by issuer1, ML-DSA-44)
      ├───────────────▶│                    │
      ├────────────────────────────────────▶│
      │       each peer checks the client's ML-DSA-44 certificate chain and signature
      │                ├── ② invoke issueCredential ───────────▶│
      │                │                    ├──────────────────▶│
      │                │◀──── read set / write set ─────────────┤
      │◀── ③ endorsement (ML-DSA-44) ───────┤
      │  checks both endorsements, assembles and signs the transaction (ML-DSA-44)
      ├── ④ signed transaction ───────────────────────────────────────────────────────▶│
      │                                         a block closes at 10 transactions or 2 s; signed (ML-DSA-44)
      │                │◀────────── ⑤ block ────────────────────────────────────────────┤
      │                │  validate endorsement policy, every signature, read-set versions; commit, update goleveldb
      │◀── ⑥ commit event (valid) ──┤
```
**Figure 3-4.** Transaction flow of one credential issuance under ML-DSA-44 identities

### 3.6 Chaincode Functions and Access Control

The chaincode has twelve functions, listed in Table 3-V. Every write checks the caller's role attribute, validates all inputs and throws on failure, so an invalid write fails endorsement instead of committing an error. Records are written as deterministic JSON so that both endorsing peers produce identical write sets. Figure 3-5 shows the role check that every write function calls: the caller's certificate must carry the role the function requires, otherwise the transaction is rejected.

**Table 3-V. CHAINCODE FUNCTIONS**

| Function | Type | Access | Validation and behaviour |
| --- | --- | --- | --- |
| init | Setup | Deployment | Initialises the chaincode |
| registerHolder | Write | Issuer | Validates identifier and names; refuses re-registration |
| bindHolderKeys | Write | Issuer | Exact ML-KEM-768 (1,184 B) and ML-DSA-44 (1,312 B) key lengths |
| issueCredential | Write | Issuer | Holder must have a bound ML-KEM key; issue time within ±300 s of the transaction; issuer organisation equals the submitting MSP; valid CID; field hashes; signature and key |
| updateExpiry | Write | Issuer | Not for revoked credentials; needs a new commitment hash and signature |
| suspendCredential, restoreCredential | Write | Issuer | Only active, respectively suspended, credentials; time and reason recorded |
| revokeCredential | Write | Issuer | Permanent; signature and key kept for audit |
| getCredential, getHolder | Read | Channel members | Lookup of one record |
| getCredentials, getHolders | Read | Channel members | Batch read of up to 1,000 records, full or summary view |

```javascript
_checkAccess(ctx, role) {
    if (!ctx.clientIdentity.assertAttributeValue('role', role)) {
        throw new Error(`Access denied. Only a ${role} official may access this function.`);
    }
}
```
**Figure 3-5.** Role check applied by every write function of the chaincode

### 3.7 Data Placement and Sources of Truth

The ledger and IPFS are the authoritative sources of all credential data, and the operational database holds no authoritative credential data, as Table 3-VI shows. Encrypting sensitive columns in a conventional database was rejected, because it would leave the database, and whoever controls its keys, as the arbiter of what a credential says. Every value that feeds a trust decision is read from the ledger.

**Table 3-VI. DATA PLACEMENT AND SOURCES OF TRUTH**

| Data item | Authoritative store | Protection |
| --- | --- | --- |
| Credential attribute values and their salts | IPFS (encrypted envelope); plaintext only on the holder's phone | ML-KEM-768 + AES-256-GCM |
| Salted field hashes, commitment hash, issuer signature and key | Ledger | SHA3-256 with 128-bit salts; ML-DSA-44 |
| Credential metadata (holder, type, issue time, expiry, status, CID) | Ledger | Endorsed writes; covered by the signed commitment |
| Holder name, identifier and public keys | Ledger | Issuer-only writes; exact key lengths |
| Holder private keys | Holder's phone only | Platform secure storage |
| Emirates ID mapping, e-mail, sessions, logs, subscriptions, staff | MySQL (never on the ledger) | Database access control |

### 3.8 Deployment Architecture

Figure 3-6 shows the deployment. QPortal is a static Flutter web build served by Vercel; QWallet is installed on the holder's phone. Both call the API through a Tailscale Funnel endpoint that forwards HTTPS requests to the university's virtual machine, where an Nginx gateway serves only the API and the Go backend runs in its own container. The six Fabric containers belong to one Docker Compose project with an automatic restart policy, so the network and its ledgers come back after a restart; IPFS runs as a systemd user service and MySQL on the host. A self-hosted GitHub Actions runner deploys the backend and the gateway on every push to the main branch: it runs the unit tests in the build image first, keeps the running image as a rollback point, restarts the service and polls its health endpoint for up to 60 seconds, restoring the previous image automatically if the new one is unhealthy. The Fabric network, the chaincode and the database schema are changed only by hand. The source code of every part is listed in the Appendix (Table 13-I).

```text
  Browser ──loads──▶ Vercel CDN (QPortal static build)
  Browser ──API calls──┐
  Phone (QWallet) ─────┴──▶ Tailscale Funnel (public HTTPS for /api) ──▶ university VM
  University VM · Ubuntu · 2 vCPUs · 30 GiB RAM · 246 GB disk
  ├── Docker: qchain-web-gateway (Nginx, /api only) ──▶ qchain-api (Go, port 3000)
  │           Fabric network, Compose project qchainpqc: peer0.general, peer0.government, orderer0,
  │           ca.general, ca.government, chaincode-qchaincode (restart: unless-stopped)
  ├── Host services: IPFS (Kubo, systemd user service) · MySQL 8
  └── GitHub Actions runner (self-hosted): on every push to main of qchain-backend
        push ─▶ sync VM checkout ─▶ go test (builder stage) ─▶ keep running image as previous
             ─▶ build, restart ─▶ /health for 60 s ─▶ healthy: done | unhealthy: roll back to previous
```
**Figure 3-6.** Deployment on the university virtual machine, Vercel and the holders' phones, with the continuous-deployment pipeline

## 4. System Analysis

### 4.1 User Roles

The system serves the five actors described in Table 4-I. Issuers, verifiers and IT administrators are staff of participating organisations and use QPortal; holders use QWallet; researchers use the benchmarking tools.

**Table 4-I. SYSTEM ACTORS AND RESPONSIBILITIES**

| Actor | Application | Responsibilities |
| --- | --- | --- |
| Issuer | QPortal | Registers holders, issues credentials, updates expiry dates, suspends, restores and revokes credentials, reviews issuance history |
| Holder | QWallet | Activates the wallet (key generation), receives and views credentials, presents selected attributes by QR code or OTP, approves or rejects verifier subscriptions |
| Verifier | QPortal | Verifies presentations by QR code or OTP, reviews verification history, subscribes to status changes of verified credentials |
| System administrator | QPortal and host | Manages staff and roles, reviews audit logs, operates the Fabric network and its cryptographic material |
| Researcher | Benchmark tools | Runs the benchmarks, compares algorithms and machines, analyses storage and scalability |

### 4.2 Use Cases

**Issuer.** An issuer registers a holder, issues a credential from a schema of named attributes and manages its lifecycle: updating the expiry date produces a new signed commitment, suspension is reversible and revocation is permanent. **Holder.** A holder activates the wallet once, which generates the post-quantum key pairs and registers the public keys, then retrieves credentials, views them after on-device decryption and creates presentations that disclose only the chosen attributes; the holder also decides whether a verifier may follow a credential's status. **Verifier.** A verifier scans a presentation QR code or enters an OTP, receives the verdict with the result of every check, consults the verification history and, with the holder's approval, subscribes to status alerts. **System administrator.** An administrator manages staff accounts and roles, reviews the audit log and operates the network. **Researcher.** A researcher benchmarks the algorithms and the lifecycle and measures storage and scalability.

### 4.3 Use Case Diagram

Figure 4-1 presents the use case diagram of the system with the relationships between the actors and their use cases.

*Figure 4-1. Use case diagram of the QChain system*

## 5. UI/UX Design

QChain has two user interfaces: QWallet, the holder's mobile wallet, and QPortal, the web portal of issuing and verifying organisations. Both keep the cryptography behind plain actions such as issuing, presenting and checking a credential. This chapter presents their main screens.

### 5.1 QWallet

Figure 5-1 shows the final onboarding screen, which completes the wallet set-up by generating the holder's quantum-resistant key pairs, ML-KEM-768 for encryption and ML-DSA-44 for digital signatures, on the phone. The private keys never leave the device; once they are generated and their public keys registered, the screen confirms that the wallet is ready.

Figure 5-2 shows the Home screen, the main dashboard of QWallet. It displays the holder's name and identity status and an overview of credential states, with colour indicators for valid, suspended, revoked and expired credentials, together with recent activity and quick access to favourite documents.

*Figure 5-1. Wallet Set-Up Screen* · *Figure 5-2. Home Screen*

Figure 5-3 shows the Document Details screen, which gives the full information of a credential: the issuing authority, the credential identifier, the digital signature, the ledger anchor and the validity status, as read from the ledger and decrypted on the phone.

Figure 5-4 shows the Selective Disclosure screen, where the holder chooses which fields to reveal before sharing a credential. Hidden fields, and their salts, never leave the device.

*Figure 5-3. Document Details* · *Figure 5-4. Selective Disclosure*

Figure 5-5 shows the QR Presentation screen, which shares the signed presentation through a single-use QR code with an expiry timer; the holder can generate a new code or switch to a six-digit one-time password.

Figure 5-6 shows the Manage Subscriptions screen. A verifier can ask to follow the status of a credential; the request appears here, and the holder accepts or rejects it, keeping control over who is told about later changes.

*Figure 5-5. QR Presentation* · *Figure 5-6. Manage Subscriptions*

### 5.2 QPortal

QPortal gives each role its own workspace. The issuer workspace covers issuance and credential management, and the verifier workspace covers verification by QR code, identifier or one-time password, together with subscriptions to credential status.

The issuer dashboard in Figure 5-7 shows the credentials issued today, unread alerts, upcoming expiries and recent issuing activity, with quick actions for single issuance, batch issuance and schema management. Single issuance is a guided five-step process; Figure 5-8 shows its data-entry step, which captures only the attributes that the chosen template defines, and the final step requires an explicit confirmation before the credential is signed with the issuer's ML-DSA-44 key.

*Figure 5-7. Issuer dashboard*

*Figure 5-8. Single issuance: entering the credential attributes*

The credential details in Figure 5-9 combine the system and holder information with the editable fields and a cryptographic section that shows the signature algorithm (ML-DSA-44), the signature hash, the ledger transaction and the IPFS reference. The management page in Figure 5-10 revokes credentials permanently, suspends them temporarily and restores suspended ones; every action is a ledger transaction.

*Figure 5-9. Credential details*

*Figure 5-10. Revocation and suspension management*

The verifier dashboard in Figure 5-11 shows the verifications completed today, pending alerts and recent results, with quick actions to scan a QR code, verify manually or process a batch. Figure 5-12 shows the result that appears directly after scanning a holder's QR code, with the holder's disclosed details and the outcome of the checks.

*Figure 5-11. Verifier dashboard*

*Figure 5-12. Verification result after a successful scan*

When scanning is not possible, the holder generates a six-digit one-time password in the wallet and the verifier enters it, as Figure 5-13 shows. Every method ends in the same result card: Figure 5-14 shows it for a credential whose data no longer matches its ledger record, which is flagged as tampered and must not be accepted.

*Figure 5-13. Manual verification by one-time password*

*Figure 5-14. Verification result for a tampered credential*

## 6. Implementation

This chapter describes how each layer is implemented; the post-quantum protocols themselves are described in Chapter 7.

### 6.1 Programming Languages and Technologies

Table 6-I lists the languages and technologies of each part of the system.

**Table 6-I. LANGUAGES AND TECHNOLOGIES**

| Part | Language | Main technologies |
| --- | --- | --- |
| Fabric network | Go | Modified Fabric, Fabric CA and fabric-lib-go; Docker Compose |
| Chaincode | JavaScript | Fabric Contract API, chaincode as a service, node:test |
| Backend | Go | Fabric Gateway client, liboqs-go, MySQL driver, IPFS HTTP API |
| QWallet | Dart | Flutter, liboqs, PointyCastle, platform secure storage |
| QPortal | Dart | Flutter web, served by Vercel |
| Benchmarking harness | Go, C, Python | liboqs 0.16.0, OpenSSL 3.0, Docker; Python for the analysis |

### 6.2 Blockchain Layer

The chaincode stores two document types, holders and credentials. A credential record contains the commitment version, the holder identifier, the issuing identity and organisation, the status, the credential type, the issue time, the expiry date, the IPFS content identifier (CID), the field hashes, the commitment hash, the issuer's signature and public key, and the times of lifecycle events; it contains no attribute values. The credential identifier is derived from the Fabric transaction identifier, so it is unique and identical on both peers, and transaction times come from the deterministic transaction timestamp rather than a peer's clock. Figure 6-1 shows the issuance function `issueCredential`. A suite of twelve chaincode tests covers the functions, access control, input validation, the full lifecycle and the batch reads.

```javascript
async issueCredential(ctx, holderID, credentialType, issuedAt, expiryDate, issuerOrgID,
    ipfsCID, fieldHashesJSON, credentialHash, issuerSignature, issuerPublicKey) {
    this._checkAccess(ctx, 'issuer');

    const holder = await this._getHolder(ctx, holderID);
    if (!holder.KemPublicKey) {
        throw new Error(`Holder ${holderID} has not bound an ML-KEM public key`);
    }
    this._requireText(credentialType, 'credentialType');
    this._requireIssuedAt(ctx, issuedAt);
    this._requireExpiry(expiryDate);
    const mspID = ctx.clientIdentity.getMSPID();
    if (issuerOrgID !== mspID) {
        throw new Error(`issuerOrgID ${issuerOrgID} does not match the submitting organisation ${mspID}`);
    }
    if (typeof ipfsCID !== 'string' || !CID.test(ipfsCID)) {
        throw new Error('ipfsCID must be a non-empty IPFS CID');
    }
    const fieldHashes = this._parseFieldHashes(fieldHashesJSON);
    this._requireSignature(credentialHash, issuerSignature, issuerPublicKey);
    // ... builds the credential record (ID, holder, issuer, status, CID,
    // field hashes, credential hash, ML-DSA-44 signature and public key)
    return JSON.stringify({ success: true, credentialID: credID });
}
}
```
**Figure 6-1.** Credential issuance in the chaincode (excerpt)

### 6.3 IPFS Layer

IPFS stores only encrypted envelopes. The backend issues a credential only after its envelope has been sealed to the holder's bound ML-KEM-768 key and added to IPFS; the returned CID becomes part of the signed commitment, so the envelope cannot be swapped without invalidating the signature, and the node pins every envelope so that it is retained. Because the envelope is content-addressed, any change to it changes its CID and is detected.

### 6.4 Backend API

The backend exposes 39 REST endpoints and a health check, grouped in Table 6-II; the mobile endpoints serve QWallet and the others QPortal. Every request is validated before any ledger or database access and errors use one JSON structure. Status, dates, keys, hashes and signatures are always read from the ledger, and a read fails rather than falling back to the database when the ledger is unavailable. Holder and credential identifiers are always generated by the server, and signed issue times and session expiries use UAE local time with an explicit offset.

**Table 6-II. BACKEND API ENDPOINTS BY GROUP**

| Group | Count | Endpoints |
| --- | --- | --- |
| Credential lifecycle | 8 | registerHolder, issueCredential, updateCredential, revokeCredential, suspendCredential, restoreCredential, getAllCredentials, getCredentialDetail |
| Verification | 3 | resolveSession, getVerificationHistory, getVerificationDetail |
| Holders, dashboard and audit | 3 | getHolders, getDashboardStats, getAuditLogs |
| Subscriptions and alerts | 6 | requestSubscription, getSubscriptions, deleteSubscription, unsubscribe, getSubscriptionAlerts, acknowledgeAlert |
| Staff and directory | 5 | getStaff, getDirectory, inviteStaff, updateStaffRole, deleteStaff |
| Mobile (QWallet) | 14 | checkKeys, registerHolderKeys, getEnvelope, getHolderProfile, getCredentialsByHolder, toggleFavorite, getActivity, generateOTP, generatePresentation, getCatalog, fetchDocument, getSubscriptions, approveSubscription, rejectSubscription |
| Health | 1 | health |

## 7. Post-Quantum Security Mechanisms

This chapter describes the mechanisms that protect a credential across its lifecycle. It starts from the threat model, then follows a credential from issuance to verification; Figure 7-1 shows how the mechanisms interact during wallet activation, issuance and retrieval.

### 7.1 Threat Model and Security Requirements

The design considers an external attacker who records network traffic and ledger data today and obtains a quantum computer later; honest-but-curious operators of a peer, the IPFS node or the database; a verifier who tries to learn undisclosed attributes; a holder who tries to alter a disclosed value; and a participant who tries to tamper with ledger records or impersonate a network identity. It assumes that the holder's phone protects its secure storage and that the issuer's signing key is protected on the backend host. Table 7-I states the requirements and how QChain meets them.

**Table 7-I. SECURITY REQUIREMENTS AND HOW QCHAIN MEETS THEM**

| Property | Requirement | Mechanism in QChain | Status |
| --- | --- | --- | --- |
| Confidentiality | Attributes readable only by the holder, also against future quantum attacks | Per-field ML-KEM-768 / HKDF-SHA3-256 / AES-256-GCM envelopes; private key only on the phone | Met |
| Integrity | Any change to committed data is detected | Commitment recomputed from the ledger; salted SHA3-256 field hashes; endorsed writes | Met |
| Authenticity | A credential provably comes from its issuer | ML-DSA-44 signature checked against the trusted issuer key | Met |
| Non-repudiation | Holders cannot deny a presentation | Holder ML-DSA-44 signature bound to one credential | Met |
| Privacy | Verifiers learn only the disclosed attributes | 128-bit per-field salts; undisclosed fields never leave the wallet | Met |
| Quantum-safe ledger identities | Network identities secure against quantum attacks | ML-DSA-44 MSP: every identity, endorsement, block and node TLS certificate | Met |
| Authorisation | Only authorised officials write to the ledger | Role attributes in ML-DSA-44 certificates, enforced by the chaincode | Met |
| Availability | Services recover from restarts; failed deployments cause no outage | Container restart policy; health-checked deployment with rollback | Met on one host |

```text
  QWallet               QPortal (issuer)        QChain API                     Fabric ledger            IPFS
  A. Wallet activation: the phone generates ML-KEM-768 and ML-DSA-44 key pairs; private keys stay in secure storage
     ├── 1 registerHolderKeys (public keys only) ──────────▶│
     │                                                       ├── 2 bindHolderKeys (issuer role, exact lengths) ─▶│
  B. Issuance
                        ├── 4 issueCredential (holder, type, attributes, expiry) ─▶│
                        │      5 read the holder's ML-KEM-768 public key from the ledger
                        │      6 per attribute: 128-bit salt, field hash = SHA3-256(salt:key:value)
                        │      7 seal attributes + salts to the holder (ML-KEM-768 · HKDF-SHA3-256 · AES-256-GCM)
                        │                                    ├── 8 add encrypted envelope ──────────────────────▶│
                        │                                    │◀── 9 CID ──────────────────────────────────────────┤
                        │      10 commitment {v, holderID, type, issuedAt, issuerOrgID, expiryDate, cid, fieldHashes}
                        │         hash = SHA3-256(canonical JSON); signature = ML-DSA-44(issuer key, hash)
                        │                                    ├── 11 issueCredential tx (no attribute values) ─▶│
                        │                                    │      12 validate inputs and role; endorse, order, commit
  C. Retrieval
     ├── 15 getEnvelope ────────────────────────────────────▶│ ── 16 cat CID ──▶ IPFS ── 17 ciphertext ──▶ QWallet
     18 decrypt on the phone and check the values against the on-chain field hashes
```
**Figure 7-1.** Sequence of wallet activation, credential issuance and retrieval

### 7.2 Issuer-Side ML-DSA-44 Signed Commitments

The issuer does not sign the credential body. It signs a **commitment**: a canonical JSON object containing a version number, the holder identifier, the credential type, the issue time, the issuing organisation, the expiry date, the CID of the encrypted envelope and the salted hash of every attribute. The canonical form, built by `commitmentCanonicalJSON`, uses sorted keys, no whitespace and no HTML escaping and always includes every key, so that an empty expiry is covered too. `CommitmentHash` hashes it with SHA3-256 and `SignCommitment` signs the hash with the issuer's ML-DSA-44 key; Figure 7-2 shows both functions. During verification the backend **rebuilds the commitment from the ledger fields** and never trusts the stored hash, so changing any committed field invalidates the signature. The chaincode accepts the signed issue time only within 300 seconds of the transaction's timestamp, which prevents back-dating.

```go

// CommitmentHash returns SHA3-256(canonical commitment JSON) as lowercase hex.
func CommitmentHash(c CredentialCommitment) (string, error) {
	canonical, err := commitmentCanonicalJSON(c)
	if err != nil {
		return "", err
	}
	return Sha3Hex(canonical), nil
}

// SignCommitment hashes the commitment and signs the hash with the issuer key.
// Returns (credentialHash, signatureHex).
func SignCommitment(c CredentialCommitment, privateKeyHex string) (string, string, error) {
	hash, err := CommitmentHash(c)
	if err != nil {
		return "", "", err
	}
	sig, err := PqcSign(hash, privateKeyHex)
	if err != nil {
		return "", "", fmt.Errorf("sign commitment: %w", err)
	}
	return hash, sig, nil
}
```
**Figure 7-2.** Commitment hashing and ML-DSA-44 signing in the backend

### 7.3 Salted Field Hashes for Selective Disclosure

For every attribute the backend draws a 16-byte (128-bit) random salt and computes the field hash SHA3-256(salt : name : value) in `CommitAttributes`; Figure 7-3 shows the salt generation and the salted hash of every attribute. Only the hashes reach the ledger; the salts travel inside the holder's encrypted envelope. Without its salt, the hash of a low-entropy value such as a grade cannot be reversed by guessing, and equal values in two credentials have unrelated hashes, following RFC 9901 [34]. When a holder discloses a field, the presentation carries its value and salt, and the verifier accepts it only if the recomputed hash equals the hash on the ledger; a salt supplied for an undisclosed field is rejected, because it would leak that field.

```go
// SaltedFieldHash is the per-field fingerprint committed on-chain.
func SaltedFieldHash(salt, key, value string) string {
	return Sha3Hex(salt + ":" + key + ":" + value)
}


// CommitAttributes draws a salt for every attribute and returns the salts (for
// the holder's envelope) and the salted field hashes (for the chain).
func CommitAttributes(attrs map[string]string) (salts, fieldHashes map[string]string, err error) {
	salts = make(map[string]string, len(attrs))
	fieldHashes = make(map[string]string, len(attrs))
	for key, value := range attrs {
		salt, err := newFieldSalt()
		if err != nil {
			return nil, nil, err
		}
		salts[key] = salt
		fieldHashes[key] = SaltedFieldHash(salt, key, value)
	}
	return salts, fieldHashes, nil
}
```
**Figure 7-3.** Per-attribute salts and salted field hashes at issuance

### 7.4 Holder-Side ML-KEM-768 Envelope Encryption

The credential body exists in one place only: an envelope on IPFS encrypted to the holder's ML-KEM-768 public key, whose structure is shown in Figure 7-4. `sealAttributes` encapsulates a fresh shared secret to the holder's key; for every field it draws a random 256-bit data key, encrypts the field with AES-256-GCM, and wraps the data key under a single-use key-wrapping key derived with HKDF-SHA3-256 from the shared secret, an envelope context and the field name. Figure 7-5 shows this loop: one encapsulation per recipient, then a fresh data key, an AES-256-GCM ciphertext and a wrapped key for every field. Every derived key is used once, so no AES-GCM key and nonce pair is ever reused, and each attribute can be decrypted on its own.

```text
  Holder's ML-KEM-768 public key (1,184 B, bound on the ledger)
     │  ML-KEM-768 encapsulation → kemCt (1,088 B) + 32-byte shared secret
     ▼
  per field: HKDF-SHA3-256(shared secret, envelope context | field) → single-use key-wrapping key
  per field: random 256-bit data key → AES-256-GCM(value, random 96-bit nonce); the data key is wrapped (48 B)
  Envelope on IPFS (JSON, addressed by its CID):
    { "_qc_env": "qchain-env", "v": 2, "kemAlg": "ML-KEM-768", "aeadAlg": "AES-256-GCM", "kdf": "HKDF-SHA3-256",
      "credId": "env-<random>", "wraps": [ { "recipient": "holder", "kemCt": "<1,088 B>" } ],
      "fields": [ { "key": "College", "nonce": "<12 B>", "ct": "<value + 16 B tag>", "wrap": { "holder": "<48 B>" } },
                  ..., { "key": "_salts", ... } ] }
  On the ledger: fieldHashes { "College": SHA3-256(salt:College:value), ... } and the ML-DSA-44-signed commitment.
  The private key never leaves the holder's phone; the backend, the issuer and the peers cannot decrypt.
```
**Figure 7-4.** Structure and key schedule of the holder-encrypted credential envelope

A key design decision was to keep the decryption keys **on the holder's phone** rather than at the issuing organisation. Organisation-held keys would let the organisation, or anyone who compromises its servers, read every credential it has issued. With holder-held keys the backend can seal an envelope but never open it, and issuance fails if the holder has not bound an ML-KEM key.

```go
// sealAttributes builds a per-field envelope over attrs, wrapped to every
// recipient. contextID is the HKDF context binding stored as the envelope's
// credId. Fields are emitted in sorted key order.
func sealAttributes(contextID string, attrs map[string]json.RawMessage, recipients []Recipient) (*Envelope, error) {
	// ... (checks the recipients, fills the envelope header: ML-KEM-768,
	//      AES-256-GCM, HKDF-SHA3-256; error checks omitted in this excerpt)
	ss := make(map[string]string, len(recipients)) // recipient -> shared secret hex
	for _, r := range recipients {
		kemCt, s, err := kemEncap(r.PubHex)
		env.Wraps = append(env.Wraps, KemWrap{Recipient: r.Name, KemCt: kemCt})
		ss[r.Name] = s
	}
	for _, key := range SortedKeys(attrs) {
		dataKey := make([]byte, 32)
		_, err := io.ReadFull(rand.Reader, dataKey)
		nonce, ct, err := aesSeal(hex.EncodeToString(dataKey), attrs[key])
		f := EncField{Key: key, Nonce: nonce, Ct: ct, Wrap: map[string]string{}}
		for _, r := range recipients {
			kwk, err := deriveKey(ss[r.Name], hkdfFieldInfo(contextID, key))
			w, err := wrapKey(kwk, dataKey)
			f.Wrap[r.Name] = w
		}
		env.Fields = append(env.Fields, f)
	}
	return env, nil
}
```
**Figure 7-5.** Per-field envelope sealing with ML-KEM-768, HKDF-SHA3-256 and AES-256-GCM (excerpt)

### 7.5 Holder-Side ML-DSA-44 Presentations

A presentation is a canonical JSON payload with the credential identifier, the disclosed fields, their salts and a timestamp. `buildSignedDisclosedPayload` in the wallet hashes it with SHA3-256 and signs the hash with the holder's ML-DSA-44 key, whose public key was bound to the holder's ledger record at activation. Figure 7-6 shows the wallet assembling the payload from the chosen fields and their salts, hashing it and signing the hash. The backend stores the payload exactly as signed and registers it as a session identified by a QR token or a six-digit one-time password; sessions expire after 120 seconds and can be redeemed only once. The payload must name the credential for which the session was created, so a signed presentation of one credential cannot be replayed for another. Figure 7-7 follows one presentation from the wallet to the verifier's verdict, through the single-use session, the ledger reads and the five checks.

```dart
static Future<({String disclosedPayloadJson, String holderSignatureHex})>
    buildSignedDisclosedPayload({
  required String credentialID,
  required Map<String, dynamic> disclosedFields,
}) async {
  // ... (checks the input, reads the ML-DSA-44 private key from
  //      secure storage, keeps only the fields the holder chose)
  // One salt per disclosed field — never the salts of hidden fields.
  final salts = saltsForDisclosure(trimmedCredID, bodyOnly.keys);

  final disclosedPayload = <String, dynamic>{
    'credentialID': trimmedCredID,
    'disclosedFields': bodyOnly,
    'salts': salts,
    'timestamp': DateTime.now().toIso8601String(),
  };
  final payloadJson = canonicalJsonEncode(disclosedPayload);
  final payloadHash = sha3Hex(payloadJson);
  final holderSignatureHex = signPayload(payloadHash, dsaPrivHex);
  // ... returns the payload JSON and the holder signature
}
```
**Figure 7-6.** Building and signing a presentation in QWallet (excerpt)

```text
  Holder · QWallet     Verifier · QPortal       QChain API             MySQL              Fabric ledger
  builds {credentialID, disclosed fields, salts, timestamp}, signs SHA3-256(payload) with ML-DSA-44
      ├── ① generatePresentation / generateOTP ─────▶│
      │                                              ├── ② single-use session, 120 s ─▶│
      │◀────────── ③ QR token or 6-digit OTP ────────┤
      ├── ④ shows QR / reads OTP ─▶│
      │                            ├── ⑤ resolveSession ─▶│
      │                            │                      ├── ⑥ load and delete session ─▶│
      │                            │                      ├── ⑦ read credential and holder ──────────────▶│
      │                            │       checks: exists · active · issuer ML-DSA-44 signature over the
      │                            │       recomputed commitment · salted field hashes · holder signature
      │                            │                      ├── ⑧ verification log ─▶│
      │                            │◀─ ⑨ verdict, each check, disclosed fields ─┤
```
**Figure 7-7.** Presentation and verification of a credential

### 7.6 Verification Pipeline

When a verifier redeems a session, `verifyPresentation` reads the credential and holder records from the ledger and runs five checks: the credential exists on the ledger; its status is active; the issuer's ML-DSA-44 signature verifies over the commitment recomputed from the ledger, using the trusted issuer key from the backend's configuration; every disclosed value and salt matches its field hash; and the holder's ML-DSA-44 signature verifies over a payload that names this credential. Figure 7-8 shows the three cryptographic checks: the issuer signature over the recomputed commitment, the field hashes and the holder signature bound to the credential. All checks run even after one fails, so the portal can show every result, and the verdict reports the most significant failure in a fixed order: not found, revoked, suspended, expired, invalid signature, hash mismatch, invalid field hashes, invalid holder signature.

```go
// Issuer signature over the commitment rebuilt from the chain — never the
// stored hash — and only from the trusted issuer key.
if hash, err := pqc.CommitmentHash(pqc.CommitmentFromChain(cred)); err == nil {
	out.HashMatches = hash == cred.CredentialHash
	if in.TrustedIssuerKey != "" && strings.EqualFold(cred.PublicKey, in.TrustedIssuerKey) {
		out.SignatureValid, _ = pqc.PqcVerify(hash, cred.Signature, cred.PublicKey)
	}
}

if payloadErr == nil {
	out.FieldHashesValid = pqc.VerifyDisclosedFields(payload.DisclosedFields, payload.Salts, cred.FieldHashes) == nil

	payloadID := strings.TrimSpace(payload.CredentialID)
	boundToCredential := payloadID != "" && (payloadID == in.DisplayID || payloadID == in.FabricID)
	if boundToCredential && in.Holder != nil && in.Holder.DsaPublicKey != "" && in.HolderSignature != "" {
		out.HolderSignatureValid, _ = pqc.PqcVerify(pqc.Sha3Hex(in.RawPayload), in.HolderSignature, in.Holder.DsaPublicKey)
	}
}
```
**Figure 7-8.** Issuer-signature, field-hash and holder-signature checks of the verification engine (excerpt)

### 7.7 Post-Quantum Identities at the Ledger Layer

The mechanisms above protect the credential; the ML-DSA-44 membership service provider (Section 3.5) protects the ledger that anchors it. Every proposal, endorsement, transaction and block is signed with ML-DSA-44, and peers and the orderer authenticate each other with ML-DSA-44 TLS certificates. A quantum adversary can therefore neither forge an issuer's credential signature nor impersonate the peers, orderer or officials that write the ledger.

### 7.8 Security Review Findings

After the mechanisms were implemented, the team reviewed them against the threat model; Table 7-II lists the findings. The review also confirmed that no AES-GCM key and nonce pair is reused, that issuance fails without a holder key, that no secret is written to logs or responses, and that all database queries are parameterised.

**Table 7-II. SECURITY REVIEW FINDINGS AND RESOLUTIONS**

| # | Finding | Severity | Resolution | Status |
| --- | --- | --- | --- | --- |
| 1 | The holder key-binding transaction did not check the caller's role | High | Issuer role check added; covered by the chaincode tests | Fixed |
| 2 | A presentation was not bound to the credential it was presented for | Medium | Payload must name the session's credential; check fails closed | Fixed |
| 3 | Residual server-side holder key generation and an organisation decryption key | Medium | Removed; only the holder's phone can decrypt | Fixed |
| 4 | A caller-supplied holder identifier could overwrite a holder record | Medium | Identifiers always generated by the server; regression tests | Fixed |
| 5 | Envelope retrieval does not check that the caller is the holder | Low | An envelope is ciphertext that only the holder's key opens | Accepted (low risk) |

## 8. System Testing and Validation

The system is tested at four levels: unit tests of the backend and the chaincode, an end-to-end suite that exercises every API endpoint against the deployed system, a deployment gate that runs the unit tests before every backend deployment, and the validation of the post-quantum network itself. Table 8-I summarises the suites.

**Table 8-I. TEST SUITES AND RESULTS**

| Suite | Framework | Scope | Result |
| --- | --- | --- | --- |
| Backend unit tests | Go testing | 58 test functions: commitment, salts, envelopes, interoperability with the wallet, ML-DSA signing, verification with ledger- and holder-side tampering, wallet identities | All passed (one needs a live network) |
| Chaincode tests | node:test | 12 tests: functions, role-based access, input validation, lifecycle, batch reads | All passed |
| End-to-end API suite | Bash and curl on the deployed system | 123 assertions in 17 groups covering all 39 endpoints and the health check | 123 / 123 passed |
| Deployment gate | GitHub Actions | Unit tests in the build image before every deployment | Enforced |

### 8.1 Unit Tests

The backend unit tests exercise each cryptographic module in isolation: a golden test fixes the canonical form of the commitment; other tests show that a change to any committed field changes the hash, that envelopes round-trip with their salts, that a tampered ciphertext fails authentication and that the backend's key-derivation context matches the wallet's. The verification tests cover valid presentations, selective disclosure, ledger-side and holder-side tampering, an untrusted issuer key, expiry and every lifecycle state, and further tests cover the wallet identities and the ML-DSA-44 Gateway signer. The chaincode tests run the smart contract against a simulated ledger.

### 8.2 End-to-End API Testing

The end-to-end suite drives the deployed system through a complete scenario on the post-quantum network: it registers holders, generates real ML-KEM-768 and ML-DSA-44 key pairs, binds them on the ledger, issues credentials, retrieves and decrypts the envelope as the holder, creates signed presentations, verifies them, exercises the full lifecycle and every portal and wallet function, and checks the audit log. Table 8-II lists the seventeen groups; the run passed all 123 assertions.

**Table 8-II. END-TO-END TEST GROUPS**

| # | Test group | Assertions | What the group demonstrates |
| --- | --- | --- | --- |
| 1 | API health | 1 | Service reports healthy |
| 2 | Holder registration | 6 | Server-generated identifiers; a collision cannot overwrite a holder |
| 3 | Keys before activation | 5 | No keys bound; profile read from the ledger |
| 4 | Holder key generation | 1 | Real ML-KEM-768 and ML-DSA-44 key pairs |
| 5 | Key registration | 7 | Wrong-length key rejected; keys bound and read back from the ledger |
| 6 | Credential issuance | 7 | Invalid attribute rejected; envelope on IPFS; details from the ledger |
| 7 | Envelope retrieval and decryption | 7 | Holder decrypts; one salt per field |
| 8 | Presentation session | 2 | Session created with UAE local expiry |
| 9 | Five-check verification | 10 | All checks pass; hidden field not disclosed; sessions single-use |
| 10 | Tamper detection | 9 | Tampered value, wrong salt, forged signature, missing or extra salts, wrong credential rejected |
| 11 | OTP flow | 2 | OTP generated, redeemed and verified |
| 12 | Credential lifecycle | 21 | Suspend, restore, expiry re-signing, revocation and their verification outcomes |
| 13 | History and dashboard | 8 | Verification history and statistics from the ledger |
| 14 | Wallet endpoints | 12 | Catalogue, document fetch, wallet list, favourites, activity |
| 15 | Subscriptions and alerts | 16 | Request, approve, reject, alert on suspension, acknowledge, unsubscribe |
| 16 | Staff management | 7 | Invite, duplicate rejected, role update, removal |
| 17 | Audit logs | 2 | Staff actions recorded |

### 8.3 Tamper and Negative Testing

The negative tests show that every manipulation the threat model considers is detected:

- a disclosed value altered by the holder is rejected with FIELD_HASHES_INVALID, even though the holder's signature over the altered payload is valid;
- a correct value with a wrong salt fails the field-hash check, and a forged holder signature is rejected with HOLDER_SIGNATURE_INVALID;
- presentations without salts, with a salt for an undisclosed field, or naming another credential are rejected, and a redeemed session cannot be redeemed again;
- suspended, expired and revoked credentials report SUSPENDED, EXPIRED and REVOKED, and a revoked credential cannot be restored or edited;
- altering any committed field on the ledger, or signing with an untrusted key, causes SIGNATURE_INVALID.

### 8.4 Validation of the Post-Quantum Network

The deployed network was validated directly, as Table 8-III shows. The size of a real credential transaction on it was also measured: two `issueCredential` transactions with three and four attributes occupied 46,961 and 47,091 bytes. A transaction on a network with classical identities and the same ML-DSA-44 credential signature takes about 21.2 to 21.4 KB at these attribute counts (Section 10.10), so the ML-DSA-44 identities make a credential transaction about 2.2 times larger: it carries the submitter's ML-DSA-44 certificate (4,157 bytes), the client's signature (2,420 bytes) and two endorsements, each with an ML-DSA-44 certificate and signature. A block still holds ten such transactions. The comparison rests on two transactions and is therefore indicative.

**Table 8-III. VALIDATION OF THE DEPLOYED POST-QUANTUM NETWORK**

| Check | Method | Result |
| --- | --- | --- |
| Identity algorithm | Public-key algorithm of every certificate in the network's crypto material, and of the identities in the newest block | 58 / 58 ML-DSA-44 (OID 2.16.840.1.101.3.4.3.17) |
| Credential signature | Size of the issuer signature stored on the ledger | 2,420 bytes, the ML-DSA-44 size |
| Functional test | End-to-end suite on the post-quantum network | 123 / 123 |
| Data consistency | Ledger, MySQL and IPFS compared record by record | Identical |
| Restart | Docker restarted with the network running | All containers back with their data |

## 9. Benchmarking Methodology and Harness

The benchmarking study measures QChain's post-quantum cryptography in the way the system uses it, on three machines of different kinds. This chapter states the research questions, the algorithms, the machines, the measurement harness, the statistics and the rules that decide whether a measurement can be trusted, and the network configuration of the ledger experiments.

### 9.1 Research Questions and Experiments

The study answers six research questions: what the primitives cost (RQ1); what a complete encrypted credential lifecycle costs (RQ2); how much QChain's integration pattern adds to the C library (RQ3); how sensitive the results are to the hardware, the runtime and the library build (RQ4); what the storage consequences are on the ledger and IPFS, and whether payload size changes Fabric's latency (RQ5); and whether issuer-side and holder-side ML-DSA signing are equivalent (RQ6). Table 9-I lists the fifteen experiments and the machines that ran them; experiments whose result is a size or a ratio that does not depend on the processor ran on the Mac only, by design.

**Table 9-I. THE FIFTEEN EXPERIMENTS AND THE MACHINES THAT RAN THEM**

| ID | Experiment | What is measured | Machines |
| --- | --- | --- | --- |
| E0 | Environment | Processor, kernel, timer, versions, library build; correctness gates | All three |
| E1 | Primitive latency | Key generation, signing, verification, encapsulation and decapsulation of 30 algorithms | All three |
| E2 | Integration cost | Library object reused, re-initialised per call, and QChain's production pattern | All three (VM and laptop without SLH-DSA) |
| E3 | Integration layers | C library, Go-to-C call, Go binding, Go standard library | Mac |
| E4 | Build sensitivity | Portable and native liboqs builds; build without OpenSSL | Mac, laptop; VM (no OpenSSL) |
| E5 | Native execution | E1's cells natively on macOS, outside the container | Mac |
| E6 | Message size | Signing and verification from 64 B to 1 MiB | Mac |
| E7 | Rejection timing | Corrupted signatures, wrong messages, tampered ciphertexts | Mac |
| E8 | Sizes | Keys, signatures and ciphertexts | Mac (identical everywhere) |
| E9 | Memory | Stack, C heap, Go heap and process memory per operation | All three |
| E10 | Issuer–holder equivalence | Issuer and holder signing interleaved, equivalence test | All three |
| E11 | Lifecycle cost | Activation, issuance, retrieval, presentation, verification through QChain's code | Mac (176 pairs); VM, laptop (16 suites) |
| E12 | Concurrency | Throughput and latency with several workers | All three |
| E13 | Ledger and IPFS bytes | Transaction, block and IPFS sizes from a model calibrated on a captured block | Mac |
| E14 | Fabric latency | Commit latency against payload size on QChain's network rebuilt in Docker | Mac |

### 9.2 Algorithms Under Test

Table 9-II lists the 30 algorithms: every final NIST parameter set, the two schemes selected for standardisation and five classical baselines. The 25 post-quantum algorithms come from liboqs 0.16.0 [64], called from Go through its binding; the classical ones come from OpenSSL 3.0 and cross the same Go-to-C boundary. The two classical key-encapsulation mechanisms implement DHKEM as specified in RFC 9180 and reproduce its published test vectors [31].

**Table 9-II. ALGORITHMS UNDER TEST**

| Group | Algorithms | Count |
| --- | --- | --- |
| Signatures: final standards | ML-DSA-44, -65, -87 (FIPS 204); SLH-DSA with SHA2 and SHAKE at 128, 192 and 256 bits, small (s) and fast (f) variants (FIPS 205) | 15 |
| Signatures: selected | Falcon-512, Falcon-1024 and their padded variants | 4 |
| Signatures: classical | ECDSA P-256, Ed25519, RSA-3072 (PSS) | 3 |
| KEMs: final standard | ML-KEM-512, -768, -1024 (FIPS 203) | 3 |
| KEMs: selected | HQC-1, HQC-3, HQC-5 | 3 |
| KEMs: classical | DHKEM(X25519), DHKEM(P-256) | 2 |

### 9.3 Measurement Machines

Table 9-III describes the three machines. Every measurement ran in the same Linux container image, pinned to one CPU, with the same compiler, libraries and Go version, so that the machines differ only in their hardware and host. The Mac is an ARM laptop, the laptop an x86 processor with AVX2 vector instructions, and the university's virtual machine an x86 virtual CPU without AVX2.

**Table 9-III. THE THREE MEASUREMENT MACHINES**

| Item | Mac | Laptop | VM |
| --- | --- | --- | --- |
| Host | MacBook Air (M1, 2020), 8 GB; macOS; Docker in Colima | HP laptop, AMD Ryzen 5 7520U (4 cores, 8 threads), 8 GB; Windows 11; Docker on WSL2 | Proxmox (KVM) virtual machine, 2 virtual CPUs, 30 GiB; Ubuntu; Docker |
| Processor seen | Apple M1 core (ARMv8, implementer 0x61) | AMD Ryzen 5 7520U with Radeon Graphics | QEMU Virtual CPU version 2.5+ |
| Vector instructions | ARM NEON, SHA-3 | AVX2, AES-NI | No AVX2; AES-NI |
| Container | 1 pinned CPU, 3 GiB, no swap | 1 pinned CPU, 3 GiB, no swap | 1 pinned CPU, 4 GiB, no swap |
| Software | Go 1.24.13, liboqs 0.16.0, OpenSSL 3.0.22, gcc 12.2 | Same | Same |
| Timer resolution | 41 ns | 40 ns | 54 ns |

### 9.4 Measurement Harness

The harness is an independent code base that never touches the live system; Figure 9-1 shows it on the Mac, and the other two machines use launchers with the same checks. A launcher checks the machine before every run and refuses to start on battery power, with other applications open or with background load above a limit. Inside the container an orchestrator runs the measurement blocks in a seeded, shuffled order, each in a fresh process, and decides after every block whether its measurement is valid; at the end the results are copied out and their checksums verified. To measure QChain itself rather than an approximation, the harness contains verbatim copies of the backend's production functions (envelope sealing, commitment, signing, verification) and of the wallet's decryption, checked against the repository by a script.

*Figure 9-1. The measurement harness, shown on the Mac*

### 9.5 Statistics and Data Quality

Every measurement block is one fresh process for one experiment, algorithm, operation and repetition, with its own keys and inputs prepared by a separate process. Fast operations receive 1,000 untimed warm-up calls and 10,000 timed samples; slow operations are sampled for at least 60 seconds and 30 samples. Every sample is stored, outliers are never removed and every output is checked for correctness outside the timed region. The result of a cell is the median, over five repetitions, of the per-repetition medians, with bootstrap 95 % intervals.

For every block the harness records the CPU use of everything else on the machine, the thermal state, the power source and any CPU throttling, and it runs a fixed canary benchmark every 10 minutes. A block disturbed by an external cause is repeated, up to three attempts, and its headline result is its first undisturbed attempt; the thresholds were calibrated on each machine by a trial run and frozen before the main runs. Table 9-IV summarises the data quality. Seventeen laptop blocks had no undisturbed attempt; a sensitivity analysis over 5,030 cells shows that using all attempts instead of the headline attempts changes the medians by 0.0 % in the median case and by at most 5.5 %, so no conclusion depends on the flagging rules.

**Table 9-IV. DATA QUALITY OF THE MAIN RUNS**

|  | Mac | Laptop | VM |
| --- | --- | --- | --- |
| Main runs | 14 plans | 8 plans | 7 plans |
| Measurement attempts | 11,507 | 6,672 | 5,997 |
| Attempts flagged for an external cause | 5.0 % | 11.5 % | 10.1 % |
| Main cause of flags | Other CPU activity | Other CPU activity | Canary deviation |
| Blocks without an undisturbed attempt | 0 | 17 | 0 |

### 9.6 Network Configuration of the Ledger Experiments

The two ledger experiments, E13 and E14, measured QChain's Fabric network in the configuration it had while they ran, before the network moved to ML-DSA-44 identities. In that configuration only the credential's own issuer signature was post-quantum: the Fabric identities and TLS certificates were ECDSA P-256, and both peers shared one CouchDB state database. E14 rebuilt that network in Docker from its configuration files, with a synthetic chaincode whose transactions carry payloads sized like QChain's credentials, and E13 modelled its transaction and block sizes from a captured block. Table 9-V compares that configuration with the deployed network described in Chapter 3; the results of E13 and E14 describe the former.

**Table 9-V. NETWORK OF THE LEDGER EXPERIMENTS AGAINST THE DEPLOYED NETWORK**

|  | Ledger experiments (E13, E14) | Deployed network |
| --- | --- | --- |
| Fabric identities and node TLS | ECDSA P-256 | ML-DSA-44 |
| Credential signature | ML-DSA-44 (and the other schemes under test) | ML-DSA-44 |
| State database | One CouchDB shared by both peers | One goleveldb per peer |
| Fabric build | Hyperledger Fabric 2.5.15 | Modified Fabric, 3.1.5 line |
| Chaincode | Synthetic payload chaincode (E14) | qchaincode 2.0 as a service |

## 10. Benchmarking Results

This chapter presents the results of all fifteen experiments on all three machines. Times are medians over five repetitions; the charts show the important algorithms and family averages, and the tables give the complete data. The family averages are geometric means, because the members of a family can differ by orders of magnitude.

### 10.1 Primitive Latency

Table 10-I and Table 10-II list the median time of every operation of the 30 algorithms on the three machines. ML-DSA-44 signs in 60, 79 and 206 µs and verifies in 28, 32 and 79 µs on the Mac, the laptop and the VM. The classical schemes are cheaper to sign and to generate keys, the lattice schemes cheaper to verify and to exchange keys, and SLH-DSA and HQC are the slow families.

**Table 10-I. SIGNATURE SCHEMES: MEDIAN LATENCY PER CALL ON THE THREE MACHINES**

| Algorithm | KeyGen Mac | KeyGen Laptop | KeyGen VM | Sign Mac | Sign Laptop | Sign VM | Verify Mac | Verify Laptop | Verify VM |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| ML-DSA-44 | 30.2 µs | 32.0 µs | 77.3 µs | 60.2 µs | 79.2 µs | 206 µs | 28.4 µs | 32.0 µs | 78.7 µs |
| ML-DSA-65 | 53.0 µs | 55.0 µs | 132 µs | 95.5 µs | 125 µs | 321 µs | 47.6 µs | 53.1 µs | 108 µs |
| ML-DSA-87 | 90.9 µs | 85.2 µs | 183 µs | 140 µs | 153 µs | 415 µs | 83.5 µs | 81.3 µs | 184 µs |
| Falcon-512 | 4.80 ms | 5.62 ms | 11.6 ms | 145 µs | 210 µs | 3.67 ms | 21.7 µs | 41.0 µs | 36.2 µs |
| Falcon-padded-512 | 4.80 ms | 5.61 ms | 11.5 ms | 145 µs | 208 µs | 3.66 ms | 21.8 µs | 41.0 µs | 38.0 µs |
| Falcon-1024 | 14.9 ms | 15.8 ms | 33.0 ms | 290 µs | 417 µs | 7.91 ms | 42.6 µs | 81.0 µs | 78.3 µs |
| Falcon-padded-1024 | 14.8 ms | 15.8 ms | 33.1 ms | 287 µs | 419 µs | 7.93 ms | 41.9 µs | 80.7 µs | 79.4 µs |
| SLH-DSA-SHA2-128f | 888 µs | 890 µs | 981 µs | 20.7 ms | 20.8 ms | 22.7 ms | 1.25 ms | 1.26 ms | 1.39 ms |
| SLH-DSA-SHA2-128s | 56.8 ms | 56.0 ms | 61.1 ms | 431 ms | 430 ms | 470 ms | 428 µs | 438 µs | 471 µs |
| SLH-DSA-SHA2-192f | 1.30 ms | 1.29 ms | 1.40 ms | 33.8 ms | 34.3 ms | 36.7 ms | 1.82 ms | 1.84 ms | 2.00 ms |
| SLH-DSA-SHA2-192s | 82.7 ms | 81.6 ms | 90.0 ms | 758 ms | 774 ms | 838 ms | 633 µs | 641 µs | 724 µs |
| SLH-DSA-SHA2-256f | 3.42 ms | 3.46 ms | 3.67 ms | 69.5 ms | 71.5 ms | 76.7 ms | 1.87 ms | 1.91 ms | 2.07 ms |
| SLH-DSA-SHA2-256s | 54.7 ms | 54.9 ms | 58.7 ms | 668 ms | 702 ms | 749 ms | 928 µs | 953 µs | 1.04 ms |
| SLH-DSA-SHAKE-128f | 1.12 ms | 1.70 ms | 1.62 ms | 26.2 ms | 41.3 ms | 37.6 ms | 1.56 ms | 2.37 ms | 2.25 ms |
| SLH-DSA-SHAKE-128s | 70.8 ms | 113 ms | 103 ms | 553 ms | 862 ms | 789 ms | 539 µs | 780 µs | 776 µs |
| SLH-DSA-SHAKE-192f | 1.64 ms | 2.48 ms | 2.36 ms | 43.2 ms | 67.0 ms | 61.7 ms | 2.28 ms | 3.53 ms | 3.27 ms |
| SLH-DSA-SHAKE-192s | 104 ms | 166 ms | 150 ms | 982 ms | 1.50 s | 1.40 s | 786 µs | 1.16 ms | 1.13 ms |
| SLH-DSA-SHAKE-256f | 4.34 ms | 6.05 ms | 6.20 ms | 89.3 ms | 125 ms | 127 ms | 2.34 ms | 3.27 ms | 3.36 ms |
| SLH-DSA-SHAKE-256s | 69.0 ms | 96.1 ms | 100 ms | 887 ms | 1.22 s | 1.25 s | 1.16 ms | 1.62 ms | 1.67 ms |
| ECDSA P-256 | 25.9 µs | 38.4 µs | 40.8 µs | 19.9 µs | 24.4 µs | 30.9 µs | 54.5 µs | 58.9 µs | 78.4 µs |
| Ed25519 | 32.1 µs | 41.7 µs | 46.7 µs | 30.3 µs | 38.1 µs | 40.3 µs | 81.6 µs | 101 µs | 102 µs |
| RSA-3072-PSS | 402 ms | 524 ms | 510 ms | 1.60 ms | 1.53 ms | 2.18 ms | 33.0 µs | 36.9 µs | 53.3 µs |

**Table 10-II. KEY-ENCAPSULATION MECHANISMS: MEDIAN LATENCY PER CALL ON THE THREE MACHINES**

| Algorithm | KeyGen Mac | KeyGen Laptop | KeyGen VM | Encaps Mac | Encaps Laptop | Encaps VM | Decaps Mac | Decaps Laptop | Decaps VM |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| ML-KEM-512 | 8.33 µs | 11.9 µs | 24.0 µs | 9.12 µs | 12.8 µs | 26.1 µs | 10.1 µs | 14.5 µs | 26.2 µs |
| ML-KEM-768 | 13.3 µs | 20.3 µs | 37.1 µs | 14.0 µs | 20.6 µs | 38.9 µs | 16.1 µs | 23.7 µs | 40.3 µs |
| ML-KEM-1024 | 20.2 µs | 28.5 µs | 55.5 µs | 21.0 µs | 29.4 µs | 53.7 µs | 23.7 µs | 34.0 µs | 58.6 µs |
| HQC-1 | 894 µs | 1.08 ms | 1.23 ms | 1.79 ms | 2.16 ms | 2.48 ms | 2.73 ms | 3.30 ms | 3.73 ms |
| HQC-3 | 2.35 ms | 3.27 ms | 3.66 ms | 4.67 ms | 6.51 ms | 7.27 ms | 7.04 ms | 9.80 ms | 10.9 ms |
| HQC-5 | 5.92 ms | 7.96 ms | 8.85 ms | 11.8 ms | 15.9 ms | 17.7 ms | 17.8 ms | 23.9 ms | 26.8 ms |
| X25519 | 31.9 µs | 39.9 µs | 42.6 µs | 67.4 µs | 77.8 µs | 90.3 µs | 37.5 µs | 39.2 µs | 45.1 µs |
| ECDH P-256 | 26.0 µs | 37.5 µs | 38.5 µs | 110 µs | 131 µs | 138 µs | 99.5 µs | 117 µs | 126 µs |

Figure 10-1 compares signature key generation. ML-DSA-44 generates a key pair in 30–77 µs, close to ECDSA P-256 (26–41 µs) and Ed25519 (32–47 µs), whereas the Falcon and SLH-DSA family averages need 8–20 ms and RSA-3072 0.4–0.5 s. Figure 10-2 shows signing: ML-DSA-44 signs in 60, 79 and 206 µs, three to seven times slower than ECDSA P-256 (20–31 µs) but far faster than the Falcon average on the VM without AVX2 (5.4 ms) and the SLH-DSA average (168–211 ms).

*Figure 10-1. Signature key generation: QChain's ML-DSA-44, family averages and the classical schemes*

*Figure 10-2. Signing: QChain's ML-DSA-44, family averages and the classical schemes*

Figure 10-3 shows verification, the operation a verifier performs for every presentation. ML-DSA-44 verifies in 28–79 µs, faster than ECDSA P-256 (55–78 µs) on the Mac and the laptop and level with it on the VM; RSA-3072 (33–53 µs) and Falcon (30–58 µs on average) are comparable, while SLH-DSA verification takes 1.1–1.4 ms.

*Figure 10-3. Verification: QChain's ML-DSA-44, family averages and the classical schemes*

Figure 10-4, Figure 10-5 and Figure 10-6 show the three operations of the key-encapsulation mechanisms. ML-KEM-768 needs 13–40 µs for each of them, less than X25519 (32–90 µs) and ECDH P-256 (26–138 µs) for encapsulation and decapsulation, while the HQC family average needs 2.3–10.3 ms, about 170 to 430 times ML-KEM-768.

*Figure 10-4. KEM key generation: QChain's ML-KEM-768, family averages and the classical schemes*

*Figure 10-5. Encapsulation: QChain's ML-KEM-768, family averages and the classical schemes*

*Figure 10-6. Decapsulation: QChain's ML-KEM-768, family averages and the classical schemes*

ML-DSA signing has a heavy tail on every machine, with a 99th percentile 2.8 to 4.1 times its median, caused by its rejection sampling; the other lattice operations stay under 2.8 times. A fresh process also pays a one-off cost on its first randomised call, about 0.9 ms more than the warm median on the Mac and 1.7 to 1.9 ms on the laptop and the VM. Table 10-III lists, for six key operations, the 99th percentile as a multiple of the median and the extra time of the first call on each machine.

**Table 10-III. TAIL LATENCY AND FIRST-CALL COST OF SELECTED OPERATIONS**

| Operation | p99 ÷ median Mac | p99 ÷ median Laptop | p99 ÷ median VM | First call extra Mac | First call extra Laptop | First call extra VM |
| --- | --- | --- | --- | --- | --- | --- |
| ML-DSA-44 signing | 3.62× | 3.95× | 4.01× | 906 µs | 1.86 ms | 2.02 ms |
| ML-DSA-44 verification | 1.15× | 1.85× | 1.30× | 21.4 µs | 85.5 µs | 133 µs |
| Falcon-512 signing | 1.15× | 1.43× | 1.06× | 912 µs | 1.71 ms | 1.93 ms |
| ML-KEM-768 encapsulation | 1.31× | 2.04× | 1.50× | 855 µs | 1.81 ms | 1.75 ms |
| ML-KEM-768 decapsulation | 1.15× | 1.77× | 1.48× | 21.8 µs | 69.1 µs | 104 µs |
| ECDSA P-256 signing | 1.28× | 2.31× | 1.64× | 358 µs | 770 µs | 691 µs |

### 10.2 Key, Signature and Ciphertext Sizes

Table 10-IV lists the public key, secret key and signature or ciphertext of all 30 algorithms. The sizes are the same on every machine and match FIPS 203 and 204 exactly; Falcon and ECDSA signatures vary in length, so their medians over 1,000 signatures are given. ML-DSA-44's 2,420-byte signature is 34 times an ECDSA signature, while ML-KEM-768's 1,088-byte ciphertext is the smallest among the post-quantum KEMs at its security level.

**Table 10-IV. SIZES OF KEYS, SIGNATURES AND CIPHERTEXTS (BYTES)**

| Algorithm | Public key | Secret key | Signature or ciphertext | Algorithm | Public key | Secret key | Signature or ciphertext |
| --- | --- | --- | --- | --- | --- | --- | --- |
| ML-DSA-44 | 1,312 | 2,560 | 2,420 | SLH-DSA-SHAKE-192f | 48 | 96 | 35,664 |
| ML-DSA-65 | 1,952 | 4,032 | 3,309 | SLH-DSA-SHAKE-192s | 48 | 96 | 16,224 |
| ML-DSA-87 | 2,592 | 4,896 | 4,627 | SLH-DSA-SHAKE-256f | 64 | 128 | 49,856 |
| Falcon-512 | 897 | 1,281 | 655 | SLH-DSA-SHAKE-256s | 64 | 128 | 29,792 |
| Falcon-padded-512 | 897 | 1,281 | 666 | ECDSA P-256 | 65 | 32 | 71 |
| Falcon-1024 | 1,793 | 2,305 | 1,271 | Ed25519 | 32 | 32 | 64 |
| Falcon-padded-1024 | 1,793 | 2,305 | 1,280 | RSA-3072-PSS | 422 | 1,793 | 384 |
| SLH-DSA-SHA2-128f | 32 | 64 | 17,088 | ML-KEM-512 | 800 | 1,632 | 768 |
| SLH-DSA-SHA2-128s | 32 | 64 | 7,856 | ML-KEM-768 | 1,184 | 2,400 | 1,088 |
| SLH-DSA-SHA2-192f | 48 | 96 | 35,664 | ML-KEM-1024 | 1,568 | 3,168 | 1,568 |
| SLH-DSA-SHA2-192s | 48 | 96 | 16,224 | HQC-1 | 2,241 | 2,321 | 4,433 |
| SLH-DSA-SHA2-256f | 64 | 128 | 49,856 | HQC-3 | 4,514 | 4,602 | 8,978 |
| SLH-DSA-SHA2-256s | 64 | 128 | 29,792 | HQC-5 | 7,237 | 7,333 | 14,421 |
| SLH-DSA-SHAKE-128f | 32 | 64 | 17,088 | X25519 | 32 | 32 | 32 |
| SLH-DSA-SHAKE-128s | 32 | 64 | 7,856 | ECDH P-256 | 65 | 32 | 65 |

### 10.3 Hardware, Runtime and Build Sensitivity

Table 10-V gives, for each family, the ratio of the E1 medians between the machines. The hardware effect is large and depends on the family. Against the Mac, the laptop is 1.10 times slower for ML-DSA and 1.44 times for ML-KEM, and the VM 2.56 and 2.75 times. The AVX2 effect, laptop against VM, is 2.4 times for ML-DSA, 1.8 times for ML-KEM and 17 times for Falcon signing, which relies on floating-point vector code; SLH-DSA with SHA-2 and HQC are portable code and hardly depend on the processor. In 83 to 89 of the 90 cells, every repetition on one machine is slower than every repetition on the other.

**Table 10-V. HOW MUCH SLOWER ONE MACHINE IS THAN ANOTHER, BY FAMILY (E1 MEDIANS)**

| Family | Laptop ÷ Mac | VM ÷ Mac | VM ÷ Laptop |
| --- | --- | --- | --- |
| ML-DSA | 1.10 | 2.56 | 2.42 |
| Falcon | 1.44 | 2.31 | 2.07 |
| SLH-DSA | 1.22 | 1.28 | 1.05 |
| ML-KEM | 1.44 | 2.75 | 1.83 |
| HQC | 1.34 | 1.50 | 1.12 |
| Classical | 1.22 | 1.34 | 1.08 |

The production build of liboqs, which selects optimised code at run time, is within 2 % of a build tuned for the machine, whereas a portable build makes ML-DSA 1.4 (Mac) and 2.35 (laptop) times slower and Falcon signing up to 27 times slower. Table 10-VI compares each build variant, and native macOS, with the production build in the container, family by family. Running in Docker on the Mac costs nothing measurable: native macOS takes 0.97 to 1.09 times the container's time.

**Table 10-VI. BUILD AND RUNTIME VARIANTS AGAINST THE PRODUCTION BUILD, BY FAMILY**

| Family | Mac portable | Mac native | Laptop portable | Laptop native | VM without OpenSSL | Native macOS ÷ container |
| --- | --- | --- | --- | --- | --- | --- |
| ML-DSA | 1.38 | 1.00 | 2.35 | 0.99 | 1.00 | 1.03 |
| Falcon | 2.20 | 1.00 | 2.12 | 1.00 | 1.00 | 1.09 |
| SLH-DSA | 1.00 | 1.00 | 1.00 | 0.92 | 0.99 | 0.97 |
| ML-KEM | 1.45 | 1.01 | 1.76 | 0.98 | 0.95 | 0.99 |
| HQC | 1.00 | 1.00 | 1.00 | 0.97 | 1.01 | 0.97 |
| Classical | 1.00 | 1.00 | 1.00 | 1.00 | 1.00 | 0.98 |

### 10.4 Integration Cost

QChain's deployed integration pattern creates a new library object for every call and exchanges keys, signatures and ciphertexts as hexadecimal text. It adds 6 to 13 % to ML-DSA calls and 14 to 28 % to ML-KEM calls, mostly from the hexadecimal conversion, and up to 2.2 times to classical calls, which parse the key again on every call. Table 10-VII lists this overhead per family on each machine. The Go binding itself adds a median of 0.3 % to the C library over the 46 operations that take under a millisecond; an empty call across the Go-to-C boundary costs about 28 ns.

**Table 10-VII. WHAT QCHAIN'S PRODUCTION PATTERN ADDS, BY FAMILY (PRODUCTION ÷ REUSED KEY)**

| Family | Mac | Laptop | VM |
| --- | --- | --- | --- |
| ML-DSA | 1.116 | 1.129 | 1.059 |
| Falcon | 1.020 | 1.021 | 1.005 |
| SLH-DSA | 1.002 | not run | not run |
| ML-KEM | 1.257 | 1.279 | 1.137 |
| HQC | 1.007 | 1.005 | 1.003 |
| Classical | 1.136 | 1.200 | 1.154 |

### 10.5 Credential Lifecycle Cost

The lifecycle cost is the sum of five stages, wallet activation, issuance, retrieval, presentation and verification, for a credential with 10 attributes of which 5 are disclosed, measured through QChain's own code. This is cryptographic computation on the measurement machine, not the time a user waits, which is dominated by the network and the ledger (Section 10.11). For QChain's production suite, ML-DSA-44 with ML-KEM-768, it is **0.69 ms on the Mac, 0.96 ms on the laptop and 1.32 ms on the VM**, which is 0.86, 0.78 and 1.13 times the classical ECDSA P-256 with ECDH P-256 suite. Figure 10-7 charts seven representative suites on the three machines, and Table 10-VIII lists all 16 suites with their ratio to the classical suite. The post-quantum suite is therefore cheaper than the classical one on both machines with optimised vector code and 13 % dearer on the VM without AVX2.

*Figure 10-7. Lifecycle cost per credential of seven key algorithm suites on the three machines*

**Table 10-VIII. LIFECYCLE COST OF THE 16 ALGORITHM SUITES (MS, SUM OF THE FIVE STAGES)**

| Suite | Mac | Laptop | VM | ÷ classical Mac | ÷ classical Laptop | ÷ classical VM |
| --- | --- | --- | --- | --- | --- | --- |
| ML-DSA-44 + ML-KEM-512 | 0.66 | 0.92 | 1.27 | 0.82 | 0.74 | 1.08 |
| ML-DSA-44 + ML-KEM-768 | 0.69 | 0.96 | 1.32 | 0.86 | 0.78 | 1.13 |
| Ed25519 + X25519 | 0.79 | 1.16 | 1.14 | 0.97 | 0.94 | 0.98 |
| ML-DSA-65 + ML-KEM-768 | 0.86 | 1.17 | 1.82 | 1.07 | 0.96 | 1.55 |
| ECDSA P-256 + ECDH P-256 | 0.81 | 1.23 | 1.18 | 1.00 | 1.00 | 1.00 |
| ML-DSA-87 + ML-KEM-1024 | 1.16 | 1.49 | 2.27 | 1.44 | 1.21 | 1.92 |
| Falcon-512 + ML-KEM-512 | 5.53 | 6.72 | 19.19 | 6.85 | 5.47 | 16.32 |
| ML-DSA-44 + HQC-1 | 6.11 | 7.61 | 8.79 | 7.59 | 6.20 | 7.45 |
| Falcon-1024 + ML-KEM-1024 | 16.03 | 17.68 | 49.98 | 19.80 | 14.39 | 42.84 |
| ML-DSA-65 + HQC-3 | 15.08 | 21.14 | 23.88 | 18.69 | 17.21 | 20.34 |
| SLH-DSA-SHA2-128f + ML-KEM-512 | 45.59 | 46.09 | 49.29 | 56.54 | 37.53 | 41.95 |
| ML-DSA-87 + HQC-5 | 36.96 | 49.89 | 56.18 | 45.62 | 40.64 | 47.89 |
| SLH-DSA-SHA2-192f + ML-KEM-768 | 73.72 | 75.34 | 80.93 | 91.45 | 61.14 | 69.24 |
| SLH-DSA-SHA2-256f + ML-KEM-1024 | 147.69 | 152.10 | 160.93 | 183.23 | 123.78 | 136.73 |
| RSA-3072-PSS + ECDH P-256 | 421.79 | 488.16 | 568.29 | 524.56 | 398.14 | 481.59 |
| SLH-DSA-SHA2-128s + ML-KEM-512 | 923.96 | 916.80 | 994.07 | 1147.16 | 746.08 | 840.14 |

Table 10-IX breaks the production suite's cost into its five stages on each machine: issuance is the most expensive stage and verification the second. The cost of a suite is set by its signature scheme: across the Mac's 176 signature and KEM pairs, the lattice and classical KEMs change the total by at most a factor of 1.3, whereas HQC multiplies it by 8 to 52; Falcon suites cost 8 to 74 times and SLH-DSA suites 65 to 3,000 times the ECDSA with X25519 suite. A chained run of all stages costs 9 % (Mac), 17 % (laptop) and 13 % (VM) more than the sum of its stages, and issuance and verification grow linearly with the number of attributes, by about 10 µs per attribute on the Mac and 14 to 15 µs on the other machines.

**Table 10-IX. LIFECYCLE STAGES OF THE PRODUCTION SUITE, ML-DSA-44 + ML-KEM-768 (MS)**

| Machine | Activation | Issuance | Retrieval | Presentation | Verification | Total |
| --- | --- | --- | --- | --- | --- | --- |
| Mac | 0.083 | 0.210 | 0.136 | 0.112 | 0.150 | 0.689 |
| Laptop | 0.101 | 0.324 | 0.188 | 0.155 | 0.189 | 0.958 |
| VM | 0.157 | 0.422 | 0.189 | 0.281 | 0.259 | 1.319 |

### 10.6 Issuer and Holder Signing

Issuer-side signing (over a commitment hash) and holder-side signing (over a presentation hash) were interleaved in one process and tested for equivalence with two one-sided tests, within each machine's own margin. All nine tests show equivalence; the two sides differ by less than 1.1 % on every machine. Table 10-X lists each test with the medians of both sides, their difference and the machine's margin.

**Table 10-X. ISSUER-SIDE AGAINST HOLDER-SIDE ML-DSA SIGNING: EQUIVALENCE TESTS**

| Machine | ML-DSA set | Issuer (µs) | Holder (µs) | Holder against issuer | Margin | Equivalent |
| --- | --- | --- | --- | --- | --- | --- |
| Mac | ML-DSA-44 | 60.1 | 60.1 | -0.15 % | ±2.0 % | yes |
| Laptop | ML-DSA-44 | 79.5 | 78.2 | -1.04 % | ±6.1 % | yes |
| VM | ML-DSA-44 | 212.8 | 210.7 | -0.30 % | ±7.0 % | yes |
| Mac | ML-DSA-65 | 96.4 | 96.0 | -0.11 % | ±2.1 % | yes |
| Laptop | ML-DSA-65 | 126.2 | 126.6 | +0.20 % | ±2.0 % | yes |
| VM | ML-DSA-65 | 329.8 | 329.0 | -0.15 % | ±3.3 % | yes |
| Mac | ML-DSA-87 | 139.1 | 138.8 | +0.08 % | ±2.0 % | yes |
| Laptop | ML-DSA-87 | 154.0 | 153.9 | +0.33 % | ±2.0 % | yes |
| VM | ML-DSA-87 | 423.2 | 420.6 | -0.57 % | ±21.7 % | yes |

### 10.7 Memory

Table 10-XI lists the peak stack use of nine representative operations on the three machines. Heap use is identical on the three machines and stack use differs by at most 3 %. An ML-DSA-44 signing call uses about 49 KiB of stack, ML-DSA-87 about 111 KiB, Falcon-512 44 KiB and SLH-DSA about 4.4 KiB, against about 3 KiB for ECDSA; HQC decapsulation needs 64 to 206 KiB.

**Table 10-XI. PEAK STACK USE OF ONE CALL (KIB)**

| Operation | Mac | Laptop | VM |
| --- | --- | --- | --- |
| ML-DSA-44 signing | 49.0 | 49.3 | 48.4 |
| ML-DSA-87 signing | 111.0 | 111.3 | 110.4 |
| Falcon-512 signing | 42.6 | 44.0 | 43.4 |
| SLH-DSA-SHA2-128f signing | 4.4 | 4.4 | 4.4 |
| ECDSA P-256 signing | 2.5 | 3.1 | 2.6 |
| ML-KEM-768 decapsulation | 17.3 | 18.0 | 17.1 |
| HQC-1 decapsulation | 64.1 | 64.3 | 64.3 |
| HQC-5 decapsulation | 205.7 | 205.9 | 205.9 |
| X25519 decapsulation | 2.8 | 3.0 | 3.0 |

### 10.8 Concurrency

With several workers, the production suite's issuance reaches 4,487, 8,414 and 12,730 calls per second with one, two and four workers on the Mac, and 2,629, 4,641, 7,607 and 9,599 with one, two, four and eight workers on the laptop; the VM, with two virtual CPUs, reaches 1,996 and 3,000 with one and two workers. Efficiency at four workers is 0.71 to 0.78 on the Mac and the laptop, and the 99th percentile grows 2 to 5 times from one worker to the highest count. Figure 10-8 plots the speed-up of issuance and verification against the number of workers; the dashed line marks perfect scaling.

*Figure 10-8. Throughput speed-up of the production suite against the number of workers*

### 10.9 Message Size and Rejection Timing

Table 10-XII lists signing and verification times from 64 bytes to 1 MiB on the Mac. ML-DSA-44 signing takes 59.8 µs for 64 bytes, 62.5 µs for 1 KiB, 228 µs for 100 KiB and 1.79 ms for 1 MiB; QChain signs a 64-character hexadecimal hash, so message size does not matter for it. Rejecting a corrupted signature, a wrong message or a tampered ciphertext takes as long as accepting a valid input in 51 of 52 comparisons (within 3.1 %), so no timing difference reveals why an input failed; the exception is ECDH on P-256, which refuses a malformed public point after 0.14 of the normal time, by design.

**Table 10-XII. SIGNING AND VERIFICATION TIME AGAINST MESSAGE SIZE ON THE MAC (µS)**

| Operation | 64 B | 1 KiB | 10 KiB | 100 KiB | 1 MiB |
| --- | --- | --- | --- | --- | --- |
| ML-DSA-44 signing | 59.8 | 62.5 | 77.5 | 228.3 | 1,793.0 |
| ML-DSA-44 verification | 28.4 | 29.8 | 44.5 | 198.0 | 1,735.2 |
| Falcon-512 signing | 147.0 | 149.1 | 162.0 | 317.1 | 1,865.6 |
| Falcon-512 verification | 22.1 | 23.6 | 38.3 | 190.9 | 1,735.0 |
| SLH-DSA-SHA2-128f signing | 20,731.9 | 20,743.0 | 20,808.3 | 21,373.4 | 27,188.0 |
| Ed25519 signing | 30.3 | 31.7 | 44.5 | 172.2 | 1,491.2 |

### 10.10 Ledger and IPFS Storage

Storage, not computation, is the real price of post-quantum signatures. On the network configuration of Section 9.6, an `issueCredential` transaction with 10 attributes takes **22.3 KB** with an ML-DSA-44 issuer signature against 7.9 KB with ECDSA, 2.8 times as much, and a block still holds ten of them. Figure 10-9 compares the transaction size for seven issuer signature schemes, with the number that fits in one block, and Table 10-XIII lists all 22 schemes at 1, 5 and 10 attributes. A million credentials need about 22 GB per peer, and every peer stores the whole ledger. SLH-DSA-SHA2-256f would make the transaction 26 times larger and fit only two per block. The encrypted IPFS envelope of a 10-attribute credential is 5,948 bytes with ML-KEM-768, against 3,838 bytes with X25519, and grows by about 340 bytes per attribute.

*Figure 10-9. Ledger bytes of one issueCredential transaction by the issuer's signature scheme (10 attributes)*

**Table 10-XIII. LEDGER BYTES OF ONE ISSUECREDENTIAL TRANSACTION BY SIGNATURE SCHEME**

| Signature scheme | 1 attribute | 5 attributes | 10 attributes | Per block (10 attributes) | GB per million (10 attributes) |
| --- | --- | --- | --- | --- | --- |
| Ed25519 | 6,363 | 6,980 | 7,750 | 10 | 7.8 |
| ECDSA P-256 | 6,524 | 7,141 | 7,911 | 10 | 7.9 |
| RSA-3072-PSS | 9,204 | 9,821 | 10,591 | 10 | 10.6 |
| Falcon-512 | 12,188 | 12,805 | 13,575 | 10 | 13.6 |
| Falcon-padded-512 | 12,232 | 12,849 | 13,619 | 10 | 13.6 |
| Falcon-1024 | 18,239 | 18,857 | 19,627 | 10 | 19.6 |
| Falcon-padded-1024 | 18,275 | 18,893 | 19,663 | 10 | 19.7 |
| ML-DSA-44 | 20,912 | 21,529 | 22,299 | 10 | 22.3 |
| ML-DSA-65 | 27,028 | 27,645 | 28,415 | 10 | 28.4 |
| ML-DSA-87 | 34,861 | 35,478 | 36,248 | 10 | 36.2 |
| SLH-DSA-SHA2-128s | 37,543 | 38,164 | 38,934 | 10 | 38.9 |
| SLH-DSA-SHAKE-128s | 37,543 | 38,164 | 38,934 | 10 | 38.9 |
| SLH-DSA-SHA2-192s | 71,084 | 71,701 | 72,471 | 7 | 72.5 |
| SLH-DSA-SHAKE-192s | 71,084 | 71,701 | 72,471 | 7 | 72.5 |
| SLH-DSA-SHA2-128f | 74,476 | 75,093 | 75,863 | 6 | 75.9 |
| SLH-DSA-SHAKE-128f | 74,476 | 75,093 | 75,863 | 6 | 75.9 |
| SLH-DSA-SHA2-256s | 125,421 | 126,038 | 126,808 | 4 | 126.8 |
| SLH-DSA-SHAKE-256s | 125,421 | 126,038 | 126,808 | 4 | 126.8 |
| SLH-DSA-SHA2-192f | 148,844 | 149,461 | 150,231 | 3 | 150.2 |
| SLH-DSA-SHAKE-192f | 148,844 | 149,461 | 150,231 | 3 | 150.2 |
| SLH-DSA-SHA2-256f | 205,677 | 206,294 | 207,064 | 2 | 207.1 |
| SLH-DSA-SHAKE-256f | 205,677 | 206,294 | 207,064 | 2 | 207.1 |

### 10.11 Fabric Transaction Latency

On the network rebuilt for E14, a transaction sent alone takes 2.05 to 2.07 s whatever its payload, from 1 byte to 50 KB, because the orderer cuts a block for a lone transaction only at its two-second batch timeout; endorsement takes 13 to 23 ms. Bursts of 40 transactions complete in a median of 164 ms with the smallest payload, 195 ms with ML-DSA-44-sized payloads and 705 ms with SLH-DSA-SHA2-256f-sized payloads. Figure 10-10 compares the median commit latency of transactions sent one at a time and in bursts of 40, and Table 10-XIV adds the envelope sizes, endorsement times and read conflicts. Fabric's own latency is therefore two to three orders of magnitude above the cost of the cryptography. In the bursts, 410 of 1,800 transactions failed with read conflicts because both peers of that network shared one CouchDB state database; the deployed network gives each peer its own goleveldb database.

*Figure 10-10. Fabric commit latency against payload size, transactions sent one at a time and in bursts of 40*

**Table 10-XIV. FABRIC TRANSACTION LATENCY AGAINST PAYLOAD SIZE (MS)**

| Payload like | Envelope (B) | Alone: total | Alone: endorsement | Burst: total | Burst: read conflicts |
| --- | --- | --- | --- | --- | --- |
| Smallest transaction | 4,927 | 2,054 | 12.8 | 164 | 30 of 200 |
| ECDSA P-256 | 7,140 | 2,054 | 13.0 | 196 | 90 of 200 |
| Falcon-512 | 12,804 | 2,054 | 13.5 | 179 | 0 of 200 |
| ML-DSA-44 | 21,532 | 2,056 | 13.3 | 195 | 60 of 200 |
| ML-DSA-65 | 27,644 | 2,057 | 13.7 | 247 | 90 of 200 |
| ML-DSA-87 | 35,473 | 2,056 | 14.1 | 253 | 90 of 200 |
| SLH-DSA-SHA2-128s | 38,169 | 2,058 | 14.5 | 240 | 50 of 200 |
| SLH-DSA-SHA2-128f | 75,089 | 2,062 | 16.1 | 336 | 0 of 200 |
| SLH-DSA-SHA2-256f | 206,249 | 2,075 | 22.6 | 705 | 0 of 200 |

### 10.12 Summary of the Benchmarking Results

Table 10-XV answers the six research questions.

**Table 10-XV. ANSWERS TO THE RESEARCH QUESTIONS**

| Question | Answer |
| --- | --- |
| RQ1 primitive cost | ML-DSA-44 signs in 60–206 µs and verifies in 28–79 µs depending on the machine; ML-KEM-768 encapsulates in 14–39 µs. Lattice schemes verify and exchange keys faster than the classical ones on ARM and AVX2 hardware and sign 2–7 times slower; SLH-DSA signing and HQC are roughly 200 to 25,000 times slower than their lattice counterparts. |
| RQ2 lifecycle | The production suite's full lifecycle costs 0.69–1.32 ms, 0.78–1.13 times the classical suite; the signature scheme sets the cost, which grows linearly with the attributes. |
| RQ3 integration | The Go binding adds 0.3 %; QChain's hexadecimal pattern adds 6–13 % to ML-DSA and 14–28 % to ML-KEM. |
| RQ4 hardware and build | Optimised ARM and AVX2 code is 1.8–2.8 times faster than the VM's for ML-DSA and ML-KEM; Docker on the Mac and a native build change nothing; a portable build costs 1.4–2.35 times. |
| RQ5 storage and Fabric | 22.3 KB per ML-DSA-44 credential transaction (7.9 KB with ECDSA), ten per block, about 22 GB per million credentials per peer; Fabric's latency does not depend on payload size up to 50 KB. |
| RQ6 equivalence | Issuer-side and holder-side ML-DSA signing are equivalent in all nine tests. |

## 11. Conclusion and Results

QChain shows that a digital credential system can be post-quantum at both of its layers. At the credential layer, issuers sign canonical commitments with ML-DSA-44, credential contents are encrypted with ML-KEM-768 to keys that never leave the holder's phone, salted SHA3-256 field hashes allow holders to disclose only chosen attributes, and every presentation carries the holder's own ML-DSA-44 signature. At the ledger layer, the team built a Hyperledger Fabric network in which every identity, endorsement, block and node TLS certificate uses ML-DSA-44, by modifying Fabric, Fabric CA and Fabric's cryptographic library, and deployed it with the backend on the university's server and QPortal on Vercel.

The system passed all 123 end-to-end assertions, every unit and smart-contract test and every tamper test, and all 58 identity and node certificates of the network were confirmed to be ML-DSA-44. The benchmarking study of 30 algorithms on three machines showed that QChain's complete cryptographic lifecycle costs 0.69 to 1.32 ms per credential, between 0.78 and 1.13 times its classical equivalent, that hardware and vector instructions matter more than the algorithm choice within the lattice family, and that storage is the real price: an ML-DSA-44 credential transaction is 2.8 times the size of an ECDSA one, while Fabric's own latency is two to three orders of magnitude above the cost of the cryptography.

The project contributes:

- ledger-anchored post-quantum commitments, verified against the ledger rather than a stored hash;
- holder-held post-quantum confidentiality, so that neither the issuer nor any ledger peer can read a credential after issuance;
- quantum-safe selective disclosure without zero-knowledge machinery, and post-quantum holder binding of every presentation;
- no plaintext credential data on the ledger;
- a purely post-quantum Fabric identity layer, without a classical fallback, unlike hybrid designs [41, 42];
- a multi-machine measurement of the post-quantum credential lifecycle through the code of a deployed system.

Together these results give UAE institutions a working, measured path to post-quantum credentials that meets the national migration requirements.

## 12. References

1. C. Gidney, "How to factor 2048 bit RSA integers with less than a million noisy qubits," arXiv preprint arXiv:2505.15917, 2025. [Online]. Available: https://arxiv.org/abs/2505.15917.
2. M. Mosca and M. Piani, "Quantum Threat Timeline Report 2025," Global Risk Institute, Toronto, 2026. [Online]. Available: https://globalriskinstitute.org/publication/quantum-threat-timeline-report-2025b/.
3. National Institute of Standards and Technology, "Module-Lattice-Based Digital Signature Standard (FIPS 204)," U.S. Department of Commerce, Gaithersburg, MD, 2024. [Online]. Available: https://doi.org/10.6028/NIST.FIPS.204.
4. National Institute of Standards and Technology, "Module-Lattice-Based Key-Encapsulation Mechanism Standard (FIPS 203)," U.S. Department of Commerce, Gaithersburg, MD, 2024. [Online]. Available: https://doi.org/10.6028/NIST.FIPS.203.
5. National Institute of Standards and Technology, "Transition to Post-Quantum Cryptography Standards (NIST IR 8547, Initial Public Draft)," NIST, Gaithersburg, MD, 2024. [Online]. Available: https://doi.org/10.6028/NIST.IR.8547.ipd.
6. Aletihad News Center, "UAE approves National Encryption Policy, issues executive regulation," Aletihad, 27 November 2025. [Online]. Available: https://en.aletihad.ae/news/uae/4625341/uae-approves-national-encryption-policy--issues-executive-re. [Accessed 26 September 2026].
7. European Data Protection Board, "Guidelines 02/2025 on processing of personal data through blockchain technologies, Version 2.0," EDPB, Brussels, 2026. [Online]. Available: https://www.edpb.europa.eu/system/files/2026-07/edpb_guidelines_202502_blockchain_v2_en.pdf.
8. IPFS Documentation, "Privacy and encryption," IPFS Docs, 2025. [Online]. Available: https://docs.ipfs.tech/concepts/privacy-and-encryption/. [Accessed 26 September 2026].
9. Google Quantum AI and Collaborators, "Quantum error correction below the surface code threshold," Nature, vol. 638, no. 8052, pp. 920-926, 2025. [Online]. Available: https://doi.org/10.1038/s41586-024-08449-y.
10. IBM Quantum, "IBM lays out clear path to fault-tolerant quantum computing," IBM Quantum Computing Blog, 10 June 2025. [Online]. Available: https://www.ibm.com/quantum/blog/large-scale-ftqc. [Accessed 26 September 2026].
11. National Institute of Standards and Technology, "Stateless Hash-Based Digital Signature Standard (FIPS 205)," U.S. Department of Commerce, Gaithersburg, MD, 2024. [Online]. Available: https://doi.org/10.6028/NIST.FIPS.205.
12. R. Perlner, "FIPS 206 Status Update: FN-DSA (Falcon)," NIST Computer Security Resource Center, 2025. [Online]. Available: https://csrc.nist.gov/csrc/media/presentations/2025/fips-206-fn-dsa-(falcon)/images-media/fips_206-perlner_2.1.pdf. [Accessed 26 September 2026].
13. National Institute of Standards and Technology, "Status Report on the Fourth Round of the NIST Post-Quantum Cryptography Standardization Process (NIST IR 8545)," NIST, Gaithersburg, MD, 2025. [Online]. Available: https://doi.org/10.6028/NIST.IR.8545.
14. National Institute of Standards and Technology, "Considerations for Achieving Cryptographic Agility: Strategies and Practices (NIST CSWP 39)," NIST, Gaithersburg, MD, 2025. [Online]. Available: https://doi.org/10.6028/NIST.CSWP.39.
15. National Cyber Security Centre, "Timelines for migration to post-quantum cryptography," NCSC Guidance, 20 March 2025. [Online]. Available: https://www.ncsc.gov.uk/guidance/pqc-migration-timelines. [Accessed 26 September 2026].
16. NIS Cooperation Group, "A Coordinated Implementation Roadmap for the Transition to Post-Quantum Cryptography," European Commission, Shaping Europe's Digital Future, 23 June 2025. [Online]. Available: https://digital-strategy.ec.europa.eu/en/library/coordinated-implementation-roadmap-transition-post-quantum-cryptography. [Accessed 26 September 2026].
17. Advanced Technology Research Council, "UAE Cyber Security Council partners with ATRC to advance national post-quantum security transition," ATRC News, 11 May 2026. [Online]. Available: https://www.atrc.gov.ae/news/uae-cyber-security-council-partners-atrc-advance-national-post-quantum-security-transition. [Accessed 26 September 2026].
18. T. Mallick, M. Zeldin, M. Cenk and C. Nita-Rotaru, "Quantum Disruption: An SOK of How Post-Quantum Attackers Reshape Blockchain Security and Performance," arXiv preprint arXiv:2512.13333, 2025. [Online]. Available: https://arxiv.org/abs/2512.13333.
19. J. Gomes, S. Khan and D. Svetinovic, "Fortifying the Blockchain: A Systematic Review and Classification of Post-Quantum Consensus Solutions for Enhanced Security and Resilience," IEEE Access, vol. 11, pp. 74088-74100, 2023. [Online]. Available: https://doi.org/10.1109/ACCESS.2023.3296559.
20. Y. Wang and E. S. Ismail, "A Review on the Advances, Applications, and Future Prospects of Post-Quantum Cryptography in Blockchain and IoT," IEEE Access, vol. 13, pp. 112962-112977, 2025. [Online]. Available: https://doi.org/10.1109/ACCESS.2025.3584473.
21. S. Al-Janabi, "Post-Quantum Blockchain: Challenges and Opportunities," arXiv preprint arXiv:2508.17071, 2025. [Online]. Available: https://arxiv.org/abs/2508.17071.
22. R. Campbell, "Hybrid Post-Quantum Signatures for Bitcoin and Ethereum: A Protocol-Level Integration Strategy," The Journal of The British Blockchain Association, vol. 9, no. 1, pp. 1-13, 2026. [Online]. Available: https://doi.org/10.31585/jbba-9-1-(2)2026.
23. M. Vidaković and K. Miličević, "Performance and Applicability of Post-Quantum Digital Signature Algorithms in Resource-Constrained Environments," Algorithms, vol. 16, no. 11, p. 518, 2023. [Online]. Available: https://doi.org/10.3390/a16110518.
24. E. D. Demir, B. Bilgin and M. C. Onbasli, "Performance Analysis and Industry Deployment of Post-Quantum Cryptography Algorithms," arXiv preprint arXiv:2503.12952, 2025. [Online]. Available: https://arxiv.org/abs/2503.12952.
25. W. Beullens, "Breaking Rainbow Takes a Weekend on a Laptop," in Advances in Cryptology - CRYPTO 2022, Lecture Notes in Computer Science, vol. 13508, Cham, 2022. [Online]. Available: https://doi.org/10.1007/978-3-031-15979-4_16.
26. P. Thanalakshmi, A. Rishikhesh, J. M. Marceline, G. P. Joshi and W. Cho, "A Quantum-Resistant Blockchain System: A Comparative Analysis," Mathematics, vol. 11, no. 18, p. 3947, 2023. [Online]. Available: https://doi.org/10.3390/math11183947.
27. The Linux Foundation, "Announcing the Post-Quantum Cryptography Alliance (PQCA)," Linux Foundation Press Release, 6 February 2024. [Online]. Available: https://www.linuxfoundation.org/press/announcing-the-post-quantum-cryptography-alliance-pqca. [Accessed 26 September 2026].
28. IPFS Documentation, "Persistence, permanence, and pinning," IPFS Docs, 2025. [Online]. Available: https://docs.ipfs.tech/concepts/persistence/. [Accessed 26 September 2026].
29. S. Khaleelullah, S. T. Vangapalli, M. Gaddam, V. S. Hanumakonda and U. K. G. Gangapuram, "Verification of Academic Records Using Hyperledger Fabric and IPFS," in 2023 3rd International Conference on Pervasive Computing and Social Networking (ICPCSN), Salem, India, 2023. [Online]. Available: https://doi.org/10.1109/ICPCSN58827.2023.00040.
30. R. A. Jaafar and S. N. Alsaad, "Enhancing Educational Certificate Verification With Blockchain and IPFS: A Decentralized Approach Using Hyperledger Fabric," TEM Journal, vol. 12, no. 4, pp. 2385-2395, 2023. [Online]. Available: https://doi.org/10.18421/TEM124-51.
31. R. Barnes, K. Bhargavan, B. Lipp and C. A. Wood, "Hybrid Public Key Encryption (RFC 9180)," Internet Research Task Force (IRTF), 2022. [Online]. Available: https://doi.org/10.17487/RFC9180.
32. M. Barbosa, D. Connolly, J. D. Duarte, A. Kaiser, P. Schwabe, K. Varner and B. Westerbaan, "X-Wing: The Hybrid KEM You've Been Looking For," IACR Communications in Cryptology, vol. 1, no. 1, 2024. [Online]. Available: https://doi.org/10.62056/a3qj89n4e.
33. World Wide Web Consortium, "Verifiable Credentials Data Model v2.0," W3C Recommendation, 15 May 2025. [Online]. Available: https://www.w3.org/TR/vc-data-model-2.0/. [Accessed 26 September 2026].
34. D. Fett, K. Yasuda and B. Campbell, "Selective Disclosure for JSON Web Tokens (RFC 9901)," Internet Engineering Task Force (IETF), 2025. [Online]. Available: https://www.rfc-editor.org/rfc/rfc9901.
35. International Organization for Standardization, "ISO/IEC 18013-5:2021 Personal identification - ISO-compliant driving licence - Part 5: Mobile driving licence (mDL) application," ISO/IEC, Geneva, 2021. [Online]. Available: https://www.iso.org/standard/69084.html.
36. T. Wood, K. Thomas, M. Dean, S. Kannan and R. Learney, "Towards Post-Quantum Verifiable Credentials," in Proceedings of the 19th International Conference on Availability, Reliability and Security (ARES 2024), Vienna, 2024. [Online]. Available: https://doi.org/10.1145/3664476.3669932.
37. J. Bootle, V. Lyubashevsky, N. K. Nguyen and A. Sorniotti, "A Framework for Practical Anonymous Credentials from Lattices," in Advances in Cryptology - CRYPTO 2023, Lecture Notes in Computer Science, Cham, 2023. [Online]. Available: https://doi.org/10.1007/978-3-031-38545-2_13.
38. D. Margaria, A. Pino, A. Vesco, G. D’Alconzo, A. J. Di Scala, E. Guglielmino and C. Sanna, "Implementation of a Post-Quantum Anonymous Verifiable Credential Framework," in 2025 IEEE Symposium on Computers and Communications (ISCC), 2025. [Online]. Available: https://doi.org/10.1109/iscc65549.2025.11325782.
39. A. Pino, D. Margaria and A. Vesco, "On PQ/T Hybrid Verifiable Credentials and Presentations to Build Trust in IoT Systems," in 2024 9th International Conference on Smart and Sustainable Technologies (SpliTech), 2024. [Online]. Available: https://doi.org/10.23919/splitech61897.2024.10612295.
40. A. Solavagione and A. Vesco, "Transition of Self-Sovereign Identity to Post-Quantum Cryptography," in 2025 International Conference on Quantum Communications, Networking, and Computing (QCNC), 2025. [Online]. Available: https://doi.org/10.1109/qcnc64685.2025.00035.
41. A. Holcomb, G. Pereira, B. Das and M. Mosca, "PQFabric: A Permissioned Blockchain Secure from Both Classical and Quantum Attacks," in 2021 IEEE International Conference on Blockchain and Cryptocurrency (ICBC), Sydney, Australia, 2021. [Online]. Available: https://doi.org/10.1109/ICBC51069.2021.9461070.
42. S. Abbas, A. Sultana and G. Kaddoum, "Quantum-Safe Blockchain in Hyperledger Fabric," IEEE Networking Letters, vol. 7, no. 1, pp. 61-65, 2025. [Online]. Available: https://doi.org/10.1109/lnet.2024.3522966.
43. V. K. Kasula, S. B. Rakki and R. Banoth, "Enhancing Hyperledger Fabric Security with Lightweight Post-Quantum Cryptography and National Cryptographic Algorithms," in 2025 37th Conference of Open Innovations Association (FRUCT), Narvik, Norway, 2025. [Online]. Available: https://doi.org/10.23919/FRUCT65909.2025.11008110.
44. S. Grebnev, Y. Melnichuk, I. Poltavskaya, O. Turchenko, A. Kurochkin and A. Polubelov, "Analysis and challenges of implementing post-quantum cryptographic algorithms in Ethereum and Hyperledger Fabric networks," Journal of Computer Virology and Hacking Techniques, vol. 22, no. 1, p. 3, 2025. [Online]. Available: https://doi.org/10.1007/s11416-025-00585-x.
45. N. L. Ouadrassi, M. Said Boukhryss, A. Azmani and K. J. Hassani, "Toward Quantum-Safe Permissioned Blockchains: Evaluating Hybrid ML-DSA-65/ECDSA Signatures on Hyperledger Fabric," IEEE Access, vol. 14, pp. 146672-146703, 2026. [Online]. Available: https://doi.org/10.1109/access.2026.3735938.
46. M. Kuzlu, M. Pipattanasomporn, L. Gurses and S. Rahman, "Performance Analysis of a Hyperledger Fabric Blockchain Framework: Throughput, Latency and Scalability," in 2019 IEEE International Conference on Blockchain (Blockchain), 2019. [Online]. Available: https://doi.org/10.1109/blockchain.2019.00003.
47. T. Zhukabayeva, A. u. Rehman, N. Tariq and E. Benkhelifa, "Hyperledger Fabric Based Post Quantum Cryptography Healthcare Application Using Discrete Event Simulation," IEEE Access, 2024. [Online]. Available: https://doi.org/10.1109/access.2024.3513153.
48. Hyperledger Foundation, "What's new in Hyperledger Fabric v3.0," Hyperledger Fabric Documentation, 2024. [Online]. Available: https://hyperledger-fabric.readthedocs.io/en/release-3.0/whatsnew.html. [Accessed 26 September 2026].
49. J. Massimo, P. Kampanakis, S. Turner and B. Westerbaan, "Internet X.509 Public Key Infrastructure - Algorithm Identifiers for the Module-Lattice-Based Digital Signature Algorithm (ML-DSA) (RFC 9881)," Internet Engineering Task Force (IETF), 2025. [Online]. Available: https://www.rfc-editor.org/rfc/rfc9881.
50. The Go Authors, "Go 1.27 Release Notes," The Go Programming Language, August 2026. [Online]. Available: https://go.dev/doc/go1.27. [Accessed 26 September 2026].
51. D. Commey, B. Appiah, G. S. Klogo, W. Bagyl-Bac, J. D. Gadze, Y. Alsenani and G. V. Crosby, "Performance Analysis and Deployment Considerations of Post-Quantum Cryptography for Consumer Electronics," arXiv preprint arXiv:2505.02239, 2025. [Online]. Available: https://arxiv.org/abs/2505.02239.
52. J. Algar-Fernandez, A. Villacís-Vanegas, Y. Amaro-Aular and M. Cano, "Benchmarking Post-Quantum Signatures and KEMs on General-Purpose CPUs Using a TCP Client–Server Testbed," Computers, vol. 15, no. 2, p. 116, 2026. [Online]. Available: https://doi.org/10.3390/computers15020116.
53. D. Lucaciu and D. E. Popescu, "Cross-Platform Performance and Security Evaluation of Post-Quantum Cryptographic Algorithms on Resource-Constrained Devices," Applied Sciences, vol. 16, no. 12, p. 5781, 2026. [Online]. Available: https://doi.org/10.3390/app16125781.
54. Y. Oleksiichuk, "Empirical Performance Evaluation of NIST Post-Quantum Cryptographic Algorithms ML-KEM, ML-DSA, and SLH-DSA across JDK Versions," Cybersecurity: Education, Science, Technique, vol. 1, no. 33, p. 165, 2026. [Online]. Available: https://doi.org/10.28925/2663-4023.2026.33.1210.
55. R. Chhetri, S. Dhakal and A. Gautam, "Benchmarking NIST-Standardised ML-KEM and ML-DSA on ARM Cortex-M0+: Latency, Rejection-Sampling Variance, and Memory on the RP2040," arXiv preprint arXiv:2603.19340, 2026. [Online]. Available: https://arxiv.org/abs/2603.19340.
56. N. Puch, C. H. Ta and M. Beckel, "Benchmarking Post-Quantum Cryptography in Lightweight Virtualization Environments on Embedded Hardware," arXiv preprint arXiv:2609.23902, 2026. [Online]. Available: https://arxiv.org/abs/2609.23902.
57. O. Gillot, W. J. Buchanan and M. G. Tehrani, "Energy Consumption of Post-Quantum Cryptography on Constrained and General-Purpose Architectures," Cryptography, vol. 10, no. 4, p. 55, 2026. [Online]. Available: https://doi.org/10.3390/cryptography10040055.
58. J. Lopez, V. Cadena and M. S. Rahman, "Evaluating Post-Quantum Cryptographic Algorithms on Resource-Constrained Devices," arXiv preprint arXiv:2507.08312, 2025. [Online]. Available: https://arxiv.org/abs/2507.08312.
59. S. Hoque, A. Aydeger, E. Zeydan and M. Liyanage, "Analysis of Post-Quantum Cryptography in User Equipment in 5G and Beyond," arXiv preprint arXiv:2507.17074, 2025. [Online]. Available: https://arxiv.org/abs/2507.17074.
60. S. Hoque and A. Aydeger, "Energy-Aware System-Level Evaluation of Post-Quantum TLS on Embedded User Equipment over a Disaggregated 5G Network," arXiv preprint arXiv:2607.03988, 2026. [Online]. Available: https://arxiv.org/abs/2607.03988.
61. A. Kudaloor and A. Aijaz, "Toward Quantum-Safe 6G: Experimental Evaluation of Post-Quantum Cryptography Techniques," arXiv preprint arXiv:2605.06881, 2026. [Online]. Available: https://arxiv.org/abs/2605.06881.
62. J. A. Montenegro, R. Rios and J. Lopez-Cerezo, "A performance evaluation framework for post-quantum TLS," Future Generation Computer Systems, vol. 175, p. 108062, 2026. [Online]. Available: https://doi.org/10.1016/j.future.2025.108062.
63. M. J. Kannwischer, "Keynote: mlkem-native & mldsa-native: Open-Source High-Speed High-Assurance PQC," in Proceedings of the 2025 Cloud Computing Security Workshop (CCSW), 2025. [Online]. Available: https://doi.org/10.1145/3733812.3765539.
64. Open Quantum Safe Project, "liboqs 0.16.0: C library for prototyping and experimenting with quantum-resistant cryptography," GitHub, 9 July 2026. [Online]. Available: https://github.com/open-quantum-safe/liboqs/releases/tag/0.16.0. [Accessed 10 October 2026].
65. A. G. Schemitt, H. F. d. Silva, R. C. Lunardi, D. Kreutz, R. B. Mansilha and A. F. Zorzo, "Assessing the Impact of Post-Quantum Digital Signature Algorithms on Blockchains," arXiv preprint arXiv:2510.09271, 2025. [Online]. Available: https://arxiv.org/abs/2510.09271.
66. P. Juaristi, I. Agudo, R. Rios and L. Ricci, "Benchmarking Post-quantum Cryptography in Ethereum-Based Blockchains," in Lecture Notes in Computer Science, 2025. [Online]. Available: https://doi.org/10.1007/978-3-031-82349-7_23.
67. S. A. Holmes, "Impact of Post-Quantum Signatures on Blockchain and DLT Systems," in Distributed Ledger Technology Workshop (DLT 2023), CEUR Workshop Proceedings, Bologna, Italy, 2023.
68. B. Saviozzi, R. Singh, A. Lele and H. Tewari, "Building Quantum-Resilient Blockchains with ML-DSA Digital Signature Integration," in 2026 4th International Conference on Advanced Network Technologies and Applications (APAN), 2026. [Online]. Available: https://doi.org/10.1109/apan70967.2026.11709608.
69. M. Arakawa, "Post-Quantum Cryptography for Verifiable Credentials: Comprehensive Performance Evaluation Including Selective Disclosure," TechRxiv preprint, 2025. [Online]. Available: https://doi.org/10.36227/techrxiv.176118722.22701360/v1.
70. M. Kirubakaran and V. Vijayarajan, "PQ-FundusChain: a postquantum blockchain framework for secure and privacy-preserving retinal fundus image sharing in teleophthalmology," Frontiers in Digital Health, vol. 8, p. 1933562, 2026. [Online]. Available: https://doi.org/10.3389/fdgth.2026.1933562.
71. A. Olushola and S. P. Meenakshi, "Design and implementation of an authenticated post-quantum session protocol using ML-KEM (Kyber), ML-DSA (Dilithium), and AES-256-GCM," Frontiers in Physics, vol. 13, p. 1723966, 2026. [Online]. Available: https://doi.org/10.3389/fphy.2025.1723966.
72. L. F. Prikutse, R. E. Turkson, A. Yussiff, A. Yussiff and M. G. Asante-Mensah, "A Commitment-Based Hybrid Post-Quantum Cryptographic Model for Multi-File Cloud Storage," arXiv preprint arXiv:2608.13138, 2026. [Online]. Available: https://arxiv.org/abs/2608.13138.
73. G. Fabrizio, A. Sperotto and R. Van Rijswijk-Deij, "Measuring The Impact of Post-Quantum Cryptography on Complex Applications: A Case Study on Federated Identity Management," in 2025 9th Network Traffic Measurement and Analysis Conference (TMA), 2025. [Online]. Available: https://doi.org/10.23919/tma66427.2025.11097002.
74. Y. Kotukh, M. Korobchynskyi, M. Rudenko, A. Honcharuk, I. Kiris and V. Voitko, "Type Safe OCaml Bindings for the Open Quantum Safe Post-Quantum Cryptography Library," in 2026 16th International Conference on Advanced Computer Information Technologies (ACIT), 2026. [Online]. Available: https://doi.org/10.1109/acit69831.2026.11699949.
75. A. Shaw, "quantum-safe: Bridging the Post-Quantum Production Gap with a Hybrid-by-Default Python Cryptography Library," arXiv preprint arXiv:2605.17061, 2026. [Online]. Available: https://arxiv.org/abs/2605.17061.
76. C. M. Frost, "A Reproducibility Protocol for Cross-Implementation Evaluation of Post-Quantum ACVP Test Vectors," arXiv preprint arXiv:2608.13784, 2026. [Online]. Available: https://arxiv.org/abs/2608.13784.

---

## 13. Appendix

### 13.1 Gantt Chart

Figure 13-1 shows the project schedule with the progress of every task.

*Figure 13-1. Project Gantt chart, Fall 2026*

### 13.2 Source Code Repositories

The source code of QChain is kept in the team's GitHub organisation; Table 13-I lists the public repositories.

**Table 13-I. QCHAIN SOURCE CODE REPOSITORIES**

| Repository | Content | Address |
| --- | --- | --- |
| qchain-backend | The QChain API: post-quantum protocols, ledger, IPFS and MySQL access, continuous deployment | [github.com/teamqchain/qchain-backend](https://github.com/teamqchain/qchain-backend) |
| qchain-network | The post-quantum Fabric network: configuration, scripts and chaincode | [github.com/teamqchain/qchain-network](https://github.com/teamqchain/qchain-network) |
| qchain-fabric | Hyperledger Fabric with ML-DSA-44 identities | [github.com/teamqchain/qchain-fabric](https://github.com/teamqchain/qchain-fabric) |
| qchain-fabric-ca | Fabric CA issuing ML-DSA-44 certificates | [github.com/teamqchain/qchain-fabric-ca](https://github.com/teamqchain/qchain-fabric-ca) |
| qchain-fabric-lib-go | The post-quantum cryptographic provider for Fabric | [github.com/teamqchain/qchain-fabric-lib-go](https://github.com/teamqchain/qchain-fabric-lib-go) |
| qchain-portal | QPortal web portal | [github.com/teamqchain/qchain-portal](https://github.com/teamqchain/qchain-portal) |
| qchain-wallet | QWallet mobile wallet | [github.com/teamqchain/qchain-wallet](https://github.com/teamqchain/qchain-wallet) |
| qchain-kpi | Benchmarking harness, data, analysis, figures and tables | [github.com/teamqchain/qchain-kpi](https://github.com/teamqchain/qchain-kpi) |
| qchain-docs | Project documentation and reports | [github.com/teamqchain/qchain-docs](https://github.com/teamqchain/qchain-docs) |
| teamqchain.github.io | Project website | [github.com/teamqchain/teamqchain.github.io](https://github.com/teamqchain/teamqchain.github.io) |
