# FRP 定制版说明

## 这个项目是做什么的

`Codex Desktop Remote FRP` 是一个面向 Windows Codex Desktop 的 Android 远程控制客户端。

它让你可以在手机上：

- 查看 Codex Desktop 的任务列表和实时执行状态
- 查看回答、命令、文件修改、Diff 和使用量
- 处理命令、文件和权限审批
- 输入后续指令、排队消息、停止当前任务
- 创建新任务、切换模型和推理强度
- 在手机上查看运行进度和完成提醒

电脑上的 Codex、项目文件、Git 仓库和登录凭据不会被上传到第三方托管 IDE。手机通过 Bridge 控制电脑上的 Codex Desktop。

> 本项目不是 OpenAI 官方产品，与 OpenAI 没有关联或背书。

## 本定制版与上游的区别

上游项目：

<https://github.com/melompeel/codex-desktop-remote>

本仓库基于上游 `9446653` 版本修改，主要增加：

1. 仅放行一个指定的公网 FRP HTTPS 地址：

   ```text
   https://frp-sea.com:29641
   ```

2. 该公网地址必须使用 HTTPS，普通 HTTP 会被客户端拒绝。

3. 内置 `frp-sea.com` 当前 SakuraFrp 自签名证书，只对该域名作为专用信任锚。

4. Debug APK 使用独立包名：

   ```text
   com.alphapi.codexremote.frp
   ```

   因此可以与上游原版 APK 同时安装，不会因为签名不同而覆盖失败。

5. 应用名改为：

   ```text
   Codex Desktop Remote FRP
   ```

除上述改动外，功能逻辑保持上游版本一致。

## 下载 APK

前往 Releases：

<https://github.com/917349559/codex-desktop-remote-frp/releases/latest>

APK 不提交到 Git 源码历史，而是作为 Release 资产发布。

当前本地构建产物：

```text
release/frp/CodexRemote-Android-v0.6.1-frp-debug.apk
```

校验文件：

```text
release/frp/SHA256SUMS.txt
```

当前 APK SHA-256：

```text
26B393ABFA19B733EA7735A8757B5115454FF4CC4D9D711DC3387DF27249D85C
```

## 手机端连接

1. 在 Windows 上启动上游的 `CodexRemoteManager.exe`，或者使用仓库中的 Bridge 脚本。

2. 确认 Bridge 监听 `8765`。

3. 在 Android App 中填写：

   ```text
   https://frp-sea.com:29641
   ```

4. 输入 Bridge 当前显示的六位临时配对码。

5. 配对完成后，手机令牌会使用 Android Keystore 加密保存。

配对码有效期为 10 分钟。配对码过期后，需要重新生成，不需要重新配置域名。

## FRP 与 TLS 说明

当前公网端点使用 SakuraFrp 自动 TLS 生成的自签名证书：

```text
Subject: SakuraFrp Automatic TLS sn.686763065
SAN: frp-sea.com
SHA-256:
7D:DF:8D:61:10:EE:A2:2C:06:29:A2:6E:75:D2:26:B1:B8:AD:CC:F3:B8:EE:7A:07:68:8E:8B:2B:D6:BC:6A:48
Valid until: 2027-10-16
```

APK 通过 Android Network Security Config 只对：

```text
frp-sea.com
```

额外信任这个证书，不会关闭全局 TLS 校验，也不会信任其他任意自签名网站。

如果 SakuraFrp 更换证书、重建隧道或签发新的证书，Android 会再次报告：

```text
CertPathValidatorException: Trust anchor for certification path not found
```

此时需要重新提取证书、更新：

```text
android/app/src/main/res/raw/frp_sea_cert.pem
```

然后重新编译 APK。

## 公网穿透安全边界

FRP 地址位于公网时，Bridge 会成为可被公网尝试访问的控制接口。请至少保持：

- 使用 HTTPS/WSS，不使用公网 HTTP
- 不公开配对码
- 配对后及时使用独立的设备令牌
- 不把 FRP 管理密钥提交到仓库
- 不使用弱口令或固定六位密码
- 仅在个人可信设备上安装 APK
- 定期在 Bridge 中撤销不再使用的设备
- 发现异常流量时立即关闭 FRP 隧道

本项目只放行一个固定端点：

```text
frp-sea.com:29641
```

修改域名或端口后，需要同步修改 `BridgeEndpoint.kt` 并重新编译。

## 从源码构建

### 环境

- JDK 21
- Android SDK 35
- Android Build Tools 34 或更高
- Windows 11

项目路径如果含中文或空格，请按项目 `AGENTS.md` 的要求使用 `subst`：

```powershell
$repo = (Resolve-Path .).Path
subst R: $repo
try {
    $env:JAVA_HOME = "C:\path\to\jdk-21"
    $env:ANDROID_HOME = "$env:LOCALAPPDATA\Android\Sdk"
    Set-Location R:\android
    .\gradlew.bat --no-daemon clean testDebugUnitTest assembleDebug lintDebug
} finally {
    Set-Location $repo
    subst R: /D
}
```

生成的 APK：

```text
android/app/build/outputs/apk/debug/app-debug.apk
```

### 本地验证

已验证：

```text
testDebugUnitTest: 93 tests, 0 failures
assembleDebug: success
lintDebug: success
```

当前没有在真实 Android 手机上完成本版本的安装验证。

## 上游与许可证

本仓库保留上游 MIT License。上游代码和界面版权归原作者所有。

- 上游项目：<https://github.com/melompeel/codex-desktop-remote>
- 上游基准提交：`9446653`
- 本仓库：<https://github.com/917349559/codex-desktop-remote-frp>

使用前请自行确认 FRP、证书、Codex Desktop 版本和手机端安全设置。
