# Evidence needed

> **Current Switch client (2026-10-13): Harbor**, at
> `~/REPOS/Openpak/emulators/harbor`. Harbor is an independent
> OpenPak project based on Eden. Citron, Ryujinx (desktop and Android), and
> the former OpenPak Eden checkouts have been removed. References to those
> forks below record historical evidence only: do not run their commands,
> reinstall them, or select them for current work. Use Harbor's README and
> current handoff for builds and testing; old results do not establish Harbor parity.

[`design.md`](design.md) is hypothesis until these three observations exist.
Cheapest and most decisive first. Stop at the first step that settles the
adapter shape — later steps only matter once earlier ones fire.

Everything here is an observation of what the console or the emulator does.
No Sunblink server code, no game binaries in the repository. Captures stay
off-repo, in `scratch/` per the conventions doc; only decoded findings come
back as text.

## Tooling that already exists

- `prds/switch-catalogue-prd.md` §Facts from our dumps: `exefs.sh`,
  `analyze.py`, `gsid.py`, `keys.py`, `xref.py` for middleware strings, plus
  `romfs.sh`, `stack.py`, `meta.py` for Unity `global-metadata.dat`.
- Emulator DNS redirect log: boot the title in Ryujinx/Eden with DNS pointed
  at OpenPak and read which hostnames it asks for. That log is the source of
  truth wherever public lists are silent.
- Online-compatibility playbook: `docs` playbooks volume, §0 for launch
  personas (host/joiner retired 2026-09-26 — launch the fork directly).

## Step 1 — Which hosts does the Switch build resolve, in what order?

**Answers:** whether this is Sunblink-hosted, Unity Relay, or something else;
what the adapter must answer.

1. From a dump of the Switch build (base or update in
   `/mnt/media/Emulation/roms`): run the catalogue tools. Record Unity
   version, `SDK MW+...` middleware strings (Photon? PlayFab? Unity Netcode /
   Relay? EOS? none?), and any `sunblink.com`, `unity3d.com`,
   `cloud.unity.com`, or provider host strings with the code that loads them.
2. Boot in Ryujinx/Eden with OpenPak DNS, go to title screen, open the
   Multiplayer app, generate a code as host; on a second instance, attempt
   Join Friend. Record every hostname in resolution order with timestamps,
   which resolve before vs after the multiplayer menu, and which ports the
   console then opens to each (`tcpdump -n port 53` on the host plus emulator
   DNS log).

**Kill condition:** if the title resolves only Nintendo NEX/NPLN hosts, the
premise of this repository is wrong — it belongs in `platforms/switch/games/switch-nex` or
`platforms/switch/games/npln`, and this repo is deleted.

## Step 2 — Does the console pin a certificate?

**Answers:** whether a DNS redirect is enough, or a MITM also needs a CA trust
patch (hardware) / trusted stub cert (emulator).

Point the Step 1 hostname(s) at a local TLS stub presenting a self-signed
certificate and log the ClientHello (SNI, ciphers, ALPN, abort vs proceed).
Record any cleartext HTTP User-Agent or SDK headers that pin the stack
(e.g. Unity, DirtySock, EOS SDK version strings).

**Kill condition:** none — this only sets the difficulty of step 3.

## Step 3 — THE DECIDER: whose address does the guest connect to?

**Answers:** relay vs rendezvous-only vs full session server. This is the one
question that scopes the build.

With the MITM from step 2 (or, cheaper, a live retail session with
`tcpdump` on gameplay traffic): host an island on one persona, join by code
on a second, and record whose address the guest is told to connect to — a
Sunblink/Unity datacentre address, or the host console's address (direct or
NAT-traversed).

- **Datacentre/relay address** → OpenPak must run that relay leg. Scope the
  build to the observed relay protocol, nothing more.
- **Host console address** → OpenPak only needs the rendezvous (register +
  redeem codes, session handoff). Peers carry gameplay themselves.
- **Every session fails before an address is exchanged** → record how far it
  got (auth? version check? platform check?) — that gate is the first build.

If the MITM cannot get that far, the fallback is AS/WHOIS on the peer address
from a retail session: datacentre range vs residential tells relay vs P2P
without decrypting anything.

## What would *not* count as evidence

- PC / Apple Arcade / mobile captures. Different builds authenticate
  differently; only the Switch build decides this repository.
- Inference from another Sunblink title. Useful as a lead, never as a
  confirmed fact for this title.
- Anything read out of a non-free implementation's source. If a step feels blocked without one,
  it needs a capture instead.
