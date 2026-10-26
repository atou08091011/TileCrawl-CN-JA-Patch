# Tile Crawl 中 / 日语言补丁（TileCrawlPatch）

[日本語版 README](README.ja.md)

为 Steam 游戏 **Tile Crawl Demo** 增加 **简体中文** 与 **日本語** 界面的非官方补丁。

- 在游戏内 `Settings → Language` 里直接出现 `中文` / `日本語` 两个选项，随时切换
- 已翻译 **9 张本地化表 / 589 条文本 × 2 语言**（无空条目、无回退原文）
- 内置 **Noto Sans SC / Noto Sans JP** 免费字体（SIL OFL 1.1），中日文各自使用自己的字形，互不串用
- 单文件安装器（约 22 MB），**自动检索游戏目录**，可只装一种语言，**一键卸载回退**，安装前自动备份

![主菜单中文界面](assets/screenshot_menu_zh.png)

---

## 目录

- [快速开始](#快速开始)
- [功能特性](#功能特性)
- [安装器用法](#安装器用法)
- [命令行用法](#命令行用法)
- [卸载与回退](#卸载与回退)
- [备份机制](#备份机制)
- [它改了游戏的哪些文件](#它改了游戏的哪些文件)
- [从源码构建](#从源码构建)
- [目录结构](#目录结构)
- [已知问题](#已知问题)
- [许可与第三方组件](#许可与第三方组件)

---

## 快速开始

1. 从 Releases 下载 `TileCrawlPatchInstaller.exe`
2. 双击运行（若被 SmartScreen 拦截，点「更多信息 → 仍要运行」）
3. 安装器会自动找到游戏目录 → 勾选 `简体中文` / `日本語` → 点「安装」
4. 启动游戏 → `Settings → Language` → 选择 `中文` 或 `日本語`

> 安装时请**先关闭游戏**。安装器不会删除游戏任何文件，只做追加/新增，并会在安装前备份被修改的文件。

## 功能特性

| 项目 | 说明 |
| --- | --- |
| 新增语言 | 简体中文（zh-Hans）、日本語（ja） |
| 翻译覆盖 | 9 张 String Table，589 条文本/语言（含菜单、设置、提示类文本） |
| 字体 | Noto Sans SC（简体中文）/ Noto Sans JP（日本語），运行时按字符集动态生成 TMP 字体图集 |
| 字形隔离 | 中日文共用同一批汉字码位，但字形设计不同；补丁在中/日文环境下**只挂载当前语言的字体**，杜绝「中文里出现日式字形」；其他语言环境下两个都挂载，保证语言列表里的「中文 / 日本語」正常显示 |
| 溢出修复 | 游戏的 `Ellipsis` 截断 + 固定字号会把较宽的中日文标签截断为 0 个字符（表现为**标签空白**）；补丁会让这类标签的字号可缩小并改为不截断 |
| 防语言重置 | 游戏在启动后会再跑一次 `LocalizationSettings` 初始化协程，把语言重置为系统默认的 `en`，吃掉玩家选的中文/日文（表现为「打开设置页语言自己跳」）；补丁会识别这种非玩家操作的重置，并在下一帧纠正回玩家的选择 |
| 安装器语言 | 简体中文 / 日本語 / English，界面文字与日志全部本地化，布局随文本长度自适应 |
| 可回退 | 一键卸载；每次安装自动备份；Steam「验证游戏文件完整性」也可还原 |

## 安装器用法

界面元素（自上而下）：

- **界面语言**：切换安装器自身的显示语言
- **游戏目录**：自动检索（扫描 Steam 安装目录下的所有库）或手动「浏览」
- **安装语言**：勾选 `简体中文` / `日本語`（可只勾选一种）
- **状态**：显示当前是否已安装、已安装版本、已安装语言，以及目录是否可写
- **日志**：安装 / 卸载全过程
- 按钮：**安装 / 更新** · **卸载** · **打开备份目录** · **重新检测**

安装器会自动检测已有安装；若已安装则先卸载旧版本再装新版本，并生成新的备份。

## 命令行用法

安装器同时支持无界面模式，便于批处理或排障：

```bat
TileCrawlPatchInstaller.exe --detect
TileCrawlPatchInstaller.exe --install  "<游戏目录>" zh-Hans,ja
TileCrawlPatchInstaller.exe --install  "<游戏目录>" ja
TileCrawlPatchInstaller.exe --uninstall "<游戏目录>"
```

输出同时打印到控制台并追加到同目录的 `installer_cli.log`。

## 卸载与回退

三种方式，任选其一：

1. 安装器 → **卸载**
2. 命令行 `--uninstall "<游戏目录>"`
3. Steam 右键游戏 → 属性 → 已安装文件 → **验证游戏文件的完整性**（会把被修改的 json 还原）

卸载会：删除补丁 DLL、移除 `ScriptingAssemblies.json` / `RuntimeInitializeOnLoads.json` 中的补丁条目、删除 `LocalizationPatch` 数据目录。备份目录默认保留。

## 备份机制

每次安装（含更新前的自动卸载）都会在

```
<游戏目录>\tile-crawl_Data\_tilecrawl_patch_backup\backup_YYYYMMDD-HHMMSS\
```

生成一份备份，内含被修改的 `ScriptingAssemblies.json`、`RuntimeInitializeOnLoads.json` 以及 `record.json`（记录时间、游戏目录、补丁 DLL / 数据目录安装前的存在状态）。

安装器的「打开备份目录」按钮可直接打开该目录；需要强制还原时可把备份里的 json 手动拷回 `tile-crawl_Data\`。

## 它改了游戏的哪些文件

| 文件 | 动作 |
| --- | --- |
| `tile-crawl_Data\Managed\TileCrawlLocPatch.dll` | 新增（补丁本体，约 26 KB） |
| `tile-crawl_Data\ScriptingAssemblies.json` | 追加一条程序集名（供 Mono 加载） |
| `tile-crawl_Data\RuntimeInitializeOnLoads.json` | 追加一条 `RuntimeInitializeOnLoad` 钩子（游戏启动时自动运行补丁） |
| `tile-crawl_Data\LocalizationPatch\` | 新增数据目录（译文 json、字符集 txt、字体 otf、manifest.json、patch.log） |

补丁**不修改**游戏原有的任何资源包或程序集，只是「加挂」一个程序集。游戏更新后若这些 json 被覆盖，重新运行一次安装器即可。

## 从源码构建

前置条件：

- .NET SDK（netstandard2.1）
- Python 3.8+（安装器 UI 与数据生成）
- PyInstaller（`pip install pyinstaller`）

```bash
# 1) 编译补丁 DLL（默认指向 Steam 默认安装路径，可用 -p:GameManagedDir 覆盖）
cd plugin
dotnet build -c Release -o out --no-restore -p:GameManagedDir="D:\...\tile-crawl_Data\Managed"

# 2) 重新生成译文与字符集（可选；data/ 已包含生成结果）
cd .. && python src/gen_data.py

# 3) 组装 payload/（字体缺失时自动从 Google Fonts 仓库下载）
python src/build.py

# 4) 打包成单文件 exe
cd installer
pyinstaller TileCrawlPatchInstaller.spec --noconfirm --distpath ../dist --workpath ../build/pyinstaller
```

产物：`dist\TileCrawlPatchInstaller.exe`

## 目录结构

```
TileCrawlPatch/
├── VERSION                 补丁版本号
├── assets/                 README 用截图
├── data/                   生成产物：译文 json、字符集 txt
│   ├── shared.json             表结构元数据
│   ├── strings_zh-Hans.json   简体中文译文（589 条）
│   ├── strings_ja.json         日本語译文（589 条）
│   └── chars_*.txt             需要预热的字符集
├── installer/
│   ├── app.py              安装器 UI（tkinter）
│   ├── i18n.json           安装器界面文案（中/日/英）
│   └── TileCrawlPatchInstaller.spec
├── plugin/
│   ├── PatchEntry.cs       补丁本体（Unity 运行时注入）
│   └── TileCrawlLocPatch.csproj
├── src/
│   ├── patchlib.py         安装/卸载/备份/检索核心逻辑
│   ├── build.py            组装 payload/
│   ├── gen_data.py         译文与字符集生成
│   └── gen_shared.py / catbin.py   从游戏资源提取原始文本的辅助脚本
└── payload/                构建中间产物（安装器要拷贝的文件集合）
```

## 已知问题

- **汉字统一（Han Unification）**：中日共用同一批码位，「画」「面」等字在 Noto Sans SC 与 JP 中字形完全相同，这属于汉字本身的共用字形，不是字体串用；真正有差异的字（如「语 / 频 / 门 / 设」）在中日文环境下会分别使用各自字体。
- 乌克兰语等语言的本地名中含西里尔字母 `ї`（U+0457），Noto Sans SC/JP 不含该字形，预热日志会报 `missing=1`，对中日文显示无影响。

## 许可与第三方组件

- 本补丁的**代码与脚本**：MIT License，见 [LICENSE](LICENSE)
- **Noto Sans SC / Noto Sans JP**：SIL Open Font License 1.1，© Google LLC，见 [THIRD-PARTY-NOTICES.md](THIRD-PARTY-NOTICES.md)
- 本补丁为**非官方**粉丝作品，与游戏开发者无关；不含任何游戏原始资源，请自行购买/下载游戏本体。
