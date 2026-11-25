# home-assistant-for-tos

> TerraMaster TOS 7 App Center — **Docker application** package for
> [Home Assistant](https://github.com/home-assistant/core) (upstream
> **2026.11.3**).

## What this is

A four-file TOS Docker application archive (`homeassistantdocker.tar.gz`) that
installs Home Assistant on TOS 7 through the App Center:

- `config.ini` — TOS application metadata (`application_type: docker`, id
  `homeassistantdocker`)
- `homeassistantdocker.lang` — 23-language superset (TOS requires 14)
- `homeassistantdocker.svg` — application icon
- `docker-compose.yml` — one service, image
  `homeassistant/home-assistant:2026.11.3`

The archive is built for both store architectures (`x86_64`, `aarch64`) from
the same sources; only the `platform` field differs.

## How it works

| | |
|---|---|
| Container image | `homeassistant/home-assistant:2026.11.3` (Docker Hub, official multi-arch image, used unmodified) |
| Image digests | index `sha256:24e541ddae5818bf8eb27898b9ba00d0c344a655aa5097e9e9bd4b19356be847`; amd64 `sha256:fbbd521b…`; arm64 `sha256:ee500645…` |
| Open | `http://<NAS-IP>:8107/` (container port 8123) |
| Runs as | non-root, `user: "1000:1000"` |
| Persistent data | `/config` → `/Volume1/DockerAppData/homeassistantdocker/config` |
| Logs | `docker logs homeassistantdocker` (stdout/stderr) and `config/home-assistant.log` |
| Health check | `curl -fsS http://127.0.0.1:8123/manifest.json`, 180 s first-start grace |

## Repository layout

```text
home-assistant/
├── PRIVACY.md                   # privacy policy, third-party endpoint table (C3/C8)
├── NOTICE.md                    # upstream licence, image digests, trademarks (C2)
├── CHANGELOG.md                 # per-version remediation notes
├── LICENSE                      # MIT for this packaging
├── README.md                    # this file
├── src/
│   ├── config.ini               # TOS metadata (id, version, port, help URL)
│   ├── homeassistantdocker.lang # 23-language descriptions (C9)
│   ├── homeassistantdocker.svg  # icon (Simple Icons, CC0)
│   └── docker-compose.yml       # one service, healthcheck, volume, x-app-meta
├── scripts/
│   ├── build.sh                 # builds one architecture, runs tools/verify.py
│   └── build-dual.sh            # builds aarch64 + x86_64 and runs the official -c
├── tools/
│   └── verify.py                # review-standards self-check (hard gate)
├── test/
│   ├── install-and-verify.sh    # on-device install test (official tool -i; admin)
│   └── compose-resolved/        # last staged Compose file (build artefact)
├── TOSAppSelfTestingTool        # official TerraMaster checker
└── out/                         # built assets (git-ignored)
```

The source repository is public and contains everything needed to rebuild the
packages. The archive itself contains exactly the four files TOS expects.

## Install and first run

1. Install **DockerEngine** from App Center if it is not already installed.
2. Install **Home Assistant** from App Center.
3. Open `http://<NAS-IP>:8107/` and complete onboarding: create the owner
   account, set your home location, and (optionally) opt in to analytics.
4. Add your first integration under **Settings → Devices & services**.

First start can take **a few minutes** on a NAS, especially on ARM models: the
container's HTTP port answers quickly but the onboarding page becomes usable
only once Home Assistant has finished initialising. The package's health check
allows 180 s of grace; reload the page if it is not ready yet.

No account with the packager, licence key or registration is required.

> App Center's **manual** install page accepts `.deb` packages only — it cannot
> install a Docker application tarball. Docker packages are installed through
> the App Center catalog (online install) or, for testing, with the official
> `TOSAppSelfTestingTool -i` on a test NAS. This is a platform limitation, not
> a package defect.

Requires about **2 GB RAM** free for Home Assistant and its dependencies.

## Data and uninstall

All persistent data is below one directory:

`/Volume1/DockerAppData/homeassistantdocker/`

| Host path | Container path | Contents |
|---|---|---|
| `…/homeassistantdocker/config` | `/config` | `configuration.yaml`, `automations.yaml`, `.storage/` (accounts, integrations, registries), `home-assistant_v2.db` (history + statistics), `home-assistant.log`, `tts/`, `deps/`, `blueprints/`, backups |

The container's `/tmp` is discarded with the container; nothing else is
written on the NAS.

**Backup** (stop the container for a consistent copy):

```bash
docker compose -p homeassistantdocker stop
sudo tar -C /Volume1/DockerAppData -czf ha-backup-$(date +%F).tar.gz homeassistantdocker
docker compose -p homeassistantdocker start
```

Home Assistant's own **Settings → System → Backups** creates a restorable
archive in `config/backups/`.

**Uninstall and keep data:** uninstall the app in App Center (or
`docker compose -p homeassistantdocker down`). The data directory is left in
place; reinstalling reuses the existing installation.

**Uninstall and delete everything:**

```bash
docker compose -p homeassistantdocker down -v
sudo rm -rf /Volume1/DockerAppData/homeassistantdocker
```

This permanently removes the configuration, database, accounts and backups.
There is no packager-side copy. See `PRIVACY.md` §2–§4 for the full data
inventory and deletion channel.

## Privacy

The full policy is in **[PRIVACY.md](PRIVACY.md)** and is also the package's
`help` link (`config.ini` → `help`, `tosapp.json` → `help`). In short:

- Home Assistant runs entirely on your NAS; the packager operates no server
  and receives no data.
- **Usage analytics is opt-in and off by default.** It can be enabled or
  disabled at **Settings → System → Analytics**; when enabled it sends the
  data described in `PRIVACY.md` §6.1 to `analytics-api.home-assistant.io`.
- By default the app contacts only the Home Assistant alert feed
  (`alerts.home-assistant.io`, every 3 h), brand images
  (`brands.home-assistant.io`) and — if you set a home location — Met.no
  weather (`api.met.no`).
- Every optional endpoint (Marketplace/GitHub, cloud integrations, TTS/LLM
  providers, DDNS, notifications, cloud backups, Docker Hub) is listed with
  what it sends and where, in `PRIVACY.md` §6.2. They are only used when you
  configure them.

## Notices

See **[NOTICE.md](NOTICE.md)** for the upstream Apache-2.0 licence, the exact
image tag and multi-arch digests, the bundled Alpine/Python/s6 components,
the Simple Icons (CC0) icon, the Home Assistant / TerraMaster trademarks and
the non-affiliation statement. The packaging files are MIT licensed
(`LICENSE`).

## Build

Requirements: Python 3, `tar`/`gzip`, and the bundled
`TOSAppSelfTestingTool` executable for the official check. Building needs no
network access.

```bash
# both architectures, self-check + official -c
bash scripts/build-dual.sh 2026.11.3-2

# one architecture
bash scripts/build.sh 2026.11.3-2 x86_64
bash scripts/build.sh 2026.11.3-2 aarch64
```

The build:

1. stamps `version`/`platform` into `tosapp.json` and `config.ini`;
2. removes CRLF/BOM/AppleDouble files;
3. runs `tools/verify.py` (archive layout, JSON validity, required fields,
   14+ languages, version consistency, Docker-Hub-only images, fixed tag,
   reserved/recommended host ports, non-root user, per-service
   healthcheck/TZ/restart, data path below the application data root, no
   `/Volume*` wildcard, no comments inside sequence blocks, `x-app-meta`
   position, no literal secrets, icon limits);
4. packs a deterministic GNU tar + gzip archive and writes `.sha256`;
5. `build-dual.sh` runs `TOSAppSelfTestingTool -c` on both packages.

## Verification

| Gate | Command | Result |
|---|---|---|
| Build self-check | `scripts/build-dual.sh 2026.11.3-2` | `verify OK`, two packages |
| Official check | `TOSAppSelfTestingTool -c out/*.tar.gz` | `check passed` (x86_64 + aarch64) |
| Runtime staging | `python3 stage.py work/home-assistant` | containers healthy, HTTP response, uid 1000 |

Runtime verification uses an isolated Compose project (temporary host port,
temporary data directory) and never touches `/Volume1/DockerAppData`.

## Upgrade notes — 2026.5.4 → 2026.11.3

The app data (configuration, accounts, integrations, database) is compatible
and is **migrated automatically on first start**. There is no downgrade path:

1. **Back up first** — Settings → System → Backups, or copy the data folder
   (`PRIVACY.md` §4). The database is upgraded in place.
2. Install the update and wait for the first start to finish (it can take
   several minutes while the recorder schema is migrated).
3. Check **Settings → System → Logs** for migration errors.
4. Upstream changes worth knowing:
   - **2026.11** replaces HACS with the built-in **Marketplace**; an existing
     HACS installation is migrated automatically and installed projects keep
     receiving updates.
   - **Entity naming changed** (backward-incompatible upstream): some entity
     names shown on dashboards/automations may look different after the
     upgrade. Nothing is deleted.
   - The image now bundles **Python 3.14**; custom Python integrations that
     are not compatible must be updated by their authors.
5. `home-assistant_v2.db` grows during the migration; make sure the data
   volume has free space.

## Limitations and security notes

- The UI is **plain HTTP** on port 8107 (container 8123). Put it behind a TLS
  reverse proxy before exposing it outside the LAN; Home Assistant's own
  authentication is enabled by default.
- Some integrations need direct access to USB/Bluetooth/Zigbee/Z-Wave
  hardware. The container has no `privileged` flag, no host network and no
  Docker socket; devices must be passed through explicitly if you need them.
- The image is multi-arch (`linux/amd64`, `linux/arm64`), matching the
  `x86_64` and `aarch64` packages.
- Port 8107 is shared with other applications in the store by design; if
  another installed app already uses it, uninstall one of them or change the
  mapping in your own Compose override. Port conflicts are not a package
  defect and the package port is not changed for them (RUNBOOK §12).

## Troubleshooting

| Symptom | What to check |
|---|---|
| App Center install completes but the page is not ready | First start takes minutes. Wait and reload; `docker logs homeassistantdocker` shows progress |
| Page cannot be opened at all | `docker ps` — is `homeassistantdocker` running? `docker logs` for image pull errors; check that host port 8107 is free |
| Image pull fails | Docker Hub unreachable → configure a registry mirror in DockerEngine settings |
| Login forgotten | Stop the container and use Home Assistant's `auth` reset procedure on `config/.storage/auth` (back up first) |
| Database errors after upgrade | Restore the backup taken before the upgrade |

## Support

- Packaging issues: <https://github.com/Moechz/home-assistant/issues>
- Upstream documentation: <https://www.home-assistant.io/docs/>
- Privacy / data requests: **moechz@163.com**

---

Packaged for TOS 7 by Moechz. Upstream image used unmodified from Docker Hub.
Not affiliated with Nabu Casa, the Open Home Foundation or TerraMaster.
