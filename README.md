qBittorrent for OpenWrt
========================

Сборка qBittorrent (webui-only, без GUI) и LuCI-приложения для OpenWrt с использованием Qt6 и libtorrent-rasterbar.

[![Build OpenWrt packages](https://github.com/akorshun/luci-app-qbittorrent/actions/workflows/build.yml/badge.svg)](https://github.com/akorshun/luci-app-qbittorrent/actions/workflows/build.yml)
[![Latest release](https://img.shields.io/github/v/release/akorshun/luci-app-qbittorrent?include_prereleases)](https://github.com/akorshun/luci-app-qbittorrent/releases)

## Скачать готовые пакеты

Готовые сборки лежат в [Releases](https://github.com/akorshun/luci-app-qbittorrent/releases):

- **`nightly`** — собирается автоматически на каждый push в `master` (rolling prerelease).
- **`vX.Y.Z`** — стабильные релизы (создаются push-ом git-тега `v*`).

Сборка идёт под две версии OpenWrt:

| OpenWrt | Формат | Toolchain |
|---------|--------|-----------|
| 24.10.6 | `.ipk` | GCC 13.3 / musl |
| 25.12.4 | `.apk` | GCC 14.3 / musl |

Поддерживаемые архитектуры:

| Архитектура | Типичные устройства |
|-------------|---------------------|
| `aarch64_cortex-a53` | RPi 3/4, NanoPi R2S/R4S, бюджетные ARMv8 SoC |
| `aarch64_cortex-a72` | RPi 4, RK3399, OrangePi 4 |
| `aarch64_cortex-a76` | RK3588, NanoPi R6S, BPi-R3-Mini |
| `aarch64_generic` | Generic ARMv8 (Mikrotik RB5009, Xiaomi AX9000 и др.) |
| `x86_64` | x86 роутеры, мини-PC, виртуалки |

> 32-битные ARM/MIPS/PowerPC не входят в сборку: на их дефолтных кернелах в OpenWrt не включён `KERNEL_IO_URING`, без которого недоступен `liburing` — обязательная зависимость `qt6base`. На таких таргетах нужно собирать Qt6 локально с патчем или с другим toolchain.

Каждый zip покрывает одну архитектуру + версию SDK, имя: `qbittorrent-<arch>-<sdk>.zip`.

## Установка

Узнайте архитектуру роутера:

```sh
# OpenWrt 24.10
opkg print-architecture

# OpenWrt 25.12
apk info --print-arch
```

Скачайте подходящий zip из [Releases](https://github.com/akorshun/luci-app-qbittorrent/releases), распакуйте, закиньте `.ipk`/`.apk` в `/tmp/qbt` на роутере (например через `scp`).

### OpenWrt 24.10 (ipk)

```sh
opkg update
cd /tmp/qbt
opkg install ./*.ipk

/etc/init.d/qbittorrent enable
/etc/init.d/qbittorrent start
```

`opkg` сам подтянет `boost-*`, `libopenssl3` и остальные deps из официальных репозиториев OpenWrt и установит локальные пакеты в правильном порядке зависимостей.

### OpenWrt 25.12 (apk)

```sh
apk update
cd /tmp/qbt
apk add --allow-untrusted ./*.apk

/etc/init.d/qbittorrent enable
/etc/init.d/qbittorrent start
```

### Ручной порядок (если автоматика не справилась)

Если `opkg install ./*.ipk` ругается на конфликт версий или порядок — ставьте вручную:

```sh
# 1. Зависимости из официальных репов
opkg update
opkg install boost-system boost-program_options boost-thread \
             boost-filesystem boost-random libopenssl3

# 2. Qt6 base-модули (в порядке зависимостей)
opkg install ./libQt6Core_*.ipk
opkg install ./libQt6Network_*.ipk
opkg install ./libQt6Sql_*.ipk
opkg install ./libQt6Xml_*.ipk

# 3. Qt6 плагины
opkg install ./qt6-plugin-libqopensslbackend_*.ipk
opkg install ./qt6-plugin-libqsqlite_*.ipk

# 4. libtorrent → qbittorrent → LuCI app
opkg install ./rblibtorrent_*.ipk
opkg install ./qbittorrent_*.ipk
opkg install ./luci-app-qbittorrent_*.ipk
```

Для 25.12 заменить `opkg install` на `apk add --allow-untrusted`, расширения файлов на `.apk`.

## После установки

- Web UI: `http://<router-ip>:8080`
- Логин: `admin`, временный пароль — в логах:
  ```sh
  logread | grep -i qbittorrent
  ```
- В LuCI: **Services → qBittorrent** — управление автозапуском и параметрами демона.

## Сборка из исходников

Если хотите собрать локально через OpenWrt SDK / buildroot:

```sh
git clone https://github.com/akorshun/luci-app-qbittorrent package/qbittorrent-feed
make menuconfig   # LuCI → Applications → luci-app-qbittorrent
make package/luci-app-qbittorrent/compile V=s
```

## CI

Workflow [`build.yml`](.github/workflows/build.yml) использует [openwrt/gh-action-sdk](https://github.com/openwrt/gh-action-sdk) и собирает 29 архитектур × 2 SDK = 56 джобов параллельно. Артефакты живут 30 дней; релизы — бессрочно.

## Версии

| Компонент | Версия |
|-----------|--------|
| qBittorrent | 5.2.0 |
| libtorrent-rasterbar | 2.0.12 |
| Qt6 | 6.11.1 |

## Лицензия

- `luci-app-qbittorrent` — Apache 2.0
- `qbittorrent` — GPL-2.0+
- `rblibtorrent` — BSD
- `qt6base`, `qt6tools` — LGPL-3.0
