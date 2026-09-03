# IDA MCP 用法（本机约定）

单一入口：`idapro` → `http://127.0.0.1:13337/mcp`  
启动：`skills/ida-reverse/scripts/start.ps1`（`--unsafe`，无 `ext=dbg`）  
卡住 / Cursor `user-idapro` error：`scripts/recover.ps1`。每分钟 `watchdog.ps1` 会自己拉起；supervisor 死锁超过 3 分钟也会 `-Force`。

## 不要做

- 不要再配第二个 IDA MCP（`ida-pro-mcp`、blacktop、iida、stdio 第二套 supervisor）
- 不要默认开 `?ext=dbg` / `dbg_*`（调试扩展另案授权，不是静态 MCP 默认路径）
- CFF / 混淆导致 Hex-Rays 失败时：先 `recover.ps1` 修好 MCP，再用入口对齐反汇编或 `py_eval`；**禁止**用猜的磁盘 Capstone 字节冒充已打开的 IDB
- 旁路字符串、目录名、合成表不是成功条件；打开 `file:line` 再引用

## 工具怎么选

| 目的 | 工具 |
|---|---|
| 开库 / 列会话 | `idb_open` / `idb_list`（大文件优先 `open.ps1`） |
| 伪代码 | `decompile`；CFF 失败不要死磕 |
| 建函数 | `define_func` + `define_code` |
| 搜字节 / 指令 | `find_bytes` / `insn_query` |
| 任意 VA 线性反汇编、批量普查 | **`py_eval` / `py_exec_file`** |
| 写 IDB 注释 | `set_comments` / `rename` |

## py_eval 模板

`database` 必填（`idb_list` 的 session_id）。

```python
# 任意 VA 线性 20 条（不要求已是函数）
import idc
ea = 0x180065132
rows = []
for _ in range(20):
    rows.append(f"{ea:#x} {idc.generate_disasm_line(ea, 0)}")
    nxt = idc.next_head(ea)
    if nxt == idc.BADADDR: break
    ea = nxt
result = "\n".join(rows)
```

```python
# 批量 define_func
import ida_funcs
for ea in (0x181EF2460, 0x181DD7ED0):
    ida_funcs.add_func(ea)
result = "ok"
```

长脚本用 `py_exec_file`（绝对路径），不要把几百行塞进 py_eval。

## 超大函数 / 未分析完的 IDB

MCP `decompile` 巨函数可能把 worker 打挂。需要归档时用 IDA GUI **导出文件**，不要再挂一套 MCP。
