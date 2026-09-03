# 2026-08-20 TPShell leftover UNK follower lift

## 场景分类
二进制分析 / 自定义 VM opcode

## 目标概述
离线 Capstone+Unicorn 分类三条 leftover UNK follower（ORADD.NOT / LOADZX.DW.PTR / STORE.ADD.QW），恢复 word XOR 与 L2 MBA。

## Scope 摘要（脱敏）
- auth_basis: own_system
- network_profile: offline
- asset_types: [memory-dump, custom-vm]

## 角色
- lead_role: lead
- specialists: [cre, cie]

## 完整执行链路

1. `parse_records` + `exec_chain` 锚定 UNK `@ip` 与 leftover hex。
2. Unicorn 停在 `48 8B 55 60` / HANDLER0；指针类 opcode 用 FAKE 页完成 consume。
3. 逐 u16 打补丁 `{0,0xFFFF,0x1234}` 证明 word MBA ≡ XOR。
4. 打补丁 L2 `{0,0xFFFFFFFF,0x12345678,real}`；next 取最后一跳寄存器 jmp（未映射也算）。
5. 只信 live helper 上的 MBA，不抄首个 `jmp` 后的线性死码。

## Evidence 链摘要（脱敏）

| E-id | severity | status | source_type | 可复用命令模式 | 关联 Finding |
|------|----------|--------|-------------|----------------|--------------|
| E-lift-follow-new3 | info | validated | command | `python scripts/prove_follow_new3.py` | F-unk-follow-3 |

## Finding / Path 摘要
- top_finding: 三条 follower 已分类；H3 L2 为 `(dw-K1)^K2`，H1/H2 L2 可折叠为 XOR
- path_type: solve
- path_one_liner: leftover UNK → Unicorn consume/keys → live-island L2 → paste-ready `disp_*` / `if h==`

## 踩坑记录

| 问题 | 原因 | 解决方案 | 耗时 |
|------|------|---------|------|
| H2/H3 consume=0 INV | src 是指针，`[0]` 未映射 | 把 vreg 槽位 seed 成 FAKE 再 sweep L2/dest | 短 |
| L2 next=None | 未映射 fetch 时只记了异常 | 用最后一次 `jmp r*` 目标当 next | 短 |
| 线性反汇编像第三条 word MBA | 首个 `jmp` 后是死码 | 跟 Unicorn flow / helper land | 短 |

## 工具链发现
- Python 3.12 + Capstone + Unicorn；dump `TPShell64_memdump.dll`；不改 `hybrid_vm.py`

## 关键代码/命令

```
python scripts/lift_follow_new3.py
python scripts/prove_follow_new3.py
```

## 对本包的改进建议
- 路由 PRIMARY 落到 dsl-vm-reverse（JS VM）是误匹配；native leftover VM 应走 reverse-engineering / ida-reverse。无需改 routing（hint 含 dest-env 已足够）。

## 可复用的模式/脚本片段
- 指针 opcode：FAKE 页 + dest retarget
- L2：`next = H + sext32(mba)`；先试 XOR/ADD/NOTADD，再看 live island
- word：`0/0xFFFF/0x1234` 必做；L2：`0/0xFFFFFFFF/0x12345678+real` 必做且不预设 XOR

## 进化动作
- [x] 无需更新（routing 误匹配已记录，不改共享矩阵）
