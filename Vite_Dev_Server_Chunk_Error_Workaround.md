# Vite Dev Server Chunk Error Workaround

## 觀念

Vite chunk error 常見原因是 dependency optimizer cache 壞掉、過期，或 Vite 重新產生出會在 runtime 爆掉的 optimized dependency chunk。

常見錯誤長這樣：

```text
/node_modules/.vite/deps/chunk-xxxx.js
Uncaught TypeError: ...
504 Outdated Optimize Dep
```

這不一定代表 feature code 壞掉。它可能只是瀏覽器還在載入舊的 optimized module graph，也可能是 Vite optimizer 產出的 dependency chunk 本身有問題。

## Workaround 步驟

1. 停掉所有正在跑的 Vite dev server。

   先確認沒有多個 `npm run start` 或 `vite` process 同時存在。如果同時跑兩個 dev server，Chrome 可能會繼續載到舊的 module graph。

2. 清掉 Vite optimizer cache。

   只清這個目錄：

   ```text
   node_modules/.vite
   ```

   不要 `git reset`，不要刪 source code，也不要刪其他無關目錄。

3. 用強制重新 optimize 的方式重啟 Vite。

   ```bash
   npm run start -- --force
   ```

   一般日常開發不一定要加 `--force`；但遇到 chunk error 時建議加，因為它會強制 Vite 重新 optimize dependencies。

   平常正常啟動通常只需要：

   ```bash
   npm run start
   ```

4. 等第一次載入完成。

   這個 repo 第一次建立 module graph 可能比較慢。請等最多 120 秒，不要 10 秒沒有反應就判定 dev server 卡死。

5. 強制刷新 Chrome。

   使用 `Ctrl + Shift + R`。如果還看到同一個舊 chunk hash，就關掉該 tab 重新開。

   舊 chunk 範例：

   ```text
   chunk-3OQZB6BT.js?v=90a4e484
   ```

   如果 dev server 對這個 chunk URL 回 `504 Outdated Optimize Dep`，代表 server 已經更新了，是瀏覽器還握著舊的 module graph。

6. 如果清 cache 並加 `--force` 後還是發生 chunk error，就要檢查 chunk 來源。

   打開出錯的 `/node_modules/.vite/deps/chunk-*.js` 檔案或 URL，查看裡面的 source comment，回推是哪個 `node_modules` dependency 產生的。

   如果停掉 Vite 後 chunk 名稱會變，但每次都指向同一個 source dependency，就不要把它當成單純 stale cache。這比較像 optimizer-generation problem。

   這時候要做 source-level fix。例如 VITE-001 的 root cause 是 `@mui/material/Popper`，穩定修法是在只需要定位功能時改用 `@mui/base/PopperUnstyled`。

## 給 AI 的 Prompt

```text
請處理 MyEdit Vite dev server chunk exception。不要執行 git reset，不要修改無關檔案。

目標：
1. 檢查是否有多個 npm run start / vite process 正在跑，並停止舊的 Vite process。
2. 只清除 node_modules/.vite optimizer cache。
3. 使用 npm run start -- --force 重新啟動。
4. 等待最多 120 秒，確認 https://poc.myedit.online/ 是否回 200。
5. 如果 console 仍指向 /node_modules/.vite/deps/chunk-*.js，請用 curl 讀取該 chunk，並根據 chunk 裡的 source comment 回推 node_modules source。
6. 如果舊 chunk URL 回 504 Outdated Optimize Dep，請判定為瀏覽器仍載入舊 module graph，提醒使用者 Ctrl + Shift + R 或關掉 tab 重開。
7. 如果 forced re-optimization 後仍是同一個 dependency crash，請回報 root cause，並提出最小 source-level fix。
```
