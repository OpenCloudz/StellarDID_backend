# StellarDID — Backend

> API server powering DID resolution, credential indexing, and IPFS document management for StellarDID.

This package contains the Node.js/Express backend that sits between the frontend and the Stellar network. It handles IPFS document storage, credential indexing, DID resolution, and exposes a clean REST API so the frontend never talks directly to Soroban RPC or IPFS.

---

## Overview

The backend handles three responsibilities the frontend shouldn't own directly:

- **DID Resolution** — fetches a DID's document hash from the Soroban registry, retrieves the document from IPFS, verifies the hash matches, and returns the resolved DID Document
- **IPFS Management** — uploads DID Documents and Verifiable Credentials to IPFS via Pinata, returns content-addressed CIDs, and handles retrieval
- **Credential Indexing** — listens for Soroban contract events (`credential_issued`, `credential_revoked`) and maintains a queryable index so the frontend can fetch a subject's credentials without scanning the entire ledger

---

## API Reference

### DID

```
GET  /api/did/:did              Resolve a DID to its DID Document
POST /api/did/register          Upload a DID Document to IPFS, return hash
PUT  /api/did/:did              Upload an updated DID Document, return new hash
```

### Credentials

```
POST /api/credentials           Upload a Verifiable Credential to IPFS, return hash
GET  /api/credentials/:subject  List all credentials for a subject address
GET  /api/credentials/:id       Fetch a single credential by ID
```

### Verification

```
GET  /api/verify/:subject/:type Check if a subject holds a valid credential of a given type
```

### Health

```
GET  /api/health                Service health check
```

---

## Project Structure

```
backend/
├── src/
│   ├── index.ts                # Express app entry point
│   ├── routes/
│   │   ├── did.ts              # DID registration and resolution routes
│   │   ├── credentials.ts      # Credential upload and indexing routes
│   │   └── verify.ts           # Verification route
│   ├── services/
│   │   ├── ipfs.ts             # Pinata upload/fetch wrapper
│   │   ├── stellar.ts          # Soroban RPC calls (resolve, verify)
│   │   └── indexer.ts          # Contract event listener + credential index
│   ├── middleware/
│   │   ├── validate.ts         # Request validation (zod)
│   │   └── error.ts            # Centralised error handler
│   └── types/
│       └── index.ts            # Shared TypeScript types
├── .env.example
├── package.json
└── tsconfig.json
```

---

## Prerequisites

- Node.js 18+
- A [Pinata](https://pinata.cloud) account (free tier is sufficient for development)
- A deployed StellarDID registry contract ID (see [contracts/README.md](../contracts/README.md))

---

## Local Setup

```bash
# Clone the repo
git clone https://github.com/YOUR_USERNAME/stellardid_backend.git
cd stellardid_backend

# Install dependencies
npm install

# Copy env file and fill in values
cp .env.example .env
```

### Environment Variables

```bash
# Stellar
STELLAR_NETWORK=testnet                        # testnet | mainnet
TESTNET_RPC_URL=https://soroban-testnet.stellar.org
MAINNET_RPC_URL=https://your-mainnet-rpc-url
REGISTRY_CONTRACT_ID=CXXX...                  # deployed contract ID

# IPFS / Pinata
PINATA_API_KEY=your_pinata_api_key
PINATA_SECRET_KEY=your_pinata_secret_key
PINATA_GATEWAY=https://gateway.pinata.cloud

# Server
PORT=4000
```

```bash
# Start dev server with hot reload
npm run dev

# Build for production
npm run build
npm start
```

---

## Key Services

### `services/ipfs.ts`

Wraps Pinata's API for DID Document and Verifiable Credential storage.

```ts
// Upload a DID Document — returns IPFS CID
uploadDocument(doc: DIDDocument): Promise<string>

// Upload a Verifiable Credential — returns IPFS CID
uploadCredential(vc: VerifiableCredential): Promise<string>

// Fetch a document by CID
fetchDocument(cid: string): Promise<DIDDocument>

// Fetch a credential by CID
fetchCredential(cid: string): Promise<VerifiableCredential>
```

### `services/stellar.ts`

Wraps Soroban RPC calls to the registry contract.

```ts
// Resolve a DID → document hash (BytesN<32>)
resolveOnChain(did: string): Promise<string | null>

// Check credential validity via contract
verifyOnChain(subject: string, credentialType: string): Promise<boolean>
```

### `services/indexer.ts`

Polls for Soroban contract events and maintains an in-memory (v0.1) or database-backed (v0.2) credential index.

```ts
// Start listening for contract events
startIndexer(): void

// Query indexed credentials for a subject
getCredentials(subject: string): Promise<CredentialRecord[]>
```

---

## Contributing

Contributions to the backend are welcome. Here's how to get started.

### Finding Work

Browse [open issues](../../issues) and filter by the `backend` label. Issues labelled `good first issue` are a good starting point — drop a comment before picking one up.

### Making Changes

Create a branch off `main`:

```bash
git checkout -b feat/short-description
```

### Running Locally

Make sure you have a working `.env` file with a valid testnet contract ID before testing any Soroban-related routes. IPFS routes can be tested with a Pinata test key.

### Code Style

- TypeScript strict mode — no `any`
- Keep Soroban SDK usage inside `services/stellar.ts` only
- Keep IPFS/Pinata usage inside `services/ipfs.ts` only
- Routes should be thin — business logic lives in services
- Validate all incoming request bodies with `zod` in `middleware/validate.ts`

### Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org):

```bash
git commit -m "feat(indexer): add credential_revoked event handling"
git commit -m "fix(ipfs): retry failed Pinata uploads up to 3 times"
git commit -m "feat(routes): add GET /api/credentials/:subject endpoint"
git commit -m "refactor(stellar): extract RPC client into singleton"
```

### Pull Requests

Include in your PR description:
- What the change does
- Which issue it resolves
- How you tested it — ideally with a curl example for new routes

### Good Places to Start

| Issue | What it involves |
|---|---|
| Implement `services/ipfs.ts` | Pinata API integration, upload + fetch |
| Implement `services/indexer.ts` | Soroban event polling, credential indexing |
| Add request validation middleware | `zod` schema validation for all POST bodies |
| Add rate limiting | `express-rate-limit` on public endpoints |
| Write route tests | `supertest` + mocked services |

---

## License

Apache 2.0
