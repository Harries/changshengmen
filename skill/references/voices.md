# Voices（MiniMax 音色表）

用户已通过 Secure Vault 授权 MiniMax 国内站（`custom.minimax`，`api.minimaxi.com`）。
调用：`python3 ~/workspace/skills/minimax/bin/minimax_tts.py speak --text "<文本>" --voice "<音色>" --out <out.mp3> [--pitch N]`
模型：`speech-02-hd`（脚本默认）。

| 用途 | 音色 | 参数 | 说明 |
|---|---|---|---|
| 苍老之声（长生门） | `Chinese (Mandarin)_Gentleman` | `--pitch -3` | 门的画外音角色，全片唯一允许的"非原生"主声 |
| 顾清风（备用） | `junlang_nanyou` | — | 仅工具拒绝生成时作画外音；**不得与原生声混用** |
| 林砚（备用） | `male-qn-qingse` | — | 同上 |
| 狗蛋（备用） | `clever_boy` | — | 同上 |
| 旁白 | `Chinese (Mandarin)_Lyrical_Voice` | — | 已废弃：用户明确不要旁白 |

规则：
- MiniMax 只用于：①苍老之声；②视频工具拒绝生成的对白（画外音 + 反应镜头）。
- 同一角色**全片只用一种声音**：原生成功就全原生；只能二选一。
- 密钥明文不可读、不记录；不复述。
