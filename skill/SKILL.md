---
name: "changshengmen-video"
description: "Produce 《长生门》 chapter videos in the established house style: cinematic (not narrated), natively lip-synced dialogue from the video model, one consistent voice per character, pure native sync sound, no synthetic beds. Use when the user asks for a 长生门 chapter/episode video, or to continue the series in the same style."
---

# Changshengmen Video

## Purpose
Make every 《长生门》 chapter video feel like one continuous film: real cinema, not a narrated slideshow. Dialogue is generated **by the video model itself** (lips + voice together), each character keeps **exactly one voice**, and the soundtrack is **100% native sync sound** — no dubbing feel, no narration, no synthetic ambience.

## Workflow
1. **台词表**：从本章小说提炼对白（短句优先，单句不超过 ~12 秒）。旁白能删则删——用户已明确旁白多余。
2. **说话镜头**：每句对白用 `media.generate_video` 生成，中文台词**逐字写进提示词**，要求嘴型逐字同步、声音与画面同步、画面无文字。见 `references/dialogue.md`。
3. **验台词**：每条用 faster-whisper 中文转写（含词级时间戳）校验；念错/含糊的重新生成，直到通过。
4. **场景空镜**：用 `media.generate_video` 生成，提示词里写明同期声（如：脚步声、风声、溪水声），无对白、无文字。
5. **组装**（见 `references/build-pipeline.md`）：统一调色 → 0.7s 交叉叠化 → 原生音频时间轴 → 烧录 ASS 字幕 → 混音。
6. **验货**：抽帧检查字幕与画面；ffprobe 确认时长/分辨率/编码。
7. **分享版**：crf 27 压到 ~50MB，`remote-storage upload-file` 上传（需用户点审批卡，超时 10 分钟需重传）。

## Output Contract
- 1920×1080, H.264 + AAC, 16:9；单章约 2:30–3:00。
- 片头 4 秒封面卡：黑底金字，`长生门 / 第X卷·<卷名> / 第X章·<章名>`（见 visual-style.md）。
- 全片视觉：`references/visual-style.md`（水墨仙侠调色、转场、片头片尾卡）。
- 全片声音：视频模型原生同期声 + MiniMax（仅苍老之声等非人角色 / 工具拒绝生成的对白画外音）。**不叠加合成底噪、鼓点、配乐**，除非用户另行要求。
- 字幕：ASS 烧录，Noto Serif CJK SC 54px，见 `references/build-pipeline.md`。
- 产物：原版 + `-share.mp4`，放 `~/workspace/changshengmen-ch01-film/video/`（按章建目录）。

## Operating Rules
1. **口型是硬指标**：说话镜头必须由视频模型按台词一体生成（口型+声音），禁止"静止脸+后期配音"。
2. **一人一声**：每个角色全片只用一个声音。原生成功用原生；工具拒绝生成的对白才用 MiniMax 画外音 + 反应镜头，且同一角色不得混用两种声音——宁可剪掉非关键台词，不制造"一个人物两个声音"。
3. **被拒绝不重试**：`media.generate_video` 返回 `integrity_check_failed` 的请求，**不得重复**（包括换参数、换镜头描述、换工具）。如实告知用户这是硬限制。
4. **无旁白**：不用旁白讲故事，用画面和对白讲。苍老之声（长生门之声）是画外音角色，保留，但它不是旁白。
5. **非关键台词可剪**：为保"一人一声/全原生"，非关键剧情台词可直接剪掉；关键剧情台词只能用 MiniMax 画外音保留——二选一，不将就。
6. **我无法试听**：我听不见生成视频的音频。台词内容用 whisper 校验，但音质、口型自然度必须由用户亲自确认，开工前如实说明。
7. **不花钱**：不接新的付费 API、不做账号风险操作。MiniMax 走用户已授权的 `custom.minimax`（见 `references/voices.md`）。
8. **先验货再上传**：成片抽帧 + ffprobe 通过后才压分享版；上传审批卡超时就等用户说"传"再重传，不擅自反复触发。
9. **同一场景不重复**：同一个空镜/场景素材在全片只用一次（最多 2 次且必须隔得足够远）。长篇对话戏（画外音、独白）必须用多角度空镜 + 人物反应镜头交叉剪辑，**不许一个镜头循环用到底**。交付前自查全片有无重复镜头（2026-10-04 第二章教训）。
10. **人物长相全系列一致**：每个角色有固定长相描述（见 `references/characters.md`），凡该角色出镜的 prompt 必须原样引用整段描述。**不许**只写"少年/老汉"这种泛称。新角色首次出镜先定版存档。每章交付前抽查同一角色是否为同一张脸（2026-10-06 用户要求）。
