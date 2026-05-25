# CLAUDE.md

此資料夾由 **Claude Code agent 直接操作**，用於 YouTube 影片下載與影音編輯（yt-dlp + ffmpeg CLI）。

## 專案結構

```
miscTools/
├── <藝人名稱>/          # 個人歌唱實況切軌
│   ├── *.mp3
│   └── _<藝人名稱>.md   # 曲目時間軸筆記
├── live/               # 聯動直播 MP4 片段（統一存放）
├── VSPO/               # ぶいすぽ成員 MP3（不含個人資料夾藝人）
├── live.md             # 聯動直播/演唱會記錄（待處理清單）
├── .yt-dlp-cache/      # yt-dlp 快取（gitignored）
├── www.youtube.com_cookies.txt  # 會員登入 cookie（gitignored）
└── .gitignore
```

## 常用 CLI 指令

```bash
# 下載 MP3（320K），自動嵌入封面與 metadata（一般影片）
yt-dlp \
  --extract-audio --audio-format mp3 --audio-quality 320K \
  --embed-thumbnail --embed-metadata \
  --remote-components ejs:github \
  --cache-dir /tmp/claude/yt-dlp-cache \
  -P "/Users/guangzhenglee/Workspace/miscTools" \
  -o "%(title)s.%(ext)s" "<URL>"

# 下載 MP3（320K）——會員影片（需登入 cookie）
yt-dlp --cookies-from-browser chrome \
  --extract-audio --audio-format mp3 --audio-quality 320K \
  --embed-thumbnail --embed-metadata \
  --remote-components ejs:github \
  --cache-dir /tmp/claude/yt-dlp-cache \
  -P "/Users/guangzhenglee/Workspace/miscTools" \
  -o "%(title)s.%(ext)s" "<URL>"

# 無損切割 MP3 並嵌入 ID3 標籤與封面
ffmpeg -ss <start> -to <end> -i input.mp3 -i cover.jpg \
  -map 0:a -map 1:v -c:a copy -c:v mjpeg -id3v2_version 3 \
  -metadata title="..." -metadata artist="..." -metadata album="..." \
  -metadata:s:v title="Album cover" -metadata:s:v comment="Cover (front)" \
  output.mp3

# 更新 metadata（不重新編碼）
ffmpeg -y -i input.mp3 -c copy -metadata title="..." input.tmp.mp3 \
  && mv input.tmp.mp3 input.mp3

# WebP 封面轉 JPG（MP3 封面只支援 JPEG/PNG）
ffmpeg -i cover.webp cover.jpg

# 下載完整視訊（MP4，供後續 ffmpeg 切割用）
yt-dlp \
  -f "bestvideo[ext=mp4]+bestaudio[ext=m4a]/best[ext=mp4]" \
  --merge-output-format mp4 \
  --remote-components ejs:github \
  --cache-dir /tmp/claude/yt-dlp-cache \
  -o "/tmp/claude/live_work/<name>.%(ext)s" "<URL>"

# 單獨下載縮圖為 JPG（供 ffmpeg 嵌入用；--embed-thumbnail 下載後會刪除 webp）
yt-dlp --write-thumbnail --convert-thumbnails jpg --skip-download \
  --remote-components ejs:github \
  --cache-dir /tmp/claude/yt-dlp-cache \
  -o "/tmp/claude/live_work/cover" "<URL>"

# 無損切割 MP4 並嵌入 metadata
ffmpeg -y -ss <start> -to <end> -i input.mp4 -c copy \
  -metadata title="..." -metadata artist="..." -metadata album="..." \
  output.mp4
```

## Metadata 與檔名慣例

### 單曲 / MV
- **檔名**：`{song} - {artist}.mp3`
- **title**：歌名（去掉 YT title 中的 `/ artist MV` 雜訊）
- **artist**：歌手名
- **album**：YouTube 原始影片標題（完整保留）
- **封面**：確認已嵌入（ffprobe 應有 video stream）

```bash
# 修正 metadata 並重新命名
ffmpeg -y -i "原始.mp3" -c copy \
  -metadata title="歌名" -metadata artist="歌手" -metadata album="YT標題" \
  "歌名 - 歌手.mp3" && rm "原始.mp3"
```

### 歌唱實況
- 先剪輯切軌，再依剪輯結果確認 metadata 格式
- 不直接套用單曲慣例

## live.md 處理規則

- `mp4` 前綴 → 輸出 MP4 到 `live/`
- `mp3:` 前綴或無前綴 → 輸出 MP3
- 含 夜乃くろむ 的 MP3 → `夜乃くろむ/`
- 其他ぶいすぽ成員 MP3（兎咲ミミ、胡桃のあ 等）→ `VSPO/`
- 時間格式：`MM:SS` 或 `HH:MM:SS`，ffmpeg 需補零為 `HH:MM:SS`

## 注意事項

- **沙盒限制**：只能寫入專案目錄（`.`），無法直接輸出到 `~/Downloads`
- **會員影片**：需加 `--cookies-from-browser chrome`，且必須前景執行（背景平行時 cookie 解密可能失敗）
- **JS Challenge**：下載時加 `--remote-components ejs:github`，否則部分格式缺失
- **cache 路徑**：統一使用 `/tmp/claude/yt-dlp-cache` 或 `.yt-dlp-cache/`
- **封面格式**：MP3 封面只接受 JPEG/PNG，WebP 需先轉換
- **`--download-sections` 直播 VOD 極慢**：應先下載完整檔，再用 ffmpeg 切割
- **封面下載**：`--embed-thumbnail` 嵌入後會刪除 webp；ffmpeg 切割前需另用 `--write-thumbnail --skip-download` 取得 JPG
- **直播完整視訊大小**：高畫質直播視訊通常 3–5 GiB，暫存請使用 `/tmp/claude/live_work/`
