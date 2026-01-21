# zfs_snapper

**Simple, robust, cron-friendly ZFS snapshot**  
(simply in pure bash, no GUI, no Python)

This single bash script creates and maintains snapshots of single or all zfs zpools recurively (datasets as zvols) and held snapshots for a number of days (default 5) and a number of weeks (default 2).


[![bash](https://img.shields.io/badge/language-bash-brightgreen.svg)](https://www.gnu.org/software/bash/)
[![ZFS](https://img.shields.io/badge/ZFS-on--Linux%20%7C%20FreeBSD%20%7C%20TrueNAS-blue.svg)](https://openzfs.github.io/openzfs-docs/)

## Features

- Creates **recursive** snapshots with identical names on all child datasets  
  → `tank@snapper_daily_2025-12-08_14:27:39`, `tank/home@…`, `tank/lxd1/root@…`, etc.
- Daily + weekly snapshots (weekly on Sundays)
- Automatic pruning with configurable retention
- could be combined with simple syncoid call to sync snapshots to backup pools
- TrueNAS-style **incremental replication** to a local backup pool (`backup/tank`, `backup/rpool`, …)
- Backup datasets are automatically set `readonly=on`
- Same retention policy applied on the backup side
- Works perfectly with deeply nested datasets (VMs, LXC/LXD, jails, etc.)
- Pure bash, no external tools required (except standard ZFS commands)
- Dry-run mode, verbose mode, quiet mode → perfect for cron
- Battle-tested on TrueNAS SCALE, Proxmox, Ubuntu, FreeBSD, and vanilla OpenZFS

## Requirements

- OpenZFS ≥ 2.0 (the `-I` / `-i` behavior we rely on works perfectly from 2.0+)
- Root privileges (or a user with proper ZFS permissions)
- A dedicated backup pool named `backup` (you can change the name in the script)

## Configuration
```bash
SNAP_PREFIX="snapper"                 # prefix for all snapshots
DAILY_RETENTION_DAYS=5
WEEKLY_RETENTION_DAYS=14
```

## Quick Start

```bash
# 1. Copy the script
sudo cp zfs_snapper /usr/local/sbin/zfs_snapper
sudo chmod +x /usr/local/sbin/zfs_snapper

# 2. First full replication (you only do this once)
sudo zfs_snapper --all --dosnap --verbose

# 3. Set up cron 
# Daily snapshots of all pools + pruning at 2 AM
0 2 * * * /usr/local/sbin/zfs_snapper --all --dosnap --doprune --quiet

or if you want only pool "tank" snappped:

# Daily snapshots (and show new snapshots)
0 2 * * * /usr/local/sbin/zfs_snapper tank --dosnap --doprune --list-new
```

## Usage / Options
```
Usage: zfs_snapper [OPTIONS] [zpool ...|--all]

  --dosnap          Actually create new snapshots
  --doprune         Actually delete old snapshots
  --verbose         Extra debug output
  --list-new        Show snapshots that would be/were created
  --list-all        Show all snapper_* snapshots at end
  --quiet|-q        Minimal output
  --all             Process all pools
  -h|--help         This help

Examples:
  zfs_snapper                  → full dry-run (show what would happen)
  zfs_snapper --dosnap         → create new snapshots only
  zfs_snapper --doprune        → prune old snapshots only
  zfs_snapper --dosnap --doprune   → normal daily run (both)
```
