# RetroBoxDB MegaDrive

English | [中文说明](README.zh-CN.md)

A single-file SQLite archive design for Sega Mega Drive / Genesis preservation: exact ROM identities, deduplicated storage, DAT validation across No-Intro snapshots, dump provenance, internal-header metadata, RetroAchievements hash matching, English/Chinese names and checksummed TorrentZip export plans. It is the cartridge sibling of [RetroBoxDB-NES](https://github.com/rshi0212/RetroBoxDB-NES).

**The public Catalog contains no ROM bodies, original DAT/DB/Dumplog payloads, compressed content groups or media.** The populated database stays local. The Catalog keeps metadata, expected checksums, header fields, archive recipes, provenance and the processing source code; it cannot restore or export files.

| Download / document | Purpose |
| --- | --- |
| [RetroBoxDB.MegaDrive.Catalog.sqlite](https://github.com/rshi0212/RetroBoxDB-MegaDrive/releases/latest/download/RetroBoxDB.MegaDrive.Catalog.sqlite) | Metadata-only SQLite (42 MiB), attached to GitHub Releases |
| [Technical design](RetroBoxDB.Cartridge.Technical-Design.en.md) | Storage v4, adapters, incremental updates, RetroAchievements, verification |
| [中文说明](RetroBoxDB.Cartridge.zh-CN.md) | Full Chinese guide for SNES and Mega Drive |
| [RA coverage](reports/ra-megadrive-games.csv) / [summary](reports/ra-megadrive.json) | Every RetroAchievements game with achievements and its match status |
| [Build report](reports/megadrive-build-report.json) | Import, scan, diff, No-Intro, packages, names and audit results |

## Collection

| | |
| --- | --- |
| Local No-Intro ZIPs (incl. Aftermarket/Private) | 5,281 files, 3.93 GiB |
| Populated database | 0.95 GiB (24.2% of the source ZIPs) |
| ROM records / games / releases | 3,687 / 1,581 / 3,503 |
| DAT 20260714-063411 coverage | 3,398/3,486 |
| DAT 20260927-122056 coverage | 3,398/3,504 |
| DAT diff (old → new) | 3,474 unchanged, 18 added, 11 renamed, 1 case change |
| No-Intro DB Export + Dump Log 20260927-122056 | 3,640 archives, 4,142 file identities, 9,152 sources, 2,017 documented hardware assertions; Dump Log 868 Verified |

## Storage chosen by measurement

The NES layout (8 KiB blocks, XOR deltas, 2 MiB LZMA groups) was not copied blindly. A 10% random family sample compared seven strategies; MD sample: 408 MiB ZIP → 217 MiB with the NES v3 engine → 146 MiB with storage v4 (−33%). Storage v4 deduplicates 64 KiB blocks by SHA256 and packs them in No-Intro family order (parent and clones together) into solid LZMA2 groups of at most 32 MiB. Here, 3.87 GiB of unique blocks are stored as 0.90 GiB in 143 groups. Every block keeps its own SHA256, every object is verified against CRC32/MD5/SHA1/SHA256, and the v3 engine refuses v4 files rather than misreading them. See [assessment/data/cart-storage-experiment.json](assessment/data/cart-storage-experiment.json).

## RetroAchievements

A snapshot of the public RA API (console 1) is stored in `ra_games` / `ra_hashes`; no credentials are stored. Each ROM's RA hash is computed with rcheevos rules. Of 612 RA games with achievements, **520 have a matching local ROM** (673 ROMs), **none is missing among games No-Intro lists**, 1 matches only a No-Intro DB bad dump, and 91 (65 hacks, 14 official sets that target translation patches or subsets).

```sql
SELECT * FROM v_rom_ra_matches WHERE has_achievements;
SELECT * FROM v_dat_ra_matches WHERE has_achievements AND NOT local_rom_available;
```

## Names

2,694 of 2,869 rows translated (1,226 unique names); 2,550 direct + 117 inherited releases; 2,656 local ROMs.

```sql
SELECT * FROM v_release_effective_chinese_names WHERE catalog_title LIKE '%Mario%';
```

## Using the Catalog

```bash
python3 -B -c 'import sqlite3,sys; c=sqlite3.connect(sys.argv[1]); s=c.execute("SELECT content FROM resources WHERE name=?",("engine.py",)).fetchone()[0]; c.close(); exec(compile(s,"RetroBoxDB:engine.py","exec"))' ./RetroBoxDB.MegaDrive.Catalog.sqlite audit
```

Replace `audit` with `stats`, `checksums FILE_ID` or `help`. The catalog engine is query-only and reports `payloads_verified=false`.

## Building and growing your own database

Python 3.10+ standard library only. `tools/build_cart_db.py megadrive FULL.sqlite --catalog CATALOG.sqlite` builds from local No-Intro inputs (read-only). `tools/update_cart_db.py FULL.sqlite --discover --ra --catalog CATALOG.sqlite` adds new DATs, DB Export/Dump Log snapshots and ROMs idempotently and repacks new revisions into their family's solid group. Tests: `python3 -B -m unittest tests/test_cart.py tests/test_game_names.py`.

Audit of the populated database: 3,691 objects, 143 solid groups, 5,044 archive plans; SQLite integrity and foreign keys clean; no errors.
