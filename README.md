# nubia Z17s (NX595J) 解锁 Bootloader + Magisk Root 实录

> **设备**：nubia Z17s，型号 NX595J，骁龙 835 (MSM8998)，Android 7.1.1 / nubia UI 5.1，国行 `NX595J_CNCommon_V2.22`
> **实测日期**：2026-09-17
> **结果**：✅ 成功获得 Magisk root（Magisk 26.4）
> **所有下载链接均于 2026-09-17 逐个复核，返回 200**

---

## 目录

- [一句话结论](#一句话结论)
- [全部下载链接](#全部下载链接实测核对) ← 所有文件、直链、字节数、SHA256、备用源
- [三个关键难点](#三个关键难点)
- [完整流程](#完整流程)
- [已知副作用](#已知副作用)
- [环境与工具链备注](#环境与工具链备注)
- [参考资料](#参考资料)

---

## 一句话结论

可以 root。这台机器有一条努比亚自己的隐藏解锁命令，**不需要申请解锁码、不需要账号等级、没有等待期**。但过程有三个地方极容易卡死，全部记录在下面 —— 很多人放弃就是因为卡在这三点。

---

## 全部下载链接（实测核对）

**通用校验方法**（下载完先跑一遍，防止下到半截的残包）：

```powershell
# PowerShell
Get-FileHash .\文件名 -Algorithm SHA256 | Format-List

# CMD
certutil -hashfile 文件名 SHA256
```

```bash
# Linux / macOS
sha256sum 文件名
```

---

### ★ 官方链接失效了怎么办 —— 离线包

**本项目用到的全部文件 —— 包括 2.2 GB 的官方固件 —— 都已归档进本仓库的一个 release**，不再依赖第三方链接是否还活着：

# **https://github.com/huntina6/nubia-z17s-root-guide/releases/tag/offline-tools-v1**

| 文件 | 字节数 | 永久直链 |
|---|---|---|
| `platform-tools-latest-windows.zip` | 8,044,989 | [下载](https://github.com/huntina6/nubia-z17s-root-guide/releases/download/offline-tools-v1/platform-tools-latest-windows.zip) |
| `platform-tools_r28.0.2-windows.zip` | 11,067,637 | [下载](https://github.com/huntina6/nubia-z17s-root-guide/releases/download/offline-tools-v1/platform-tools_r28.0.2-windows.zip) |
| `usb_driver_r13-windows.zip` | 8,682,039 | [下载](https://github.com/huntina6/nubia-z17s-root-guide/releases/download/offline-tools-v1/usb_driver_r13-windows.zip) |
| `twrp_recovery_3.7.0_9_for_nx595j-new-partitions-20221116.img` | 33,805,612 | [下载](https://github.com/huntina6/nubia-z17s-root-guide/releases/download/offline-tools-v1/twrp_recovery_3.7.0_9_for_nx595j-new-partitions-20221116.img) |
| `Magisk-v26.4.apk` | 12,526,383 | [下载](https://github.com/huntina6/nubia-z17s-root-guide/releases/download/offline-tools-v1/Magisk-v26.4.apk) |
| `twrp_recovery_3.7.0_13_for_nx595j-fbe.img` | 39,068,972 | [下载](https://github.com/huntina6/nubia-z17s-root-guide/releases/download/offline-tools-v1/twrp_recovery_3.7.0_13_for_nx595j-fbe.img) |
| `NX595J_Z69_EN_VNG0N_V112.zip.part1.rar` | 1,258,291,200 | [下载](https://github.com/huntina6/nubia-z17s-root-guide/releases/download/offline-tools-v1/NX595J_Z69_EN_VNG0N_V112.zip.part1.rar) |
| `NX595J_Z69_EN_VNG0N_V112.zip.part2.rar` | 1,117,718,202 | [下载](https://github.com/huntina6/nubia-z17s-root-guide/releases/download/offline-tools-v1/NX595J_Z69_EN_VNG0N_V112.zip.part2.rar) |

**全部文件的 SHA256：**

```
45F4D63113E895EBDE0C90F194099A4676B6AC653BD28D54314A9E022BBC1A99  platform-tools-latest-windows.zip
6A721560633BABEBC74F9330D7184A23D74D18B835CC769F81BBF977575B3800  platform-tools_r28.0.2-windows.zip
360B01D3DFB6C41621A3A64AE570DFAC2C9A40CCA1B5A1F136AE90D02F5E9E0B  usb_driver_r13-windows.zip
1BADBC96BCF169D0DB89167D3520F5D78F9B21A427754DED937A47184C197010  twrp_recovery_3.7.0_9_for_nx595j-new-partitions-20221116.img
A2818E8B0E6FC0C4467808996B0885F1B231BD4CF03A3D0D7416AA9C7BC0A410  twrp_recovery_3.7.0_13_for_nx595j-fbe.img
543A96FE26C012D99BAF3A3AA5A97B80508D67CC641AF7C12CE9F7B226B2B889  Magisk-v26.4.apk
F823A20E74666AF35ED43FFCC70E0ED0BB7BD4A198F3E929CA5CEADA41D79A6F  NX595J_Z69_EN_VNG0N_V112.zip.part1.rar
0DB70C82AE37B52B291827ED29632D7A0CF4A391A868598BE4A6B41436666F0E  NX595J_Z69_EN_VNG0N_V112.zip.part2.rar
```

**固件解压后**（`NX595J_Z69_EN_VNG0N_V112.zip`，2,376,008,975 字节）的 SHA256：

```
F75E8CC068B5D7D2503B39E07E72BBD28A34375D6BD2DFBB7217CAB639F69058
```

### 官方固件为什么是分卷的 —— 用法说明

固件原文件 **2,376,008,975 字节（2.21 GiB）**，**超过 GitHub 单个附件 2 GiB 的硬上限**，无法作为单个附件上传，所以按 **1200 MB** 切成两卷归档（RAR 存储模式，无压缩，不损失内容）。

**用法：两卷都下齐，放在同一个文件夹里，解压任意一卷即可**得到完整的 zip：

- **图形界面**：用 WinRAR / 7-Zip / Bandizip 打开 `NX595J_Z69_EN_VNG0N_V112.zip.part1.rar` → 解压 → 得到 `NX595J_Z69_EN_VNG0N_V112.zip`
- **命令行（Windows）**：`"C:\Program Files\WinRAR\rar.exe" x NX595J_Z69_EN_VNG0N_V112.zip.part1.rar`
- **命令行（Linux / macOS）**：`unrar x NX595J_Z69_EN_VNG0N_V112.zip.part1.rar`（或 `7z x`）

解出来的文件必须**正好是 2,376,008,975 字节**，SHA256 等于上面那串 —— 两者都对上，才是完好的官方固件。

---

### 1. Google platform-tools（adb + fastboot 工具包）

**本次实际使用的版本（推荐）：**

| 项目 | 内容 |
|---|---|
| 直链 | `https://dl.google.com/android/repository/platform-tools-latest-windows.zip` |
| 大小 | 8,044,989 字节（7.67 MB） |
| SHA256 | `45F4D63113E895EBDE0C90F194099A4676B6AC653BD28D54314A9E022BBC1A99` |

**旧版备用**（实测行为与最新版无差异，如遇新版兼容问题可换）：

| 项目 | 内容 |
|---|---|
| 直链 | `https://dl.google.com/android/repository/platform-tools_r28.0.2-windows.zip` |
| 大小 | 11,067,637 字节（10.55 MB） |
| SHA256 | `6A721560633BABEBC74F9330D7184A23D74D18B835CC769F81BBF977575B3800` |

**解压后内部文件校验值**（可用来确认工具确实是官方原版）：

| 文件 | 大小 | SHA256 |
|---|---|---|
| `adb.exe` | 8,273,560 字节（7.89 MB） | `B4A6B455702684652CCCF7B46258B29E653538904359A58FD4931CF3EF286B3F` |
| `fastboot.exe` | 2,429,080 字节（2.32 MB） | `B2D9CBFF4CE9AE7EB448CFC831BAFC867935F50F5BE38F8F81057FD7EB3B8D86` |

> 实测 `adb version` 输出 37.0.1。解压路径要求**纯英文、无空格**，例如 `D:\adb`；路径含中文会导致部分命令找不到文件。

---

### 2. Google USB Driver（fastboot 模式驱动）

**这一条是本次流程里最容易卡死人的环节，务必读完「[难点 1](#难点-1windows-缺-fastboot-驱动且-google-的驱动包不覆盖这台设备)」再动手。**

| 项目 | 内容 |
|---|---|
| 直链 | `https://dl.google.com/android/repository/usb_driver_r13-windows.zip` |
| 大小 | 8,682,039 字节（8.28 MB） |
| SHA256 | `360B01D3DFB6C41621A3A64AE570DFAC2C9A40CCA1B5A1F136AE90D02F5E9E0B` |

解压后关键文件路径：`usb_driver\android_winusb.inf`
（压缩包内套了一层同名目录，实际是 `usb_driver\usb_driver\android_winusb.inf`，别被绕晕）

> ⚠️ **这个驱动包不含本机的硬件 ID（`VID_18D1&PID_D00D`）**，它只列了 Nexus/Pixel 的 `4EE0` / `2C10` / `9004` / `9006` / `4D00`。所以**不会自动匹配**，必须手动强制安装。详见难点 1。

---

### 3. TWRP recovery 镜像

社区维护仓库：`https://github.com/Cyborg2017/android_device_nubia_nx595j-twrp`
发布页（所有版本都在这里）：`https://github.com/Cyborg2017/android_device_nubia_nx595j-twrp/releases`

**★ 主用版本 —— 3.7.0_9（本次实测通过，原厂 Android 7.1.1 用它）：**

| 项目 | 内容 |
|---|---|
| 文件名 | `twrp_recovery_3.7.0_9_for_nx595j-new-partitions-20221116.img` |
| 直链 | `https://github.com/Cyborg2017/android_device_nubia_nx595j-twrp/releases/download/twrp_recovery_3.7.0_9/twrp_recovery_3.7.0_9_for_nx595j-new-partitions-20221116.img` |
| 大小 | 33,805,612 字节（32.24 MB） |
| SHA256 | `1BADBC96BCF169D0DB89167D3520F5D78F9B21A427754DED937A47184C197010` |
| 发布日期 | 2022-11-14 |

**备选版本 —— 3.7.0_13-FBE（刷了新系统的设备才需要）：**

| 项目 | 内容 |
|---|---|
| 文件名 | `twrp_recovery_3.7.0_13_for_nx595j-fbe.img` |
| 直链 | `https://github.com/Cyborg2017/android_device_nubia_nx595j-twrp/releases/download/twrp_recovery_3.7.0_13-fbe/twrp_recovery_3.7.0_13_for_nx595j-fbe.img` |
| 大小 | 39,068,972 字节（37.26 MB） |
| 发布日期 | 2023-01-16 |
| 适用场景 | 说明写着「Decrypt automatically on android 11/12/13 devices」，即刷了 Android 11/12/13 系统的人才需要。原厂 7.1.1 用不上，**若主用版本出现 TWRP 无法启动或触摸失灵，再换它试** |

> 仓库里总共只有这两个 release，没有更老的版本可选。

---

### 4. Magisk

**★ 主用版本 —— v26.4（本次在 Android 7.1.1 上实测刷入成功）：**

| 项目 | 内容 |
|---|---|
| 直链 | `https://github.com/topjohnwu/Magisk/releases/download/v26.4/Magisk-v26.4.apk` |
| 大小 | 12,526,383 字节（11.95 MB） |
| SHA256 | `543A96FE26C012D99BAF3A3AA5A97B80508D67CC641AF7C12CE9F7B226B2B889` |

> **关键细节**：Magisk 的 `.apk` **本身就是可刷入的 zip 包**，不需要另外找 zip。刷机时直接把 `Magisk-v26.4.apk` 复制一份、改名为 `Magisk-v26.4.zip` 就能交给 TWRP 安装（本次就是这么做的，两个文件 SHA256 完全相同）。

**其它版本（未在 7.1.1 上验证，列出仅供参考）：**

| 版本 | 日期 | 说明 |
|---|---|---|
| `v31.0` | 2026-09-04 | 目前最新，状态是 **Pre-release**。这一版把 App 界面用 Jetpack Compose + Material 3 全部重写，还加了内置终端。`https://github.com/topjohnwu/Magisk/releases/download/v31.0/Magisk-v31.0.apk`。**Android 7.1.1 上是否可用未经验证**，新框架对老系统常有兼容问题，不建议第一次刷就用它 |
| `v30.4` ~ `v30.7` | 2025-10 ~ 2026-02 | 同样未验证 |
| `v26.x` | 2023 | 26.4 是这一系列最后一个，也是社区在老机器上验证最充分的版本 |

全部版本列表：`https://github.com/topjohnwu/Magisk/releases`

---

### 5. 官方固件包（救砖用）

**努比亚官方 ROM 服务器根地址**：`http://rom.download.nubia.com/`
（根路径返回 **403** —— 说明服务器存在且仍在运营，只是禁止列目录，不是域名失效）

**可用文件（实测 200 可直接下载）：**

| 文件名 | 直链 | 字节数 | 大小 |
|---|---|---|---|
| `NX595J_Z69_EN_VNG0N_V112.zip` | `http://rom.download.nubia.com/Europe&Asia/Z17S/NX595J_Z69_EN_VNG0N_V112.zip` | 2,376,008,975 | 2265.94 MB |
| `NX595J_Z69_EN_VNG0N_V111.zip` | `http://rom.download.nubia.com/Europe&Asia/Z17S/NX595J_Z69_EN_VNG0N_V111.zip` | 2,386,906,528 | 2276.33 MB |

**探测过的无效路径（不用再试了）：**

| 路径 | 结果 |
|---|---|
| `.../Z17S/NX595J_Z69_EN_VNG0N_V106.zip` | ❌ 404（已下架） |
| `http://rom.download.nubia.com/China/Z17S/` | ❌ 404 |
| `http://rom.download.nubia.com/CN/Z17S/` | ❌ 404 |
| `http://rom.download.nubia.com/Mainland/Z17S/` | ❌ 404 |
| `.../Z17S/NX595J_CNCommon_V2.22.zip` | ❌ 404 |
| `http://rom.download.nubia.com/Asia/Z17S/` | ❌ 404 |

> ⚠️ **官方服务器上只有 EN 国际版**。CN 国行版（`NX595J_CNCommon_*`）的所有候选路径都 404，找不到可直连的国行固件。你的国行机器只能拿到 EN 版作为备用。

> ⚠️ **这是 OTA 卡刷包，不是 EDL 线刷包 —— 救不了硬砖。** 它需要在 recovery 里刷入，价值是「系统起不来但还能进 recovery 时，有个官方系统可以刷回去」。真正能救「完全黑屏、任何按键无反应」的是 9008 模式用的 firehose 线刷包（含 `prog_firehose_ddr.elf`），**这个找不到公开下载**。

> ⚠️ **型号必须完全匹配。** 努比亚 Z17 / Z17 mini / Z17s 的固件包互不兼容，刷错轻则无限重启、重则彻底黑砖。认准文件名里的 `NX595J`。

**下载完成后一定要核对字节数和 SHA256**（官方没有公布校验值，下面两个值是本次实测算出来的，可作为唯一基准）：

```powershell
(Get-Item .\NX595J_Z69_EN_VNG0N_V112.zip).Length              # 应等于 2376008975
(Get-FileHash .\NX595J_Z69_EN_VNG0N_V112.zip -Algorithm SHA256).Hash
# 应等于 F75E8CC068B5D7D2503B39E07E72BBD28A34375D6BD2DFBB7217CAB639F69058
```

> **官方链接失效时**，用本文档开头的[离线包](#-官方链接失效了怎么办--离线包) —— 该固件已按 1200 MB 切成两卷（`...V112.zip.part1.rar` / `...part2.rar`）归档在同一个 release 里，解压后同样要做上面这两项核对。

---

### 6. 其它参考链接

| 内容 | 链接 |
|---|---|
| XDA 讨论帖（解锁命令来源） | `https://xdaforums.com/t/z17s-nx595j-unlock-bootloader-twrp.3762206/` |
| 解锁黑名单项目（ZTE/nubia 章节） | `https://github.com/zenfyrdev/bootloader-unlock-shame` / 仓库名 `bootloader-unlock-wall-of-shame` |
| TWRP 官方设备列表（**无 nubia Z17s**，所以只能用社区版） | `https://twrp.me/Devices/` |
| 高通 9008 驱动（救硬砖用） | 搜 `Qualcomm HS-USB QDLoader 9008 driver` |
| 努比亚官网下载中心 | 官网 → 服务 → 下载中心（有 Nubia USB Driver 与官方助手工具） |

> **链接失效时的替代检索方法**：TWRP 和 Magisk 的发布页地址是稳定的（`/releases`），直接在页面里找对应版本即可；固件包若哪天 404，用 `site:rom.download.nubia.com NX595J` 或到 XDA 帖里找当时的存档链接。

---

### 7. 下载实战技巧（本次踩过的坑）

**① 先测速，再下大文件。** 本次环境里有条失效的代理环境变量（`127.0.0.1:12903`），会让下载降到约 80 KB/s；加 `--noproxy '*'` 走直连后，同一地址首段可达 6 MB/s（快 75 倍）。

```bash
# 各下 3 MB 对比速度，再决定用哪条线路
curl.exe -s -L --noproxy '*' -r 0-3145727 -o t1.bin -w "直连 speed=%{speed_download}B/s\n" <URL>
curl.exe -s -L --proxy http://127.0.0.1:7897 -r 0-3145727 -o t2.bin -w "代理 speed=%{speed_download}B/s\n" <URL>
```

**② 大文件务必用断点续传**，努比亚的 CDN 对持续大文件会限速（本次约 400 KB/s），中途断线很常见：

```bash
curl.exe -L --noproxy '*' -C - --retry 6 --retry-all-errors \
  --connect-timeout 30 --speed-time 90 --speed-limit 2048 \
  -o NX595J_Z69_EN_VNG0N_V112.zip.part "<URL>"
```

**③ 下完以字节数为准，不要靠肉眼看进度条。** 本次就因为「任务退出码 0」误判成下载完成，实际只下到 118 MB / 2266 MB。用 `-o` 写到 `.part` 文件，下完比对字节数再改名。

**④ 别用 `--retry` 掩盖错误。** `curl ... | Out-File` 这类管道会把 curl 的真实退出码吃掉（管道返回的是 `Out-File` 的码）。要取退出码，让 curl 单独执行、紧接着读 `$LASTEXITCODE`。

---

## 三个关键难点

### 难点 1：Windows 缺 fastboot 驱动，且 Google 的驱动包不覆盖这台设备

设备在 fastboot 模式下的 USB 标识：

```
VID_18D1&PID_D00D          ← Android 通用 fastboot PID
CompatibleId: USB\Class_FF&SubClass_42&Prot_03
```

而 Google USB Driver 的 `android_winusb.inf` **不含 `D00D`** —— 它只列举了 Nexus/Pixel 那几个：`4EE0` / `2C10` / `9004` / `9006` / `4D00`。它也没有 `Class_FF/SubClass_42`（fastboot 接口类）的通配条目。所以**不会自动匹配**，必须手动强制安装。

**先确认症状**（PowerShell 里查设备状态）：

```powershell
Get-PnpDevice -PresentOnly | Where-Object { $_.InstanceId -match 'VID_18D1' } |
  Select-Object Status, Class, FriendlyName, InstanceId

# 看问题代码 —— Code 28 就是"驱动未安装"
Get-PnpDeviceProperty -InstanceId 'USB\VID_18D1&PID_D00D\CBADD6E3' -KeyName 'DEVPKEY_Device_Problem'
```

**正确装法（9 步，一步都不能跳）：**

1. 右键开始菜单 → 打开**设备管理器**
2. 找到带黄色感叹号的 `Android` 设备（通常在「其他设备」下）
3. 右键 → **更新驱动程序**
4. 选 **「浏览我的电脑以查找驱动程序」**
5. ⚠️ **不要点「浏览…」按钮**，点这一屏下面那行蓝色链接 **「让我从计算机上的可用驱动程序列表中选取」**
6. 点 **「从磁盘安装…」**
7. 在**文件名输入框**里直接粘贴完整路径（不用一层层点目录）：
   ```
   C:\你的路径\usb_driver\usb_driver\android_winusb.inf
   ```
8. 型号列表里选 **`Android ADB Interface`** 或 **`Android Bootloader Interface`**（两者 Service 都是 WinUSB，实测都能用）
   - 如果列表里看不到这两项，回上一步**取消勾选「仅显示兼容硬件」**再选
9. 弹出「不推荐安装此驱动程序」→ 选 **「是」** → 等感叹号消失

**装完后让设备重新枚举一次**（拔插 USB 或 `adb reboot bootloader`），否则新驱动不会加载。

> **最大的卡点**：如果第 5 步点了「浏览…」按钮，会弹出「**浏览文件夹**」对话框 —— **那个对话框只列文件夹、不列文件**（它的设计就是让你选文件夹），而且选完文件夹后 Windows 拿 inf 里的硬件清单自动匹配，因为不含 `D00D` 必定失败，报「找不到适用于此设备的驱动程序」。很多人在这里反复尝试、以为文件丢了，**实际上文件一直在**（压缩包里套了两层同名目录 `usb_driver\usb_driver\`，也容易看混）。**「选文件夹」和「从磁盘安装」是两条不同的路，后者才是能强制指定的那个。**

**怎么判断装对了**：

```powershell
Get-PnpDeviceProperty -InstanceId 'USB\VID_18D1&PID_D00D\<你的序列号>' -KeyName 'DEVPKEY_Device_Service'
# 期望输出：WinUSB
```

### 难点 2：fastboot 的输出全在 stderr，PowerShell 完全抓不到

这条坑了整整一轮排查。**现象是「命令没有任何输出」，极容易误判成驱动坏了、通信断了。**

- `fastboot getvar` / `oem` / `reboot` 的**所有信息性输出都写到 stderr**
- 只有 `fastboot devices` 走 stdout
- 下面这些 PowerShell 写法**一律返回空字符串**（实测过，全部无效）：

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
print("rc =", p.returncode)
print(p.stderr)      # ← 真正的输出在这里
```

**更关键的：怎么区分「命令真的失败了」和「只是拿不到输出」。** 三个可靠信号：

1. **设备状态变了没有** —— `fastboot reboot` 输出全空，但设备 10 秒后确实重启进了系统，说明命令执行了
2. **累加型字段变了没有** —— `fastboot oem device-info` 里的 `fastboot unlock count` 每执行一次解锁就 +1，即使看不到解锁回显，count 变了就证明命令下发成功
3. **`fastboot devices` 走 stdout，是唯一能正常读到的命令** —— 它能列出设备，不代表通信正常（它只读 USB 描述符里的序列号，不走 fastboot 协议）

**通用教训**：拿不到输出时，**先怀疑「是不是输出被吞了」，而不是「命令失败了」**。

### 难点 3：bootloader 命令被大幅裁剪，进 recovery 只能靠屏幕菜单

最耗时间的一处。实测这条设备对 fastboot 命令的裁剪程度：

| 命令 | 结果 |
|---|---|
| `fastboot oem reboot-recovery` | ❌ `FAILED (remote: 'unknown command')` |
| `fastboot boot twrp.img` | ❌ 镜像发送 OKAY，但 `Booting FAILED (remote: 'unknown command')` |
| `fastboot oem poweroff` | ❌ `unknown command` |
| `fastboot oem reboot recovery` | ❌ 直接挂起超时 |
| `fastboot reboot recovery` | ⚠️ **返回 `OKAY` 却实际进了系统**（假成功，最坑） |
| `fastboot reboot` | ✅ 正常 |
| `fastboot flash recovery <img>` | ✅ 正常 |
| `fastboot oem nubia_unlock NUBIA_NX595J` | ✅ 正常 |
| `fastboot oem device-info` | ✅ 正常 |
| `fastboot getvar <任意>` | ✅ 正常 |

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

> **关于 `fastboot reboot recovery` 的假成功**：它返回 OKAY 并让设备正常启动进系统，而**原厂系统在启动时会检测 recovery 分区并还原它**（系统里有 `ro.expect.recovery_id` 属性，说明存在自动恢复机制）。所以刷完 TWRP 必须立刻用菜单进 recovery，**绝不能让它正常开机**。
>
> **关于长按电源键**：在 fastboot 模式下长按电源键，很多设备是「重启」而不是「关机」—— 一重启就进系统、recovery 立刻被覆盖。这也是本次前两次尝试失败的真正原因。**优先用屏幕菜单，别靠按键组合。**

---

## 完整流程

### 前置准备

先按上面的[下载链接](#全部下载链接实测核对)把文件全部下好、逐个核对 SHA256 / 字节数：

```
platform-tools.zip                       7.67 MB     工具链
usb_driver_r13-windows.zip               8.28 MB     fastboot 驱动
twrp_recovery_3.7.0_9_for_nx595j-*.img   32.24 MB    recovery
Magisk-v26.4.apk                         11.95 MB    root（改名 .zip 可刷）
NX595J_Z69_EN_VNG0N_V112.zip             2265.94 MB  救砖固件（可选但建议）
```

### 步骤 1：备份数据

```bash
adb pull /sdcard/ ./backup/
```

本机实测用户数据约 1 GB（含 `/sdcard/Android` 的应用数据），其中照片视频等真正重要的部分只有约 50 MB。

备份完**核对文件数与总字节数**：

```powershell
$f = Get-ChildItem .\backup -Recurse -File
$f.Count; ($f | Measure-Object Length -Sum).Sum
```

本次实测：1159 个文件 / 1001.6 MB。

### 步骤 2：装 fastboot 驱动

见上面[难点 1](#难点-1windows-缺-fastboot-驱动且-google-的驱动包不覆盖这台设备)。装好后让设备重新枚举一次（拔插 USB 或 `adb reboot bootloader`）。

### 步骤 3：解锁 Bootloader

```bash
adb reboot bootloader
fastboot oem nubia_unlock NUBIA_NX595J
```

**型号必须大写并加 `NUBIA_` 前缀。**

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

> **小技巧**：`fastboot unlock count` 会累加，可以拿它判断命令是否真的下发成功了 —— 即使你看不到输出（见难点 2）。

**如果命令无反应**：回到开发者选项，把「OEM 解锁」开关**反复关闭再打开几次**再试（XDA 上多位用户报告过这个怪癖）。

#### ⚠️ 关键：这是「会话级」解锁，重启即失效

实测结论，**这条非常重要**：

- 解锁后 `Device unlocked: true`
- **但设备一重启就变回 `false`**
- `ro.boot.flash.locked` 始终为 `1`，`ro.boot.verifiedbootstate` 始终 `green`
- 数据也**从未被清空**（`df /data` 前后都是约 6.4 GB / 6417740 KB），证明解锁没有真正持久化

**试过但无效的持久化手段**（不用再重复踩了）：

- `settings put global oem_unlock_enabled 1` —— 能写入并读回 `1`，但 `sys.oem_unlock_allowed` 仍是 `0`，**且重启后被重置**
- Android 7.1.1 的努比亚 ROM **没有** Android 8 才引入的 `OemLockService` 机制，重启后 `sys.oem_unlock_allowed` 属性直接消失
- `ro.oem_unlock_supported = 1`（硬件支持），但 `sys.oem_unlock_allowed = 0`

**推论**：努比亚 bootloader 的解锁标志只在**当前 fastboot 会话内**有效。**必须趁状态还在立刻刷写，中途绝不能重启。**

> **好消息**：正因为解锁没持久化，**数据全程没有被清空**。网上所有「解锁必清数据」的说法在这台机器上不成立。

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

> 备用方式：菜单里选 `Power off` 关机，然后按住「音量上 + 电源键」。但实测**菜单更可靠**。

**判断成功**：

```bash
adb devices                        # 状态应显示 recovery（不是 device）
adb shell "getprop ro.twrp.version"  # 应输出 3.7.0_9-0
adb shell "ls /twres"              # 应看到 fonts images languages ui.xml
```

> 如果 `adb devices` 显示 `device` 而不是 `recovery`，且 `getprop` 里能查到 `zygote` / `sys.boot_completed=1`，说明进了**系统**而不是 recovery —— recovery 已被原厂覆盖，回到步骤 4 重刷。

### 步骤 6：刷入 Magisk

**不要推到 `/sdcard`** —— 这台机器 data 分区加密，TWRP 环境下 `/sdcard` 不可用（`ls` 出来全是乱码文件名）。**推到 `/tmp`（TWRP 的内存文件系统）：**

```bash
# apk 改名为 zip 即可，Magisk 的 apk 本身就是可刷入的 zip
copy Magisk-v26.4.apk Magisk-v26.4.zip

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

> **App 首页显示的版本号有时会和实际安装的对不上**（例如显示「无法获取版本号」）。以命令行的三个检查为准。

---

## 已知副作用

- 解锁后依赖设备完整性的功能（**指纹支付、部分银行 App**）可能失效
- 系统 OTA 升级功能失效
- 官方保修失效（Z17s 早已过保，影响不大）
- 开机时可能显示解锁状态警告
- **本次解锁是会话级的，重启即回锁** —— 所以上述副作用在你重启后可能反而不明显，但同样意味着将来再刷 recovery 要重走一遍流程

---

## 环境与工具链备注

这些是排查过程中踩到的、和刷机本身无关但会浪费大量时间的坑：

**PowerShell / Windows：**

- `fastboot` 输出走 stderr 抓不到（见难点 2）
- PowerShell 工具的 stdout 会被吞，必须 `Out-File` 写文件再读
- `cmd /c` 可能被安全策略拦截
- 脚本中某条 `adb shell` 返回非零时，整个 PowerShell 命令会以 exit 1 结束，**并且可能完全不写文件**（表现像「脚本没跑」）。调试时先跑最简命令（只 `adb devices`）确认链路，再逐步加回复杂逻辑
- 含 `su` 的命令会挂起，导致脚本超时且文件不生成 → 调试时把 su 单独拿出来、加 timeout 包裹
- `.inf` 文件在资源管理器里默认隐藏扩展名，会显示成 `android_winusb`，别以为文件不存在

**网络：**

- 下载大文件前先测速（见[下载技巧](#7-下载实战技巧本次踩过的坑)）
- XDA Forums 常被 bunny.net 拦截返回 403，拿不到正文时改用搜索引擎摘要或 `web.archive.org` 镜像

**判断设备状态：**

- **判断数据是否丢失要看 `df /data`，不要看 `ls`** —— 系统运行中 `ls /sdcard` 会返回空、`ls /storage/emulated/0` 会返回一堆随机文件名（sdcardfs / 加密视图的正常表现），但数据完好无损。本次实测刷机前后 `df /data` 都是 6417740 KB
- 区分系统 / recovery：`ps` 里有 `zygote64` / `system_server`、`sys.boot_completed = 1` 说明在完整系统里；TWRP 环境有 `ro.twrp.version` 和 `/twres` 目录
- 设备在哪个模式：`VID_19D2` 是中兴/努比亚厂商 ID（MTP 模式下 FriendlyName 直接显示型号 NX595J），`VID_18D1&PID_D00D` 是 Android 通用 fastboot 接口。查 `InstanceId` 比 `FriendlyName` 可靠

---

## 文件说明

- `nubia-z17s-root-guide.html` —— 分阶段操作手册，浏览器直接打开即可。含可勾选的准备清单（带真实下载链接）、一键复制的命令行、排错对照表、救砖分级流程

---

## 参考资料

- XDA 讨论帖 *Z17S (NX595J) Unlock Bootloader & TWRP* —— 提供 `fastboot oem nubia_unlock` 命令与 OEM 开关反复触发的经验
- 开源项目 `zenfyrdev/bootloader-unlock-wall-of-shame`（ZTE / nubia 章节）—— 确认努比亚骁龙机型的解锁命令格式为 `fastboot oem nubia_unlock NUBIA_<MODEL>`
- GitHub `Cyborg2017/android_device_nubia_nx595j-twrp` —— Z17s 的 TWRP 镜像来源
- GitHub `topjohnwu/Magisk` —— Magisk 官方发布页
- 努比亚官方 ROM 服务器 `rom.download.nubia.com` —— 官方固件直链来源
- LineageOS Wiki（NX563J 安装页）—— 佐证「刷完 recovery 不得直接重启进系统」这一通用约束

---

## 免责声明

刷机有风险。本文档为单机实测记录，不能保证适用于所有批次的 NX595J。按此操作导致的任何后果请自行承担。**动手前务必备份数据**，并尽量准备好可用的固件包。
