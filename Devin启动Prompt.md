# Devin 启动 Prompt（无头测试套件）

以下英文部分可直接原样喂给 Devin：

---

Goal: Set up deterministic headless CI tests for a Godot 4 parry-window prototype (GDScript), so combat timing is verified without a GUI and without wall-clock flakiness.

Repo layout to create (or match if it already exists):
- combat/parry_window.gd — pure parry judgment, integer physics ticks only (file provided; do not rewrite its logic)
- tests/test_parry_window.gd — GUT test suite (file provided)
- addons/gut/ — GUT addon (latest GUT release compatible with Godot 4.x)
- .github/workflows/test.yml — CI workflow

Tasks:
1. Install a Godot 4.x stable headless-capable binary in the CI environment (Linux). No display server, no Xvfb needed: use `--headless` (dummy display + audio drivers).
2. Add the GUT addon and place the two provided .gd files at the exact paths above. The test suite preloads `res://combat/parry_window.gd`, so paths must match.
3. Run the suite with this exact command:
   godot --headless --path . --fixed-fps 60 -s addons/gut/gut_cmdln.gd -gdir=res://tests -ginclude_subdirs -gexit
   `--fixed-fps 60` forces every engine iteration to advance exactly 1/60s of simulation time, locking frame pacing to physics stepping 1:1 regardless of CI machine speed.
4. Wire the same command into GitHub Actions on every push/PR. Import assets first if needed (`godot --headless --path . --import`).

Hard rules (do not violate):
- All combat timing lives in `_physics_process` and is counted in integer ticks via `Engine.get_physics_frames()`. Never use `_process` delta, `Time.get_ticks_msec()`, or `create_timer()` for judgments or test waits.
- Tests wait by counting `await get_tree().physics_frame`, never by sleeping.
- Do not raise `physics/common/max_physics_steps_per_frame` to "fix" failures. Under `--fixed-fps 60` the catch-up cap must never trigger; if it does, that is a real bug — fail loudly.
- The suite includes a determinism guard asserting exactly 60 ticks after 60 awaited physics frames. It must pass unmodified.

Acceptance criteria:
1. CI is green on a clean checkout.
2. Mutation check: temporarily change PARRY_WINDOW_TICKS from 8 to 7 in combat/parry_window.gd — the boundary sweep test MUST fail. Revert afterwards. A suite that stays green under this mutation is rejected.
3. Report the exact CI run log showing both the green run and the intentionally-failing mutation run.

---

## 使用说明（给你本人，不发给 Devin）

- 两个 .gd 文件随本 Prompt 一起交付，Devin 拿到即可开工。
- 本地跑法：装好 GUT 插件后，在项目根目录执行上面第 3 步那条命令。
- 验收第 2 条（变异测试）是防「假通过」的关键：故意改坏一个常量，测试必须变红，否则说明套件没真正覆盖判定逻辑。
