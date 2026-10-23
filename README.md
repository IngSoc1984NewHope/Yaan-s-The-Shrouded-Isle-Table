# YaanS The Shrouded Isle Cheat Table v0.1 (2026-10-21)

**Status / 状态：Experimental / 实验性（静态检查完成，尚未在游戏进程内测试）**

This Cheat Engine `.CT` table was rebuilt from the supplied `MonoFull.TXT` class dump and the older `Shinkansen The Shrouded Isle V1000.CT`. It does **not** depend on the old GameManager.Update AOB signature.

这个 CE 表格依据所提供的 Mono 转储与 Shinkansen 旧表格重新制作，不再依赖旧版 `GameManager.Update` 的特征码扫描。**无法仅凭静态转储保证在你的游戏进程内可用。**

## Functions / 功能

| Entry / 勾选项 | Effect / 作用 |
|---|---|
| `[01] Initialize Mono Engine` | **Enable first.** Resolve game classes, discover fields, find the GameManager singleton, and start the update timer. / **首先勾选**：初始化 Mono 和数据定位。 |
| `[02] MAX Town Statistics = 100` | Hold Ignorance, Fervor, Discipline, Penitence, Obedience at **100**. / 将五项城镇统计值维持为 **100**。 |
| `[03] MAX All Family Approvals = 100` | Hold each active `FamilyData.approval` at **100**. / 将当前所有家族的好感度维持在 **100**。 |
| `[04] AUTO Reveal All Virtues + Vices` | Every few seconds, call the game's `TraitData.RevealTrait(true)` for unrevealed character traits. / 通过游戏原生方法自动完全揭示角色善行与罪孽，也能处理后续出现的人物。 |
| `[05] SHOW Character Roster` | Open a read-only character roster with family, person name, actual positive traits and negative traits, and reveal-state values. / 弹出人物名单窗口，列出家族、角色、善行与罪孽。 |
| `[06] Reveal NOW` | Trigger the full-reveal method once immediately. / 立即运行一次全部揭示。 |
| `[07] Diagnostic Snapshot` | Display current Mono addresses, stat values, family approval, character count, trait count and reveal state. / 显示诊断信息，以定位测试问题。 |

## Setup and recommended test order / 安装与测试顺序

1. **Back up the save game / 备份存档。** Changing memory values and trait knowledge can persist after saving. / 内存修改与善行罪孽的揭示状态有可能被保存。
2. Launch The Shrouded Isle and **load an active campaign/save**, rather than staying only on the title screen. / 启动游戏，进入实际存档。
3. Open Cheat Engine **7.x**, attach to the game process, then open `YaanSTheShroudedIsleTable_v0.1_21102026.CT`. / 使用 CE 7.x 附加游戏进程并打开本 CT 文件。
4. Enable **`[01]`**. / 先勾选 `[01]`。
5. Enable **`[07]`** once to check that the diagnostics show five stats, all current families, and a nonzero character count. / 先勾选 `[07]` 检查读取结果是否合理；验证后可以取消勾选。
6. Enable **`[02]`**, check the five town stats in game, and proceed. / 再勾选 `[02]`，检查五项统计值。
7. Enable **`[03]`**, check each family approval. / 再勾选 `[03]`，核实所有家族好感。
8. Enable **`[06]`** once, then reopen the in-game character/family panels. Optionally enable **`[04]`** for continuous reveal of new characters. / 勾选 `[06]` 一次揭示现有角色；重新进入人物界面观察。需要持续揭示新角色时，勾选 `[04]`。
9. Enable **`[05]`** to inspect every character's virtue/vice without relying on the game UI. For refreshed roster data, uncheck and check `[05]` again. / 勾选 `[05]` 查看角色名单；重新读取需要取消后再勾选。

**Important / 注意：** `[05]`, `[06]`, `[07]` are action entries. Uncheck and recheck an entry if you want to run the action again. `[02]`–`[04]` are continuously active switches. `[01]` should stay enabled while using other functions. / `[05]`、`[06]`、`[07]` 属于一次性动作；其他三个功能需保持勾选；整个测试期间保持 `[01]` 勾选。

## Implementation / 实现说明

The old table uses GameManager fields at `town=0x3C`, `peopleList=0xAC`, `familyDataList=0xB0`, whereas the supplied Mono dump shows `town=0x40`, `peopleList=0xB0`, and `familyDataList=0xB4`. This is why blindly fixing the old fixed-address/AOB table would not be sufficient.

旧表格与 Mono 转储存在偏移差异。新版表格直接使用 Mono API：`mono_findClass`, `mono_class_enumFields`, `mono_class_getStaticFieldAddress`, `mono_class_findMethod`, `mono_invoke_method`，并按当前进程位宽解析托管 `List<T>`。字段根据运行时反射计算，而非写死 dump 中的地址。

Town statistics use `GameManager.town -> TownData.rd -> ResourceData`. Family approval uses `GameManager.familyDataList -> FamilyData.approval`. Character traits use `GameManager.peopleList -> PersonData.traits -> TraitData`. The full reveal calls the **original** `TraitData.RevealTrait(true)` rather than guessing the numeric enum value of `TraitRevealState`.

## Known limitations / 已知限制

- **No in-game runtime validation yet / 尚未进行游戏内验证。** XML structure and Lua syntax were checked, but live Mono collection, object layout and effect must still be tested in the running game.
- A stat or family approval can momentarily change between timed updates (about 1.8 s). / 因为采用周期更新，变化可能短暂出现在两次更新之间。
- The roster is a **snapshot**, not a live-updating UI. / 名单是快照，不会自动实时更新。
- After enabling full reveal, close/reopen the in-game character panel if its contents do not immediately refresh. / 若人物面板没有立即更新，先退出再重新进入。
- The `[04]` periodic full reveal calls game code from Cheat Engine. If the game becomes unstable, disable `[04]` and test `[06]` independently. / 自动揭示通过 CE 调用原生游戏方法；如不稳定，先关闭 `[04]` 并单独使用 `[06]`。
- Disabling entries stops their future updates; **previous changes are not undone**. / 取消勾选不会恢复已经写入的数值或被揭示的资料。
- If the game is updated or its classes change significantly, the field scanner may refuse to initialize. / 游戏重大更新后可能需要再次适配。

## Troubleshooting / 诊断

- **Cannot enable `[01]`**: make sure the game process is attached, the game uses the Mono runtime, and the campaign is loaded. Check Cheat Engine's `View -> Show Cheat Table Lua Script` or `Memory View -> Tools -> Lua Engine` and the CE error dialog. / 检查所选进程、Mono 状态和 CE 错误消息。
- **`[07]` shows zero families or people**: the save may not have reached gameplay initialization; try after starting the campaign. / 可能尚未进入实际游戏阶段。
- **Stats/approval don't stay at 100**: confirm `[01]` is active and the diagnostics show valid data. / 核实 Mono 初始化和诊断结果。
- **`RevealTrait` raises an exception**: disable `[04]`; provide the exact error text and `[07]` diagnostic output, so we can adjust the method-invocation implementation. / 如果原生揭示方法报错，停止自动揭示并提供错误文本和诊断结果。

Offline single-player game modification only / 仅供离线单机使用。
