Issue #5 — Initialise Express + TypeScript project
Labels: setup, backend
Description:
Bootstrap the Node.js backend with Express, TypeScript, and the base folder structure outlined in the README.
Acceptance Criteria:

 package.json configured with dev, build, and start scripts
 TypeScript strict mode enabled in tsconfig.json
 Express server starts on PORT from .env and responds to GET /api/health with { status: "ok" }
 nodemon or ts-node-dev used for hot reload in dev
 src/routes/, src/services/, src/middleware/, src/types/ directories scaffolded with index files


Issue #6 — Set up environment variable config and validation
Labels: setup, backend
Description:
Add a centralised config module that loads, validates, and exports environment variables so the rest of the app never reads process.env directly.
Acceptance Criteria:

 .env.example lists all required and optional variables with comments
 src/config.ts uses zod to parse and validate env vars on startup
 Server fails fast with a clear error message if a required variable is missing
 STELLAR_NETWORK, REGISTRY_CONTRACT_ID, PINATA_API_KEY, PINATA_SECRET_KEY, and PORT are all validated
 Config is exported as a typed object, not raw strings


Issue #7 — Implement centralised error handling middleware
Labels: setup, backend
Description:
Add an Express error handler that catches all thrown errors and returns consistent JSON error responses.
Acceptance Criteria:

 src/middleware/error.ts exports an Express error handler middleware
 All errors return { error: string, code?: string } JSON with the appropriate HTTP status
 Unhandled 404 routes return { error: "Not found" } with status 404
 Errors from Soroban RPC and IPFS are caught and mapped to readable messages
 Error handler is registered last in src/index.ts


Issue #8 — Scaffold Soroban RPC service
Labels: setup, backend
Description:
Create src/services/stellar.ts with the Stellar SDK client configured for testnet and mainnet, ready for contract calls.
Acceptance Criteria:

 SorobanRpc.Server instance created from config values
 resolveOnChain(did: string): Promise<string | null> implemented and returns the document hash or null if not found
 verifyOnChain(subject: string, type: string): Promise<boolean> implemented via contract simulation
 Network switching (testnet/mainnet) driven by STELLAR_NETWORK env var
 Function is tested against the deployed testnet contract and returns expected values


Issue #9 — Scaffold IPFS service with Pinata
Labels: setup, backend
Description:
Create src/services/ipfs.ts with Pinata upload and fetch functions for DID Documents and Verifiable Credentials.
Acceptance Criteria:

 uploadDocument(doc: DIDDocument): Promise<string> uploads to Pinata and returns the IPFS CID
 fetchDocument(cid: string): Promise<DIDDocument> retrieves and parses a DID Document by CID
 uploadCredential(vc: VerifiableCredential): Promise<string> and fetchCredential(cid: string) implemented with same pattern
 Hash of uploaded content is verified against the CID before returning
 Failed uploads retry up to 3 times before throwing