# [RE] Delta Force 全量工程逆向 Pass-1（本地安装）

## 任务类型
二进制分析 / 启动链 / 壳 / 反作弊全栈静态测绘

## 目标描述
对 `E:\LP game\Delta Force` 做不排除组件的工程逆向：Bootstrap、Shipping、TenProtect6、ACE UM+内核多变体、SGuard、launcher、pak 元数据。用户明确要求「按照工程不进行排除」。

## Scope 摘要（已脱敏）
- auth_basis: own_system
- network_profile: offline
- asset_types: [ue4-shipping, tpshell, ace-um, ace-sys, sguard, launcher, pak-meta]

## 角色
- lead_role: lead
- specialists: [cre, cie]

## 实际执行路径
1. master-route → reverse-engineering R0
2. case-init + scope 全安装树
3. rabin2 PE triage / 哈希 / Authenticode
4. ACE 34 文件清单 + 驱动字符串设备面
5. 节表确认 .tvm0 / .std / .detourd
6. findings + report + architecture

## Evidence 链摘要
| E-id | source_type | 可复用模式 | Finding |
|------|-------------|------------|---------|
| E-001 | hash+sig | 发行商三分 | F-004 |
| E-003 | pe-import | Shipping 第一导入 = 壳 | F-002 |
| E-008/E-013 | export+section | 单导出 + .tvm0 + 零字符串 | F-002 |
| E-009 | driver-strings | 设备名 + notify/Ob/Flt | F-003 |
| E-010 | um-export | InitAceClient 族 | F-003 |
| E-015 | idw | D3D12 函数地址采集 | F-003 |

## Finding / Path
- top: TenProtect6 强制入口 + ACE 全栈并行，二者均 .tvm0
- path: stub → Shipping → TPShell!1 → UE；ACE Base/CORE/GAME/SSC 内核旁路

## 踩坑
| 现象 | 原因 | 正确做法 |
|------|------|----------|
| case-guard offline fail | 缺 offline sample cue | scope 写明 offline_sample 路径 |
| Get-ChildItem -Include 无结果 | 无 -Recurse | Where-Object Extension |
| ShippingBase -zz 空 | VM/加密 | 看节表 .tvm0 而非 strings |
| 初版 scope 排除 AC | 用户要求不排除 | 立即改 scope 全量分析 |

## 工具链
- rabin2 6.2.0 @ C:\Users\Harry_win10\Tools\radare2\bin\rabin2.exe
- reverse-skill case scripts

## 本次未执行（列入 W9+）
- IDA 打开 TPShell / ACE-Base64 / ACE-BASE.sys
- 运行时设备枚举与 IOCTL 探测
- pak 解包
- 动态进程树

## 可复用模式
- 腾讯系 UE 射击：ShippingBase 文件名伪装 + TPShell64 PDB + .tvm0
- ACE 驱动：固定 BASE/GAME + CORE 动态设备名模板 + 多 sys 变体同树分发
