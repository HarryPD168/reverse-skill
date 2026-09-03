# 2026-08-13 HWW v2.17.2 LIVE IDA 红队 follow-up（四问闭合）

- 案件：`work/hww-v217-defensive-re` auth=granted
- PRIMARY：ida-reverse；不开驱动、无 PoC、无仓库改动
- 复用：`hww-launcher-live` / `hww-gui-live` / `hww-engine-live`；`idb_close hwwv2drv` 后 `open.ps1` → `hww-wmiprv-live`

## 有效做法

- worker=4 满时先关本轮不用的 `hwwv2drv`
- 嵌套 MZ：`find_bytes 4D 5A 90 00` 得 8 VA → VA→fileoff → PE SizeOfRawData 与契约 size 分别 SHA
- iqvw 双 SHA = 26624（无 overlay）vs 34568（+7944 Authenticode）
- rust 静态串：`xrefs_to` 为 0 时再 `find_bytes` 64-bit fat-ptr；仍 0 则判 dead .rdata

## 四问结论

1. wmiprv `7EBBDCC7`：cloak 路径 LIVE；`CIMWIN32_LOAD` 0 xref leftover
2. F15：嵌套 iqvw **就是** 契约 pin `4429F32D` / 34568
3. DSE `@0x1403a2788`：0 xref / 0 fat-ptr / 0 listing → dead 但仍链入
4. GUI sticky env blob 与 `Desktop\spoofer\crates\hww-gui-tauri` 均在
