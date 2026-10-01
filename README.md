# 寒山小说发布工具 · macOS 版

从 Windows 版 `app(1).exe`（PyInstaller + Python 3.10 + tkinter + Playwright）逆向还原并移植到 macOS。

原始程序是一个网文自动发布工具，支持番茄 / 起点 / 七猫 / 飞卢 / 微信 / 书旗 / 晋江 / 息壤 / 咪咕
九个平台，通过 Playwright 驱动真实浏览器完成登录、建书、发章节、自动签约、全勤检查等流程。

---

## 怎么拿到 macOS 应用

### 方式一：用本仓库的 GitHub Actions 自动构建（推荐，不需要 Mac）

1. 打开本仓库的 **Actions** 标签页
2. 左侧选「**构建 macOS 应用**」
3. 右侧点「**Run workflow**」，架构选 `arm64`（Apple Silicon）或 `x86_64`（老 Intel Mac）
4. 等 5-10 分钟
5. 在该次运行的 **Artifacts** 区下载 `寒山小说发布工具-macOS-arm64.zip`
6. 解压得到 `寒山小说发布工具.app`，拷到 Mac 双击

> 公开仓库的 macOS runner 免费无限量。构建发生在 GitHub 托管的真 macOS 机器上，
> 产出的是**自包含**的 .app（内置 Python 运行时），不依赖目标机器上装没装 Python。

### 方式二：在 Mac 上本地构建

```bash
cd novel_publisher_mac
chmod +x *.sh *.command assets/*.sh
./build_mac.sh
```

前置条件：Mac 上要有带 tkinter 的 Python 3.10
（[官网安装包](https://www.python.org/downloads/release/python-31011/) 自带；
Homebrew 需 `brew install python@3.10 python-tk@3.10`）。

### 方式三：不打包，直接跑

```bash
cd novel_publisher_mac
chmod +x 启动.command
./启动.command
```

---

## 首次打开被 macOS 拦截

CI 构建出来的 .app 是 **ad-hoc 签名**，不是 Apple 开发者证书签发的。
在你自己的机器上：

- 右键 App → **打开** → 再点「打开」
- 或终端执行：`xattr -dr com.apple.quarantine 寒山小说发布工具.app`

要分发给别人又不想让对方做这一步，需要 Apple Developer 账号（$99/年），
详见 `novel_publisher_mac/README_MAC_BUILD.md` 第 4 节。

---

## 数据目录

应用的可写文件都在 App 包外面：

```
~/Library/Application Support/寒山小说发布工具/
├── config.ini           界面配置
├── auth.json            登录态（Playwright storage_state）
├── 签约信息.txt          自动签约用
├── Shortcuts/
│   ├── contrast.json    笔名 → 调试端口 映射
│   └── <笔名>.command   「浏览器分身」启动器（Windows 版是 .lnk）
└── <端口>/              各分身的浏览器 user-data-dir
```

彻底重置：删掉整个目录即可。

---

## 工程结构

```
.
├── .github/workflows/build-macos.yml    CI：用 macOS runner 打包 .app
└── novel_publisher_mac/
    ├── app.py                           主程序（5700+ 行，135 个函数/方法）
    ├── novel_publisher/                 13 个业务模块
    ├── macos_support.py                 macOS 平台适配层
    ├── platform_patch.py                启动时替换 Windows 专有实现
    ├── browser_detector.py              Chrome/Edge/Chromium 探测
    ├── novel_publisher_mac.spec         PyInstaller 配置
    ├── rthook_frozen_macos.py           macOS 运行时钩子
    ├── gen_icns.py                      纯 Python 生成 .icns
    ├── build_mac.sh / make_dmg.sh       打包脚本
    ├── selftest_business.py             业务自检
    └── README_MAC_BUILD.md              完整打包与排错文档
```

---

## macOS 与原版的差异

| 项 | Windows 原版 | macOS 版 |
|---|---|---|
| 创建分身 | `.lnk`（win32com） | `.command` 脚本（启动参数完全一致） |
| 打开文件/目录 | `os.startfile` | `open` / `open -R` |
| 数据目录 | `%APPDATA%\寒山小说发布工具` | `~/Library/Application Support/寒山小说发布工具` |
| 托盘图标 | `icon.run()` 子线程 | `run_detached(darwin_nsapplication=NSApp)` |
| 结束占用进程 | `taskkill` | `os.kill(pid, 15)` |
| 界面字体 | 微软雅黑 | PingFang SC |

---

## 说明

本工具仅用于学习和研究。还原与移植过程严格以原始字节码为准：
`app.py` 中 134 个 code object 全部在场，83/134 的 `co_code` 逐字节相同，
127/134 的函数签名完全相同，135 处关键字调用参数与原始字节码一致。
