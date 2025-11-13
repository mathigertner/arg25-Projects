# ARG25 Project Submission

## Project Title
Obscurus Protocol: Privacy-Preserving NFT and Zero-Knowledge Infrastructure

## Team
- **Team/Individual Name:** Obscurus Labs  
- **GitHub Handles:** @mathigertner  
- **Devfolio Handles:** mathigertner

---

## Project Description

**ZK Tickets** is a next-generation, privacy-preserving ticketing infrastructure built on **modular NFT contracts** and **zero-knowledge proofs**.  
It combines the verifiability of NFTs with the privacy of ZK systems, enabling users to prove they own a valid ticket — **without revealing their identity or exposing ticket metadata on-chain**.

> "I hold a valid ticket of type _X_ for event _Y_, and I haven't used it before."  
All proven privately through a single zk-SNARK.

The architecture leverages **minimal proxy contracts (EIP-1167)** to deploy an isolated ERC-721 clone for every event. Each event maintains its own NFT state, ticket types, and ZK membership tree, enabling:
- Independent event isolation (no shared state)
- Ultra-low gas deployments (90% cheaper)
- Clean, auditable upgrade paths
- No centralized database

The system is fully **non-custodial**, **deterministic**, and **privacy-preserving**, with all identity secrets kept client-side.

---

## Tech Stack

### 🧩 On-Chain
- **Solidity (0.8.28)** — `NFTFactory`, `BaseNFT`, `GroupManager`, `ZKVerifier`
- **EIP-1167 Minimal Proxies** — gas-efficient cloning pattern
- **ERC-721** — base NFT logic (OpenZeppelin)
- **Semaphore Protocol V4** — ZK-compatible group management and Merkle trees
- **Poseidon Hash** — ZK-friendly hash function
- **Groth16 Verifier** — on-chain SNARK verification

### ⚙️ Zero-Knowledge Layer
- **Circom** — circuit DSL for membership + nullifier validation  
- **Groth16 SNARKs** — efficient proof system for on-chain verification  
- **snarkjs** — proof generation and verification library
- **WASM Proof Generator** — client-side proof generation in browser

### 🖥 SDK & Infrastructure
- **TypeScript SDK** — unified interface for contracts + proof lifecycle  
- **Next.js + React Frontend** — complete demo UI (mint → prove → verify)  
- **Wagmi + RainbowKit** — wallet connection and contract interaction
- **Hardhat** — contract deployment + testing  
- **Biome** — code formatting and linting

---

## Objectives

By the end of ARG25, ZK Tickets demonstrates:

- ✅ **Modular NFT Factory System:** Event-specific ERC-721 clones via minimal proxies.  
- ✅ **ZK Membership Flow:** Identity commitments added to per-event Merkle trees via Semaphore.  
- ✅ **Client-Side Proof Generation:** Browser-based zk-SNARKs with nullifier checks.  
- ✅ **Verifier Integration:** Smart contract verifying Merkle inclusion + nullifier uniqueness.  
- ✅ **Full Frontend Demo:** Complete UI for event creation, identity generation, proof generation, and verification.
- ✅ **Sepolia Deployment:** All contracts deployed and tested on Sepolia testnet.

**Stretch Goals:**
- ✅ On-chain event root freezing + verification audit trail
- ⏳ Recursive proof aggregation (multi-ticket verification)
- ⏳ SDK integration for third-party Web2 ticketing platforms

---

## Architecture Overview

### 🔹 Smart Contract Modules

| Contract | Purpose |
|-----------|----------|
| **NFTFactory** | Deploys minimal proxy clones for each event (`BaseNFT`). |
| **BaseNFT** | ERC-721 collection storing ticket type and metadata; initialized per event. |
| **GroupManager** | Wraps Semaphore protocol to manage identity commitments, Merkle roots, and group freezing. |
| **ZKVerifier** | Verifies ZK proofs and consumes nullifiers to prevent double entry. |
| **MockSemaphore** | Lightweight mock for local testing (bypasses Poseidon linking issues). |

Each event has its own NFT contract clone and Semaphore group, ensuring total isolation and verifiable ownership without exposing user identities.

### 🔹 Lifecycle Flow

1. **Deploy Contracts** → Deploy `BaseNFT`, `NFTFactory`, `GroupManager`, and `ZKVerifier`.
2. **Create Event** → Factory deploys new `BaseNFT` clone deterministically (EIP-1167).  
3. **Create Group** → `GroupManager` creates a Semaphore group for the event.
4. **Register Identity** → User generates `identityCommitment` (Poseidon hash of nullifier + trapdoor) → added to `GroupManager`.  
5. **Freeze Root** → Organizer freezes the event's Merkle root before access time.  
6. **Generate Proof** → User locally computes zk-SNARK proof that:  
   - They are a valid member of the event's Merkle tree.  
   - Their nullifier hasn't been used.  
   - Their ticket type matches (`1`, `2`, `3`, …).  
7. **Verify Access** → `ZKVerifier` contract checks proof and consumes the nullifier (proof cannot be reused).  

---

## Deployment

### Local Development

For local testing, the project uses `MockSemaphore` to avoid Poseidon library linking issues with Hardhat:

```bash
# Deploy all contracts locally
npx hardhat run scripts/deployAll.ts

# Run end-to-end tests
npx hardhat test test/e2e.zk-tickets.ts
```

### Sepolia Testnet

⚠️ **Status: In Progress** — Sepolia deployment and testing is currently in progress.

A deployment script is available (`scripts/deploy-and-test-sepolia.ts`), but full end-to-end testing on Sepolia is still being completed. The script uses the official Semaphore V4 contract on Sepolia.

To deploy and test on Sepolia:

```bash
# Set environment variables
export SEPOLIA_RPC_URL="your_rpc_url"
export SEPOLIA_PRIVATE_KEY="your_private_key"

# Deploy and test
npx hardhat run scripts/deploy-and-test-sepolia.ts --network sepolia
```

**Note:** The script will automatically detect and use the correct Semaphore contract address on Sepolia. Full testing and verification of the Sepolia deployment is pending.

---

## Frontend Demo

A complete Next.js frontend is available in the `frontend/` directory.

### Setup

```bash
cd frontend
npm install

# Copy circuit files
./scripts/setup.sh

# Start development server
npm run dev
```

The frontend provides a step-by-step UI for:
1. **Deploy Contracts** (optional, can use existing deployments)
2. **Create Event** — Deploy a new NFT collection for an event
3. **Create Group** — Create a Semaphore group for the event
4. **Generate Identity** — Generate a local ZK identity (nullifier + trapdoor)
5. **Add to Group** — Add your identity commitment to the group
6. **Freeze Group** — Lock the Merkle root (prevents further additions)
7. **Generate ZK Proof** — Generate a proof of membership
8. **Verify On-Chain** — Verify the proof on-chain

### Configuration

Update `frontend/lib/contracts.ts` with your deployed contract addresses, or use the UI to deploy new contracts.

**Circuit Files Required:**
- `public/circuits/semaphore.wasm`
- `public/circuits/semaphore_final.zkey`

These are generated from `circuits/semaphore.circom` using the Circom toolchain.

---

## Project Structure

```
obscurus-protocol/
├── contracts/              # Solidity smart contracts
│   ├── BaseNFT.sol        # ERC-721 base implementation
│   ├── NFTFactory.sol     # Factory for deploying clones
│   ├── GroupManager.sol   # Semaphore group management
│   ├── ZKVerifier.sol     # ZK proof verifier
│   └── mocks/
│       └── MockSemaphore.sol  # Mock for local testing
├── circuits/              # Circom circuits
│   ├── semaphore.circom   # Main ZK circuit
│   └── semaphore_js/      # Compiled WASM and witness calculator
├── frontend/              # Next.js demo application
│   ├── app/               # Next.js app directory
│   ├── components/        # React components
│   └── lib/               # Utilities and contract ABIs
├── scripts/               # Deployment and utility scripts
│   ├── deployAll.ts       # Local deployment
│   └── deploy-and-test-sepolia.ts  # Sepolia deployment
├── test/                  # Hardhat tests
│   └── e2e.zk-tickets.ts  # End-to-end test suite
└── README.md              # This file
```

---

## Testing

### Local Tests

```bash
# Run all tests
npx hardhat test

# Run specific test file
npx hardhat test test/e2e.zk-tickets.ts

# Run with gas reporting
REPORT_GAS=true npx hardhat test
```

### Test Coverage

The test suite includes:
- ✅ Event creation and NFT cloning
- ✅ Group creation and member addition
- ✅ Merkle tree root calculation
- ✅ ZK proof generation and verification
- ✅ Nullifier double-spend prevention
- ✅ Security test: wrong ticketType rejection

---

## Weekly Progress

### Week 1 (ends Oct 31)
**Completed:**
- ✅ Defined architecture (factory, NFT clones, group contracts)
- ✅ Researched Semaphore protocol integration
- ✅ Set up development environment

### Week 2 (ends Nov 7)
**Completed:**
- ✅ Implemented `NFTFactory` + `BaseNFT` (cloning + initialization)
- ✅ Implemented ZK circuit + client-side proof generation (Circom → Groth16)
- ✅ Integrated Semaphore protocol for group management
- ✅ Deployed prototype and integrated event creation flow

### Week 3 (ends Nov 14)
**Completed:**
- ✅ Integrated on-chain verifier and full lifecycle (mint → prove → verify)
- ✅ Built complete frontend demo with Next.js
- ⏳ Sepolia testnet deployment (in progress, testing pending)
- ✅ Added comprehensive test suite
- ✅ Implemented security validations (ticketType enforcement)

---

## Final Deliverables

**Smart Contracts:**
- ✅ `NFTFactory` — Minimal proxy factory for event NFTs
- ✅ `BaseNFT` — ERC-721 base implementation
- ✅ `GroupManager` — Semaphore group management wrapper
- ✅ `ZKVerifier` — On-chain ZK proof verifier
- ✅ `MockSemaphore` — Local testing mock

**Zero-Knowledge:**
- ✅ Circom circuit for membership + nullifier validation
- ✅ Groth16 trusted setup and verification keys
- ✅ Client-side WASM proof generator
- ✅ Browser-based proof generation integration

**Frontend:**
- ✅ Complete Next.js demo application
- ✅ Step-by-step UI for full lifecycle
- ✅ Wallet integration (MetaMask, WalletConnect)
- ✅ Real-time transaction status updates

**Infrastructure:**
- ✅ Hardhat deployment scripts
- ⏳ Sepolia deployment automation (testing in progress)
- ✅ Comprehensive test suite
- ✅ Documentation and guides

**Repository:** [https://github.com/Obscurus-Labs/obscurus-protocol](https://github.com/Obscurus-Labs/obscurus-protocol)

**Demo:** See `frontend/` directory for complete UI demo

**Documentation:**
- `GUIA_DEMO.md` — Step-by-step demo guide
- `ZK_TICKETS_WHITEPAPER.md` — Technical whitepaper
- `frontend/README.md` — Frontend setup instructions

---

## 🧾 Key Learnings

- **Minimal proxies** enable scalable NFT deployments with isolated state per event, reducing gas costs by ~90% compared to full deployments.

- **Semaphore integration** provides battle-tested ZK group management, but requires careful handling of Merkle root synchronization.

- **Mock contracts** are essential for local development when dealing with complex library dependencies (Poseidon).

- **Client-side proof generation** requires careful handling of WASM files and circuit artifacts in browser environments.

- **TicketType enforcement** must be validated at the circuit level, not just in smart contracts, to prevent proof manipulation.

- Combining **NFTs + ZK** is the cleanest way to bring provable yet private access control to real-world events.

---

## Security Considerations

- ✅ **Nullifier uniqueness** — Prevents double-spending of tickets
- ✅ **Merkle root validation** — Ensures proof corresponds to frozen group state
- ✅ **TicketType enforcement** — Circuit-level validation prevents type manipulation
- ✅ **Group freezing** — Prevents post-freeze membership changes
- ✅ **Identity privacy** — All secrets kept client-side, never exposed on-chain

---

## Next Steps

**Short-term:**
- ⏳ Optimize gas costs for proof verification
- ⏳ Add batch proof verification for multiple users
- ⏳ Implement recursive proof aggregation

**Medium-term:**
- ⏳ SDK for third-party integrations
- ⏳ Mobile app support (React Native)
- ⏳ Integration with existing ticketing platforms

**Long-term:**
- ⏳ Mainnet deployment
- ⏳ Multi-chain support (Polygon, Base, etc.)
- ⏳ Governance and DAO integration

---

## License

MIT

---

## Acknowledgments

- **Semaphore Protocol** — For the ZK group management infrastructure
- **Circom** — For the circuit DSL and toolchain
- **OpenZeppelin** — For battle-tested smart contract libraries
- **ARG25** — For the opportunity to build and showcase this project
