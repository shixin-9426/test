# 局域网传输 v1.4.2（布局优化 & Bug 修复版源码）

基于上传的 `局域网传输-v1.4.1.apk`（`com.toolbox.lantransfer`）反编译分析与完整重构，保留原有应用图标与核心架构（零 AndroidX 冗余依赖，仅依赖 `com.google.zxing:core:3.5.3`）。

## 一、已修复的 Bug
1. **默认保存目录提示显示原始 `%1$s` 占位符**：修复 `MainActivity.refreshSaveDir()` 中 `saveNoteText` 与 `saveNote` 重复绑定并在默认目录下无参调用 `getString(R.string.save_note)` 覆盖格式化文本的问题。
2. **文件导入不完整报错恒显示 `0/总大小`**：修复 `Storage.importUri()` 先删除文件再调用 `target.length()` 导致实写字节数恒为 0 的问题。
3. **自定义目录重名重试失效**：修复 `Storage.saveToTree()` 同毫秒连续两次调用按秒取时间戳的 `renameAttempt()` 导致第 2、3 次候选名完全相同的问题。
4. **安装 APK 回退分支闪退**：为 `MainActivity.openReceived()` 中 `ACTION_INSTALL_PACKAGE` 回退分支补充 `try-catch`。
5. **主线程轮询卡顿/ANR 风险**：将 `Net.localIp()`、`QRUtil.encode()` 和 `Storage.receivedEntries()` 移至后台单线程执行器异步执行。
6. **Wi-Fi 切换后前台通知栏地址不更新**：新增 `LanService.updateNotificationIfNeeded()` 在 IP 变化时实时更新通知。
7. **HTTP 传输与安全修复**：
   - `HttpServer.serveDownload()` 启用 `asciiName()` 修复中文文件名下载乱码风险，并新增 **HTTP Range (206 Partial Content)** 断点续传支持；
   - 修复 `handleUpload()` 解析 `boundary=` 区分大小写及逐字节排空流的性能问题；
   - 修复 `ReceivedProvider` 缺少规范路径校验（`getCanonicalPath()`）的路径穿越风险。
8. **Web 端未选文件点上传导致按钮永久卡死**：补充空文件校验，并升级为支持实时进度条、上传速率、取消上传和拖拽上传的 XHR 上传方案。

## 二、布局与交互优化
- **Android 端**：安全区适配（`fitsSystemWindows` + 浅色状态栏）、移除易误触返回箭头、合并服务状态/地址/可折叠二维码为主控卡、共享/已接收改为分段 Tab 切换、补全手机端已接收文件删除与清空操作、所有按钮增加 `ripple` 点击反馈、增大删除按钮触控热区、去除列表末行多余分割线。
- **Web 端**：桌面端自适应双栏网格布局、拖拽上传区与已选文件管理、实时进度条与速率、已接收文件支持网页端直接下载、5 秒自动无刷新同步列表。

## 三、编译打包方法
1. 使用 **Android Studio** 打开本目录，等待 Gradle 同步完成；
2. 点击 **Build -> Build Bundle(s) / APK(s) -> Build APK(s)**，或在终端执行：
   ```bash
   ./gradlew assembleRelease
   ```
