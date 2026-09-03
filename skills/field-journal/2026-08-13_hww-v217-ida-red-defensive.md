# 2026-08-13 HWW v2.17.2 LIVE IDA 红队（自有产品 · 防御）

- 目标：自有 HWW LIVE pin `2B5C8A75` + dist/ProgramData 客户 PE
- PRIMARY：ida-reverse；auth=granted / own_system / offline
- 不做：驱动重建/加载、exploit PoC、commit

## 复用

- IDA session `hwwv2drv`（driver\v2\HwwV2.sys）
- ProgramData LIVE：`hww-launcher-live` / `hww-gui-live` / `hww-engine-live`
- 先验 dump：`work/hww-v217-defensive-re\`（未再 unpack）

## 有效做法

- `start.ps1` + `open.ps1` 拉起 worker；`idb_list` 复用已开库
- 大 PE 用 `survey_binary(minimal)` + `find_regex`；驱动用 `analyze_function`/`decompile`
- `find_regex` 不要写 `\\.`（re 会报 missing {）
- idalib max-workers=4：第 5 个 open 会失败，先关旧 session

## 结论摘要

- 2B5C8A75：OEM / SOFTWARE\HWW / NIC allowlist 明文已不在；F03 IOC 与 APC 注入路径仍在
- LIVE GUI：iqvw/byovd 字符串 0；HMAC 盐 ASCII 仍在（HYN-005）
- LIVE engine：DSE 剧本字符串仍在（0 xref）；hard-remint 是新披露面
- LIVE launcher：8×MZ + 契约 JSON + build-identity + iqvw PDB
