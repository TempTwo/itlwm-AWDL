# itlwm-AWDL（Intel 无线网卡魔改版）

> 为 macOS 15 Sequoia 上的 Intel 无线网卡补上 **AWDL / AirDrop** 能力。

## 项目来源

本项目是 fork，代码来源链条如下：

- 上游：[OpenIntelWireless/itlwm](https://github.com/OpenIntelWireless/itlwm)（作者：钟先耀 / zxystd）
- 二级 fork：[laobamac/itlwm](https://github.com/laobamac/itlwm)（Sequoia / Tahoe 适配）
- 直接来源：[crazyhulk/itlwm](https://github.com/crazyhulk/itlwm) 的 `awdl-airdrop` 分支

本仓库在直接来源的基础上整理、补全并以正式魔改版发布，**保留了完整的提交历史与原始许可证**。

## 本版改动（相对直接来源）

1. **AWDL 虚拟接口创建**：允许在 Sequoia（`__MAC_15_2`）上创建 AWDL VIF，同时保持 P2P 关闭，避免关联 AP 时的启动延迟。
2. **Action Frame 发送**：实现 `outputActionFrame` 与 `bpfOutput80211Radio`，把 awdld 经 BPF 写入 `awdl0` 的 AWDL action frame 交给固件发送。
3. **AWDL 接收注入**：新增 `ic_awdl_rx` 回调，处理 802.11 action category 127，把收到的 AWDL 帧注入 `awdl0`，供 awdld 通过 BPF 读取。
4. **数据发送路径**：恢复 `requestPacketTx` 中的 AWDL 分支。
5. 补充 AWDL 相关辅助头文件（`itlwm/awdl/`）。

## 适配情况

| 项目 | 说明 |
|---|---|
| 实测网卡 | **Intel AX211** |
| 实测系统 | **macOS 15 Sequoia**（15.2+，实测 15.8） |
| 其他 Intel 网卡 | 沿用上游 `IOPCIMatch` 支持列表，AWDL 功能未逐一验证 |
| macOS Tahoe 26.x | 仓库保留上游 Tahoe target，但本魔改的 AWDL 改动针对 Sequoia |
| 蓝牙侧配合 | 需同时使用 [BlueToolFixup-FixA2DP-BLE](https://github.com/TempTwo/BlueToolFixup-FixA2DP-BLE) 并添加 `-btlfxadvsets` |

> 未验证的网卡/系统组合请自行测试，不保证可用。

## 安装方法

### 方式一：使用 Release（推荐）

1. 从 [Releases](https://github.com/TempTwo/itlwm-AWDL/releases) 下载 `AirportItlwm.kext`（Sequoia 版）。
2. 放入 OpenCore 的 `EFI/OC/Kexts/`。
3. 在 `config.plist` 的 `Kernel → Add` 中加入该 kext，设置 `Enabled = true`。
4. 确保**没有**同时加载其他 Wi-Fi kext（`itlwm.kext` 与 `AirportItlwm.kext` 只能二选一）。
5. 重启。

### 方式二：自行编译

需要 Xcode 与 MacKernelSDK：

```sh
xcodebuild -project itlwm.xcodeproj -scheme AirportItlwm-Sequoia \
  -configuration Release ARCHS=x86_64 CODE_SIGNING_ALLOWED=NO build
```

### 可选启动参数

- `itlwm_he=1`：启用 Wi-Fi 6（802.11ax / HE），默认关闭。

## AirDrop 使用前提

1. Intel 网卡加载本 kext；
2. 蓝牙侧加载 BlueToolFixup 魔改版，并添加 `-btlfxadvsets`；
3. 重启后确认系统中存在 `awdl0` 接口；
4. 与 iPhone / iPad 之间测试 AirDrop。

## 已知限制

- 仅在 **Intel AX211 + macOS 15 Sequoia** 上完整验证。
- WPA3-only、强制 PMF、MLO 不支持（沿用上游限制）。
- 其他网卡/系统可能需要额外调整，欢迎提交 Issue。

## 开源声明

本项目基于 **GPL-2.0** 许可证发布，完整许可证见 [LICENSE](./LICENSE)。

- 原始代码版权归 OpenIntelWireless、钟先耀（zxystd）及各位贡献者所有。
- 本仓库仅对现有代码进行魔改与整理，**不主张原始代码版权**。
- 按 GPL-2.0 要求，分发时必须保留版权声明与许可证文本。
- 本项目按“现状”提供，不提供任何明示或暗示担保；替换 kext 有风险，请自行备份 EFI。

## 致谢

感谢 OpenIntelWireless 社区、laobamac 的 Sequoia/Tahoe 适配，以及所有提交 PR 与反馈的贡献者。
