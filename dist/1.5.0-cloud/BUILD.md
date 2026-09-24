# OpenSteamTool 1.5.0（含 `[cloud]` 云存档宿主）— 内部构建

**这不是上游 release。** 本目录是自编译产物，用于土豆 Steam 工具箱的内核（借库/lua 入库）。

## 与上游 1.4.8 release 的差别

| | 上游 1.4.8（2026-06-13 编译） | 本构建 |
|---|---|---|
| 云存档重定向宿主 `[cloud]` | ❌ 无（DLL 里搜不到任何 `cloud` 串） | ✅ 有 |
| 内核签名中转 `[remote] url_template` | ✅ 有 | ✅ 有 |

`[cloud]` = 在 Steam 进程内加载 `cloud_redirect.dll`（CloudRedirect，MIT），把 lua 入库游戏的
Steam 云存档 RPC 重定向到自备存储；正版拥有的游戏仍走 Valve 原生云。

## 来源与构建

- 源码：上游 `OpenSteam001/OpenSteamTool` main 分支快照（2026-08-30 拉取，源码树未做任何修改）
- 工具链：Visual Studio 2022 Community / MSVC 14.38.33130 + CMake 3.27.2 + Ninja
- 命令（源码必须放在**纯 ASCII 路径**下，否则 protoc 的中文路径会被 MSVC 按 GBK 咬坏）：

```bash
cmake -S src -B build -G "Visual Studio 17 2022" -A x64 -DOPENSTEAMTOOL_VERSION=1.5.0 \
      -DCMAKE_CXX_FLAGS="/DWIN32 /D_WINDOWS /W3 /GR /EHsc /utf-8"
cmake --build build --config Release --target OpenSteamTool dwmapi xinput1_4
```

> ⚠️ `/utf-8` 必须与 MSVC 默认 flags 一起写。只给 `/utf-8` 会覆盖掉默认里的 `/EHsc`，
> toml++ 会切到 noex 模式并报 `IPCLoader.cpp: error C2593 "operator =" 不明确`。

## 校验

```
<SHA256SUMS.txt 同目录>
```

## 安装

三个文件一起复制到 Steam 根目录（`C:\Program Files (x86)\Steam`），
安装前先退出 Steam（否则 DLL 被占用）。`opensteamtool.toml` 保持原样即可。

启用云存档重定向需要额外三样（本构建只提供宿主，不含这些）：

1. `<Steam>\cloud_redirect.dll`（CloudRedirect v2.6.5 官方成品）
2. `opensteamtool.toml` 增加 `[cloud]` 段：`enabled = true`（可选 `library = "cloud_redirect.dll"`）
3. `%APPDATA%\CloudRedirect\config.json` 指定存储 provider（`s3` / `folder` 等）与凭据

未配置 1/2 时 `[cloud]` 保持关闭，行为与 1.4.8 完全一致。
