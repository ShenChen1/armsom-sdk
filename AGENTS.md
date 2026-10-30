# Repository Guidelines

## 项目结构与模块组织

本目录是由 Repo 管理的 ArmSoM/Rockchip Linux SDK，根目录本身不是 Git 仓库；在对应子仓库内检查和提交修改。

- `device/rockchip/common/`：构建脚本、钩子与公共配置；根目录 `build.sh`、`Makefile`、`README.md` 链接至此。
- `device/rockchip/.chips/rk3576/`：板级 defconfig、分区表与打包配置。
- `kernel-6.1/`、`kernel-5.10/`：内核、驱动与设备树；`kernel/` 是可切换的链接。
- `u-boot/`：引导程序；`rkbin/`、`armsom-initrd/`：固件二进制与初始内存盘；`prebuilts/`、`tools/`：工具链和工具。
- `debian11/`、`debian12/`、`ubuntu22.04/`、`ubuntu24.04/`：根文件系统构建、覆盖文件与调试资源。
- `output/`：生成配置、日志和镜像；`rockdev/` 链接到 `output/firmware/`。

## 构建与开发命令

在 SDK 根目录使用 Bash。先确认开发板、根文件系统和内核版本；不要直接套用 README 中其他芯片的示例。

```bash
./build.sh chip       # 交互选择芯片和板级配置
./build.sh uboot      # 构建引导程序
./build.sh kernel     # 构建内核及配置要求的启动镜像、deb 包
./build.sh ubuntu     # 构建 Ubuntu 根文件系统；Debian 使用 debian
./build.sh firmware   # 整理分区镜像
./build.sh updateimg  # 打包 update.img
```

`./build.sh` 执行默认完整镜像构建；`./build.sh help` 查看可用目标。日志位于 `output/sessions/`。板级配置必须匹配硬件，例如 `armsom_cm5_rk3576_ubuntu_server_defconfig` 适用于 CM5 对应配置。

## 编码风格与命名

遵循所属子项目及相邻代码风格；内核 C、设备树和公共构建脚本使用现有制表符缩进，Makefile 配方必须使用制表符。保留 `RK_*` 配置命名、`*_defconfig` 板级命名及 `<priority>-<hook>.sh` 钩子命名。内核提供 `.clang-format` 和 `scripts/checkpatch.pl`，避免整文件格式化造成无关差异。

## 测试与验证

内核包含 KUnit 和 `tools/testing/selftests/`；按修改范围选择测试，本 SDK 没有统一的根目录测试命令或覆盖率门槛。

- Shell 修改先运行 `bash -n <script.sh>`，并验证正常输入与错误输入。
- 内核补丁可运行 `kernel/scripts/checkpatch.pl --no-tree <patch>`，再使用目标板配置编译。
- 新功能添加针对性测试；修复缺陷先添加能复现问题的回归测试。
- 板上外设测试入口如 `ubuntu22.04/overlay-debug/rockchip-test/rockchip_test.sh`；将相应测试资源部署到目标板后运行，按模块选择测试。

记录板型、defconfig、内核版本、命令和结果，分别说明编译验证与真实硬件验证。

## 提交与 Pull Request

当前清单使用浅克隆，可见历史包含 `add armsom-sige7s`、`fix: del waveshare_backlight` 等标题，未形成统一前缀规范。使用简短、明确的英文标题；每次提交只处理一个问题，并在对应子仓库操作。

PR 说明问题、行为变化、关联 Issue、假设、受影响板型及验证结果；界面变化附截图，启动或驱动变化附相关日志。修改前完整阅读相关文件、定义和调用路径，比较至少两种方案后选择最简单的可行方案。

## 配置与产物管理

持久配置修改应落在受版本控制的 defconfig 或覆盖文件中。不要提交 `output/`、生成的镜像、deb 包或环境快照；分享日志前移除凭据和密钥。
