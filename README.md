# FullScreenClock

一个基于 Electron 和 HTML 的全屏时钟应用。

## 目录

- [环境准备](#环境准备)
- [本地开发](#本地开发)
- [沙盒权限问题](#沙盒权限问题)
- [打包配置](#打包配置)
- [GitHub Actions 自动构建](#github-actions-自动构建)

---

## 环境准备

### 1. 克隆仓库

    git clone https://github.com/MarcusEddie/fullscreenclock.git
    cd fullscreenclock

### 2. 安装 Node.js

**Ubuntu/Debian**

    # 安装 Node.js 22.x（当前 LTS）
    curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
    sudo apt-get install -y nodejs
    
    # 验证
    node -v
    npm -v

**macOS**

    brew install node
    node -v
    npm -v

**Windows**

访问 [Node.js 官网](https://nodejs.org/) 下载 LTS 版本安装包，双击运行安装程序，打开 PowerShell 验证：

    node -v
    npm -v

### 3. 安装项目依赖

    npm install

### 4. 验证 Electron 安装

    npx electron -v

---

#### 4.1 沙盒权限问题

在 Linux 系统上运行 npx electron -v 时，可能会遇到沙盒权限错误：

    The SUID sandbox helper binary was found, but is not configured correctly.

**解决方案一：禁用沙盒（开发环境推荐）**

修改 package.json 的 start 脚本：

    {
      "scripts": {
        "start": "electron . --no-sandbox"
      }
    }

或者设置环境变量后运行：

    export ELECTRON_DISABLE_SANDBOX=1
    npx electron -v

**解决方案二：配置 SUID 沙盒（生产环境推荐）**

    sudo chown root:root node_modules/electron/dist/chrome-sandbox
    sudo chmod 4755 node_modules/electron/dist/chrome-sandbox

    然后再验证：
    npx electron -v

---
## 本地开发

### 启动应用

    npm start

---


## 打包配置

### package.json build 配置

    {
      "build": {
        "appId": "com.yourname.fullscreenclock",
        "productName": "FullScreenClock",
        "linux": {
          "target": ["AppImage", "deb"],
          "category": "Utility"
        },
        "win": {
          "target": ["portable"]
        },
        "mac": {
          "target": ["dmg"]
        }
      }
    }

### Windows 打包

**格式说明**

| 格式 | 说明 | 生成文件 |
|------|------|----------|
| portable | 便携版，单文件直接运行，无需安装 | FullScreenClock 1.0.0.exe |
| nsis | 安装程序，引导用户安装到系统，支持卸载 | FullScreenClock Setup 1.0.0.exe |
| nsis-web | 在线安装程序，安装时下载文件 | 小体积安装程序 |
| msi | Windows Installer 格式，适合企业部署 | .msi 文件 |
| appx | Windows 应用商店格式（需 Windows 10 打包） | .appx 文件 |

**打包命令**

    # portable 版（64 位）
    npx electron-builder --win portable --x64

    # portable 版（32 位）
    npx electron-builder --win portable --ia32

    # nsis 安装程序（64 位）
    npx electron-builder --win nsis --x64

    # 同时打包 portable 和 nsis
    npx electron-builder --win portable nsis --x64

    # 跳过代码签名（无证书时使用）
    npx electron-builder --win --x64 -c.win.signAndEditExecutable=false

**package.json 配置示例**

    "win": {
      "target": [
        {
          "target": "portable",
          "arch": ["x64"]
        },
        {
          "target": "nsis",
          "arch": ["x64", "ia32"]
        }
      ]
    }

### Linux 打包

**格式说明**

| 格式 | 说明 | 适用发行版 |
|------|------|------------|
| AppImage | 单文件便携版，无需安装 | 所有发行版 |
| deb | Debian 安装包 | Ubuntu, Debian |
| rpm | RPM 安装包 | Fedora, CentOS, RHEL |
| snap | Snap 包 | 支持 Snap 的发行版 |
| flatpak | Flatpak 包 | 支持 Flatpak 的发行版 |

**打包命令**

    # AppImage（64 位）
    npx electron-builder --linux AppImage --x64

    # AppImage（ARM64）
    npx electron-builder --linux AppImage --arm64

    # deb 包（64 位）
    npx electron-builder --linux deb --x64

    # 同时打包 AppImage 和 deb
    npx electron-builder --linux AppImage deb --x64

**package.json 配置示例**

    "linux": {
      "target": [
        {
          "target": "AppImage",
          "arch": ["x64", "arm64"]
        },
        {
          "target": "deb",
          "arch": ["x64"]
        }
      ],
      "category": "Utility"
    }

### macOS 打包

**格式说明**

| 格式 | 说明 |
|------|------|
| dmg | 磁盘镜像，标准 macOS 分发格式 |
| pkg | 安装包格式 |
| mas | Mac App Store 格式 |
| zip | 压缩包格式 |

**打包命令**

    # dmg（Intel Mac）
    npx electron-builder --mac dmg --x64

    # dmg（Apple Silicon）
    npx electron-builder --mac dmg --arm64

    # dmg（通用版，同时支持 Intel 和 Apple Silicon）
    npx electron-builder --mac dmg --universal

**package.json 配置示例**

    "mac": {
      "target": [
        {
          "target": "dmg",
          "arch": ["x64", "arm64"]
        }
      ],
      "category": "public.app-category.utilities"
    }

### 跨平台打包注意事项

| 打包目标 | Windows 上打包 | macOS 上打包 | Linux 上打包 |
|----------|----------------|--------------|--------------|
| Windows | ✅ | ✅ | ✅（需要 Wine） |
| macOS | ❌ | ✅ | ❌ |
| Linux | ❌ | ✅ | ✅ |

建议使用 GitHub Actions 进行跨平台打包。

---

## GitHub Actions 自动构建

### 配置文件

位置：.github/workflows/build.yml

    name: Build Electron App

    on:
      push:
        branches: [main]
      workflow_dispatch:

    jobs:
      build-windows:
        runs-on: windows-latest
        steps:
          - name: Checkout code
            uses: actions/checkout@v4

          - name: Setup Node.js
            uses: actions/setup-node@v4
            with:
              node-version: '22'

          - name: Install dependencies
            run: npm install

          - name: Build Windows portable
            run: npm run build:win

          - name: Upload artifact
            uses: actions/upload-artifact@v4
            with:
              name: FullScreenClock-Windows-x64
              path: dist/*.exe

      build-linux:
        runs-on: ubuntu-latest
        steps:
          - name: Checkout code
            uses: actions/checkout@v4

          - name: Setup Node.js
            uses: actions/setup-node@v4
            with:
              node-version: '22'

          - name: Install dependencies
            run: npm install

          - name: Build Linux AppImage
            run: npm run build:linux

          - name: Upload artifact
            uses: actions/upload-artifact@v4
            with:
              name: FullScreenClock-Linux-x64
              path: |
                dist/*.AppImage
                dist/*.deb

      build-macos:
        runs-on: macos-latest
        steps:
          - name: Checkout code
            uses: actions/checkout@v4

          - name: Setup Node.js
            uses: actions/setup-node@v4
            with:
              node-version: '22'

          - name: Install dependencies
            run: npm install

          - name: Build macOS dmg
            run: npm run build:mac

          - name: Upload artifact
            uses: actions/upload-artifact@v4
            with:
              name: FullScreenClock-macOS
              path: dist/*.dmg

### 配置说明

**触发条件 (on)**

| 配置 | 说明 |
|------|------|
| push: branches: [main] | 推送到 main 分支时触发 |
| workflow_dispatch | 允许在 GitHub 页面手动触发 |

**运行环境 (runs-on)**

| 值 | 说明 |
|----|------|
| ubuntu-latest | 最新版 Ubuntu |
| windows-latest | 最新版 Windows Server |
| macos-latest | 最新版 macOS |

**常用 Actions**

| Action | 说明 |
|--------|------|
| actions/checkout@v4 | 检出仓库代码 |
| actions/setup-node@v4 | 安装 Node.js |
| actions/upload-artifact@v4 | 上传构建产物 |

### 下载构建产物

1. 打开仓库的 Actions 页面
2. 点击对应的 workflow run
3. 在页面底部的 Artifacts 部分下载

---

## 常见问题

**Windows 打包时下载很慢**

electron-builder 首次打包需要下载代码签名工具，可跳过签名：

    npx electron-builder --win --x64 -c.win.signAndEditExecutable=false

**Linux ARM 架构无法打包 Windows 版本**

ARM Linux 上的 Wine 无法运行 x64 Windows 程序，建议使用 GitHub Actions 自动打包。

**macOS 打包提示签名错误**

没有 Apple 开发者证书时，可跳过签名：

    npx electron-builder --mac -c.mac.identity=null

---

## License

ISC
ENDOFFILE
