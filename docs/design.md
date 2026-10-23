# Design — Sunblink backend (Hello Kitty Island Adventure, Switch)

## 1. The game

Hello Kitty Island Adventure (Sunblink / Sanrio) is a narrative life sim.
Switch release 30 January 2025 (Apple Arcade 28 July 2023, Steam same day as
Switch, PlayStation 5 August 2025). Unity engine — Switch 1 and Switch 2 builds
run different Unity versions and cannot play together.

## 2. Multiplayer shape (public, wiki-observed)

- 2 players max: one host, one guest visiting the host island.
- Invite codes: host generates a temporary code in the in-game Multiplayer app
  (Tablet); guest enters it via Join Friend (in-game app or title screen).
  Codes expire within minutes and can rotate when the menu reopens.
- Same platform only, same update version required. No cross-platform play.
- On Switch: Nintendo Switch Online subscription required, plus in-game
  Permissions / Allow Online toggle. Device clock must be correct (out-of-sync
  time breaks multiplayer).
- Guest keeps own inventory, shares host tools/stamina, limited quest access;
  friendship levels/emotes/abilities gate co-op content. None of this needs a
  server decision yet — it is game logic once the session exists.

Source: official Hello Kitty Island Adventure Wiki, Multiplayer page
(fetched 2026-10-13).

## 3. What the network must do (hypothesis, unconfirmed)

Invite codes that expire in minutes imply a rendezvous:

1. Host registers session (island, version, platform) and gets a short code.
2. Guest redeems the code and learns where to connect.
3. Host and guest join a 2-peer session (direct P2P, platform relay, or
   Sunblink relay).

What is NOT known: hostname(s), ports, TLS pinning, auth token (Nintendo
NSA id_token? Sunblink account? none?), session transport (UDP? WebSocket?
Unity Relay? custom?), NAT traversal. No dump strings or captures exist yet.

## 4. Why existing adapters do not cover it

- `platforms/switch/games/switch-nex`, `platforms/switch/games/npln`: Nintendo-server titles. No evidence
  this title uses NEX, NPLN, Pia, or Eagle.
- `servers/photon`, `servers/playfab`, `servers/eos`, `servers/demonware`,
  `platforms/switch/games/klei`, `servers/t17`, `platforms/switch/games/battlenet`: each answers a named
  provider host with observed identifiers (AppId, tenant, etc.). No such
  identifier has been observed for this title.
- Conclusion: new adapter, scoped to whatever host the boot capture names.
  One process per OpenPak convention (plain HTTP inside pod network, Traefik
  terminates console TLS, NSA token verified via nx-baas internal API, health
  endpoint, block of ten in 22600–22699).

## 6. Connectivity service (mandatory)

The adapter must consume `openpak/connectivity` (internal HTTP on 20150,
authenticated per-backend token namespace) rather than inventing its own NAT
layer, per the shared connectivity plan (internal research notes):

- Register each player independently with its own generation, even when host
  and guest share a public IP (household / CGNAT). Never key identity by IP.
- Issue STUN/TURN via `POST /v1/ice` mapped into the client's native ICE
  response; preserve the client's TTL/cache conventions.
- If the session transport proves to be native UDP peer-to-peer, allocate a
  pair relay (`POST /v1/relays`), verify each side's destination-specific
  mapping toward the relay endpoint through the game's own handshake, renew
  with `observed` candidates, then `bind` exact endpoints only. Unverified
  stock clients stay unsupported — no IP-only learning fallback.
- Keep matchmaking, invite codes, and session handoff in this adapter;
  endpoint leases, TURN credentials, and relay sockets live in connectivity.

## 5. The three unknowns that matter, in order

1. Hostnames + ports the Switch build resolves (decides the adapter's shape).
2. Certificate pinning (decides whether DNS redirect suffices).
3. Session address family — relay/datacentre vs host console (decides whether
   OpenPak runs a relay or only a rendezvous).

See [`evidence-needed.md`](evidence-needed.md) for how to answer each.
