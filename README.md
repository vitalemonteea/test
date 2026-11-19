# Parry Window Prototype — 无头测试套件

首个 Boss「起手重劈」的格挡判定原型，全部判定基于整数 physics tick，CI 全程 headless、确定性运行。

## 目录结构

```
combat/parry_window.gd          # 格挡判定纯函数（8 tick 视窗 + 4 tick 输入缓冲）
tests/test_parry_window.gd      # GUT 测试套件（边界全扫描 + 确定性守卫）
.github/workflows/test.yml      # CI：下载 Godot → 跑 GUT → 变异验证（必须变红）
project.godot                   # 最小项目配置（60Hz 物理步進）
Devin启动Prompt.md               # 喂给 Devin 的完整启动 Prompt
```

## 运行前提

需要先安装 GUT 插件到 `addons/gut/`（Godot Asset Library 搜 "GUT"，或从
https://github.com/bitwes/Gut 下载对应 Godot 4.x 的 release 解压进去）。
本仓库不含插件本体，Devin 的启动 Prompt 里已包含安装步骤。

## 本地运行

```
godot --headless --path . --fixed-fps 60 -s addons/gut/gut_cmdln.gd -gdir=res://tests -ginclude_subdirs -gexit
```

## CI 验收逻辑

1. 正常跑一遍：必须全绿。
2. 变异验证：把 `PARRY_WINDOW_TICKS` 从 8 改成 7 再跑一遍，**必须变红**
   （硬编码 tick 48 的命名用例会失败）。套件在突变下仍绿 = 拒收。
3. 自动还原常量，输出验证结论。
