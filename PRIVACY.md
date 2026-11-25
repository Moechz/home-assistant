# Privacy Policy — Home Assistant for TerraMaster TOS (`homeassistantdocker`)

**Last updated:** 25 November 2026
**Data controller / contact:** Moechz — moechz@163.com
**Applies to:** version `2026.11.3-2` and later of the TOS application
`homeassistantdocker`, the container image it runs and this repository.

## 1. Summary

- Home Assistant runs **on your NAS**. The packager (Moechz) does not operate a
  server, does not receive your data and cannot access your instance.
- There is **no packager telemetry**, no account with the packager and no
  registration, licence key or e-mail address required to use the app.
- Home Assistant itself is **local-first**. Out of the box it contacts the Home
  Assistant alert feed and brand-image host, and — if you set a home location
  during onboarding — the Norwegian Met.no weather service (see §6).
- **Usage analytics is opt-in and off by default.** Nothing is sent to the
  Home Assistant analytics receiver unless you switch it on yourself
  (§6.1). This was verified in the 2026.11.3 image: every analytics preference
  defaults to `false`, and the onboarding screen shows the switches off.
- The app can reach **many third-party services**, but only the ones **you**
  configure: cloud integrations (e.g. Tuya, Xiaomi), TTS/LLM providers, DDNS
  providers, e-mail/notification services and so on. §6 lists the categories
  and the endpoints they use; you choose each one in the UI.

## 2. What data is processed and where it is stored

All persistent data lives in **one folder**, mounted as `/config` in the
container:

`/Volume1/DockerAppData/homeassistantdocker/config`

| Data | Stored at (inside the folder) | Notes |
|---|---|---|
| Owner account and additional users: name, username, salted password hash, refresh tokens | `.storage/auth`, `.storage/auth_provider.homeassistant` | No password is ever sent to the packager or to Home Assistant's project servers |
| Integrations you add, including their credentials (API keys, OAuth tokens, cloud logins) | `.storage/core.config_entries` | Stored locally in JSON; protect the data folder |
| Devices, entities, areas, floors, labels, dashboards and their settings | `.storage/core.*`, `.storage/*` | Includes the entity/device registry |
| Automation, script, scene and helper definitions | `automations.yaml`, `scripts.yaml`, `scenes.yaml`, `.storage/` | |
| State history and long-term statistics (the "recorder" database) | `home-assistant_v2.db` (+ `-wal`, `-shm`) | SQLite by default; see §3 for retention |
| Home Assistant log | `home-assistant.log`, `.1`, `.fault` | stdout/stderr is also captured by Docker (`docker logs homeassistantdocker`) |
| Text-to-speech cache | `tts/` | Regenerated on demand |
| Custom integrations/dependencies you install | `deps/`, `custom_components/` | |
| Blueprints you import | `blueprints/` | |
| Backups you create in the UI | `backups/` (default location) | Contain a full copy of the configuration and database |
| Home Assistant Cloud preferences | `.cloud/` | Only after you sign in to Home Assistant Cloud |

The container's own `/tmp` is discarded with the container. The SQLite
database, the `.storage` JSON files and the log are the only files Home
Assistant writes on the NAS; everything else is read-only image content.

**What is *not* collected:** the packager has no analytics, crash reporting or
"phone home" of its own. The package contains no telemetry code.

## 3. Retention period

| Data | Retention |
|---|---|
| Entities/state history in the recorder | **10 days** by default (`recorder.purge_keep_days`), then purged automatically |
| Long-term statistics | Kept indefinitely (they are the source for the Energy dashboard); delete them manually if you do not want them |
| Configuration, accounts, registries, automations | Until **you** delete them; there is no expiry |
| Logs | Rotated by Home Assistant; Docker keeps container stdout/stderr according to the daemon's log rotation |
| TTS cache | Until cleared; safe to delete at any time |
| Backups | Until you delete them; they are files in your data folder |
| Analytics (only if **you** opt in) | Home Assistant's analytics receiver keeps the per-installation record for at most **60 days** after the last update |
| Device-database snapshots (Labs preview, off by default) | Only while the Labs feature is enabled; stored by the Open Home Foundation device database |

## 4. Your rights and the deletion channel

| Right | How |
|---|---|
| Access / export | **Settings → System → Backups → Create backup** produces a full archive of `/config` (configuration, database, accounts, integrations). You can also copy `/Volume1/DockerAppData/homeassistantdocker/` at any time. |
| Correct | Edit users, integrations, entities and automations in the Home Assistant UI; edit the YAML files while the container is stopped. |
| Delete (partial) | Delete entities, devices, history ranges, statistics, integrations, users or backups in the UI. |
| Delete (everything) | Uninstall the app in App Center **with "delete data"** ticked, or stop the container and remove the folder: `sudo rm -rf /Volume1/DockerAppData/homeassistantdocker`. This is irreversible. |
| Analytics data | **Settings → System → Analytics**: switch the options off. The receiver deletes the record automatically within 60 days of the last update. |
| Assistance | E-mail **moechz@163.com** with the subject "Home Assistant data request"; a reply follows within 30 days. |

The packager holds no copy of your data, so access and deletion are normally
completed on your own NAS. The e-mail channel exists for questions about this
policy and for requests concerning the repository or the store listing.

## 5. Security measures

- The container runs as **uid/gid 1000**, never as root, with no `privileged`
  mode, no `cap_add`, no host network and no Docker socket.
- Only the application's own port is published (host **8107** → container 8123).
- The image is referenced by a **fixed tag** (`homeassistant/home-assistant:2026.11.3`,
  multi-arch index `sha256:24e541ddae5818bf8eb27898b9ba00d0c344a655aa5097e9e9bd4b19356be847`),
  never `:latest`; the package is rebuilt when upstream ships security fixes.
- Home Assistant stores passwords as salted hashes and issues its own
  long-lived refresh tokens; protect the data folder, because integration
  credentials are stored there in clear text JSON.
- The UI is plain **HTTP** on the LAN. Do not expose port 8107 directly to the
  internet: put it behind a TLS reverse proxy (or use a VPN) and enable
  Home Assistant's own authentication (on by default).
- The data folder is owned by the application user; back it up before
  uninstalling and keep NAS backups encrypted if they leave the device.

## 6. Third-party sharing and API disclosure (C8)

The packager shares **nothing**. The table below lists every endpoint the
application can contact, what is sent, where it goes and whether it is on by
default. "Default" means: with a fresh install where you complete onboarding
and do not add any integration.

### 6.1 On by default (core / onboarding)

| # | Endpoint | What is sent | Destination / region | When |
|---|---|---|---|---|
| 1 | `https://alerts.home-assistant.io/alerts.json` | A plain GET; the server sees your public IP and User-Agent | Home Assistant project (Open Home Foundation), hosted in the **US/EU** | Every 3 hours. Set `homeassistant_alerts:` disabled or block the host to stop it |
| 2 | `https://brands.home-assistant.io/...` | A GET for integration brand images; the server sees your IP and User-Agent | Home Assistant project (Open Home Foundation), **US/EU** | When the UI shows integration logos (your browser makes the request) |
| 3 | `https://aa015h6buqvih86i1.api.met.no/weatherapi/locationforecast/2.0/complete` | Your home latitude/longitude (as set in onboarding) and the request headers | **MET Norway**, Norway (EEA) | When the Met.no weather entity is set up/refreshed. Remove the Met.no integration to stop it |
| 4 | `registry-1.docker.io` (Docker Hub) | Anonymous image pull; Docker Hub sees your IP | Docker, **US** (or your configured registry mirror) | Install and package updates only |
| 5 | `https://my.home-assistant.io/redirect/...` | The target path only; the server sees your IP and User-Agent | Home Assistant project, **US/EU** | Only when you click a "My Home Assistant" badge/link |
| 6 | `https://analytics-api.home-assistant.io/v1` | **Off by default — only if you opt in.** Basic: installation UUID, version, installation type, country (derived server-side from your IP). Usage: names/versions of your integrations, database engine, energy/HTTP-certificate flags. Statistics: counts of integrations/users/entities/automations. Diagnostics (Supervisor/OS only, **not** this Container install): crash report to Sentry | Home Assistant analytics receiver on Cloudflare, **US/global** | 15 minutes after start, then every 24 h, while enabled |
| 7 | `https://device-database.eco-dev-aws.openhomefoundation.com/api/v1/snapshot/1` | **Off by default (Labs preview).** Device-database snapshot: device/entity metadata, no account data | Open Home Foundation device database, **US** | Only while the Labs "device database snapshots" feature is enabled |

Nothing else is contacted by default. Home Assistant does **not** send your
configuration, automation contents, entity states, camera streams or location
history to the packager or to the Home Assistant project.

### 6.2 Optional features you can turn on

| Feature | Endpoint(s) | What is sent | Region |
|---|---|---|---|
| Marketplace (new in 2026.11, replaces HACS) | `api.github.com`, `codeload.github.com`, optional China mirrors (e.g. `ghproxy`/community mirrors) | Repository and release metadata; the projects you download | GitHub, **US**; mirrors as chosen by you |
| Version / update checks (`version` integration, optional) | `pypi.org/pypi/homeassistant/json`, `version.home-assistant.io`, `hub.docker.com`, `api.github.com` | The installed version and the requested project name | **US/EU** |
| Home Assistant Cloud (Nabu Casa) | `api.nabucasa.com`, `cloud.nabucasa.com` | Your Nabu Casa account, subscription status, remote-UI/voice traffic | Nabu Casa, **US** |
| Google Translate TTS | `translate.google.com` | The text you ask Home Assistant to speak | Google, **US** |
| Radio Browser | `all.api.radio-browser.info` | Search terms and station IDs; your IP | Community-run mirrors, **EU** |
| DDNS (DuckDNS / No-IP etc.) | `www.duckdns.org`, `dynupdate.no-ip.com` | Your current public IP, hostname and provider token | **US** |
| Notifications / e-mail | `ntfy.sh`, `api.telegram.org`, `graph.microsoft.com`, your SMTP server, … | The notification content and your credentials for that service | Wherever **you** chose |
| Backups to cloud | Google Drive, OneDrive, Dropbox, Azure, S3, SFTP, Synology DSM, Backblaze B2, … | The backup archive (full configuration and database) | Wherever **you** configured |
| Any integration you add (Tuya, Xiaomi Mi Cloud, Philips Hue, MQTT, Matter, Z-Wave, weather, energy, LLM/AI, …) | The vendor's own API, e.g. `openapi.tuyacn.com`/`openapi.tuyaus.com`, `api.io.mi.com`, `api.openai.com`, `api.anthropic.com`, local device IPs | Whatever that integration needs: device states, commands, your account token, and — for cloud integrations — the device/entity data the vendor requires | The vendor's region; local integrations stay on your LAN |

**How to stop any of it:** remove the integration/feature in **Settings →
Devices & services** (or Settings → System → Analytics / Labs for the two
Home Assistant services), or block the host at your router/DNS. Blocking
`alerts.home-assistant.io` and `brands.home-assistant.io` only disables update
alerts and brand images; Home Assistant keeps working.

## 7. Cookies and local storage

The Home Assistant frontend stores UI preferences (theme, sidebar order,
selected dashboard, language) in your browser's `localStorage`. The packager
sets no cookies and adds no tracking code. The app does not use third-party
analytics or advertising scripts.

## 8. Children

The app is a general-purpose home-automation tool and is not directed at
children. The packager does not knowingly collect data from anyone, including
children.

## 9. Changes to this policy

Material changes are published in this file in the repository and noted in
`CHANGELOG.md`; the "Last updated" date at the top is bumped. The policy that
applies to your installed version is the one shipped with that version's
release tag.

## 10. Contact

Moechz — **moechz@163.com** — <https://github.com/Moechz/home-assistant>
