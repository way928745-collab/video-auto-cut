# Edit a video-auto-cut project (OpenChatCut engine)

## Start with current state

1. Call `openchatcut_status` only after the OpenChatCut editor is running.
2. Call `list_projects` when the project is not already identified.
3. Call `target_project` with the selected project ID.
4. When specialized guidance applies, call `load_skill`; it requires no edit
   session or `editSessionId`.
5. Call `begin_edit_session` and keep the returned `editSessionId`.
6. Call `read_project` with that session ID before the first mutation.

Use `approvalMode: "manual"` by default. Use `"auto"` only when the user asks
for an unattended, atomic application.

## Load specialized guidance

Call `load_skill` after targeting the project and before specialized work. Common skill names include:

- `talking-head-guide`
- `transcription`
- `create-motion-graphics`
- `image-gen`
- `video-gen`
- `voice`
- `music`
- `shader-gen`
- `export`
- `verification`

Treat the live `load_skill` tool description as the current catalog. If a
loaded skill lists support files, request the relevant one with its `file`
argument.

## 制作前秒级分镜确认（必做）

1. 未明确确认前，不开始 B-roll 摆放、动效生成、字幕/音乐定稿或产线执行。
2. 先根据最终文案输出秒级制作分镜，至少包含：起止秒、画面类型（A-roll/B-roll）、镜头内容、对应文案。
3. 每个 A-roll 动效贴片必须写明：内容、位置、起止秒，并声明避开人脸与字幕区。
4. 只有用户明确回复确认后，才开始按该分镜制作；`approvalMode: manual` 不能替代这一步。（2026-10-21 安岩定稿）

## A-roll 定稿 → B-roll 后补（推荐流程）

1. 先只做 A-roll：用口播素材 + 最终文稿粗剪定稿，获得真实时间轴。口播时间线是 B-roll、动效、字幕、音乐的唯一时间基准；未定稿 A-roll 前，不按文稿估算去摆 B-roll。
2. A-roll 定稿后按需补 B-roll：不需要切开/重接 A-roll，同一时间线继续，B-roll 作为上层画面（全屏 cutaway 或 PiP 小窗）叠到指定目标窗口。
3. B-roll 时长由目标窗口决定，不由素材长度决定：
   - 素材比窗口长：先抽帧看素材，用 `sourceStart`/`sourceDuration`（或 `sourceStartMs`/`sourceEndMs`/`sourceStartSeconds`/`sourceEndSeconds`）选有效源区间，不默认从头或从尾截。
   - 一个长素材多处可用：同一素材可在不同目标位置多次放置，每次引用不同源区间。
   - 素材比窗口短：慢放（`speed`/`playbackRate` 0.1–8）、循环/重复、定格，或换素材，不硬拉长。
   - B-roll 常见 1.5–4 秒 cut-cover 盖跳切/补充画面，不需铺满每个说话段；不要在开头/结尾 3 秒内切走；密集跳切用一次较长 cutaway 覆盖；默认不与 MG 重叠。
4. 补 B-roll 也走秒级分镜确认：目标起止秒、对应哪句文案、用哪条素材、取素材哪段有效区间、全屏还是小窗、避开人脸和字幕区；确认后放置，补完可停下确认再继续后续步骤。（2026-10-21 安岩定稿）

## Apply edits

- Use IDs returned by `read_project`, discovery tools, or prior receipts.
- Keep every editor call in the same edit session.
- Re-read the project after a failed mutation or when the timeline may be
  stale.
- Use `view_timeline_frames`, `inspect_color`, or the verification skill after
  visual edits.
- Keep generation and other immediate side effects outside a proposal unless
  the exposed tool explicitly supports the draft session.

## Review and finish

1. Call `review_edit_session` after all draft edits are ready.
2. For manual mode, tell the user the proposal is ready inside the OpenChatCut editor.
3. Poll `get_edit_session` when the client needs the final state.
4. Report completion only when the status is `applied`.
5. If the status is `rejected` or `discarded`, report that exact result.

Applied operations form one atomic undo step.
