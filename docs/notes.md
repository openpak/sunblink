# Notes: sunblink

## Background

OpenPak's Sunblink backend for Nintendo Switch titles: the service a Sunblink
game reaches when a console is redirected to OpenPak. The first and only known
title is **Hello Kitty Island Adventure** (Sunblink / Sanrio, Switch
30 January 2025, Unity).

## Status

**Status: documentation only, and deliberately so.** There is no code here and
none is planned until the captures in
[`evidence-needed.md`](evidence-needed.md) show what the Switch build
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
[`design.md`](design.md) for what is known, and
[`evidence-needed.md`](evidence-needed.md) for the three captures
that decide everything.

## Ground rules

- Clean room. Read [clean-room policy](https://docs.openpak.org/contributors/clean-room/)
  before writing a line. Public protocol documentation, our own captures and
  observations, and compatibly licensed code are the only inputs.
- AGPL-3.0-only, English only, Go 1.27 when there is Go, no secrets in the
  repository.
- Raw captures, tokens, and account data stay outside every repository, in
  `scratch/` per [conventions](https://docs.openpak.org/contributors/conventions/).

This project is not affiliated with or endorsed by Nintendo, Sunblink, or
Sanrio. It contains no game code, assets, SDK material, binaries,
certificates, keys, firmware, extracted files, credentials, or raw captures.
