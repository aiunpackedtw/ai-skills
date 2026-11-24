---
name: test-first-ci
description: 叫 AI 寫或修改任何程式專案時使用：先寫自動化測試、每次改完都跑測試、用 Git 分支和 commit 保存進度，並在 GitHub 接好 CI（GitHub Actions），讓錯誤在上線前就被擋下來。適用 JavaScript／TypeScript 和 Python 專案。
---

# 測試先行＋CI

來自「AI 拆解室」長片〈AI 幫你寫程式，背後到底發生什麼事？〉。
目的：讓 AI 寫的程式「存得住、改不壞、錯誤上線前就被擋下」。

## 什麼時候用

- 新建任何程式專案。
- 修 bug、加功能、重構。
- 使用者說「幫我寫測試」「幫我接 CI」「為什麼一改就壞」。

## 規則（每次都要照做）

1. **先確認有 Git。** 專案還沒初始化就先 `git init`，並建立 `.gitignore`（至少包含 `node_modules/`、`.env`、`__pycache__/`、`dist/`）。
2. **改東西之前先開分支。** 例如 `git checkout -b fix-total`。不要直接在 main 上改。
3. **先寫測試，再寫或改程式。**
   - 修 bug：先寫一個會「重現這個 bug」的測試，確認它失敗（紅燈），再修到它通過（綠燈）。
   - 加功能：先寫描述預期行為的測試，再實作。
   - 每個核心功能至少要有：正常情況、邊界情況（空值、0、負數、超大數字）、錯誤輸入各一個測試。
4. **每次改完都跑全部測試。** 沒有全綠，不准 commit，也不准說「完成了」。
5. **小步 commit。** 每完成一個能通過測試的小改動就 commit 一次，訊息寫清楚做了什麼，例如 `fix: 總額計算漏掉負數`。
6. **金鑰絕對不進 Git。** API 金鑰、密碼放在 `.env`，程式用環境變數讀取；另外提供 `.env.example`（只有欄位名稱，沒有真實值）。在 CI 裡用 GitHub Secrets。
7. **接上 CI。** 專案還沒有 `.github/workflows/ci.yml` 就照下面範本建立，讓每次 push 和 pull request 都自動跑測試。
8. **完成時回報。** 用三行告訴使用者：新增或修改了哪些測試、目前測試結果（幾個通過）、CI 有沒有接好。

## 測試工具預設

- JavaScript／TypeScript：Vitest（專案已在用 Jest 就沿用 Jest）。`package.json` 要有 `"test": "vitest run"`。
- Python：pytest。測試放在 `tests/`，檔名 `test_*.py`。

## CI 範本：JavaScript／TypeScript

`.github/workflows/ci.yml`

```yaml
name: CI
on:
  push:
  pull_request:
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: actions/setup-node@v5
        with:
          node-version: 22
          cache: npm
      - run: npm ci
      - run: npm test
```

## CI 範本：Python

`.github/workflows/ci.yml`

```yaml
name: CI
on:
  push:
  pull_request:
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: pip
      - run: pip install -r requirements.txt pytest
      - run: pytest -q
```

## 測試範例（給使用者看懂用）

```js
// total.test.js：記三筆 100 元，總額必須是 300
import { expect, test } from "vitest";
import { total } from "./total.js";

test("三筆 100 元，總額是 300", () => {
  expect(total([100, 100, 100])).toBe(300);
});

test("沒有任何紀錄時，總額是 0", () => {
  expect(total([])).toBe(0);
});
```

## 不要做的事

- 為了讓測試通過而去改測試本身或刪掉測試（除非使用者確認原本的預期是錯的）。
- 用 `skip`、`only` 跳過失敗的測試後就回報完成。
- 把 `.env` 或任何金鑰 commit 進去。
- 一次改一大堆檔案才 commit。
