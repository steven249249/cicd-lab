# CI/CD Lab

這份文件是 Lab 手冊，會帶你完成：

1. 啟動 Fastify 應用
2. 實際 push 到 GitHub 觀察 CI
3. 在本機用 `act` 模擬 CI
4. 理解不同 branch 推送時的效果
5. 完成課堂練習

## 開始前

建議使用 GitHub Codespaces 開啟本 repo（環境已預先準備 Node / Docker / act）。

### Fork

1. 到原始 repo 頁面，點右上角 **Fork**
2. 進入你自己的 fork repo
3. 用你的 fork repo 開啟 Codespaces

先確認工具版本：

```bash
node --version
docker --version
docker compose version
act --version
```

## 本地開發

安裝依賴：

```bash
npm ci
```

啟動服務：

```bash
npm run build
npm run start
```

驗證服務：

```bash
curl http://localhost:3000/
curl http://localhost:3000/health
```

## Lab-01: Hello GitHub Actions

Push 到 GitHub 觀察 CI

### 第一次使用請先確認 Actions 已啟用

在你的 fork repo：

1. 點選上方 **Actions** 分頁
2. 如果看到啟用提示，點 **I understand my workflows, go ahead and enable them**（或同義按鈕）
3. 回到 Code 頁面繼續操作

### 實際推送一個 feature branch

把 `snippets/01_hello.yaml` 複製到 `.github/workflows/` 底下

```bash
git checkout -b feature/ci-observe
cp snippets/01_hello.yaml .github/workflows/
git add .
git commit -m "ci: add hello.yaml"
git push origin feature/ci-observe
```

### 在 GitHub Actions 頁面觀察 Hello CI 結果

1. 到你的 fork repo 的 **Actions** 頁面
2. 找到最新 `ci` workflow run
3. 查看 01_hello.yaml 的執行步驟與結果

---

## Lab-02: Run test

把 `snippets/02_run-test.yaml` 複製到 `.github/workflows/` 底下

```bash
cp snippets/snippets/02_run-test.yaml .github/workflows/
git add .
git commit -m "ci: add run-test.yaml"
git push origin feature/ci-observe
```

### 在 GitHub Actions 頁面觀察 Run test CI 結果

- 觀察 run-test.yaml 內容
- 到 GitHub Actions 頁面, 查看 run test 執行結果
- 觀察 artifact

---

## Lab-03: Run GitHub Actions with act locally

### 用 `act` 模擬 push event

`act` 是一個可以在本機執行 GitHub Actions workflow 的工具

你可以用它：

- 在不 push 到 GitHub 的情況下先驗證 workflow
- 快速重跑失敗步驟、縮短除錯時間
- 模擬 `push` / `pull_request` 等事件

官方資源：

- 官網：<https://nektosact.com/>
- GitHub 專案：<https://github.com/nektos/act>

切到要模擬的 branch，執行 `act push`

```bash
act push
```

若只想跑單一 workflow，可加 `-W`：

```bash
act push -W .github/workflows/ci.yaml
```

進階補充：若你需要在「不切 branch」情況下指定事件分支，可以使用：

```bash
act push --env GITHUB_REF=refs/heads/<branch>
```

---

## Lab-04: Conditional workflow and deploying

實際應用中, 我們可能會將每一個版本都跑過 ci, 並 build 出 image
但 image tag 要可以區分出是 feature branch 或是 release branch
方便我們理解與追蹤特定版本的 source code

觀察 `snippets/ci.yaml` 與 `snippets/cd.yaml`

將 `snippets/ci.yaml` 與 `snippets/cd.yaml` 複製到 `.github/workflows/` 底下

```bash
cp snippets/ci.yaml .github/workflows/
cp snippets/cd.yaml .github/workflows/
```

### 在 feature branch 觀察 ci.yaml 執行結果

```bash
git checkout -b feature/a
act push
```

觀察 build 出來的 image

```bash
docker images
```

### 切出 release branch 觀察 ci.yaml 執行結果

切出 release branch

```bash
git checkout -b release/1.0.0
act push
```

觀察 build 出來的 image

```bash
docker images
```

## 思考

- 如何設計 CI Pipeline 以確保程式碼品質
- 如何設計 CD Pipeline 部署到目標環境
- CI Pipeline 與 CD Pipeline 的相依關係
- 通知或報表機制

## 已實作 CI Pipeline

本專案新增了 `.github/workflows/ci_313832005.yaml`，包含以下自動檢查：

- TypeScript typecheck：使用 `npm run typecheck`，透過 `tsc --noEmit` 檢查型別錯誤
- Prettier check：使用 `npm run format:check`，檢查格式是否符合專案設定
- Test：使用 `npm test` 執行 `vitest run`，並同時輸出 JUnit 測試報告

為了讓測試結果在 GitHub Actions 結果頁面中更容易查看，pipeline 會：

- 將測試報告輸出為 `reports/vitest-junit.xml`
- 使用 `actions/upload-artifact@v4` 上傳報告檔案
- 使用 `dorny/test-reporter@v1` 將 JUnit 測試結果發佈到 GitHub Test Report 顯示區

若任一檢查失敗，pipeline 會直接回傳失敗，並在 Actions 執行紀錄中顯示對應錯誤。