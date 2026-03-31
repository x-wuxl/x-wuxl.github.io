---
layout: post
title: "Flutter 抓包终极指南：使用 reFlutter + TunProxy + Burp Suite 绕过 SSL Pinning"
date: 2026-03-31
last_modified_at: 2026-03-31
categories: [安全技术, 移动端渗透]
tags: [Flutter, Android, Burp Suite, reFlutter, SSL Pinning, 逆向分析]
keywords: "Flutter 抓包, reFlutter 教程, SSL Pinning 绕过, TunProxy 配置, Burp Suite 隐形代理, Flutter 逆向, Android 渗透测试"
description: "本文详解如何利用 reFlutter 静态补丁绕过 Flutter 自定义的 SSL Pinning 机制。通过 TunProxy 强制路由与 Burp Suite 隐形代理，配合根权限模拟器安装系统级证书，实现对 Flutter 应用 HTTPS 流量的深度截获与解密。"
author: "wuxl"
excerpt: "攻克 Flutter 流量分析难题：通过 reFlutter 补丁、TunProxy 转发及 Burp 隐形代理，全流程演示如何获取 Flutter App 的明文请求与响应，解决无法抓取 Flutter HTTPS 流量的痛点。"
toc: true
toc_sticky: true
---

Flutter 应用因使用自己的网络栈（Dart IO），普通系统代理通常无法拦截其 HTTPS 流量。本文将通过 reFlutter 静态逆向补丁技术，配合 TunProxy 虚拟网卡转发和 Burp Suite 深度解析，手把手教你构建一套完整的 Flutter 流量分析环境，彻底解决 Flutter App 抓不到包、无法解密 HTTPS 的痛点。

本次成功抓包的关键组合是：

**reFlutter**：对 APK 进行静态 patch，强制 Flutter Socket 流量重定向到指定代理端口（默认 8083），并辅助绕过部分 TLS 验证。

**TunProxy**：使用 Android VPNService 机制，进一步强制路由 App 流量到 Burp

**Burp Suite**：配置 invisible proxying 进行 MITM 解密。

**Android Studio Emulator**：root + writable-system

#### 准备工作

1. **工具准备**
   - reFlutter（最新版推荐通过 `pip install reflutter` 安装）
   - Burp Suite（Community 或 Professional 版）
   - Android Studio + AVD Emulator（推荐 API 30+，启用 root）
   - uber-apk-signer（用于 APK 签名，可以从 https://github.com/patrickfav/uber-apk-signer/releases 下载）
   - TunProxy APK（可以从 https://apkpure.com/tunproxy/com.tun.proxy 下载）
   - OpenSSL ：用于生成 Burp CA 证书 hash
   - ADB ：可以使用 adb 命令

2. **创建 Android 虚拟机**

   Android 版本选择11~14，比较稳定，兼容性好。一定要选择 Google APIs 版本的，不要选 Google Play 的镜像（Play 版通常无法轻松 root）
   

3. **Emulator 启动方式**（关键，可写系统）
   使用完整路径启动（Windows 示例）：
```powershell
   # 列出你所有的 Android 虚拟机名称
   & "C:\Users\<你的用户名>\AppData\Local\Android\Sdk\emulator\emulator.exe" -list-avds
   
   # 带 -writable-system 参数启动
   & "C:\Users\<你的用户名>\AppData\Local\Android\Sdk\emulator\emulator.exe" -avd 你的AVD名称 -writable-system
```

#### 详细操作步骤

**步骤 1：使用 reFlutter 处理 APK**
```powershell
# 安装 reflutter
pip install reflutter

# 处理 apk 文件
reflutter moke.apk
```
- 工具会自动识别 SnapshotHash 并输出 `release.RE.apk`。
- reFlutter 默认启用流量拦截模式（强制重定向到 Burp 8083 端口）。

**步骤 2：重签名 APK**

使用 uber-apk-signer 进行重签名
```powershell
# 对 apk 重签名
java -jar uber-apk-signer.jar --allowResign -a release.RE.apk
```
签名完成后得到可安装的 APK。将APK拖到安卓虚拟机进行安装，如果以前安装过则先卸载再进行安装。

**步骤 3：配置 Burp Suite Listener（8083 端口）**
- Proxy → Proxy Settings → Proxy Listeners → Add
  - Bind to port：**8083**
  - Bind to address：**All interfaces**
  - Request handling：**必须勾选 Support invisible proxying**
  - Running 一定要是勾选的状态。

**步骤 4：安装 Burp CA 证书到系统（System 标签页）**
1. Burp 导出 CA 证书 → 选择 `Certificate in DER format` → 选择导出目录 →  文件名填写为 `cacert.der`
2. 使用 OpenSSL 生成旧 hash 并重命名：
   ```bash
   openssl x509 -inform DER -in cacert.der -out cacert.pem
   openssl x509 -inform PEM -subject_hash_old -in cacert.pem | head -1
   # 这里会输出一个 hash 值，比如我的是：9a5ba575
   # 你将下面的命令替换成你输出的 hash 值
   # 后面必须加 .0
   cp cacert.pem 9a5ba575.0
   ```

3. 推送并安装：
   ```powershell
   # 这一段命令如果执行不成功
   # 后面的操作都没用！
   # 你的虚拟机必须是带 -writable-system 参数启动的才可以
   
   # 开启 root
   adb root
   # 执行 remount
   adb remount
   Disabling verity for /system
   Using overlayfs for /system
   Using overlayfs for /vendor
   Using overlayfs for /product
   Using overlayfs for /system_dlkm
   Using overlayfs for /system_ext
   Now reboot your device for settings to take effect
   
   # 如果你执行 remount 输出了以上内容，接着再执行下面的命令
   # 重启，等1~2分钟，等虚拟机完全启动好
   adb reboot
   adb root
   adb remount
   
   # 如果输出的是 remount succeeded
   # 接着下面的 adb push 执行就行了
   ```
   ```powershell
   # 先推送到 sdcard
   adb push 9a5ba575.0 /sdcard/
   adb shell
   mv /sdcard/9a5ba575.0 /system/etc/security/cacerts/
   
   # 设置权限
   chmod 644 /system/etc/security/cacerts/9a5ba575.0
   exit
   adb reboot
   ```
4. 重启虚拟机后，在 Settings → Security & privacy → More security settings → Encryption & credentials → Trusted credentials → System 标签页确认有 **PortSwigger** 证书，就表明安装成功

**步骤 5：安装并配置 TunProxy（关键成功步骤）**
- 安装 TunProxy APK（拖拽到虚拟机进行安装）。
- 打开 TunProxy，在输入框填写 `你电脑的IP:8083`
- 点击 Start，允许 VPN 连接。

**步骤 6：安装并运行目标 App**
- 卸载原版 App（避免冲突）
- 安装签名后的 reFlutter 处理过的 APK
- 启动 → 操作 App 进行网络请求

**步骤 7：验证与抓包**
- 在 Burp 的 **Proxy → HTTP history** 中查看请求信息（能看到业务接口的明文请求/响应）。

#### 常见问题与注意事项
- **remount 失败**：必须使用 `-writable-system` 启动 Emulator，并在 remount 后重启。
- **抓取的接口不全**：确认 Burp Listener 已勾选 Support invisible proxying；
- **Flutter 版本更新快**：若 SnapshotHash 不支持，需检查 reFlutter 的 enginehash.csv 或自行编译 patched engine。
- **仅用于合法安全研究**：如自家 App 测试或授权渗透。

#### 导出抓包记录进行分析
在 Burp **HTTP history** 中：
- 全选或多选记录 → 右键 → **Save items**
- 底部 **取消勾选** Base64-encode（保持明文），保存为 `名称.xml` 格式的文件，