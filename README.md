# PSYROB Public Mainnet v1.0.0


A block-DAG network with CPU mining. NGOs are elected on-chain and share the emission with miners.

- **Consensus:** GHOSTDAG ordering (k = 18), about one block every 10 seconds, up to 10 parents per block. Parallel blocks are merged; a conflicting transaction is skipped, not fatal.
- **Proof of work:** Argon2d with 32 MiB per attempt. Every attempt is signed by the miner wallet, so the work cannot be handed to a pool operator without the reward key.
- **Supply:** maximum 1,000,000,000 PSY.
  - 36 % to miners and 36 % to elected NGOs, on the same smoothly decaying schedule;
  - 12 % to the development team, in immutable vesting that nothing can change, block or pause;
  - 16 % to the Community/Treasury allocation.

  No airdrop and no developer float
  - 100 NGO seats with one-year terms. The order is mine, publish and mint, attestation, election, then finality.
  - A 21-member finality committee (6 NGO seats, 15 miner seats, quorum 15, slashing for equivocation).
  - Five operational roles elected by the NGOs. Role holders cannot vote and are paid from the Treasury.
  - An Interim Council fallback applies when fewer than 60 NGOs are elected after three months.
  - Launch ceremony: before the network starts, the three Dev Account holders each sign their seat with the Dev Account wallet and name a separate council key (`/launch`).
  - Dev council: until 60 NGOs are elected for the first time, or 12 months at most, two of the three council keys can approve small node updates, change the committee bond and the role candidacy deposit, and pause. Then the council keeps only the emergency pause. It never has access to the Treasury, balances, the 12 % or the NGO election and its rules.
  - Upgrades (`/upgrades`): every contract or node update is submitted, approved and waits a public timelock (7 days, 14 for node updates approved by the DAO). Contract updates need a DAO vote with a majority of the elected NGOs. A node older than an update in force stops instead of splitting the chain.
  - Emergency pause (`/pause`): one contract or all transactions, at most 72 hours; only the DAO can extend or end it early. Mining, votes, upgrades and the 12 % vesting continue; nobody can move PSY during a network pause.
- **Finality from the start:** until the first committee is chosen, a checkpoint needs **2 of 3 dedicated keys** (they do not mine and hold no PSY), for 12 months at most; only the DAO can replace the keys (docs/dag/DAG_SPEC.md section 9.1).
- **Safety rules** (section 18): old block data leaves memory, the node answers while it starts and syncs, never builds on blocks too old to be accepted, and bounds every list request; NGO names follow simple rules and show as unverified until attested; the miner stores its key encrypted, uses several nodes and checks every template; releases are plain, reproducible and signed (`docs/RELEASE-KEY.pem`, `scripts/verify-release.mjs`)..


## Layout

| Folder | Contents |
|---|---|
| `node/` | the full node: consensus, P2P with peer discovery, JSON-RPC, explorer and participation pages, wallet tool |
| `miner/` | the separate miner application |
| `common/` | code shared by the node and the miner: header, Argon2d proof of work, RLP, crypto, ABI codec |
| `genesis/` | contract artifacts used to build the genesis state, the genesis file, chain metadata, the genesis check and seed peers |
| `deploy/` | how the public node is operated: systemd units, the watchdog, the pre-mining check and the launch scripts (`deploy/README.md`) |
| `docs/` | the specification (`docs/dag/DAG_SPEC.md`), the research notes with 202 verified peer-reviewed sources (`docs/research/RESEARCH.md`), the security analysis (`docs/SECURITY.md`), the decisions (`docs/DECISIONS.md`) and the operator notes |

## Run a node

Requirements: Node.js 22 or later, about 1 GB of free memory, and an open TCP port for peers (39870 by default).

```sh
git clone https://github.com/psyrob-blockchain/psyrob.git
cd psyrob
npm ci
node node/bin/psyrob.js --datadir ~/.psyrob
```

The node reads seed peers from `genesis/seeds.txt` (add more with `--seeds host:port,...`). It finds further peers through address gossip and keeps 8 outbound connections. It serves the explorer and JSON-RPC on `http://127.0.0.1:8770`. See `--help` for every option.

## Mine

```sh
node node/bin/psyrob-wallet.js new --out ~/psyrob-miner.key   # prints only the address; back the file up offline
node miner/bin/psyrob-miner.js --key ~/psyrob-miner.key --node http://127.0.0.1:8770 --threads 1
```

Each thread needs 32 MiB of memory and one CPU core. **A lost key cannot be recovered**, and an NGO that loses its wallet must restart the whole process with a new one.

## License

MIT, see [LICENSE](LICENSE). Third-party code keeps its own license: `third_party/argon2/LICENSE`,
`contracts/lib/forge-std/` and the font and Bootstrap licenses in `node/web/vendor/`.
