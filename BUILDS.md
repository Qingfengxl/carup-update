# 已交付的安装包与更新源

| 文件 | versionCode | 更新源（已写进包） | 用途 |
| --- | --- | --- | --- |
| `CarUp-4.0.1-patched.apk` | 86 | `https://raw.githubusercontent.com/Qingfengxl/carup-update/main/` | 默认版，raw 直连 GitHub |
| `CarUp-4.0.1-cdn.apk` | 86 | `https://fastly.jsdelivr.net/gh/Qingfengxl/carup-update@main/` | raw 被墙时用，jsDelivr 镜像 |
| `CarUp-4.0.2-test.apk` | 87 | 同默认版（raw） | 链路自测，验证检测→下载→安装 |

两个 86 包版本号、签名一致，可以互相覆盖安装。

## 当前仓库状态

`manifest.json` 现在指向 **4.0.2-test（versionCode 87）**，所以在仓库根目录上传 `CarUp-4.0.2-test.apk` 后，车机上装 86 的包点“检查更新”就会走完整链路。

自测通过后，把官方 5.5 用 `carup-update.jks` 重签、生成新的 manifest（versionCode 92 > 87）覆盖本文件即可。

## 两个备用清单

- `manifest-cdn.json` —— 对应 4.0.1-cdn 的 sha256，若仓库里放的是 cdn 版的 `CarUp-4.0.1.apk` 就用它
- `manifest-test.json` —— 4.0.2-test 的清单原文
