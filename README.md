# sunblink

OpenPak's Sunblink backend for Nintendo Switch titles: the service a Sunblink
game reaches when a console is redirected to OpenPak. The first and only known
title is **Hello Kitty Island Adventure** (Sunblink / Sanrio, Switch
30 January 2025, Unity).

**Status: documentation only, and deliberately so.** There is no code here and
none is planned until the captures in
[`docs/evidence-needed.md`](docs/evidence-needed.md) show what the Switch build
actually dials. This is not a scaffold waiting to be filled in.

## Why a new adapter

Nothing in `servers/` answers this game today:

- Not NEX / NPLN: no Nintendo game-server strings are expected; multiplayer is
  same-platform invite codes through Sunblink, not Nintendo matchmaking.
- Not Photon, PlayFab, EOS, Demonware, Klei (`klei`), Team17 (`t17`) or
  battlenet: no AppId, tenant, or provider hostname for this title has been
  observed yet.
- The wiki-confirmed shape (2-player visit-a-friend, temporary invite codes,
  no cross-platform play, Nintendo Switch Online required on Switch) points at
  a Sunblink rendezvous plus peer/relay session — but that is a hypothesis
  until a boot log says so.

So the work starts with observation, not code. Read
[`docs/design.md`](docs/design.md) for what is known, and
[`docs/evidence-needed.md`](docs/evidence-needed.md) for the three captures
that decide everything.

## What would make this repository real

1. A boot capture showing the Switch build resolving a Sunblink (or Unity
   Relay / other middleware) host and the ports it opens. Then this repository
   implements exactly that host.
2. A session capture showing whose address the guest is told to connect to
   (Sunblink relay/datacentre vs the host console). That decides relay vs
   rendezvous-only vs full session server.

Until then: no port block claimed in [`ports.md`](../../docs/ports.md); the
third-party range 22600–22699 has room reserved (next free block 22660–22669),
and taking ten ports for a service that does not exist would be a lie in a
shared document. No `go.mod` either; it comes back with the first real package.

## Ground rules

- Clean room. Read [`docs/clean-room-policy.md`](../../docs/clean-room-policy.md)
  before writing a line. Public protocol documentation, our own captures and
  observations, and compatibly licensed code are the only inputs.
- AGPL-3.0-only, English only, Go 1.27 when there is Go, no secrets in the
  repository.
- Raw captures, tokens, and account data stay outside every repository, in
  `scratch/` per [`docs/playbooks/conventions.md`](../../docs/playbooks/conventions.md).

This project is not affiliated with or endorsed by Nintendo, Sunblink, or
Sanrio. It contains no game code, assets, SDK material, binaries,
certificates, keys, firmware, extracted files, credentials, or raw captures.
