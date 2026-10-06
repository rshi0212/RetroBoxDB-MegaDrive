MegaDrive Catalog, storage v4 (64 KiB blocks, 20 solid LZMA2 groups of up to 256 MiB). Metadata only: **no ROM payloads are published**; `compression_groups`, `chunks` and `object_chunks` are empty.

- `meta.storage` corrected: after retuning it still described the original group size.
- Naming normalized: platform codes are the Batocera system names, every populated database is `RetroBoxDB.<label>.sqlite`, and `meta.scope` / `meta.storage` are derived from the platform and the current storage parameters.
- One schema for all fifteen platforms: the header tables of every platform (including Master System, 32X, WonderSwan, NeoGeo Pocket and Pokémon Mini) and the provider-information tables exist in every Catalog; tables of other platforms and provider tables have no rows.
- RetroAchievements reports look up sibling databases (NES<->FDS, SNES<->Satellaview, WonderSwan<->WonderSwan Color, NeoGeo Pocket<->NeoGeo Pocket Color): a game whose ROM is stored there is `local_other_platform`, not a gap.
- Source: 6,215 ZIPs (nointro 5,281, retroachievements 934), 4.66 GiB (6,216 ROM files, 9.61 GiB uncompressed). Populated database: 998.9 MiB (20.9% of the ZIPs). All source ZIPs are reproduced byte-for-byte.
- Contents: 3,959 ROM records, 1,581 games, 3,503 releases; DAT versions: 20260714-063411, 20260927-122056.
- RetroAchievements: 606 of 613 games with achievements have a local ROM.
- Export (Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, all checks): whole newest-DAT set with export_set.py 18.8 MiB/s (3,398 files); single file with a cold cache 1.734 s (ROM) / 2.087 s (TorrentZip) on average.
- Full audit of the populated database: 3,963 objects, 20 groups, 5,367 archive plans, no errors.

The release workflow starts from the base Catalog pinned by SHA256 in `release/catalog-release.json`, injects the engine and documents of the tagged commit, checks every data-table digest, SQLite integrity and foreign keys, runs the Catalog audit and the repository tests. Verify the download with `SHA256SUMS`.

[中文说明](https://github.com/rshi0212/RetroBoxDB-MegaDrive/blob/main/README.zh-CN.md)
