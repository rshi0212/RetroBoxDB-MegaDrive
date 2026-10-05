# RetroBoxDB MegaDrive

English | [中文说明](README.zh-CN.md)

Single-file SQLite preservation database for Sega Mega Drive / Genesis. The public Catalog holds metadata only (checksums, DAT and provenance records, header fields, archive recipes and the processing code); it contains no ROM data and cannot restore files. The populated database stays local.

| Item | Value |
| --- | --- |
| Original size | 6,215 source ZIPs, 4.66 GiB (No-Intro 5,281, RetroAchievements sets 934); 6,216 ROM files, 9.61 GiB uncompressed |
| Stored size | populated database 998.5 MiB; public Catalog 44.7 MiB (no ROM data) |
| Ratio | 20.9% of the source ZIPs, 10.2% of the uncompressed ROM files |
| Technology | storage v4: SHA256-deduplicated 64 KiB blocks packed in No-Intro family order into solid LZMA2 groups of up to 256 MiB (256 MiB dictionary); per-block SHA256 and per-object CRC32/MD5/SHA1/SHA256 verification; source ZIPs reproduced byte-for-byte from TorrentZip plans |
| Export performance | Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, Python 3.14.4, all checks included. whole newest-DAT set with `export_set.py` (3,398 files, each checked against the DAT hashes): 18.8 MiB/s, 74 ms per file on average; single file with a cold cache (the group is decoded up to the file): ROM 1.734 s, TorrentZip 2.087 s on average |

## Downloads and documents

| File / document | Content |
| --- | --- |
| [RetroBoxDB.MegaDrive.Catalog.sqlite](https://github.com/rshi0212/RetroBoxDB-MegaDrive/releases/latest/download/RetroBoxDB.MegaDrive.Catalog.sqlite) | Public Catalog (Release asset with `SHA256SUMS`) |
| [Storage v4 guide](RetroBoxDB.Storage-v4.en.md) / [中文](RetroBoxDB.Storage-v4.zh-CN.md) | Storage evaluation, contents, RA, names and maintenance for all seven platforms |
| [Technical design](RetroBoxDB.Storage-v4.Technical-Design.en.md) | Storage format, platform adapters, incremental updates, verification |
| [RA list](reports/ra-megadrive-games.csv) / [summary](reports/ra-megadrive.json), [build report](reports/megadrive-build-report.json), [audit resolution](reports/audit-resolution-20261004.md) | Detailed data |

## Storage choice and platform specifics

Change against 32 MiB groups on real data (first 16 family-ordered groups, 472 MiB): 64 MiB −1.74%, 128 MiB −1.99%, 256 MiB −2.64%; 256 MiB chosen by the rule.

- ROM sizes are 0.1–5 MiB (larger with the SSF mapper); grouping is family-ordered with the largest measured gain from bigger groups among the 64 KiB-block platforms.
- Internal header at 0x100: system type, copyright, domestic/overseas titles, serial, device support, ROM/RAM ranges, SRAM descriptor and regions; the computed checksum is the big-endian word sum from 0x200 to the end of the file. Declared checksums often differ: homebrew and prototypes frequently store 0, and several early licensed releases do not match.
- SMD-interleaved copier files are detected and stored unchanged (none locally). The Aftermarket and Private folders hold most of the local ROMs that are in no DAT.

## Contents

| Item | Value |
| --- | --- |
| ROM records / games / releases | 3,959 / 1,581 / 3,503 |
| DAT coverage per version | 20260714-063411: 3,398/3,486; 20260927-122056: 3,398/3,504 |
| Local ROMs in no DAT | 561 |
| ROM files of the RetroAchievements set | in a No-Intro DAT 654, RA only 275, hash not in the latest RA snapshot 5 ([list](reports/ra-megadrive-collection-unknown.csv)); RA games still without a local ROM: [gap list](reports/ra-megadrive-missing.csv) |
| No-Intro DB Export + Dump Log 20260927-122056 | 3,640 archives, 4,142 file identities, 2,017 documented hardware assertions; Dump Log Verified 868 |
| RetroAchievements (console 1) | 613 games with achievements: 606 with a local ROM (942 ROMs), 0 DAT only, 0 DB file only, 7 without a No-Intro counterpart |
| Chinese names | 2,694 of 2,869 rows translated (1,226 unique); 2,656 local ROMs have a Chinese name |
| Populated-database audit | 3,963 objects, 20 groups, 5,367 archive plans, all passed |

Every source ZIP is reproduced byte-for-byte from its TorrentZip plan (`v_file_checksums.exported_bytes_equal_source`).

## Usage

```bash
# Query-only audit with the Catalog's embedded engine (also: stats, checksums FILE_ID, help)
python3 -B -c 'import sqlite3,sys; c=sqlite3.connect(sys.argv[1]); s=c.execute("SELECT content FROM resources WHERE name=?",("engine.py",)).fetchone()[0]; c.close(); exec(compile(s,"RetroBoxDB:engine.py","exec"))' ./RetroBoxDB.MegaDrive.Catalog.sqlite audit
# Populated database: export by DAT version, 1G1R, RA achievements, TorrentZip or plain ROMs
python3 -B tools/export_set.py RetroBoxDB.MegaDrive.sqlite OUT --set 1g1r --ra achievements --container torrentzip --layout ra-category
# Add new DATs, DB Export / Dump Log snapshots, ROMs and an RA snapshot incrementally
python3 -B tools/update_db.py RetroBoxDB.MegaDrive.sqlite --discover --ra --catalog RetroBoxDB.MegaDrive.Catalog.sqlite
```

Python 3.10+ standard library only. `engine.py` and the other `resources` entries are executable code; run them only from a database you built or a Release asset whose SHA256 you verified. Releases are produced by `.github/workflows/publish-catalog.yml`: it starts from the base Catalog pinned in `release/catalog-release.json`, injects the engine and documents of this commit, checks every data-table digest, runs the tests and the audit, then publishes.
