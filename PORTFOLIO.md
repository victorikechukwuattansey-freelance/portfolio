# Smart Contract Portfolio

Three Solidity projects, each built from first principles — spec before code,
adversarial tests for every invariant, and documented limitations as part of
the deliverable.

## 1. ERC-20 Token (`my-erc-20-token`)

A production-shaped ERC-20 composed from five OpenZeppelin extensions:
`ERC20Capped`, `Ownable`, `ERC20Burnable`, `ERC20Pausable`, `ERC20Permit`.

**Tool:** Foundry
**Tests:** 19 passing (`forge test`)
**Deployed:** Sepolia, verified on Etherscan

What it demonstrates:
- **Diamond-inheritance resolution** — `_update` explicitly overridden across
  `ERC20`, `ERC20Capped`, and `ERC20Pausable` with a deliberate pause-before-cap
  order
- **EIP-2612 Permit** — EIP-712 signature-based approvals with nonce/deadline/domain
  separator tests
- **Exact-error assertions** — `vm.expectRevert(abi.encodeWithSelector(...))`
  verifies not just *that* a call reverts, but *which custom error with what arguments*

Repo: https://github.com/victorikechukwuattansey-freelance/my-erc-20-token
Deployed: https://sepolia.etherscan.io/address/0xBdD7E89be61dA8fe4f66388fdd4dDFac77933d61

## 2. NFT Collection (`my-nft`)

A minimal ERC-721 with per-token metadata (`ERC721URIStorage`) and public burn
(`ERC721Burnable`).

**Tool:** Foundry
**Tests:** 6 passing (`forge test`)
**Deployed:** Sepolia, verified on Etherscan

What it demonstrates:
- **Two override resolutions** — `tokenURI` and `supportsInterface` explicitly
  resolved across `ERC721` and `ERC721URIStorage`; `_burn` deliberately *not*
  overridden because only one parent overrides it
- **Fuzz invariant** — `testFuzz_TransferConservesBalances` runs 256 iterations
  asserting `balanceOf(from) + balanceOf(to)` is unchanged
- **The `totalSupply` lesson** — base ERC-721 does not define `totalSupply`;
  the README explains why `ERC721Enumerable` is out of scope

Repo: https://github.com/victorikechukwuattansey-freelance/my-nft
Deployed: https://sepolia.etherscan.io/address/0xc91c858497795bE246E4f6aBe0627969A2b60940

## 3. NFT Staking (`nft-staking`)

Fixed-duration NFT staking with a per-stake ETH reward and pull-over-push payout.

**Tool:** Hardhat 3 (Mocha + Ethers.js + TypeScript)
**Tests:** 18 deterministic (Hardhat) + 2 fuzz invariants × 128,000 transitions (Foundry)
**Deployed:** Sepolia (production + 60-second demo variant), both verified
**Spec:** [`INVARIANTS.md`](./nft-staking/INVARIANTS.md) — 10 invariants, each
with a positive and an adversarial test

What it demonstrates:
- **Spec-first discipline** — invariants written with failure stories and test
  names *before* `Staking.sol` existed
- **Build order by failure impact** — escrow → authorization → reentrancy →
  single-unstake → rewards → duration → structural
- **Three-bucket solvency** — `address(this).balance >= rewardPool +
  Σ stakes[i].reward + Σ pendingWithdrawals`
- **CEI on both external calls** — the NFT transfer and the ETH payout each
  tested with a malicious receiver that attempts reentry
- **Versioned migration** — v1.0 (push payout, 14 tests) → v1.1
  (pull-over-push, 18 tests), each tagged and documented

- **Fuzz-tested invariants** — Foundry stateful fuzzing verifies I1 and
  I4+I10 across 128,000 randomized stake/unstake/withdraw/fund sequences.
  Sabotage-validated: removing the pool decrement in `stake` produces a
  counterexample on the first run, proving the fuzzer catches the class of
  bug it's meant to catch.

Repo: https://github.com/victorikechukwuattansey-freelance/nft-staking
Deployed (7-day): https://sepolia.etherscan.io/address/0x27DbE8BBB484BF8885d856d7f66182A7f3189aaE
Deployed (60-sec demo): https://sepolia.etherscan.io/address/0xF96DF232c796c1ac9B5a18700AF487AF090B913d

## Common discipline across all three

Every repo has:

- **A specification that predates the code** (`INVARIANTS.md`, design rationale,
  override comments)
- **Positive + adversarial tests** — every invariant has a test that would catch
  its violation, not just a test that exercises the happy path
- **Documented limitations** — the honest boundary is part of the artifact
- **Green CI** on every push (`forge fmt --check + build + test` for Foundry
  repos; `npm ci + hardhat build + test` on Node 22 for Hardhat)
- **Legible version history** — commits per invariant or extension, tags that
  mark stable states

## License

MIT across all repos.
