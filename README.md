# mybot-bk7258

BK7258 SMP 平台上的 mybot 语音助手集成工程。顶层仓库通过 Git submodule
固定 SDK 与解决方案代码，避免复制源码或把本地构建产物带入版本库。

## 仓库结构

| 路径 | 说明 | 跟踪分支 |
| --- | --- | --- |
| `bk_avdk_smp/` | BK7258 SMP SDK、Agora IoT SDK、AOSL 和平台基础组件 | `release/v3.1.1-mybot` |
| `bk_solution_ai/` | mybot SDK、BK725x 平台适配和 `projects/mybot` 双核工程 | `release/v3.1.1-mybot` |

顶层仓库中的 gitlink 固定两个子模块的准确提交；`.gitmodules` 中的 `branch`
用于执行 `git submodule update --remote` 时选择更新来源。

## 获取代码

首次克隆时同时初始化子模块：

```bash
git clone --recurse-submodules git@github.com:junlon2006/mybot-bk7258.git
cd mybot-bk7258
```

已经克隆顶层仓库但尚未获取子模块时执行：

```bash
git submodule update --init --recursive
```

## 编译 mybot 固件

工程当前支持 BK7258。`bk_avdk_smp` 与 `bk_solution_ai` 必须保持同级目录，
并显式将 `SDK_DIR` 指向前者：

```bash
make -C bk_solution_ai/projects/mybot clean SDK_DIR="$PWD/bk_avdk_smp"
make -C bk_solution_ai/projects/mybot bk7258 SDK_DIR="$PWD/bk_avdk_smp"
```

主要输出位于 `bk_solution_ai/projects/mybot/build/bk7258/mybot/package/`：

- `all-app.bin`：包含 bootloader、CP 和 AP 的完整烧录固件。
- `app_pack.rbl`：包含 CP 和 AP 的 OTA 包。
- `build_summary.txt`：各核 Flash、SRAM、ITCM 和 DTCM 使用情况。

## 内嵌语音资源

`bk_solution_ai/projects/mybot/assets/locales/` 下的 Opus-in-Ogg 文件通过 C 数组
编入 AP 固件，不使用独立的 `assets_data` 分区。修改资源后需要先手动重新生成数组：

```bash
python3 bk_solution_ai/projects/mybot/scripts/generate_assets_c.py \
  bk_solution_ai/projects/mybot/assets \
  bk_solution_ai/components/mybot/platforms/bk725x/modules/storage/mybot_assets.c
```

生成器只收集 `locales/**/*.ogg`；资源目录中的许可证和 README 不会进入固件。

## 更新子模块

拉取两个 mybot 分支的最新提交：

```bash
git submodule update --remote --merge
git status --short
```

确认子模块变更后，需要在顶层仓库提交新的 gitlink，其他开发者才能获得同一版本。
不要提交子模块内的 `build/`、`__pycache__/` 或其他本地生成文件。
