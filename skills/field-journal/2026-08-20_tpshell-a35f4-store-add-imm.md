# 2026-08-20 TPShell 0x1821A35F4 STORE.ADD.IMM.DW

## 场景分类
二进制分析 / 自定义 VM opcode

## 目标概述
离线 Capstone+Unicorn 分类 leftover UNK `0x1821A35F4`（rec124 @ip=24）。

## Scope 摘要（脱敏）
- auth_basis: own_system
- network_profile: offline
- asset_types: [memory-dump, custom-vm]

## 角色
- lead_role: lead
- specialists: [cre, cie]

## 完整执行链路

1. `parse_records` + `exec_chain` 锚定 UNK `@ip=24` leftover `aedb532ffb5d2698da8de41a`。
2. 零 seed INV@0；FAKE 页完成 consume=12。
3. 逐 u16 `{0,0xFFFF,0x1234}` 证明 word MBA ≡ XOR；imm dword `{0,0xFFFFFFFF,0x12345678,real}` ≡ XOR。
4. L2 `{0,0xFFFFFFFF,0x12345678,real}` 证明 `(~dw)-0xE51BAF47`，不是 XOR；real → HANDLER0。
5. 只信 live helper `0x182072214` / tail `0x181F9E74C`，不抄首个 `jmp` 后的线性死码。

## Evidence 链摘要（脱敏）

| E-id | severity | status | source_type | 可复用命令模式 | 关联 Finding |
|------|----------|--------|-------------|----------------|--------------|
| E-lift-a35f4 | info | validated | command | `python scripts/prove_a35f4.py` | F-unk-a35f4 |

## Finding / Path 摘要
- top_finding: STORE.ADD.IMM.DW `[sa+sb]=imm32`；L2 `(~dw)-0xE51BAF47`；next HANDLER0
- path_type: solve
- path_one_liner: leftover UNK → FAKE consume/keys → live-island L2 → paste-ready `disp_a35f4` / `if h==`

## 踩坑记录

| 问题 | 原因 | 解决方案 | 耗时 |
|------|------|---------|------|
| consume=0 INV@0 | src 是指针，`[0]` 未映射 | vreg `0x380`/`0xE8` seed 成 FAKE 再 sweep | 短 |
| rec59 同 MOV.QW6 易混 next | 那条 leftover 去已接线 `0x1820DB12A` | rec124 L2 `0x1AE48DDA` 单独证明 → H0 | 短 |
| 单独 exec_chain v62=0 | 8580 共享 vreg；fresh rem 无前置写入 | 顺序走 rec0..124 再读槽 | 短 |

## 工具链发现
- Python 3.12 + Capstone + Unicorn；dump `TPShell64_memdump.dll`；不改 `hybrid_vm.py`

## 关键代码/命令

```
python scripts/lift_a35f4.py
python scripts/prove_a35f4.py
python scripts/snippets_a35f4.py
```

## 对本包的改进建议
- 无

## 可复用的模式/脚本片段
- 指针 opcode：FAKE 页 + dest retarget
- L2：`next = H + sext32(mba)`；先试 XOR/ADD/~dw+K，再看 live island
- word：`0/0xFFFF/0x1234` 必做；L2：`0/0xFFFFFFFF/0x12345678+real` 必做且不预设 XOR
