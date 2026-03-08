# Testing — Solana Stablecoin Standard

## Test Summary

| Test Suite | Tests | Runner | Status |
|------------|-------|--------|--------|
| SDK Unit Tests (PDA helpers) | 37 | Jest | ✅ Verified passing |
| SDK Unit Tests (encoding/parsing utils) | 54 | Jest | ✅ Verified passing |
| **Total SDK Unit Tests** | **91** | **Jest** | **✅ All passing** |
| SSS-1 Integration | 31 | Mocha | Requires devnet deployment |
| SSS-2 Integration | 34 | Mocha | Requires devnet deployment |
| SSS-3 Integration | 30 | Mocha | Requires devnet deployment |
| Transfer Hook Integration | 22 | Mocha | Requires devnet deployment |
| Security Invariants | 29 | Mocha | Requires devnet deployment |
| SDK Integration | 27 | Mocha | Requires devnet deployment |
| **Total Integration Tests** | **173** | **Mocha** | Pending devnet deploy |
| **Grand Total** | **264** | — | — |

---

## Running the SDK Unit Tests (no setup required)

```bash
git clone https://github.com/helmutdeving/solana-stablecoin-standard
cd solana-stablecoin-standard
npm install -w packages/sdk
npm test -w packages/sdk
```

### Verified Output (2026-03-08)

```
PASS src/__tests__/pdas.test.ts (5.78 s)
  findSss1ConfigPda
    ✓ returns a tuple of [PublicKey, number]
    ✓ returns a valid off-curve PDA
    ✓ bump is in valid range [0, 255]
    ✓ is deterministic for same mint
    ✓ is different for different mints
    ✓ is program-specific (different program → different PDA)
    ✓ PDA is a valid base58 string
  findFreezeAuthorityPda
    ✓ returns a tuple of [PublicKey, number]
    ✓ returns a valid off-curve PDA
    ✓ is deterministic for same mint
    ✓ is different from config PDA for same mint
    ✓ is different for different mints
    ✓ bump is in valid range [0, 255]
  findWhitelistPda
    ✓ returns a tuple of [PublicKey, number]
    ✓ returns a valid off-curve PDA
    ✓ is deterministic for same mint+wallet
    ✓ is different for different wallets
    ✓ is different for different mints with same wallet
    ✓ is different from freeze PDA for same mint+wallet
    ✓ is different from config PDA
  findBlacklistPda
    ✓ returns a tuple of [PublicKey, number]
    ✓ returns a valid off-curve PDA
    ✓ is deterministic for same mint+wallet
    ✓ is different for different wallets
    ✓ is different for different mints
    ✓ bump is in valid range [0, 255]
  findAuditLogPda
    ✓ returns a tuple of [PublicKey, number]
    ✓ returns a valid off-curve PDA
    ✓ is deterministic for same mint+eventId
    ✓ is different for sequential event IDs (0 vs 1)
    ✓ handles large event IDs without error
    ✓ events 0, 1, 100 all produce unique PDAs
    ✓ is different from whitelist PDA for event 0
    ✓ is different for different mints same event
    ✓ bump is in valid range [0, 255]
  cross-PDA uniqueness
    ✓ all five PDA types produce unique addresses for same inputs
    ✓ PDAs for different mints are all unique

PASS src/__tests__/utils.test.ts
  encodeName
    ✓ encodes a short name to 32 bytes
    ✓ pads short names with zero bytes
    ✓ encodes the correct characters
    ✓ truncates names longer than 32 bytes
    ✓ handles empty string
    ✓ handles exactly 32-character names
    ✓ handles UTF-8 multi-byte characters without overflow
    ✓ returns an array of numbers
    ✓ encodes 'USD Coin' correctly
    ✓ different names produce different encodings
  encodeSymbol
    ✓ encodes a symbol to 8 bytes
    ✓ pads short symbols with zeros
    ✓ truncates symbols longer than 8 bytes
    ✓ encodes the correct bytes for BTC
    ✓ handles empty string
    ✓ handles exactly 8-char symbol
    ✓ USDC encodes correctly
  decodeName
    ✓ decodes a null-terminated byte array
    ✓ decodes a non-null-terminated array
    ✓ decodes empty bytes
    ✓ decodes Uint8Array input
    ✓ round-trips with encodeName
    ✓ round-trips with encodeSymbol
    ✓ handles all-zero array
    ✓ stops at first null byte
    ✓ decodes USDC symbol round-trip
  formatAmount
    ✓ formats zero correctly with 6 decimals
    ✓ formats 1 USDC (1_000_000 with 6 decimals)
    ✓ formats 1.5 USDC
    ✓ formats with 0 decimals
    ✓ formats with 2 decimals
    ✓ pads fractional zeros correctly
    ✓ formats 100 USDC
    ✓ formats 0.5 with 2 decimals
    ✓ formats large supply (1 billion tokens)
    ✓ returns string type
  parseAmount
    ✓ parses '1.000000' with 6 decimals
    ✓ parses '0' with 6 decimals
    ✓ parses '1.5' with 6 decimals
    ✓ parses whole numbers without decimal point
    ✓ parses with 2 decimals
    ✓ round-trips with formatAmount
    ✓ truncates excess decimal places
    ✓ returns BN instance
    ✓ parses '0.000001' (minimum USDC)
    ✓ parses large amounts
  encodeReference
    ✓ returns a hex string
    ✓ returns 128 hex chars (64 bytes)
    ✓ pads to 128 chars with zeros
    ✓ handles empty string
    ✓ truncates refs longer than 64 chars
    ✓ encodes 'A' (0x41) as first two hex chars
    ✓ different refs produce different encodings
    ✓ same ref produces same encoding

Test Suites: 2 passed, 2 total
Tests:       91 passed, 91 total
Time:        6.79 s
```

---

## Integration Tests (require devnet deployment)

The 173 integration tests cover real on-chain program interactions using
[Anchor's Mocha test framework](https://www.anchor-lang.com/docs/testing).
They require the programs to be deployed on Solana devnet.

### Running integration tests (after devnet deploy)

```bash
export ANCHOR_PROVIDER_URL=https://api.devnet.solana.com
export ANCHOR_WALLET=~/.config/solana/id.json

npm install -w tests
npm test -w tests

# Run a specific suite
npm test -w tests -- --grep "SSS-1"
```

### Integration test scenarios

#### SSS-1 (31 tests)
- Initialize Token-2022 mint with SSS-1 preset (admin, decimals, supply cap)
- Mint tokens to authorized minter
- Burn tokens by authorized burner
- Freeze/thaw accounts via freeze authority
- Pause/unpause emergency controls
- Supply cap enforcement: cap boundary (exact cap ✓, cap+1 → `SupplyCapExceeded` ✗)
- Multi-minter RBAC: second minter can mint after being granted role
- Metadata validation: name/symbol correctly stored on-chain

#### SSS-2 (34 tests)
- All SSS-1 scenarios plus:
- Initialize Transfer Hook program alongside stablecoin
- Install ExtraAccountMetaList for hook account resolution
- Blacklist account via admin: PDA created, blacklist count incremented
- Transfer attempt from blacklisted sender → blocked by hook ✗
- Transfer attempt to blacklisted receiver → blocked by hook ✗
- Remove from blacklist: transfers succeed again ✓
- Seize operation: freeze target → seize via permanent delegate → recipient balance = 0
- Compliance scenario (OFAC-style): blacklist → transfer blocked → seize → verify

#### SSS-3 (30 tests)
- Allowlist-gated operations: non-allowlisted address cannot mint
- Oracle-gated mint cap: fetch Pyth price feed and verify cap enforcement
- Staleness guard: mint rejected if Pyth feed >60s old
- Confidence guard: config update rejected if feed confidence interval >2%
- Confidential commitment: commitment stored and retrievable
- Downgrade path: SSS-3 → SSS-2 configuration migration

#### Transfer Hook (22 tests)
- Deploy hook program, verify bytecode
- `initialize_extra_account_meta_list`: ExtraAccountMetaList PDA created
- Hook invoked on every Token-2022 transfer (verified via event)
- Sender blacklist → transfer blocked at hook
- Receiver blacklist → transfer blocked at hook
- Clean state (no blacklist) → hook passes through
- `fallback` discriminator routing verified

#### Security Invariants (29 tests)
- Minter cannot invoke burn (`AccountUnauthorized`)
- Burner cannot invoke mint (`AccountUnauthorized`)
- Freezer cannot mint or burn
- Non-admin cannot update supply cap
- Double-mint prevention (idempotency guard)
- Zero-value transfer guard
- Underflow protection: burn-against-zero → `UnderflowProtection`
- Authority transfer: two-step proposal/accept — accepting without proposal fails
- Authority transfer: wrong key accepting fails

#### SDK Integration (27 tests)
- End-to-end `SSSClient` flow: init → mint → burn → freeze → thaw
- `SSS2Client` compliance: blacklist → transfer attempt → unblacklist
- `SSS3Client` oracle: price feed fetch, staleness check
- Multi-client interop: SSSClient configures token, SSS2Client enforces compliance

---

## Fuzz Testing (Trident)

Property-based fuzzing of the SSS-1 and SSS-2 programs. Covers arbitrary
sequences of mint/burn/freeze/seize with randomized role assignments.

```bash
cd programs/solana-stablecoin-standard
cargo install trident-cli
trident fuzz run sss1_fuzz   # SSS-1: supply cap, RBAC, pause semantics
trident fuzz run sss2_fuzz   # SSS-2: blacklist consistency, seize invariants
```
