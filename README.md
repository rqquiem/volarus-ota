# VolarusProject OTA update feed

Signed over-the-air update feed for VolarusProject. This repository hosts the
JSON feed and the release ZIPs that already-installed VolarusProject apps poll
to update their web/backend bundle without shipping a new APK.

It contains no application source code. The app embeds the matching public key
(`server/ota-manifest.js` `OTA_PUBLIC_KEY`); the Ed25519 signing key never
leaves CI secrets (or the git-ignored `.secrets/` directory for manual
releases). A manifest that does not verify against that public key is discarded
by the app before it downloads anything.

## Feed files

Both files live on the `main` branch and are served over HTTPS through
`raw.githubusercontent.com`:

- `latest.json` — the signed manifest for the newest release. This is the trust
  root: an Ed25519 signature covers the canonical encoding of `schema`,
  `version`, `zipUrl`, `zipSha256`, `zipBytes`, `minAppVersion`, `notes`, and
  `releasedAt`. The app verifies the signature, then verifies the downloaded
  archive's SHA-256 and byte length against this manifest.
- `changelogs.json` — the last 30 `{ version, notes, releasedAt }` entries,
  newest first, used for the in-app release notes.

The actual bundles are attached to GitHub Releases tagged `v<version>`:

```
v1.4.2/volarus-v1.4.2.zip
```

## Manifest shape

```json
{
  "schema": 1,
  "version": "1.4.2",
  "zipUrl": "https://github.com/rqquiem/volarus-ota/releases/download/v1.4.2/volarus-v1.4.2.zip",
  "zipSha256": "<64 hex chars>",
  "zipBytes": 1297681,
  "minAppVersion": "1.2.11",
  "notes": "# v1.4.2 ...",
  "releasedAt": "2026-10-11T02:25:15.954Z",
  "signature": "<base64 Ed25519 signature>"
}
```

`minAppVersion` is a native-APK floor, not the release version: it names the
oldest installed APK the bundle may run on (the first APK that shipped the OTA
client). Never set it to the version being published.

## How it is published

`.github/workflows/release.yml` in `rqquiem/VolarusProject` runs
`scripts/publish-update.cjs`, which builds and signs the bundle, creates the
`v<version>` release here, uploads the ZIP asset, and writes `latest.json` and
`changelogs.json` to `main`. See `docs/ota-pipeline.md` and
`docs/ota-production-release.md` in the application repository.

## Do not edit by hand

`latest.json` is signed. Editing any signed field — including the zip URL or
hash — invalidates the signature and every app rejects the feed. Re-sign with
the private key through the publisher instead.
