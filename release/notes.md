MegaDrive Catalog, storage v4 (64 KiB blocks, 20 solid LZMA2 groups of up to 256 MiB). Metadata only: **no ROM payloads are published**; `compression_groups`, `chunks` and `object_chunks` are empty.

- RetroAchievements ROM set imported: 934 ZIPs; 654 ROM files are also in a No-Intro DAT, 275 are only in the RA set, 5 have a hash absent from the latest RA snapshot. RA games with achievements that have a local ROM: 520 → 606.
- Source collections (`source_collections`, `v_collection_files`) and RetroAchievements links per file (`v_ra_collection`).
- ROMs outside every DAT join the family of the stored ROMs they share the most blocks with (hacks next to their original).
- Imports and DAT packaging no longer decode solid groups for already stored blocks; DAT formats (e.g. FDS/QD, NES headered/headerless) are handled separately.
- Source: 6,215 ZIPs (nointro 5,281, retroachievements 934), 4.66 GiB (6,216 ROM files, 9.61 GiB uncompressed). Populated database: 998.5 MiB (20.9% of the ZIPs). All source ZIPs are reproduced byte-for-byte.
- Contents: 3,959 ROM records, 1,581 games, 3,503 releases; DAT versions: 20260714-063411, 20260927-122056.
- RetroAchievements: 606 of 613 games with achievements have a local ROM.
- Export (Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, all checks): whole newest-DAT set with export_set.py 18.8 MiB/s (3,398 files); single file with a cold cache 1.734 s (ROM) / 2.087 s (TorrentZip) on average.
- Full audit of the populated database: 3,963 objects, 20 groups, 5,367 archive plans, no errors.

The release workflow starts from the base Catalog pinned by SHA256 in `release/catalog-release.json`, injects the engine and documents of the tagged commit, checks every data-table digest, SQLite integrity and foreign keys, runs the Catalog audit and the repository tests. Verify the download with `SHA256SUMS`.

[中文说明](https://github.com/rshi0212/RetroBoxDB-MegaDrive/blob/main/README.zh-CN.md)
