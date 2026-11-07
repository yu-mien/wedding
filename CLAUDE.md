# 婚禮喜帖網站

單一檔案的靜態網站（`index.html`），沒有 build 步驟。本機預覽：`.claude/launch.json` 裡的 `wedding`（`python3 -m http.server 8000`）。

## 參考資料

- `docs/reference/ui-ux-prompt-reference.md`：網路上分享的婚禮網站 UI/UX 指令，**僅供參考、尚未採用**。不要主動照它改網站；只有使用者明確指定要套用其中某部分時才使用。

## Mobile First

- CSS 以手機為基準撰寫，桌機調整只放在 `@media (min-width: 768px)` 裡。
- 每次改完 UI，先在手機寬度（375px）review：截圖每個區塊、確認 `document.documentElement.scrollWidth` 不超過視窗寬度、文字斷行自然、表單好點好填。
- 手機確認後，再用桌機寬度檢查有沒有跑版。至少看 1280×800、1920×1080、768×1024，用原尺寸截圖逐區塊看清楚，並確認照片沒有被截到人物。
- 首頁主視覺是影片 `assets/hero-video.mp4`（1080×1350，4:5，有 faststart），放在圓角相框裡完整顯示；`assets/hero-poster.jpg` 是影片第一格，當載入前的預覽圖。標語、名字、日期一律放在影片下方，不要壓在影片上。
- `<video>` 必須保留 `autoplay muted loop playsinline webkit-playsinline`，少了 `muted` 或 `playsinline`，iPhone 不會自動播放。換影片時記得用 `ffmpeg -c copy -movflags +faststart` 把 moov 移到檔頭，並重新產生 poster。
- 首頁影片的音軌是無聲的，沒有音樂。背景音樂另外放在 `assets/music.mp3`，右下角的音樂開關鈕會先用 HEAD 請求確認檔案存在才顯示；沒有這個檔案時按鈕自動隱藏。瀏覽器不允許有聲音的自動播放，所以音樂只會在訪客點按鈕後開始。
- 目前的 `assets/music.mp3` 是帕海貝爾〈卡農〉，由美國空軍樂團 Strolling Strings 演奏（Frank Hudson 編曲，2004）。這是美國政府作品，屬於公有領域，不需要標示作者。來源：https://commons.wikimedia.org/wiki/File:Canon_(2004)_-_Strolling_Strings_-_United_States_Air_Force_Band.mp3 ，原檔 10.3MB，已壓成 128kbps（約 3.5MB）。換音樂時要確認授權允許用在公開網站。

## 出席回函表單必須與 Google 表單保持一致

`#rsvp-form` 會把資料送到 Google 表單：
https://docs.google.com/forms/d/e/1FAIpQLSe7WhgKEuIkK121dCIaKZF2r23Dd-KlHcRcSrvckXlhJnJe9w/viewform

Google 只收下文字和它的選項**完全相同**的答案（包含全形標點、空白），所以：

- 只要更動網頁表單的欄位、選項，或選項裡的文字，就要同步更新 Google 表單的對應欄位（反過來也一樣）。
- 改網頁時，`value` 和畫面上顯示的文字都要改，`toggleDynamicFields()`、送出時判斷 `pageHistory` 的程式碼裡寫死的選項文字（例如「期待參與，一定到場！」）也要一起改。
- Claude 無法直接編輯 Google 表單：改網頁的同時，要列出 Google 表單上需要怎麼改（哪個問題、舊文字 → 新文字），請使用者去改。
- 使用者改完後，打開上面的 viewform 網址，從 `window.FB_PUBLIC_LOAD_DATA_[1][1]` 讀出每題的標題、`entry` ID、選項，逐一和網頁比對，確認一致才算完成。
- Google 表單分成三個區段（0: 基本資料、1: 出席人數及飲食、2: 喜帖），送出時的 `pageHistory` 依賴這個結構。新增或刪除區段時要一併修改送出程式碼。
