# NOTICE — third-party components and attributions

This package (`homeassistantdocker`, TOS 7 Docker application, version
`2026.11.3-1`) is an independent packaging by **Moechz** of the upstream
**Home Assistant** project. It is not affiliated with, sponsored or endorsed by
the Home Assistant project, the Open Home Foundation, Nabu Casa or TerraMaster.

## 1. Upstream application — Home Assistant Core

- Project: <https://github.com/home-assistant/core>
- Version packaged: **2026.11.3** (released 2026-11-23; the current upstream
  stable release at the time of this package)
- Licence: **Apache License 2.0** —
  <https://github.com/home-assistant/core/blob/2026.11.3/LICENSE.md>
- Copyright: © the Home Assistant contributors / Open Home Foundation.
- Relationship: the upstream application is used **unmodified**, inside the
  container image. No upstream source code is redistributed in this package.

Home Assistant is a project of the
[Open Home Foundation](https://www.openhomefoundation.org/).

## 2. Container image — `homeassistant/home-assistant:2026.11.3`

- Registry: Docker Hub — `docker.io/homeassistant/home-assistant:2026.11.3`
- Maintained by the Home Assistant project (official image, used unmodified).
- Multi-architecture OCI image index digest:
  `sha256:24e541ddae5818bf8eb27898b9ba00d0c344a655aa5097e9e9bd4b19356be847`
  - `linux/amd64` manifest:
    `sha256:fbbd521bd989893b6c350877d17e1eb1126b6a1bfb715c28e6945f1c82ae0093`
  - `linux/arm64` manifest:
    `sha256:ee500645d61b5483d0b05eebd4f2de22c390618ec178df0e1114a77d40d6d650`
- Both architectures required by the store (`x86_64`, `aarch64`) are present in
  the index. The image is pulled from Docker Hub at install time and is **not
  redistributed** by this package.
- Licence of the image contents: Apache-2.0 for Home Assistant itself, plus the
  licences of the bundled base system and Python dependencies (see §3).

## 3. Components inside the image

The official image is built by the Home Assistant project from Alpine Linux and
includes:

- **Alpine Linux** and its `musl`/BusyBox userland (various permissive
  licences; Alpine's own licence list is shipped in the image);
- **Python 3.14** and the Python standard library (PSF licence);
- **s6-overlay** (ISC licence) as the process supervisor;
- Home Assistant's Python dependencies (e.g. `aiohttp`, `SQLAlchemy`,
  `zeroconf`, `paho-mqtt`, `PyJWT`), each under its own licence. They are
  compiled/installed inside the upstream image and documented in the upstream
  `requirements*.txt`; their licence texts are shipped inside the image. This
  package does not redistribute them separately.

## 4. This packaging

The files authored for this package — `docker-compose.yml`, `config.ini`,
`homeassistantdocker.lang`, `homeassistantdocker.svg`, the build/verification
scripts and this documentation — are released under the **MIT licence** (see
`LICENSE`).

`TOSAppSelfTestingTool` is the package self-test utility provided by
TerraMaster for App Center submissions. It is redistributed here unchanged as
part of the repository's build/test tooling and remains subject to
TerraMaster's terms; it is not covered by the MIT licence of this packaging.

## 5. Trademarks

- **Home Assistant** and the Home Assistant logo are trademarks of
  **Nabu Casa, Inc.** / the Open Home Foundation. Their use here is descriptive
  (to identify the upstream software this package installs) and does not imply
  endorsement.
- **TerraMaster**, **TOS** and **App Center** are trademarks or registered
  trademarks of TerraMaster.
- All other product names, logos and brands are the property of their
  respective owners.

This package is an independent community submission and is **not affiliated
with, sponsored or endorsed by** Nabu Casa, the Open Home Foundation, the Home
Assistant project or TerraMaster.

## 6. Icon

The application icon in `src/homeassistantdocker.svg` is the Home Assistant
glyph from **[Simple Icons](https://simpleicons.org/)** (released under
**CC0 1.0 Universal**), recoloured for this package. The Home Assistant mark
itself remains a trademark of Nabu Casa, Inc.; it is used only to identify the
application being packaged.

## 7. Contact

Moechz — **moechz@163.com** — <https://github.com/Moechz/home-assistant>
