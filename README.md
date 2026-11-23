# 宿舍小屋

安卓宿舍协作 App：四只可换形象的小猫、选床位、日常状态、休息计划、今日纸条公告板和轻提醒。

## 当前可安装版本

0.3.0 云端测试版，Android 8.0 及以上。安装包内包含界面和插画；宿舍共享数据通过 HTTPS 访问腾讯云上海 CloudBase PostgreSQL。电脑可以关机，手机不必位于同一 Wi-Fi。

云端配置完成后的版本覆盖旧版热点包时会清除旧身份，原本地数据库保留。请重新创建云端宿舍，分享新的六位邀请码；舍友加入时选择空床位和小猫。后续身份保存在本机，卸载应用会丢失身份。不要将旧的“六位码@IP:18765”用于云端版。

## 提醒与免费额度

界面每 15 秒检查状态，后台每 60 秒检查新休息事件和纸条。只有醒着的人收到通知；同一事件不重复提醒。通知通道无声音，振动约 70ms，实际以手机通知设置为准。请允许通知、后台运行并避免强制停止应用。当前是后台轮询通知，尚未接入厂商远程推送。Android 15 等系统会限制 dataSync 前台服务的累计运行时间，达到上限会暂停后台同步并提示重新打开；省电策略、断网也可能延迟或中断提醒。

使用者需自行创建 CloudBase PostgreSQL 环境。免费额度、续期方式和功能以腾讯云控制台当前规则为准。仓库不提供公共宿舍服务，也不包含作者的线上连接配置。

## 代码位置

- `preview/index.html`：安卓 WebView 使用的界面、交互及分层被子渲染。
- `app/assets/`：房间、猫和状态插画素材。`room-neutral.png` 已清理固定床单花纹，`sleep-bare.png` 为被子下的猫身体素材；最终睡眠状态仍会显示动态被子。
- `android-native/`：实际打包的 Java 安卓项目；原生启动入口、HTTPS 客户端、后台服务、通知及振动。
- `cloudbase/schema.sql`：云端原子接口，私有 schema 保存房间，只有经过成员身份校验的函数公开调用；数据库管理员密码不会进入 APK。
- `cloudbase/test.mjs`：真实 PostgreSQL 引擎 PGlite 测试，运行 `cd cloudbase && npm install && npm test`。
- `server/`：本机 Python 网页预览与旧 LAN 逻辑，运行 `python3 -m unittest discover -s server -v`。
- `app/lib/main.dart`、`tools/`：早期 Flutter 原型，当前 APK 不使用这些文件。

将 `android-native/assets/cloud-config.example.json` 复制为 `cloud-config.json`，填入自己的 HTTPS RPC 地址和官方 anon Publishable Key；后者不是管理员 API Key。真实配置已加入 `.gitignore`。成员身份 token 不写入源码，快照不返回其他成员 token。请勿将云服务管理员 API Key、个人数据库、用户纸条、签名 keystore 上传仓库。

## 构建 APK

需要 JDK 17、Android SDK Platform 35 和 Build Tools 35.0.1。设置 `JAVA_HOME`、`ANDROID_SDK_ROOT` 后运行：

```sh
DORM_BUILD_DIR=/absolute/path/to/build DORM_APK_OUT=/absolute/path/to/dorm-cottage.apk sh android-native/build.sh
```

脚本生成标准单一签名 APK，支持直接安装。首次构建会在构建目录创建测试签名密钥；后续使用同一个目录以保留签名。现有手机升级必须使用原密钥。不要清除本机 `work/native-build/dorm-debug.jks`，也不要把它提交 GitHub。CI 默认新建测试密钥，其产物不保证可覆盖本机签名的旧版。

## 验证状态

云端 HTTPS 实测通过：创建与两个身份加入、并发抢床位只有一个成功、状态和床单同步、休息事件去重、自己删除纸条、拒绝删除他人纸条、退出后身份失效。PGlite 验证私有表无法由 anon 直接读取。安卓 APK 编译、包版本与签名校验通过；校园网/移动流量之间的双手机安装验收仍需实机完成。床单视觉层已改造，最新效果仍需手机截图验收。

## 开源与贡献

代码使用 MIT 许可证，详情见 LICENSE。插画为 AI 生成素材，随本项目提供，不保证在所有地区具有相同的版权保护范围。欢迎通过 Issues 反馈问题，提交复现步骤、手机系统版本和去除个人信息的截图；也欢迎提交 Pull Request。

第一次接入请先运行 `cloudbase/schema.sql`，然后配置自己的 `cloud-config.json` 并构建。未提供该文件时会使用本地局域网原型；云端版本需完成上述配置。首次构建生成的签名只用于测试。请勿发布你或舍友的身份 token、纸条内容、私钥或数据库管理员凭据。
