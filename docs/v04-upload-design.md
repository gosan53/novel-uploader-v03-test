# v0.4 安全 ZIP 上傳設計（開發分支）

## 已實作
- 前端本機 ZIP 中央目錄基本檢查。
- 從 ZIP 檔名開頭自動帶入版本號，版本號不符時顯示錯誤。
- 尚未啟用上傳按鈕；此階段不得宣稱已上傳或部署。

## 安全邊界
- GitHub Pages 為靜態前端，不得存放 GitHub Token、Cloudflare API Token 或 R2 密鑰。
- 獨立認證後端核對允許的 GitHub 身分，採短期有效工作階段及 CSRF/Origin 檢查。
- 後端限制可上傳的作品、類型、最大檔案大小及 ZIP 格式；禁止 zip-slip、壓縮炸彈及異常解壓總量。
- ZIP 原件以唯一上傳 ID 存入私人暫存物件儲存區（候選方案：Cloudflare R2）；暫存區不直接公開，也不直接部署。
- 前端按檔案分塊上傳，逐塊重試，支援失敗後安全續傳；後端核對大小與 SHA-256，最後再驗證 ZIP 結構與內容。
- 後端不得信任前端版本號，需自行解析 ZIP 檔名並核對中繼資料。
- 不使用 GitHub Contents API 儲存 75 MB ZIP；GitHub repository 不作為大型原始 ZIP 的上傳暫存服務。
- 正式公開網站 robot.learncode53.workers.dev 不接受此開發階段的部署指令。
- little-darkness-5326.learncode53.workers.dev 的部署也必須在上傳驗證完成後，另行人工確認才執行。
- robot-novel-site 母版、既有 main 分支、正式網站均不修改。

## 預計 API（尚未實作）
1. POST /api/uploads/init：授權後建立上傳工作，回傳 uploadId、分塊資訊。
2. PUT /api/uploads/:id/chunks/:index：上傳一塊，後端驗證工作階段及分塊範圍。
3. GET /api/uploads/:id：取得進度，支援恢復。
4. POST /api/uploads/:id/complete：驗證大小、雜湊、ZIP 目錄及檔名版本號，標記為已驗證候選。
5. POST /api/deployments/preview：在第二次人工確認後才允許將已驗證候選部署到 little；初期保持關閉。

## 待確認的外部資源
- Cloudflare 帳戶是否已有可供本專案使用的 R2 儲存桶及適當的費用限制。
- 是否使用 Cloudflare Access 或 GitHub OAuth 作為登入方式；不得要求使用者將永久 Token 貼入公開網頁。
- 測試專用 Worker 網址與授權設定。

## 驗收條件
- Android Chrome、iPhone Safari 各上傳相同 179 ZIP，確認伺服器接收且 SHA-256 相符。
- 模擬中斷、重試、版本不符、損毀 ZIP、未授權請求。
- 確認測試期間 robot 正式網站、little 測試網站及小說母版均未改動。
