# RetroBoxDB MegaDrive

[English](README.md) | 中文

世嘉 Mega Drive／Genesis的单文件 SQLite 保存库。公开的 Catalog 只含元数据（校验值、DAT 与来源记录、头部字段、打包配方和程序），不含 ROM 数据，不能独立恢复文件；完整库保留在本地。

| 项目 | 数值 |
| --- | --- |
| 原始大小 | 源 ZIP 6,215 个，4.66 GiB（No-Intro 5,281 个，RetroAchievements 集合 934 个）；解压后 ROM 6,216 个，9.61 GiB |
| 入库后大小 | 完整库 998.7 MiB；公开 Catalog 44.8 MiB（不含 ROM 数据） |
| 比例 | 完整库为原 ZIP 的 20.9%，为解压后 ROM 总量的 10.2% |
| 使用的技术 | 存储 v4：64 KiB 块按 SHA256 去重，按 No-Intro 游戏族顺序装入最大 256 MiB 的 LZMA2 实体组（字典 256 MiB）；逐块 SHA256、逐对象 CRC32／MD5／SHA1／SHA256 校验；源 ZIP 由 TorrentZip 配方逐字节重建 |
| 导出性能 | Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz，空闲负载，Python 3.14.4，含全部校验。按最新 DAT 整套导出（`export_set.py`，3,398 个文件，逐个按 DAT 哈希校验）：18.8 MiB/s，平均 74 毫秒／个；单个文件冷缓存（每次清空缓存，需解压所在组的前段）：ROM 平均 1.734 秒，TorrentZip 平均 2.087 秒 |

## 下载与说明

| 文件／文档 | 内容 |
| --- | --- |
| [RetroBoxDB.MegaDrive.Catalog.sqlite](https://github.com/rshi0212/RetroBoxDB-MegaDrive/releases/latest/download/RetroBoxDB.MegaDrive.Catalog.sqlite) | 公开 Catalog（Release 附件，附 `SHA256SUMS`） |
| [存储 v4 说明](RetroBoxDB.Storage-v4.zh-CN.md)／[English](RetroBoxDB.Storage-v4.en.md) | 八个平台的存储评估、内容、RA、中文名与维护 |
| [Technical design](RetroBoxDB.Storage-v4.Technical-Design.en.md) | 存储格式、平台适配、增量更新、校验 |
| [RA 清单](reports/ra-megadrive-games.csv)／[汇总](reports/ra-megadrive.json)、[构建报告](reports/megadrive-build-report.json)、[审计处理](reports/audit-resolution-20261004.md) | 逐项数据 |

## 本平台的存储选择与特殊情况

真实全量数据（按族排序的前 16 个组，472 MiB）上相对 32 MiB 组的变化：64 MiB −1.74%，128 MiB −1.99%，256 MiB −2.64%；按规则采用 256 MiB。

- ROM 为 0.1–5 MiB（SSF 映射器游戏更大）；在 64 KiB 块的平台中，加大组带来的收益在 MD 上最明显。
- 0x100 内部头部：系统类型、版权、日版／海外标题、序列号、设备、ROM／RAM 范围、SRAM 描述和地区；计算校验和为 0x200 至文件末尾的大端 16 位字之和。声明值常与计算值不符：自制和原型多填 0，部分早期授权游戏也不一致。
- SMD 交错格式会被识别并原样保存（本地没有）。不在任何 DAT 中的本地 ROM 大多来自 Aftermarket 和 Private 目录。

## 内容

| 项目 | 数值 |
| --- | --- |
| ROM 记录／游戏组／发行版本 | 3,959／1,581／3,503 |
| 各版 DAT 覆盖 | 20260714-063411：3,398/3,486；20260927-122056：3,398/3,504 |
| 不在任何 DAT 的本地 ROM | 561 |
| RetroAchievements 集合中的 ROM 文件 | DAT 中有 654，仅 RA 收录 275，哈希不在最新 RA 快照 5（[清单](reports/ra-megadrive-collection-unknown.csv)）；仍缺本地 ROM 的 RA 游戏见 [缺口清单](reports/ra-megadrive-missing.csv) |
| No-Intro DB Export＋Dump Log 20260927-122056 | 3,640 个档案、4,142 个文件身份、2,017 条有文档的硬件声明；Dump Log Verified 868 |
| RetroAchievements（console 1） | 有成就的游戏 613 个：本地有 ROM 606（942 个 ROM），ROM 在兄弟库中 0，仅 DAT 有 0，仅 DB 文件 0，无 No-Intro 对应 7 |
| 中文名 | 2,869 条记录中 2,694 条有中文（1,226 个唯一名）；本地 ROM 2,656 个有中文名 |
| 完整库审计 | 3,963 个对象、20 个组、5,367 个 ZIP 配方，全部通过 |

源 ZIP 均可由 TorrentZip 配方逐字节重建（`v_file_checksums.exported_bytes_equal_source`）。

## 使用

```bash
# 用 Catalog 内嵌引擎做只读审计（stats、checksums FILE_ID、help 同理）
python3 -B -c 'import sqlite3,sys; c=sqlite3.connect(sys.argv[1]); s=c.execute("SELECT content FROM resources WHERE name=?",("engine.py",)).fetchone()[0]; c.close(); exec(compile(s,"RetroBoxDB:engine.py","exec"))' ./RetroBoxDB.MegaDrive.Catalog.sqlite audit
# 完整库：按 DAT 版本、1G1R、RA 成就、TorrentZip／裸 ROM 组合导出
python3 -B tools/export_set.py RetroBoxDB.MegaDrive.sqlite OUT --set 1g1r --ra achievements --container torrentzip --layout ra-category
# 增量加入新 DAT、DB Export／Dump Log、ROM 与 RA 快照
python3 -B tools/update_db.py RetroBoxDB.MegaDrive.sqlite --discover --ra --catalog RetroBoxDB.MegaDrive.Catalog.sqlite
```

只需 Python 3.10+ 标准库。`resources` 中的 `engine.py` 等是可执行代码，只应从自己构建或 SHA256 已核对的 Release 附件中执行。发布由 `.github/workflows/publish-catalog.yml` 完成：工作流从 `release/catalog-release.json` 固定的基础 Catalog 出发，注入本仓库提交中的引擎与文档，核对全部数据表摘要、运行测试与审计后发布。
