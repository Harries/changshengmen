# Build Pipeline（构建管线）

每章一个构建脚本：`~/workspace/changshengmen-ch01-film/work/build_v<N>.py`
（参考 v6：`work/build_v6.py`）。分四段：

## 1. 画面
- 每个镜头：调色（GRADE）→ 精确时长（trim / `-stream_loop` 循环补足）。
- 说话镜头用 whisper 验出的起止时间裁剪。
- 按分镜顺序 concat → `tpad=clone` 垫长 0.7s → xfade 链（fade 0.7s）。
- 输出 `v<N>_video_silent.mp4`；`ffprobe` 确认时长 = 分镜总时长。

## 2. 音频（全原生时间轴）
- 每个镜头的**原生同期声**按场景起点原样放置（`native_audio` 抽取，不做任何处理）。
- 说话镜头的原生对白：增益 ~1.6（对白比环境声实）。
- 循环补足的镜头，音频同样循环补足。
- MiniMax 只放：苍老之声 / 工具拒绝的对白画外音（轻混响贴场景）。
- **不加**合成风声/鼓点/钟声/音乐垫——用户要的是纯原生。
- 归一化：峰值 0.89 → `tanh(x*1.15)*0.92`；首 0.5s 淡入、尾 2.5s 淡出。
- 输出 stereo 44.1kHz WAV → AAC 192k `.m4a`。

## 3. 字幕
- ASS 烧录：`Noto Serif CJK SC, 54px`，白字 + 半透明黑底。
- 只给**对白**打字幕（无旁白）；时间轴与音频放置点对齐。
- Style 行参考 `work/build_v6.py` 的 subs 段。

## 4. 合成与交付
- `ffmpeg -i v<N>_video_silent.mp4 -i v<N>_mix.m4a -vf subtitles=... -c:v libx264 -preset medium -crf 19` → 原版。
- 分享版：`-crf 27 -c:a aac -b:a 128k`，约 50MB，`-share.mp4`。
- 抽帧验货（关键对白点 + 转场点），确认字幕无遮挡、画面无拉伸。
- 上传：`/opt/hatch/bin/remote-storage upload-file --path <share>`，
  需用户 10 分钟内点审批卡；超时（`approval_expired`）则等用户说"传"再重传。
- 产物落盘：`~/workspace/changshengmen-ch01-film/video/`（原版 + 分享版）。
