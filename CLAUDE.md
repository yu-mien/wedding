# 婚禮喜帖網站

單一檔案的靜態網站（`index.html`），沒有 build 步驟。本機預覽：`.claude/launch.json` 裡的 `wedding`（`python3 -m http.server 8000`）。

## Git

- 不要開新分支，直接在 `main` 上 commit 並 push（使用者明確指定）。

## 參考資料

- `docs/reference/ui-ux-prompt-reference.md`：網路上分享的婚禮網站 UI/UX 指令，**僅供參考、尚未採用**。不要主動照它改網站；只有使用者明確指定要套用其中某部分時才使用。

## Mobile First

- CSS 以手機為基準撰寫，桌機調整只放在 `@media (min-width: 768px)` 裡。
- 每次改完 UI，先在手機寬度（375px）review：截圖每個區塊、確認 `document.documentElement.scrollWidth` 不超過視窗寬度、文字斷行自然、表單好點好填。
- 手機確認後，再用桌機寬度檢查有沒有跑版。至少看 1280×800、1920×1080、768×1024，用原尺寸截圖逐區塊看清楚，並確認照片沒有被截到人物。
- 首頁主視覺是照片輪播：`assets/hero/01.jpg`～`10.jpg`，依檔名順序播放，每 1.5 秒換一張，換的時候 0.5 秒淡入淡出（`setInterval` 1500ms，CSS `transition: opacity 0.5s`）。下一張還沒載完就先停在目前這張。標語、名字、日期一律放在照片下方，不要壓在照片上。
- 相框是 2:3 直式、圓角，`object-fit: contain`，相框底色 `--accent-soft`。所有照片都是 2:3 直式，剛好填滿。02 原本是橫式（3:2），使用者決定裁成 2:3 填滿相框，可以稍微切到人：從原檔（6816×4544）裁 x=1788、寬 3029 的區域，兩人置中，新郎右手臂和新娘裙襬左側會切掉一點。
- 照片原檔（02 以外都沒裁切）在使用者的 Google Drive 資料夾 https://drive.google.com/drive/folders/11XzlrAqM6mAdZl6Ke4OpjakU_6Oe1_3W （檔名 1.jpg～10.jpg，每張 6–12MB、sRGB）。放進網站前要縮小：寬 1080（1080×1620），JPEG 品質 80、progressive，套用 EXIF 方向並移除 EXIF（含 GPS），10 張合計約 1.2MB。增減照片時要同時改 `index.html` 裡 `.hero-frame` 的 `<img>` 清單。
- 原本的首頁影片（`hero-video.mp4`、`hero-poster.jpg`）已刪除，需要時可從 git 歷史找回。
- 背景音樂放在 `assets/music.mp3`，右下角的音樂開關鈕會先用 HEAD 請求確認檔案存在才顯示；沒有這個檔案時按鈕自動隱藏。瀏覽器不允許有聲音的自動播放，所以音樂只會在訪客點按鈕後開始。
- 目前的 `assets/music.mp3` 是 CHEEZE〈My Romance〉（韓劇《海岸村恰恰恰》原聲帶，℗ 2021 Stone Music Entertainment），由使用者從 Google Drive 提供，來源是 YouTube。**這首有版權、沒有取得公開網站的使用授權**，Claude 已說明風險（可能收到 DMCA 下架），使用者決定照樣使用。原檔 256kbps（5.5MB），已壓成 128kbps（約 2.8MB）並移除檔案內的標籤。之前用的是公有領域的帕海貝爾〈卡農〉（美國空軍樂團 Strolling Strings 演奏）：https://commons.wikimedia.org/wiki/File:Canon_(2004)_-_Strolling_Strings_-_United_States_Air_Force_Band.mp3 ，要換回無版權疑慮的音樂時可用。換音樂時要確認授權允許用在公開網站。

## 出席回函表單必須與 Google 表單保持一致

`#rsvp-form` 會把資料送到 Google 表單：
https://docs.google.com/forms/d/e/1FAIpQLSe7WhgKEuIkK121dCIaKZF2r23Dd-KlHcRcSrvckXlhJnJe9w/viewform

Google 只收下文字和它的選項**完全相同**的答案（包含全形標點、空白），所以：

- 只要更動網頁表單的欄位、選項，或選項裡的文字，就要同步更新 Google 表單的對應欄位（反過來也一樣）。
- 改網頁時，`value` 和畫面上顯示的文字都要改，`toggleDynamicFields()`、送出時判斷 `pageHistory` 的程式碼裡寫死的選項文字（例如「期待參與，一定到場！」）也要一起改。
- Claude 無法直接編輯 Google 表單：改網頁的同時，要列出 Google 表單上需要怎麼改（哪個問題、舊文字 → 新文字），請使用者去改。
- 使用者改完後，打開上面的 viewform 網址，從 `window.FB_PUBLIC_LOAD_DATA_[1][1]` 讀出每題的標題、`entry` ID、選項，逐一和網頁比對，確認一致才算完成。
- Google 表單分成三個區段（0: 基本資料、1: 出席人數及飲食、2: 喜帖），送出時的 `pageHistory` 依賴這個結構。新增或刪除區段時要一併修改送出程式碼。
