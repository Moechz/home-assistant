# Changelog

All notable user-visible changes to the Home Assistant TOS application package.

## 2026.11.3-2 — 2026-11-25

Supersedes the on-hold `2026.5.4-2` submission and the superseded
`2026.11.3-1` build, and remediates the review findings **C10, C2, C3, C8,
C9, T1** plus the App Center self-test result ("online installation failed —
cannot be opened").

- **C9 follow-up (this build):** removed every ASCII semicolon from the
  `descript`, `important` and `release_note` values in the 23-language file.
  The store metadata parser treats `;` as an inline INI comment, so the
  `2026.11.3-1` build's listing description was silently truncated at the
  first `;` and stored with a stray opening quote. The texts are otherwise
  identical.

### Security (T1)
- Upgraded the container image from `homeassistant/home-assistant:2026.5.4` to
  **`homeassistant/home-assistant:2026.11.3`** (upstream stable released
  2026-11-23). This rolls up six months of upstream security and dependency
  fixes (Home Assistant Core, Python 3.14 base, bundled libraries).
- Verified the official image is a multi-architecture index with
  `linux/amd64` **and** `linux/arm64` manifests
  (`sha256:24e541ddae5818bf8eb27898b9ba00d0c344a655aa5097e9e9bd4b19356be847`),
  so the aarch64 package pulls the native arm64 image rather than relying on
  emulation.
- Documented the image digests and licence obligations in `NOTICE.md`.

### Fixed
- **Install / "cannot be opened" (App Center self-test):** the health check now
  probes `/manifest.json` (HTTP 200, no redirect) and the first-start grace
  period is raised to **180 s** with 5 retries, so a slow first boot — typical
  on ARM NAS models, where Home Assistant spends minutes initialising — is no
  longer reported as a failed install. Startup was re-verified as non-root
  (`uid 1000`) with the new image; the page answers HTTP 302 to
  `/onboarding.html` as soon as the HTTP stack is up.
- **C9 (description ≠ delivered functionality):** rewrote the `en-us`,
  `zh-cn` and `zh-hk` descriptions and "important" texts. The previous
  `zh-cn`/`zh-hk` strings were copied from another application
  (beaverhabits: "习惯打卡与连续天数统计") and referenced the wrong data
  directory; they now describe Home Assistant, its data location, the
  non-root container, the opt-in analytics switch and the upgrade caveat.
  Removed the misleading "keeps all data on your NAS" claim: Home Assistant
  does contact the network by default (alert feed, brand images, Met.no) and
  can reach third-party services when the user configures integrations.
- **C10 (repository and directory compliance):** completed `README.md` with
  Repository layout, How it works, Install and first run, Data and uninstall,
  Privacy, Notices, Build, Verification, Upgrade notes, Limitations and
  Troubleshooting. Removed the stale `probe`/`entrypoint_file` metadata that
  still pointed at `daya0576/beaverhabits` and a non-existent
  `src/entrypoint.sh`.

### Added
- **`PRIVACY.md`** (C3, C8): full policy — what is stored and where, retention
  (recorder default 10 days, long-term statistics kept, analytics 60 days),
  user rights and the deletion channel, security measures, and a
  third-party endpoint table with destination/region and whether each
  endpoint is on by default.
- **`NOTICE.md`** (C2): upstream Apache-2.0 licence (verified against
  `home-assistant/core@2026.11.3`), image source and multi-arch digests,
  bundled Alpine/Python/s6 components, Simple Icons (CC0) icon attribution,
  Home Assistant / TerraMaster trademark notices and the non-affiliation
  statement.

### Changed
- `tosapp.json`: `help` now points at the public `PRIVACY.md`;
  `license`/`upstream_license` are Apache-2.0; `release_note` describes the
  remediation; the beaverhabits `probe` block and the dangling
  `entrypoint_file` were deleted; `image`, `upstream_version` and `version`
  updated.
- `src/config.ini`: `help` points at the public `PRIVACY.md`.
- `src/docker-compose.yml`: image tag, image comment with digest, health-check
  target and first-start grace period.
- `src/homeassistantdocker.lang`: regenerated from `tosapp.json` (23
  languages; `zh-cn`/`zh-hk` rewritten, English fallback for the rest).

### Migration notes — 2026.5.4 → 2026.11.3
- **Back up the data folder before updating.** The recorder database is
  migrated in place on first start and cannot be downgraded.
- The first start after the upgrade can take several minutes and the database
  grows temporarily; make sure the data volume has free space.
- **HACS is now the built-in Marketplace** (2026.11). An existing HACS
  installation is migrated automatically; installed projects keep receiving
  updates and custom repositories carry over.
- **Entity naming changed upstream** (backward-incompatible in 2026.11): some
  entity names on dashboards and in automations may read differently after the
  update. Nothing is deleted.
- The image bundles **Python 3.14**; custom Python integrations that are not
  compatible must be updated by their authors.
- No change to the port (host 8107 → container 8123), the data path or the
  container user (uid/gid 1000); a reinstall reuses the existing data.

## 2026.5.4-2 — 2026-11-15

- Repackaged the 2026.5.4 image for both architectures; submitted to the store
  (this version is the one held in review and superseded by 2026.11.3-1).

## 2026.5.4-1 — 2026-11-12

### Added
- First release. Home Assistant (`homeassistant/home-assistant:2026.5.4`, Docker Hub official image, unmodified)
  packaged as a TOS 7 Docker application, published on port 8107 and running
  as a non-root user (1000:1000).
- Persistent data is kept under `/Volume1/DockerAppData/homeassistantdocker/`.
