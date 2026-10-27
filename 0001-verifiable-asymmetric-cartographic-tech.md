# 🗺️ RFC: Verifiable Asymmetric Cartography & Programmable Objectives

Following a recent discussion across the community (involving **Please No Pictures, Ocky, Econmartin, lacal, Social Assassin, beauKode, MiMiMi, ProtoDroidBot, Shalimar, Byron DuMont, and Scetrov**), we wanted to share a consolidated summary of how the "Fog of War" map changes are landing for builders and players, alongside a concrete proposal for **Fenris Creations (FC)** to bridge the gap between in-game exploration and 3rd-party tool building using cryptographically verifiable primitives.

---

## 1. Where the Community Stands (The 4 Core Viewpoints)

* **Static vs. Dynamic Asymmetry:** Gaming communities solve static search problems fast. Hiding static star coordinates only creates temporary friction before everything of value is logged. True, lasting information asymmetry lives in **dynamic, ephemeral system state**—scanning active mining operations, infrastructure changes, and "road closures" (destroyed/offline gates) on less-traveled routes.
* **The Explorer Profession:** Removing client-side map spoilers is great *if* there are in-game mechanisms to turn cartography into shareable, sellable, or lootable data (e.g., cache drops with nav-data fragments, tradeable intel).
* **The Tribal Intel Operator (Non-Coders):** Players running tribe intel don't want to mess with coordinate math or file scraping. They just want a frictionless way to upload discovered systems and layer **permissioned, rank-based intel** onto a shared map—ideally accessible in-flight via the updated Chromium 156 (CEF) browser.
* **The 3rd-Party Builder Reality:** Right now, map builders are relying on legacy C6 coordinate data to map `solarSystemId` and crowdsourcing the rest. Everyone agrees this crutch won't survive future cycles. Builders need official, cryptographically verifiable primitives—not unverified local log hacks—to build trustless maps and data markets.

---


## 2. The Proposal: Primitives First, Community Builds the Rest

Assuming FC will not make the full map visible by default, we don't want a static coordinate dump or an easily spoofed local loopback API. We also don't want to hand FC a full system design to build. Tribal maps, intel markets, and objective contracts are things the community can and should build.

What we need from FC is the small set of things **only the game server or client can provide**: signed, verifiable records of what happened in game, and a way to get them out of the client. We've staged these as **crawl / walk / run**. Each stage is useful on its own, so builders can ship on crawl while FC decides on the rest.

### Crawl: Signed Jump & Scan Receipts

* **P1. Signed receipts.** When a player jumps or scans, the server signs a receipt (e.g. `CharacterID`, `FromSystemID`, `ToSystemID`, scan snapshot, `Timestamp`) with **Ed25519**. Sui Move verifies Ed25519 natively (`sui::ed25519::ed25519_verify`), as does any off-chain backend. Receipts are **unforgeable**: nobody can fabricate one without FC's key.
* **P2. Owner field.** Each receipt names the character / wallet it was issued to. Unforgeable is not the same as non-transferable; without an owner field a copied receipt can be used by anyone.
* **P3. Published keys.** FC publishes its attestation public keys as an **on-chain registry object**, with rotation, so contracts and backends don't hard-code keys.
* **P4. Batch signing.** FC signs a **Merkle root** over a batch of receipts, so a pilot can prove any single receipt against the root. Nobody wants to handle 1m individually signed receipts.
* **P5. Assembly hand-off (open question).** Builders need some way to get receipts from the client into their contracts. One candidate is to make this a **capability of the existing Smart Assembly interaction**:
  * An assembly declares which receipt types it accepts. When a pilot interacts with it, the client offers matching receipts and the pilot approves before anything is shared.
  * The pilot has already chosen to interact with that assembly, and its owner is known on-chain. Recon objectives map naturally onto "scan X, bring the receipt back to this assembly."
  * Not proposed: minting receipts as owned on-chain objects would remove the need for a hand-off, but FC would pay gas on every jump.
### Walk: More Receipt Types & Hand-off Paths (Optional)

Same signing, owner field and keys as crawl, covering more of the game and, if needed, a second way out of the client. For example:

* **In-game browser with a receipt bridge (if the assembly route isn't right).** There is no in-game browser today. If assembly interaction turns out to be the wrong hand-off path, the alternative is a client browser or flight-view overlay (e.g. on the embedded CEF) with a `window.ef` bridge to the local receipt store. It would need **per-site approval and selective sharing**, like a wallet "connect" prompt; exposing the whole receipt store to any page would let one malicious map harvest a pilot's intel history.
* **Objective Tracker binding.** Pilots point the native top-left Tutorial/Objective tracker at a contract-defined objective. On completion the server issues a receipt: *"character X completed objective Y defined by contract Z."* Tribes can push recon missions to pilots' HUDs and verify completion without trusting the pilot.
* **Loot receipts.** NPC caches and wrecks drop signed nav-data fragments.
* **Destruction & status receipts.** Gates going offline, assemblies destroyed: a verifiable source for the "road closures" in Section 1.

### Run: The Information Economy (Optional)

* **POD-format signing.** The server also signs receipts as **Provable Object Data (PODs)** (EdDSA on BabyJubJub, Poseidon hashing), alongside Ed25519.
* **Standard consumption GPCs.** FC publishes standard **General Purpose Circuit (GPC)** configurations for its receipt types, with Groth16 verifying keys on-chain, so every builder verifies proofs against the same circuits using `sui::groth16`.
* **What this enables.** Zero-knowledge intel: a scout can prove they hold fresh scans for a 5-jump route from System A to System B, or that an enemy assembly exists within X lightyears, **without revealing the intermediate systems or coordinates**. That is the basis for intel markets, private bounties, and cross-tribe intel sharing without trusting the seller.

---

## 3. What Builders Can Do With These

Examples, not asks. Design is up to whoever builds them, including replay protection, fair exchange, and pricing.

* **Tribal intel maps:** pilots share receipts with their tribe's permissioned map, which verifies FC's signature, with rank-based layers (rendered in flight if an in-game browser lands).
* **Recon objectives:** tribe contracts post "submit a fresh scan for System X" and pay out on a valid receipt.
* **Route-intel markets:** sellers prove what they hold (run stage) before revealing it.
* **Lootable nav data:** traded and consumed like any other item.

---

## 4. Open Questions for FC

1. **Hand-off:** is the assembly interaction the right place for receipts to leave the client, or is there a better path?
2. **In-game browser:** if assembly hand-off isn't right, is a client browser/overlay on the roadmap at all?
3. **Jump receipt contents:** which fields go in jump receipts versus scan receipts? Signed relative distance/bearing on every jump would let players triangulate absolute coordinates quickly.
4. **Cheapest additions:** which walk-stage receipt types would be cheapest for FC to add?

---

## 5. Summary of "Asks" for FC

1. **Crawl (core):** Ed25519-signed jump and scan receipts with an owner field, an on-chain key registry, batch (Merkle root) signing, and an assembly hand-off path (P1-P5).
2. **Walk (optional):** more receipt types (Objective Tracker completion, loot, destruction/status), and an in-game browser with a permissioned `window.ef` receipt bridge if the assembly route isn't right.
3. **Run (optional):** POD-format signing plus standard GPC configurations with on-chain verifying keys, enabling a verifiable information economy.
