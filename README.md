# PhysicsEasy — 看得見的物理，才學得會

**HKDSE 物理科互動學習平台**　·　繁體中文優先　·　從錯誤中學物理，而非背公式

### ▶ [**進入網站 → iotsolutionsmartcity.github.io**](https://iotsolutionsmartcity.github.io/)

---

拋體、天體、氣體、光 —— 背後是一整片可互動的物理力場。
以真實模擬與歷屆試題，把每一次做錯，變成真正搞懂的一刻。

## 平台功能

| 功能 | 說明 |
|------|------|
| **HKDSE 兩層課題導覽** | 必修 / 選修課題 → 子課題，對應官方課程架構（9 大課題） |
| **互動題目檢視器** | 題幹內概念可點擊 → 側欄即時彈出對應筆記，MC 與長題目皆支援 |
| **豐富互動筆記** | Markdown + 表格 + 可放大圖片 + `[[blank:答案\|提示]]` 填充題 + 章節目錄 |
| **概念層級錯題追蹤** | 記錄作答、迷思概念、筆記／模擬開啟次數，推導掌握度並提示弱項課題 |
| **歷屆試題下載中心** | 2012–2026，卷一 / 卷二 / 評卷參考，支援搜尋、年份與卷別篩選 |
| **高保真 3D 模擬** | React Three Fiber 場景（斜面受力、向量合成），delta-time 物理與陰影 |
| **Playground ↔ 題目閉環** | 模擬中觸發特定物理狀態（如全內反射）→ 側欄推薦相關題目；題目頁反向推薦「選做」實驗 |
| **AI 內容匯入工具** | DeepSeek 解析 PDF/DOC 試卷 → 自動抽出題目、選項、答案與圖片，寫回內容庫 |
| **AI 學習夥伴（實驗）** | 本地 Ollama + 筆記 RAG，串流繁中物理導師回覆 |

## 技術架構

| 層面 | 技術 |
|------|------|
| 框架 | Next.js 16 App Router · React 19 · TypeScript |
| 樣式 | Tailwind CSS 4 · shadcn 風格 UI primitives |
| 3D | Three.js · @react-three/fiber · @react-three/drei |
| 狀態 | Zustand + localStorage（SSR-safe hydration） |
| 內容 | JSON（題目／概念／課題）+ MDX 筆記 |
| AI · 匯入 | DeepSeek API（獨立 Express server，`ingestion-server/`） |
| AI · 導師 | Vercel AI SDK + 本地 Ollama `deepseek-r1:14b` |
| 測試 | Vitest · jsdom · `validate:content` 內容完整性檢查 |

## 文件

- [`PROJECT_HANDOFF.md`](PROJECT_HANDOFF.md) — 完整開發交接：目錄結構、內容格式、已實作功能、AI 管線、待辦事項

---

> 🚧 主平台開發中 · UI 與學生內容以繁體中文為主
