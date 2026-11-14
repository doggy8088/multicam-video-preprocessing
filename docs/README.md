# Multicam Video Processing

多機位影片處理練習專案，使用 `multicam-video-preprocessing` plugin 進行音訊同步、響度標準化與 AI 粗剪。

## Plugin 資訊

| 項目 | 內容 |
| --- | --- |
| 名稱 | `multicam-video-preprocessing` |
| 版本 | 1.7.0 |
| 作者 | [sylphlin](https://github.com/sylphlin) |
| Repo | https://github.com/sylphlin/multicam-video-preprocessing |
| 授權 | MIT |
| 規範 | [Agent Plugins 1.0](https://agent-plugins.org/) |
| 內容 | 1 skill、1 rule |
| 參考 commit | `66f6a13` (2026-10-02) |

### 功能

- MFCC 聲學時間對齊（次影格微調 < 0.125 ms），支援 2–6 機位
- EBU R128 響度標準化（-14 LUFS）
- 多機合一網格預覽影片（10 fps、1.2 Mbps、1s GOP）
- Vertex AI Gemini 3.8 Flash 多模態粗剪（靜音感知分段 30–40 分鐘）
- 8 項 EDL 語意驗證
- FCP7 XML 時間軸匯出（DaVinci Resolve / Premiere Pro / Final Cut Pro）與單次硬體加速渲染

### 三階段工作流

| Stage | 腳本 | 說明 |
| --- | --- | --- |
| 1 | `multicam_pipeline.py` | 同步、響度標準化、同步母帶、網格合成 |
| 2 | `generate_edl.py` | Gemini 多模態粗剪，產生 EDL |
| 3A | `export_fcp7_xml.py` | 匯出 FCP7 XML 時間軸（主要路徑） |
| 3B | `edl_to_video.py` | 直接渲染預覽影片（次要路徑） |

### 安裝

```bash
git clone https://github.com/sylphlin/multicam-video-preprocessing.git \
  ~/.gemini/config/plugins/multicam-video-preprocessing

# 選用：連結為單一 Skill
ln -s ~/.gemini/config/plugins/multicam-video-preprocessing/skills/multicam-video-preprocessing \
  ~/.gemini/config/skills/multicam-video-preprocessing
```

### 相依套件與雲端設定

```bash
brew install ffmpeg
pip3 install --user --break-system-packages numpy google-genai google-cloud-storage requests

gcloud auth application-default login
cd ~/.gemini/config/plugins/multicam-video-preprocessing
./setup.sh --project YOUR_GCP_PROJECT_ID   # 設定 GCS、生命週期規則、IAM 與 .env
```

> Stage 2 需要 Google Cloud Vertex AI (ADC) 與 GCS。

## 樣本素材

下載自 [Google Drive 資料夾](https://drive.google.com/drive/folders/1HqNRVqrJREjHAWtRc4cWMousbWmhur-r?usp=sharing)：

| 檔案 | 規格 | 時長 |
| --- | --- | --- |
| `CAM1_Host_720P.mp4` | 1280x720, 29.97 fps, H.264 / AAC | 約 4:02 |
| `CAM2_Guest_720P.mp4` | 1280x720, 29.97 fps, H.264 / AAC | 約 3:55 |

```bash
uvx gdown "https://drive.google.com/drive/folders/1HqNRVqrJREjHAWtRc4cWMousbWmhur-r?usp=sharing"
```

## 輸出

處理結果位於 `output/`（同步母帶、`multicam_merged_full.mp4`、`edl_full.csv`、`final_cut_full.xml` 等）。

## 使用方式

在 Antigravity 對話中以自然語言下達指令，例如：「幫我把 CAM1 與 CAM2 同步並產生粗剪時間軸」。CLI 選項請參考 plugin 內 `skills/multicam-video-preprocessing/SKILL.md`。
