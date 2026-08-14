---
dusk: v1alpha1
namespace: stout
kind: service
name: boating-accident
title: Boating Accident
relations:
  - type: runs_on
    to: host:mini-2/orbstack
observed_as:
  - service:mini-2/rackd-boating-accident
attributes:
  language: go
  url: https://boating-accident.stout.zone
  public: true
---

A self-hosted, encrypted inventory for firearms, ammo, knives and accessories.
One Go binary with a React SPA embedded through `go:embed` and SQLite behind it: no cloud, no account, no telemetry.
The repository is public and MIT licensed, so the polish bar here is the one for code the whole internet can read.

The encryption is the product rather than a feature of it, and that constrains what can be added.
A 6-digit PIN goes through Argon2id to a key-encryption key, which unwraps a random 256-bit data key used for AES-256-GCM on every content field and every uploaded file.
The data key lives only in memory while unlocked, so locking the vault or restarting the process seals it again until the PIN is re-entered, and changing the PIN re-wraps that key rather than re-encrypting the data.
Any new field that ought to be secret has to travel the same path; writing one straight to SQLite silently puts plaintext on disk next to encrypted neighbours, and nothing will complain.
All state is the SQLite database plus the encrypted upload blobs under `BOAT_DATA_DIR`.

Three constraints that look arbitrary and are not.
The binary is deliberately CGO-free (modernc sqlite, and iPhone HEIC converted in the browser rather than on the server), so reaching for a cgo dependency to do image or crypto work takes the static single-binary build with it.
Uploads are EXIF-stripped before storage, because a collection photo carrying GPS defeats the point of encrypting it.
AmmoSeek is deep-linked and never scraped, since their terms prohibit automated access, and the free spec lookup (Wikipedia full-text into a DBpedia infobox, key-less and cached) is presented for review instead of auto-filling fields, because it is community-sourced.

## Gotcha

**The Kubernetes namespace is `rackd`, this project's former name, and so is its volume.**
That is why the observed service is `service:mini-2/rackd-boating-accident` rather than something matching the repository.
The mismatch is deliberate and stays: renaming the namespace would mean moving the encrypted database and the upload blobs, and all it would buy is a tidier name.
Do not clean it up.
