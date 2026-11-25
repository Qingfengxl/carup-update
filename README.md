# CarUp 更新源（carup-update）

给车机上的 CarUp 用的在线更新源：App 请求 `<更新源>/manifest.json`，按里面的 `apk` 文件名下载同目录的 APK，校验包名、`sha256`、`size`、证书和 `versionCode` 后安装。

已预置进 APK 的更新源：

```
https://raw.githubusercontent.com/Qingfengxl/carup-update/main/
```

## 仓库结构

```
carup-update/
├── manifest.json          # 更新清单
├── CarUp-4.0.1.apk        # 正在分发的包（文件名必须和 manifest 里的 apk 一致）
└── ...                    # 以后新增的包
```

`manifest.json` 格式（字段全部必填，`notes` 可为空）：

```json
{
  "versionCode": 92,
  "versionName": "5.5",
  "apk": "CarUp-5.5.apk",
  "sha256": "64 位小写十六进制",
  "size": 6111663,
  "notes": "更新说明"
}
```

## 发新版本的步骤

1. 用 `carup-update.jks` 重签新包（必须是同一个密钥，否则车机上签名校验不过）：
   - Windows：`tools\sign_apk.bat CarUp-5.5.apk`
   - Linux/macOS：`BUILD_TOOLS=/path/to/build-tools/35.0.0 tools/sign_apk.sh CarUp-5.5.apk CarUp-5.5-signed.apk`（先 zipalign 再签）
2. 生成清单（会自己算 sha256/size，并读出 versionCode/versionName）：
   ```
   python tools/make_manifest.py CarUp-5.5.apk --notes "5.5：音频链路修复"
   ```
   想改仓库里的文件名就加 `--apk-name CarUp-5.5.apk`。
3. 把**签名后的 APK** 和新的 `manifest.json` 一起传到仓库根目录（`main` 分支）。两个文件必须同时更新。
4. 车机上打开 CarUp → 设置里的「在线更新 / 检查更新」，会提示有新版本并直接下载安装。

## 硬性约束（App 内部会校验，不满足就拒绝安装）

- 包名必须是 `com.carup.car`，证书必须和车机上已装的一致（即都用 `carup-update.jks` 签）
- `versionCode` 必须**大于**车机上已装的版本
- `sha256` 和 `size` 必须是最终上传的那份 APK 的值；签名/覆盖上传后要重新生成清单
- APK 文件名只能是字母、数字、`_`、`.`、`-`，且必须以 `.apk` 结尾（不能带路径、不能中文）
- 仓库必须是 public；私有仓库的 raw 链接匿名读不到

## 注意事项

- **密钥别丢、别公开**：`carup-update.jks`（密码 `carup2026`，别名 `carup`）是这条升级链的根。它一旦泄露，别人可以签出能被你车机接受的包；它一旦丢失，这条通道里所有包都无法再互相升级，只能卸载重装。不要把它提交到公开仓库。
- 第一次装：修改版和官方包的签名不同，**必须先卸载官方 CarUp 再装 `CarUp-4.0.1-patched.apk`**，卸载会清掉原有设置。
- 想升级官方原版（比如官方 5.5）也可以：用上面的脚本重签一遍再发，App 只认签名和版本号，不看是谁编译的。
- GitHub raw 在国内网络偶尔不通。若车机提示"manifest fetch failed"或一直转圈，可以在 App 的「编辑更新源」里换成 jsDelivr 镜像（同样指向这个仓库）：
  ```
  https://cdn.jsdelivr.net/gh/Qingfengxl/carup-update@main/
  ```
  jsDelivr 有缓存（文件更新后最多十几分钟才生效），需要立刻生效可以用它的 purge 接口。
- GitHub raw 本身也有几分钟 CDN 缓存，刚传完清单立刻检查可能还是旧内容。

## 工具

- `tools/make_manifest.py` —— 从已签名 APK 生成 `manifest.json`（依赖 build-tools 里的 `aapt2`，可用 `--aapt2` 指定路径）
- `tools/sign_apk.sh` / `tools/sign_apk.bat` —— zipalign + apksigner 固定密钥签名
