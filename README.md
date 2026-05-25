# miscTools

個人工具集 monorepo，每個子目錄是一個獨立的小工具。

---

## 工具列表

### yt-download

基於 [yt-dlp](https://github.com/yt-dlp/yt-dlp) 的 GUI 影片／音檔下載器。

**功能：**
- 解析影片資訊（標題、縮圖預覽、時長）
- 影片下載：mp4，可選 best / 1080p / 720p / 480p
- 音檔提取：mp3 / m4a / wav / flac
- 自訂輸出資料夾
- 即時下載進度條
- ffmpeg 存在偵測

---

## 系統需求

- Python 3.10 或以上
- ffmpeg（用於音檔轉換與影片串流合併，強烈建議安裝）

### 安裝 ffmpeg

**macOS（Homebrew）：**
```bash
brew install ffmpeg
```

**Windows（winget）：**
```powershell
winget install ffmpeg
```

**Ubuntu / Debian：**
```bash
sudo apt update && sudo apt install ffmpeg
```

---

## 安裝步驟

```bash
# 1. 複製此 repo
git clone <repo-url> miscTools
cd miscTools/tools/yt-download

# 2. 建立虛擬環境（建議）
python3 -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate

# 3. 安裝相依套件
pip install -r requirements.txt
```

---

## 使用方式

```bash
cd tools/yt-download
python main.py
```

1. 貼上影片網址，點擊「解析」
2. 選擇類型（影片 / 音檔）及畫質 / 格式
3. 選擇輸出資料夾（預設 ~/Downloads）
4. 點擊「下載」

---

## 未來工具規劃

> 以下為預留區塊，歡迎擴充。

- `img-compress` — 批次圖片壓縮工具
- `pdf-merge` — PDF 合併 / 拆分工具
- `rename-batch` — 批次重新命名工具
