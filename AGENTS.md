# AGENTS.md

## 仓库定位

本仓库将上游 GUI.for.SingBox 二进制打包为 Arch Linux 的 AUR 软件包 gui-for-singbox-bin。

相关仓库与页面：
- 上游仓库：https://github.com/GUI-for-Cores/GUI.for.SingBox
- 上游发布页：https://github.com/GUI-for-Cores/GUI.for.SingBox/releases
- AUR 软件包页：https://aur.archlinux.org/packages/gui-for-singbox-bin
- GitHub 托管与 CI 仓库：https://github.com/m4rsquake/gui-for-singbox-bin

## Git 远端与文件规范

本地配置了双远程仓库：
- origin: git@github.com:m4rsquake/gui-for-singbox-bin.git (GitHub 主仓库)
- aur: ssh://aur@aur.archlinux.org/gui-for-singbox-bin.git (AUR 官方源)

重要规范：
- AUR 仓库仅允许存放打包必需文件（PKGBUILD, .SRCINFO, install, rules, hook, launcher, desktop 等）。
- AGENTS.md、domain-model.md、.github 目录等非打包文件仅由 GitHub 仓库托管，由 GitHub Actions 负责过滤并发布至 AUR，禁止直接推送到 AUR 仓库。

## 自动化更新机制 (GitHub Actions)

仓库通过 .github/workflows/auto-update.yml 实现自动化监控与同步：
- 触发方式：每 4 小时定时轮询上游 Releases API，支持 workflow_dispatch 手动触发。
- 更新逻辑：
  1. 对比当前 PKGBUILD 的 pkgver 与上游最新 tag。
  2. 若有更新，在 Arch Linux 容器内运行 updpkgsums 更新校验和，并运行 makepkg --printsrcinfo > .SRCINFO。
  3. 自动提交并 push 回 GitHub 仓库。
  4. 使用 GitHub Secret (AUR_SSH_PRIVATE_KEY) 连接 AUR，仅将打包必需文件同步并推送到 AUR master 分支。

## 软件包版本模型

软件包有两个相互独立的分组：
- pkgver 跟随上游 GitHub release tag，不包含开头的 v。
- pkgrel 表示同一上游版本下的打包层变更次数。

上游版本变化时，pkgrel 重置为 1。仅打包配置或集成逻辑调整时递增 pkgrel。

当前软件包将 x86_64 二进制安装至 /opt/gui-for-singbox/GUI.for.SingBox，命令入口为 /usr/bin/GUI.for.SingBox。
依赖包括 glibc, libcap, polkit, sing-box-bin, webkit2gtk-4.1。

## 现有集成细节

- 安装脚本仅预先创建 /opt/gui-for-singbox/data 与 data/sing-box (777 权限)，其余运行时文件由 GUI 按需创建。
- 安装与升级时为 /usr/bin/sing-box 配置网络 capability (cap_net_admin, cap_net_raw, cap_net_bind_service=ep)。
- 通过 alpm hook 在 sing-box 核心升级后自动恢复 capability，卸载本包时自动移除 capability。
- 通过 polkit 规则允许本机活动会话用户管理 TUN 相关的 systemd-resolved 路由与 DNS 设置。

## 上游版本快照

截至 2026-09-11，最新上游 release 为 v1.27.0（发布于 2026-09-07）。
已通过 GitHub Actions 自动更新并同步至 AUR。
