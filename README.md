# 海普天32位单片机固件烧录程序 · 界面优化版（源码工程）

这是 **界面优化版的源代码**，不是打包好的成品。基于 ST 官方 **DfuSe Demo V3.0.6** 的源码改造：
全中文界面、WebUI 风格配色、改名「海普天32位单片机固件烧录程序」、窗口尺寸对齐原版、底部加驱动排错链接。

本目录可以直接用 VS2022 打开、直接命令行编译（已实测从零编译 0 错误，产出全部 7 个文件）。

---

## 一、这是什么，和另外两个目录的关系

| 目录 | 内容 | 用途 |
|---|---|---|
| **本目录**（`海普天烧录程序_界面优化版_源码`） | 完整源码 + 资源 + 解决方案 | 改代码、重编、二次开发 |
| `海普天32位单片机固件烧录程序_win32` | 成品包（exe + ST 的 4 个 DLL + `Driver\` + README） | 直接双击运行 |
| `hp_drvsetup.exe`（另一个独立项目） | 单文件驱动安装工具，装 **WinUSB** 驱动 | 给 libusb/WinUSB 类程序用 |

源码在这、成品在那，两边别混。源码改完要重新编译才能变成成品（第六节）。

---

## 二、目录结构

```
海普天烧录程序_界面优化版_源码\
├─ DfuSe.sln                     解决方案（6 个工程，Release|Win32 为交付配置）
├─ DfuseDemo\                    ★ 主程序「烧录程序.exe」的全部源码
│   ├─ DfuSeDemo.cpp/.h          应用入口、CWinApp
│   ├─ DfuSeDemoDlg.cpp/.h       ★ 主窗口：全部界面逻辑 + 自绘主题都在这里
│   ├─ DfuSeDemo.rc              ★ 对话框模板、文案、版本信息（GBK 编码，见第五节）
│   ├─ DfuSeDemo.rc.en.bak       中文化之前的英文原版备份
│   ├─ DfuSeDemo.rc.scale.bak    窗口缩放之前的备份
│   ├─ DfuSeDemoDlg.cpp.en.bak / MappingDlg.cpp.en.bak   中文化前的英文备份
│   ├─ TextProgressCtrl.cpp/.h   ★ 胶囊形自绘进度条
│   ├─ MappingDlg.cpp/.h         映射（Memory Mapping）窗口
│   ├─ OptionBytesDialog*.cpp/.h 选项字节 / 读保护 / 写保护 三个对话框
│   ├─ Enumeration.cpp/.h        USB 设备枚举
│   ├─ HexEdit.cpp/.h            十六进制编辑控件
│   ├─ resource.h                资源 ID（含新增的 IDC_STATIC_LINKDRIVER = 1069）
│   ├─ Include\                  本工程私有头文件（HID 相关 SDK 头，hid.lib 配套）
│   ├─ lib\hid.lib               编译必需（HID 接口）
│   └─ RES\DfuSeDemo.ico/.rc2    图标与 rc2 资源
├─ DfuFileMgr\                   「DFU 文件管理器」源码（hex ↔ dfu 转换）
├─ DfuSeCommand\                 命令行版烧录工具源码
├─ STDFU\                        STDFU.dll 源码（DFU 协议层）
├─ STDFUFiles\                   STDFUFiles.dll 源码（.dfu/.hex 文件解析）
├─ STDFUPRT\                     STDFUPRT.dll 源码（DFU 传输层）
├─ STTubeDevice\                 STTubeDevice30.dll 源码（USB 设备枚举库）
├─ Include\                      全局共享头（SETUPAPI.H、STTubeDevice*.h、USB*.H 等）
├─ Tools\GUIDGEN.DLL             编译资源时用到的工具 DLL
├─ kill.bat                      清编译残留的小脚本
└─ .gitignore
```

> `DfuseDemo\Release`、`Binary` 这些是编译产物目录，源码包里已清空；编译后会自动重建。

---

## 三、编译

**环境**：
- Visual Studio 2022（实测 17.x），安装 **「使用 C++ 的桌面开发」** 工作负载
- 必须勾选 **MFC 组件（MFC 与 ATL）**，否则 `afxwin.h` 找不到
- Windows 10/11 SDK

**命令行编译（Release / Win32，即交付配置）**：
```
"D:\Program Files\Microsoft Visual Studio\2022\Community\MSBuild\Current\Bin\MSBuild.exe" ^
  "本目录\DfuSe.sln" /p:Configuration=Release /p:Platform=Win32 /m /v:m /nologo
```

**输出**（全部落在 `Binary\Release\`）：

| 文件 | 大小 | 说明 |
|---|---|---|
| `DfuSeDemo.exe` | 3.4 MB | 主程序（静态 MFC，不需要 mfc140/vcruntime），**打包时改名 `烧录程序.exe`** |
| `STDFU.dll` | 354 KB | |
| `STDFUFiles.dll` | 65 KB | |
| `STDFUPRT.dll` | 68 KB | |
| `STTubeDevice30.dll` | 3.2 MB | |
| `DfuFileMgr.exe` | 77 KB | 动态 MFC，需 VC++ 运行库 |
| `DfuSeCommand.exe` | 55 KB | 动态 MFC，需 VC++ 运行库 |

三种配置：`Release|Win32`（交付用）、`Debug|Win32`（调试）、`Release|x64`（能编，但 ST 的 DLL 只编 32 位，x64 版没有配套 DLL，别用）。

**两个必须知道的坑**：

1. **编译前先关掉正在跑的 exe**，否则链接报 `LNK1104 无法打开文件 ...exe`：
   ```
   taskkill /F /IM 烧录程序.exe
   ```
2. **本源码包已修正了一处原件遗留问题**：原工程里 `DfuFileMgr`、`DfuSeDemo`、`DfuSeCommand` 是靠
   `..\STDFU\release\STDFU.lib` 这种**上一次编译残留的导入库**来链接的，解决方案里也没写工程依赖。
   原仓库能编是因为那些 lib 早就躺在磁盘上；干净检出的源码第一次编译会报
   `LNK1181: 无法打开输入文件“..\STDFU\release\STDFU.lib”`。
   本包给这三个工程加了 `<ProjectReference>`（指向 STDFU / STDFUFiles / STDFUPRT，并设
   `LinkLibraryDependencies=false`，只保证编译顺序、不改变链接内容），所以**从零编译一次就过**，
   已实测 EXIT=0。

---

## 四、改造点在哪（要改界面就从这几个地方下手）

### 1）配色与自绘 —— `DfuseDemo\DfuSeDemoDlg.cpp`

主题色常量在 **58–75 行**，改颜色只改这里：

```cpp
const COLORREF TH_Bg        = RGB(0xED, 0xF1, 0xF7);  // 页面底色（浅灰蓝）
const COLORREF TH_Card      = RGB(0xFF, 0xFF, 0xFF);  // 卡片白
const COLORREF TH_Primary   = RGB(0x16, 0x77, 0xFF);  // 主色（按钮/进度）
const COLORREF TH_PrimaryH  = RGB(0x3F, 0x8F, 0xFF);  // 悬停
const COLORREF TH_PrimaryD  = RGB(0x0B, 0x5C, 0xD6);  // 按下
const COLORREF TH_Track     = RGB(0xE6, 0xEC, 0xF6);  // 进度条轨道
const COLORREF TH_Text      = RGB(0x30, 0x36, 0x3D);  // 主文字
const COLORREF TH_TextSub   = RGB(0x8A, 0x91, 0x9C);  // 次要文字
```

关键函数：

| 函数 | 作用 |
|---|---|
| `InitTheme()`（约 3046 行） | 统一给各控件换字体（Microsoft YaHei UI 8pt）、把静态文本/分组框设成透明 |
| `DrawCard(CDC&, rcBox, sTitle)`（约 3191 行） | 画白色圆角卡片 + 标题，替换原来的灰色 GROUPBOX 边框 |
| `DrawFlatButton(LPDRAWITEMSTRUCT)`（约 3224 行） | 按钮 owner-draw：圆角 4px、悬停/按下/禁用三态，主按钮蓝底白字 |
| `OnDrawItem()`（约 3291 行） | 分发自绘，把小按钮转给 `DrawFlatButton` |
| `OnEraseBkgnd` / `OnCtlColor`（约 2771 行起） | 页面底色、控件透明、链接文字蓝色 |
| `OnCustomDrawList()`（约 3311 行） | 目标列表隔行浅色 + 选中蓝底 |

`DfuseDemo\TextProgressCtrl.cpp` 的 `SetPageColour(COLORREF)`（约 284 行）专门用来**消除胶囊进度条
两端的黑角**（把圆角外的像素按页面底色填充）。改了页面底色一定要同步调用它，否则黑角又回来。

### 2）中文化
- **对话框模板与文案**：`DfuseDemo\DfuSeDemo.rc`（5 个对话框、静态文本、按钮标题）
- **列表表头、17 条提示框、26 条状态串**：`DfuseDemo\DfuSeDemoDlg.cpp`
- **映射窗口 7 个列标题**：`DfuseDemo\MappingDlg.cpp`
- 有意保留英文：寄存器名（RDP / WRP / nRST_STDBY / BOR_LEV）、DFU 术语、`DfuFileMgr` 与 `DfuSeCommand` 两个辅助程序

### 3）改名（标题 + 文件属性）
`DfuSeDemo.rc` 里两处：`CAPTION "海普天32位单片机固件烧录程序"`；`VERSIONINFO` 块的
`FileDescription` / `ProductName` 同步改成同一个名字（这样任务栏、文件属性、任务管理器都显示新名）。
`CompanyName` 仍保留 STMicroelectronics 未改。

### 4）窗口尺寸（587×581，与 ST 原版对齐）
`DfuSeDemo.rc` 主对话框模板：`375×331 DLU → 326×271 DLU`（宽 ×0.87、高 ×0.82），所有控件坐标等比重算。
**这是最容易出错的一步**：改完必须重新截图核对顶部三张卡片、"升级或校验操作"卡片、进度条、底栏有没有文字截断或重叠。
缩放前的原始模板在 `DfuSeDemo.rc.scale.bak`，还原直接覆盖回去。

### 5）底部驱动排错链接
- 控件：`DfuSeDemo.rc` 里一个 `LTEXT`，ID 为 `IDC_STATIC_LINKDRIVER`（`resource.h` 第 80 行定义为 1069）
- 交互：`DfuSeDemoDlg.cpp`
  - `ON_STN_CLICKED(IDC_STATIC_LINKDRIVER, OnLinkDriverClicked)`（约 198 行）
  - 悬停变色 + 下划线 + 手形光标：约 3017 / 3029 行
  - `OnLinkDriverClicked()`（约 3040 行）：`ShellExecute("https://blog.csdn.net/fword/article/details/136837591")`
- 链接文字是蓝色 `TH_Primary`，悬停转深蓝。

---

## 五、编码铁律（踩过坑，别改）

1. **`DfuSeDemo.rc` 必须存成 GBK（代码页 936）+ LF 换行**。
   文件头是 `#pragma code_page(936)`；若用 UTF-8 保存，VS 的资源编译器会直接报错或把中文变成乱码。
   编辑器统一设成 GBK 再存，别让工具自动转 UTF-8。
2. 含中文的 `.cpp/.h` 用默认 ANSI/GBK 保存（与 rc 保持一致最省事）。
3. `.vcxproj` / `.sln` 是 UTF-8，别动编码。
4. 改 `.rc` 前先备份一份，或者用已有的 `*.en.bak` / `*.scale.bak` 回滚。

---

## 六、从源码出成品

```
1) 按第三节编译 → Binary\Release\ 得到 7 个文件（+ pdb，pdb 不要发出去）
2) 新建成品目录，拷入：
     DfuSeDemo.exe → 改名为 烧录程序.exe
     STDFU.dll / STDFUFiles.dll / STDFUPRT.dll / STTubeDevice30.dll
     DfuFileMgr.exe / DfuSeCommand.exe（可选，需要 VC++ 运行库）
3) 拷入 ST 官方驱动目录 Driver\（STDFU.inf + x86/x64 STTub30.sys + version.txt）
4) 拷入 README.md
5) 实测：双击 烧录程序.exe，确认标题为「海普天32位单片机固件烧录程序」、窗口 587×581、界面中文且配色正常
```

---

## 七、已知限制 / 后续可做

- 只支持 ST 内置 DFU Bootloader（VID 0483 的 DF11/DF12/DF13/FF03），不支持其它厂商自定义 Bootloader；
- 映射窗口、三个选项字节对话框**没有套主题**（还是系统灰白），且需要连真实 DFU 设备才会弹出，尚未实机核对；
- `DfuFileMgr`、`DfuSeCommand` 仍是 ST 原版英文界面；
- 驱动目录缺 `stdfu.cat`，64 位 Win10/11 装 ST 驱动会报 `0xE0000247`，需从 ST 官方 DfuSe 包
  （STSW-STM32080）补该文件；
- 窗口刻意与 ST 原版同尺寸（587×581），不做放大；想再小可以继续压到约 310×255 DLU 等比，但要重新核对所有控件。
