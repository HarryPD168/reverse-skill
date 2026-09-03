# [RE] UE4 Global client + WeGame + TenProtect6 + ACE inventory

## 任务类型
二进制分析 / 安装盘点

## 目标描述
对本地已安装的 UE4 射击客户端做离线静态架构盘点：启动链、签名主体、TenProtect 壳、ACE 组件清单。不碰反作弊绕过、不挂钩、不连正式服。

## Scope 摘要（已脱敏）
- auth_basis: own_system
- network_profile: offline
- asset_types: [ue4-shipping-exe, wegame-launcher, tenprotect-shell, ace-um, ace-sys]

## 角色
- lead_role: lead
- specialists: [cre, cie]

## 实际执行路径
1. master-route → reverse-engineering (R0)
2. case-init + 用户口头授权
3. 目录树 / pak 体量 / Win64 SDK 列表
4. rabin2 -I/-i/-l + FileVersion + Authenticode + SHA256
5. ACE 目录清单（UM + 4 个内核映像，不做 IOCTL）
6. 架构图 + findings

## Evidence 链摘要（已脱敏）
| E-id | source_type | 可复用分析模式 | 关联 Finding |
|------|-------------|----------------|--------------|
| E-001 | authenticode+sha256 | 发行商三分：游戏公司 / ACE 厂商 / WHQL | F-004 |
| E-002 | pe-triage | Shipping 第一导入 = TPShell64 ordinal 1 | F-002 |
| E-003 | dir-inventory | ACE 28.5 + SGuard + BASE/CORE/GAME/SSC-DRV | F-003 |
| E-004 | launch-chain | UE Bootstrap stub CreateProcessW + WeGame PaaS 双入口 | F-001 |

## Finding / Path 摘要
- top_finding: TenProtect6 是 Shipping.exe 的第一静态依赖；ACE 是旁路服务/驱动树而非 PE import
- path_type: callflow
- path_one_liner: stub/launcher → Shipping.exe → TPShell64!1 → UE4/Sail/CEF; ACE Service+sys 并行驻留

## 踩坑记录
| 现象 | 原因 | 正确做法 | 耗时 |
|------|------|---------|------|
| rabin2 -I 扫 526MB Shipping 可接受 | 不要对 Shipping 跑全量 -zz | 只对 stub/shell 抽字符串 | 短 |
| INTLConfig.ini.new 当 ini 读 | 已混淆 | 标成 blob，不要当配置解析 | 短 |
| ACE-Service rabin2 signed=false | 安全目录解析与 Authenticode 不一致 | 以 Get-AuthenticodeSignature 为准 | 短 |

## 工具链使用
- rabin2 6.2.0（tool-index 绝对路径）
- Get-FileHash / Get-AuthenticodeSignature / FileVersionInfo
- 未用 IDA/Frida（offline pass 1）

## 关键命令/代码
```
rabin2 -I -l -i <pe>
rabin2 -zz DeltaForceClient.exe
Get-FileHash -Algorithm SHA256
Get-AuthenticodeSignature
```

## 对技能包的改进建议
- R0 对「已安装 UE 游戏目录」足够；若用户只要反作弊清单可加一条 ACE/TenProtect 关键词到 MASTER-ROUTING
- 不必上 pwn-chain / edr-bypass-re（本任务明确排除绕过）

## 可复用的模式/脚本片段
- 游戏安装盘点顺序：root stub → Binaries/Win64 → 第一导入 DLL 的 PDB → AntiCheat* 目录 → Authenticode 分组
- ShippingBase 文件名像游戏模块，PDB/导出名才暴露 TenProtect

## 本次未执行
- IDA 打开 Shipping / TPShell
- ACE 驱动 IOCTL / 回调枚举
- pak 解包
- 动态进程树
