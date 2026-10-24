---
name: boot-audit
description: 审计这台机器的启动配置与安全状态（启动项、SecureBoot、SetupMode、启动顺序）。用户问"它会启动什么""能不能装别的系统""安全启动开了吗"时使用。
allowed-tools: read_env, run_command, list_dir, read_file, dump_smbios, dump_acpi, read_skill
---

# 启动审计

这台机器"会怎么启动"不写在磁盘上，也不写在 SMBIOS 里——它在 EFI 变量里。
`read_env` 是唯一能看到它的工具。

## 步骤

1. `read_env`，参数 `{"name":"Boot"}` —— 拿到全部 `Boot####` 启动项与 `BootOrder`。
2. `read_env`，参数 `{"name":"Secure"}` —— `SecureBoot` / `SecureBootEnable`。
3. `read_env`，参数 `{"name":"SetupMode"}` —— `SetupMode` 为 1 表示平台密钥可写。
4. 需要时再 `read_env`，参数 `{"name":"PK"}` —— 平台密钥是否存在。

## 怎么读

- **`BootOrder`** 是一串 4 位十六进制编号，**顺序即启动顺序**，第一位最先尝试。
  把每个编号对上 `Boot####` 里的名字，按 BootOrder 的顺序列出来。
- **`Boot####` 的显示形式**：`read_env` 会把设备路径转成文本，形如
  `PciRoot(0x0)/Pci(0x1,0x1)/Ata(Primary,Master,0x0)`，路径后面是启动项的名字。
  路径就是这台机器实际会走的那条链，直接用它回答"它会启动什么"。
- **有些固件不发布设备路径转文本的协议**。那种情况下 `Boot####` 只会显示
  `<boot entry, N bytes; ...>`，并且会说明原因。这种情况下再考虑用
  `run_command` 跑 `bcfg boot dump`（需用户确认）——不要一开始就退回去跑命令。
- **`SecureBoot = 0x01`** 表示安全启动已开启；`SetupMode = 0x01` 表示处于设置模式，
  此时任何程序都能写平台密钥——这是"安全启动形同虚设"的信号，要明确说出来。
- 变量不存在（工具报"无匹配"）本身也是结论：比如没有 `SecureBoot` 说明这台机器
  的固件不支持或未启用安全启动。

## 输出格式

## 启动顺序
1. `<名字>`（`Boot0001`）
2. …

## 安全状态
- 安全启动：已开启 / 未开启 / 固件不支持
- 设置模式：是 / 否（是 → 平台密钥可被改写）
- 平台密钥：存在 / 不存在

## 结论
一两句话说明这台机器现在会启动什么、安全边界在哪里。不要复述上面已经列出的内容。
