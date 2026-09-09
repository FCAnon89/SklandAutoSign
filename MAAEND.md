# MaaEnd 联动安装说明

SklandAutoSign 可以作为 MaaEnd 的非官方外部程序，在用户每次开始 MaaEnd 任务时静默执行森空岛签到。

本联动只使用 MaaEnd 客户端自带的“自定义程序”任务，不修改 MaaEnd、MaaFramework、Pipeline 或资源文件，也不向 MaaEnd 项目注入代码。SklandAutoSign 与 MaaEnd 及其维护者无隶属、合作或担保关系。

## 工作方式

MaaEnd 启动任务列表中的“自定义程序”时，会直接运行：

```text
SklandAutoSign.exe --maaend
```

`--maaend` 是专门供 MaaEnd 调用的单次静默签到入口。它会读取 EXE 旁 `data/settings.json` 中由当前 Windows 用户 DPAPI 加密的配置，立即执行一次签到并把结果写入 `data/sign.log`，不会弹出配置窗口，也不会创建、查询或触发 Windows 计划任务。

原有 Windows 每日任务仍使用 `--run`，功能保持不变。`--maaend` 与它只是复用相同的签到逻辑，触发机制彼此独立。

## 安装前提

- Windows 10 或 Windows 11 x64。
- MaaEnd 客户端中可以添加“自定义程序”任务。
- MaaEnd 与首次配置 SklandAutoSign 时使用同一个 Windows 用户运行，否则 DPAPI 无法解密 Token。
- 网络可以正常访问森空岛和鹰角账号服务。

## 第一步：解压并配置 SklandAutoSign

1. 下载 `SklandAutoSign-MaaEnd-Integration-win-x64.zip`。
2. 将压缩包完整解压到稳定且有写入权限的独立目录，例如：

   ```text
   D:\Tools\SklandAutoSign
   ```

   不要放在压缩包、浏览器临时目录或 MaaEnd 安装目录中。独立放置可避免 MaaEnd 更新时误删本工具。

3. 双击 `SklandAutoSign.exe`。
4. 按主程序说明获取并粘贴森空岛 Token，选择需要签到的游戏。
5. 点击“立即测试签到”，确认结果正常。
6. MaaEnd 联动不需要 Windows 计划任务，因此不要点击“启用每日任务”。只有脱离 MaaEnd 仍需要每天定时签到时，才单独启用原有每日任务。

Token 不应填写到 MaaEnd 的附加参数中。MaaEnd 只需要传入固定参数 `--maaend`。

## 第二步：在 MaaEnd 添加自定义程序

1. 打开 MaaEnd，在任务列表底部点击“添加任务”。
2. 选择“自定义程序”。
3. 填写以下设置：

   | 设置 | 值 |
   | --- | --- |
   | 程序路径 | 解压后的 `SklandAutoSign.exe` 完整路径 |
   | 附加参数 | `--maaend` |
   | 等待退出 | 开启 |
   | 已运行时跳过 | 开启 |
   | 通过 cmd 启动 | 关闭 |

4. 将这个“自定义程序”任务拖到 MaaEnd 任务列表最前面并勾选启用。
5. 保存或继续使用当前 MaaEnd 配置。

“等待退出”可让本次签到结束后再执行后续游戏任务；“已运行时跳过”可避免用户重复点击开始时产生重复进程；本程序不需要通过 `cmd /c` 启动。整个 MaaEnd 联动过程不会使用 Windows 计划任务。

## 第三步：验证联动

1. 在 MaaEnd 点击“开始任务”。
2. 等待“自定义程序”执行结束并继续后续任务。
3. 打开以下文件确认有本次记录：

   ```text
   <SklandAutoSign 解压目录>\data\sign.log
   ```

“今日已经签到”属于成功结果。如果日志提示 Token 过期，请重新双击 EXE 更新 Token，并再次执行“立即测试签到”。

## 更新

1. 关闭正在运行的 SklandAutoSign。
2. 保留原目录中的 `data` 文件夹。
3. 用新版 `SklandAutoSign.exe` 覆盖旧文件。
4. 如果路径和文件名没有改变，MaaEnd 自定义程序和 Windows 每日任务通常无需重新配置。
5. 运行一次“立即测试签到”确认接口仍然可用。

## 卸载

1. 在 MaaEnd 中删除或停用对应的“自定义程序”任务。
2. 如果曾启用 Windows 每日任务，先在 SklandAutoSign 中点击“停用每日任务”。
3. 关闭程序后删除整个 SklandAutoSign 目录。

`data/settings.json` 含有当前 Windows 用户加密后的敏感凭证。卸载时如不再使用，应一并删除；不要把 `data` 目录上传或发送给他人。

## 故障排查

### MaaEnd 启动后没有签到日志

- 确认自定义程序任务已勾选，并位于任务列表前部。
- 确认程序路径指向实际存在的 `SklandAutoSign.exe`。
- 确认附加参数只有 `--maaend`，不含 Token、引号或其他内容。
- 手动双击 EXE 并执行“立即测试签到”，排除 Token 或网络问题。

### 日志提示无法解密或配置不可用

确认 MaaEnd 和 SklandAutoSign GUI 由同一个 Windows 用户运行。配置使用当前用户范围的 Windows DPAPI 加密，复制到另一台电脑或另一个 Windows 账号后不能直接解密。

### 同一天执行了两次

如果同时启用了 Windows 每日任务和 MaaEnd 联动，它们是两个独立触发入口。通常第二次会返回“今日已经签到”。如不需要双重保障，可以停用其中一个入口；保留 MaaEnd 的“已运行时跳过”设置。
