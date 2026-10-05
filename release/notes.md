MegaDrive Catalog, storage v4 (64 KiB blocks, 17 solid LZMA2 groups of up to 256 MiB). Metadata only: **no ROM payloads are published**; `compression_groups`, `chunks` and `object_chunks` are empty.

- Solid groups merged from 32 MiB to 256 MiB, measured on the full database.
- Unified engine shared with NES, SNES, Game Boy, Game Boy Color and Game Boy Advance.
- Source: 5,281 No-Intro ZIPs, 3.93 GiB (5,282 ROM files, 8.25 GiB uncompressed). Populated database: 936.4 MiB (23.3% of the ZIPs). All source ZIPs are reproduced byte-for-byte.
- Contents: 3,687 ROM records, 1,581 games, 3,503 releases; DAT versions: 20260714-063411, 20260927-122056.
- RetroAchievements: 520 of 612 games with achievements have a local ROM.
- Export (Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, all checks): whole set in storage order 21.4 MiB/s (5,282 ROM files); single file with a cold cache 1.752 s (ROM) / 1.986 s (TorrentZip) on average.
- Full audit of the populated database: 3,691 objects, 17 groups, 5,044 archive plans, no errors.

The release workflow starts from the base Catalog pinned by SHA256 in `release/catalog-release.json`, injects the engine and documents of the tagged commit, checks every data-table digest, SQLite integrity and foreign keys, runs the Catalog audit and the repository tests. Verify the download with `SHA256SUMS`.

[中文说明](https://github.com/rshi0212/RetroBoxDB-MegaDrive/blob/main/README.zh-CN.md)
