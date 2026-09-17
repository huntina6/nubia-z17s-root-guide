# nubia Z17s (NX595J) 解锁 Bootloader + Magisk Root 实录

> **设备**：nubia Z17s，型号 NX595J，骁龙 835 (MSM8998)，Android 7.1.1 / nubia UI 5.1，国行 `NX595J_CNCommon_V2.22`
> **实测日期**：2026-09-17
> **结果**：✅ 成功获得 Magisk root

---

## 一句话结论

可以 root。这台机器有一条努比亚自己的隐藏解锁命令，**不需要申请解锁码、不需要账号等级、没有等待期**。但过程有三个地方极容易卡死，全部记录在下面 —— 很多人放弃就是因为卡在这三点。

---

## 三个关键难点

### 难点 1：Windows 缺 fastboot 驱动，且 Google 的驱动包不覆盖这台设备

设备在 fastboot 模式下的 USB 标识：

```
VID_18D1&PID_D00D          ← Android 通用 fastboot PID
CompatibleId: USB\Class_FF&SubClass_42&Prot_03
```

而 Google USB Driver 的 `android_winusb.inf` **不含 `D00D`** —— 它只列举了 Nexus/Pixel 那几个：`4EE0` / `2C10` / `9004` / `9006` / `4D00`。所以**不会自动匹配**，必须手动强制安装。

**正确装法：**

1. 设备管理器 → 找到带黄色感叹号的 `Android` 设备（Code 28）
2. 右键 → 更新驱动程序
3. 选「浏览我的电脑以查找驱动程序」
4. ⚠️ **不要点「浏览」按钮**，点下面那行链接 **「让我从计算机上的可用驱动程序列表中选取」**
5. 点 **「从磁盘安装」** → 指向 `android_winusb.inf`
6. 选 `Android ADB Interface` 或 `Android Bootloader Interface` 都可以（两者 Service 都是 WinUSB）
7. 遇到「不推荐安装此驱动程序」→ 选「是」

> **最大的卡点**：如果点了「浏览」按钮，会弹出「浏览文件夹」对话框 —— **那个对话框只列文件夹、不列文件**（因为它的设计就是让你选文件夹），而且选完文件夹后自动匹配必定失败。很多人在这里反复尝试、以为文件丢了，实际上文件一直在。

### 难点 2：fastboot 的输出全在 stderr，PowerShell 完全抓不到

这条坑了整整一轮排查。**现象是"命令没有任何输出"，极容易误判成驱动坏了、通信断了。**

- `fastboot getvar` / `oem` / `reboot` 的**所有信息性输出都写到 stderr**
- 只有 `fastboot devices` 走 stdout
- 下面这些 PowerShell 写法**一律返回空字符串**：

```powershell
(& $fb getvar product 2>&1 | Out-String)
& $fb getvar product 1> out.txt 2> err.txt
... *> logfile
$fb -v getvar product            # verbose 也一样为空
```

**解法：用 Python 的 subprocess**

```python
import subprocess
FB = r"path\to\fastboot.exe"
p = subprocess.run([FB, "oem", "device-info"], capture_output=True, text=True)
print(p.stderr)   # ← 真正的输出在这里
```

**通用教训**：拿不到输出时，**先怀疑"是不是输出被吞了"，而不是"命令失败了"**。用设备状态变化来验证命令是否真的生效（比如 `oem device-info` 里的 `unlocked` 字段、`fastboot reboot` 后能否 adb 连上）。

### 难点 3：bootloader 命令被大幅裁剪，进 recovery 只能靠屏幕菜单

最耗时间的一处。实测这条设备对 fastboot 命令的裁剪程度：

| 命令 | 结果 |
|---|---|
| `fastboot oem reboot-recovery` | ❌ `FAILED (remote: 'unknown command')` |
| `fastboot boot twrp.img` | ❌ 镜像发送 OKAY，但 `Booting FAILED (remote: 'unknown command')` |
| `fastboot oem poweroff` | ❌ `unknown command` |
| `fastboot oem reboot recovery` | ❌ 直接挂起超时 |
| `fastboot reboot recovery` | ⚠️ **返回 `OKAY` 却实际进了系统**（假成功，最坑） |

**真正的路：fastboot 模式下的屏幕自带一个可用音量键操作的菜单**

```
Reboot system now
Reboot to recovery mode        ← 选这一项
Reboot to emrg recovery mode
Boot to edl
Power off

Press volume key to select above item, and press power key to confirm

device info:
CPU SERIAL NUMBER - 0x602a97EE
SECURE BOOT - yes
SYSTEM IS ROOT - 0
BOOT IS ROOT - 0
DEVICE IS ROOT - 0
DEVICE STATE - unlocked
FASTBOOT UNLOCK COUNT - 5
```

用**音量键**移动光标，**电源键**确认。**这是唯一能进 TWRP 的途径，无法用任何命令替代。**

> 关于 `fastboot reboot recovery` 的假成功：它返回 OKAY 并让设备正常启动进系统，而**原厂系统在启动时会检测 recovery 分区并还原它**（系统里有 `ro.expect.recovery_id` 属性，说明存在自动恢复机制）。所以刷完 TWRP 必须立刻用菜单进 recovery，**绝不能让它正常开机**。

---

## 完整流程

### 前置准备

**工具与镜像：**

- Google platform-tools（adb / fastboot）
- TWRP：GitHub `Cyborg2017/android_device_nubia_nx595j-twrp`
  - `twrp_recovery_3.7.0_9_for_nx595j-new-partitions-20221116.img`（32.2 MB，通用版）
- Magisk v26.4（老系统上验证充分；官方要求 Android 6.0+）

**官方固件（可选但建议）：**

```
http://rom.download.nubia.com/Europe&Asia/Z17S/NX595J_Z69_EN_VNG0N_V112.zip
```

大小 2.21 GB，**200 OK 可直连下载**（2026-09 实测服务器仍在运营）。同目录下 V111 也可用，V106 已 404。

> ⚠️ **注意**：这是 **OTA 卡刷包**，不是 EDL 线刷包，**救不了硬砖**。它的价值是"系统起不来但还能进 recovery时，有个官方系统可以刷回去"。真正能救黑砖的是 9008 模式用的 firehose 线刷包，那个找不到公开下载。
>
> ⚠️ **官方服务器上只有 EN 国际版**，CN 国行版（`NX595J_CNCommon_*`）各候选路径全部 404，找不到直链。

### 步骤 1：备份数据

```bash
adb pull /sdcard/ ./backup/
```

本机实测用户数据约 1 GB（含 `/sdcard/Android` 的应用数据），其中照片视频等真正重要的部分只有约 50 MB。

### 步骤 2：装 fastboot 驱动

见上面「难点 1」。装好后让设备重新枚举一次（拔插 USB 或 `adb reboot bootloader`）。

### 步骤 3：解锁 Bootloader

```bash
adb reboot bootloader
fastboot oem nubia_unlock NUBIA_NX595J
```

**成功回显：**

```
(bootloader) START update nubia fastboot unlock flag!!!
(bootloader) set state to 1 ok!!!
OKAY [  0.053s]
Finished. Total time: 0.054s
```

用 `fastboot oem device-info` 确认：

```
(bootloader) Device unlocked: true          ← 从 false 变成 true
(bootloader) Device critical unlocked: false
(bootloader) fastboot unlock count: 2       ← 每执行一次解锁就 +1
```

> **小技巧**：`fastboot unlock count` 会累加，可以拿它判断命令是否真的下发成功了 —— 即使你看不到输出。

**如果命令无反应**：回到开发者选项，把「OEM 解锁」开关**反复关闭再打开几次**再试（XDA 上多位用户报告过这个怪癖）。

#### ⚠️ 关键：这是「会话级」解锁，重启即失效

实测结论，**这条非常重要**：

- 解锁后 `Device unlocked: true`
- **但设备一重启就变回 `false`**
- `ro.boot.flash.locked` 始终为 `1`，`ro.boot.verifiedbootstate` 始终 `green`
- 数据也**从未被清空**（`df /data` 前后都是约 6.4 GB），证明解锁没有真正持久化

**试过但无效的持久化手段**（不用再重复踩了）：

- `settings put global oem_unlock_enabled 1` —— 能写入并读回 `1`，但 `sys.oem_unlock_allowed` 仍是 `0`，**且重启后被重置**
- Android 7.1.1 的努比亚 ROM **没有** Android 8 才引入的 `OemLockService` 机制，重启后 `sys.oem_unlock_allowed` 属性直接消失
- `ro.oem_unlock_supported = 1`（硬件支持），但 `sys.oem_unlock_allowed = 0`

**推论**：努比亚 bootloader 的解锁标志只在**当前 fastboot 会话内**有效。**必须趁状态还在立刻刷写，中途绝不能重启。**

### 步骤 4：刷入 TWRP

```bash
fastboot flash recovery twrp_recovery_3.7.0_9_for_nx595j-new-partitions-20221116.img
```

**成功回显**（`avb footer` 的 warning 可以忽略）：

```
Warning: skip copying recovery image avb footer (recovery partition size: 0, recovery image size: 33805612).
Sending 'recovery' (33013 KB)                      OKAY [  0.931s]
Writing 'recovery'                                 OKAY [  0.295s]
Finished. Total time: 1.231s
```

### 步骤 5：立刻进 recovery ★

**⚠️ 不要用任何 reboot 命令。** 正确做法是在 fastboot 界面：

1. 按**音量键**把光标移到 **`Reboot to recovery mode`**
2. 按**电源键**确认

> 备用方式：菜单里选 `Power off` 关机，然后按住「音量上 + 电源键」。

**判断成功**：

```bash
adb devices                        # 状态应显示 recovery（不是 device）
adb shell "getprop ro.twrp.version"  # 应输出 3.7.0_9-0
adb shell "ls /twres"              # 应看到 fonts images languages ui.xml
```

### 步骤 6：刷入 Magisk

**不要推到 `/sdcard`** —— 这台机器 data 分区加密，TWRP 环境下 `/sdcard` 不可用（`ls` 出来全是乱码文件名）。**推到 `/tmp`（TWRP 的内存文件系统）：**

```bash
adb push Magisk-v26.4.zip /tmp/
adb shell "twrp install /tmp/Magisk-v26.4.zip"
```

**成功回显：**

```
Installing zip file '/tmp/Magisk-v26.4.zip'
***********************
 Magisk 26.4 Installer
***********************
- Mounting /system
- No vbmeta partition, patch vbmeta in boot image
- Encrypted data, keep forceencrypt
- Target image: /dev/block/sde18
- Device platform: arm64-v8a
- Constructing environment
- Unpacking boot image
- Checking ramdisk status
- Stock boot image detected
- Patching ramdisk
- Repacking boot image
- Flashing new boot image
- Unmounting partitions
- Done
```

然后重启：

```bash
adb shell "twrp reboot"
```

### 步骤 7：装 Magisk App 并验证

```bash
adb install Magisk-v26.4.apk
```

**验证 root 装好了（不依赖授权）：**

```bash
adb shell "which su"                  # → /sbin/su
adb shell "ls /sbin"                  # → magisk magisk32 magisk64 magiskinit magiskpolicy su resetprop supolicy
adb shell "pm list packages magisk"   # → package:com.topjohnwu.magisk
```

**最终验证（需要授权）：**

```bash
adb shell su -c id                    # → uid=0(root) ...
```

> ⚠️ **`su -c id` 会挂起，这是正常的，不是故障。** 它在等 Magisk App 弹出授权对话框 —— 在手机上点「允许」即可。写脚本时务必加超时包裹：`adb shell "timeout 5 su -c id"`，否则整个脚本会卡死。

---

## 已知副作用

- 解锁后依赖设备完整性的功能（**指纹支付、部分银行 App**）可能失效
- 系统 OTA 升级功能失效
- 官方保修失效（Z17s 早已过保，影响不大）
- 开机时可能显示解锁状态警告

---

## 环境与工具链备注

这些是排查过程中踩到的、和刷机本身无关但会浪费大量时间的坑：

**PowerShell / Windows：**

- `fastboot` 输出走 stderr 抓不到（见难点 2）
- PowerShell 工具的 stdout 会被吞，必须 `Out-File` 写文件再读
- 脚本中某条 `adb shell` 返回非零时，整个 PowerShell 命令会以 exit 1 结束，**并且可能完全不写文件**（表现像"脚本没跑"）。调试时先跑最简命令（只 `adb devices`）确认链路，再逐步加回复杂逻辑
- 含 `su` 的命令会挂起，导致脚本超时且文件不生成 → 调试时把 su 单独拿出来、加 timeout 包裹

**网络：**

- 下载大文件时注意环境变量里的代理。本次实测环境里有条失效代理会让下载降到 ~80 KB/s，加 `--noproxy '*'` 直连后同一地址可达 6 MB/s（快 75 倍）。**下大文件前先各下 3 MB 测速对比**
- XDA Forums 常被 bunny.net 拦截返回 403，拿不到正文时改用搜索引擎摘要或 DeepWiki 镜像

**判断设备状态：**

- **判断数据是否丢失要看 `df /data`，不要看 `ls`** —— 系统运行中 `ls /sdcard` 会返回空、`ls /storage/emulated/0` 会返回一堆随机文件名（sdcardfs / 加密视图的正常表现），但数据完好无损
- 区分系统 / recovery：`ps` 里有 `zygote64` / `system_server`、`sys.boot_completed = 1` 说明在完整系统里；recovery 环境没有这些进程

---

## 文件说明

- `nubia-z17s-root-guide.html` —— 分阶段操作手册，浏览器直接打开即可。含可勾选的准备清单、一键复制的命令行、排错对照表、救砖分级流程

---

## 参考资料

- XDA 讨论帖：*Z17S (NX595J) Unlock Bootloader & TWRP* —— 提供 `fastboot oem nubia_unlock` 命令与 OEM 开关反复触发的经验
- 开源项目 `zenfyrdev/bootloader-unlock-wall-of-shame`（ZTE / nubia 章节）—— 确认努比亚骁龙机型的解锁命令格式为 `fastboot oem nubia_unlock NUBIA_<MODEL>`
- GitHub `Cyborg2017/android_device_nubia_nx595j-twrp` —— Z17s 的 TWRP 镜像来源
- LineageOS Wiki（NX563J 安装页）—— 佐证「刷完 recovery 不得直接重启进系统」这一通用约束

---

## 免责声明

刷机有风险。本文档为单机实测记录，不能保证适用于所有批次的 NX595J。按此操作导致的任何后果请自行承担。**动手前务必备份数据**，并尽量准备好可用的固件包。
