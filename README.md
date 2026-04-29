
# HEX Merge Tool — 使用說明文件

> **版本 Version:** v0.02  
> **類型 Type:** 純前端網頁工具 / Pure front-end web application  
> **格式支援 Format:** Intel HEX (`.hex`)
> **線上使用 Live Demo:** [https://willylo123.github.io/hex_merge_tool/](https://willylo123.github.io/hex_merge_tool/)

---

## 目錄 Table of Contents

1. [工具簡介 Overview](#工具簡介-overview)
2. [適用場景 Use Cases](#適用場景-use-cases)
3. [功能說明 Features](#功能說明-features)
4. [操作流程 How to Use](#操作流程-how-to-use)
5. [檔名命名規則 Output Filename Logic](#檔名命名規則-output-filename-logic)
6. [校驗碼說明 Checksum Explained](#校驗碼說明-checksum-explained)
7. [注意事項 Limitations & Notes](#注意事項-limitations--notes)
8. [技術細節 Technical Details](#技術細節-technical-details)

---

## 工具簡介 Overview

**HEX Merge Tool** 是一個執行於瀏覽器中的靜態網頁工具，無需安裝、無需伺服器，可將兩個 **Intel HEX 格式**的韌體檔案合併為一個單一輸出檔。

> A browser-based static web tool — no installation or server required — that combines two **Intel HEX** firmware files into a single merged output.

---

## 適用場景 Use Cases

此工具特別適合嵌入式韌體開發流程中的以下情境：

| 場景 Scenario | 說明 Description |
|---|---|
| Bootloader + Application | 主程式與開機載入程式分開編譯，合併後一次燒錄 |
| RS232 燒錄版本整合 | 將含有 `(RS232)` 的通訊韌體與主韌體合併 |
| 雙核心/雙分區韌體 | 兩個獨立編譯的 HEX 區段需要合併成單一燒錄檔 |

> Commonly used in embedded firmware workflows: merging a Bootloader with an Application HEX, or combining an RS232 communication firmware with the main firmware for single-pass programming.

---

## 功能說明 Features

- **拖放或點擊上傳** — 支援拖放（Drag & Drop）或點擊選擇 `.hex` 檔案
  *(Drag-and-drop or click-to-browse file selection)*

- **即時校驗碼顯示** — 每個載入的檔案即時顯示 16-bit 資料總和校驗碼（CKS）
  *(Real-time 16-bit data-sum checksum display for each loaded file)*

- **自動檔名處理** — 根據命名規則自動產生輸出檔名
  *(Automatic output filename generation based on naming rules)*

- **合併後統計資訊** — 顯示 File 1 / File 2 / 合併後總行數
  *(Post-merge stats: line counts for File 1, File 2, and total)*

- **校驗碼驗證警告** — 若原始 HEX 行校驗碼不符，會顯示警告
  *(Warning if any HEX record has a checksum mismatch)*

- **瀏覽器端直接下載** — 全程不上傳伺服器，資料不離開本機
  *(All processing happens locally; no data is sent to any server)*

---

## 操作流程 How to Use

```
步驟 1  載入 File 1（基礎檔）
        Load File 1 — the base firmware file

步驟 2  載入 File 2（附加檔）
        Load File 2 — the file to be appended

步驟 3  確認兩個檔案的 CKS 校驗碼無誤
        Verify both CKS checksums look correct

步驟 4  點擊「Merge Files」按鈕
        Click the "Merge Files" button

步驟 5  點擊下載連結取得合併後的 .hex 檔案
        Click the download link to save the merged .hex file
```

### 載入方式 File Loading Methods

**方式 A — 拖放 Drag & Drop**  
將 `.hex` 檔案直接拖曳至對應的虛線區域即可。

**方式 B — 點擊選擇 Click to Browse**  
點擊虛線框，開啟系統檔案選擇視窗。

---

## 檔名命名規則 Output Filename Logic

輸出檔名依據 **File 2** 的檔名自動決定：

| File 2 檔名包含 Contains | 輸出規則 Output Rule | 範例 Example |
|---|---|---|
| `(RS232)` 字樣 | 移除 `(RS232)` 部分 | `FW_v1.0_(RS232).hex` → `FW_v1.0.hex` |
| 不含 `(RS232)` | 附加 `_merge` 後綴 | `FW_v1.0.hex` → `FW_v1.0_merge.hex` |

> The output filename is derived from File 2's name. If it contains `(RS232)`, that substring is stripped. Otherwise, `_merge` is appended before the extension.

---

## 校驗碼說明 Checksum Explained

工具顯示的 **CKS** 為「16-bit 資料總和校驗碼」，計算方式如下：

1. 讀取所有 **Type 00（資料記錄）** 的位元組
2. 將所有位元組數值相加（無限制累加）
3. 取結果的最低 16 位元（`& 0xFFFF`）
4. 以 4 位十六進制顯示（例如：`CKS: 0x 3F2A`）

```
CKS = (Σ all data bytes) & 0xFFFF
```

> The displayed CKS is a 16-bit masked data sum across all Type-00 records. It is a quick sanity check — it does **not** replace the per-line Intel HEX checksum built into the format.

**注意 Note:** 此校驗碼為快速核對用途，並非 Intel HEX 格式本身的逐行校驗。每行的內建校驗碼（最後 2 位）仍會在載入時自動驗證，若不符會顯示警告。

---

## 注意事項 Limitations & Notes

- 工具目前僅處理 **Type 00（資料）** 與 **Type 01（EOF）** 記錄；Type 02–05（擴充位址）會被保留但不額外驗證。
  *(Only Type 00 data and Type 01 EOF records are actively validated; extended address records Type 02–05 are passed through as-is.)*

- **不檢查位址重疊**：若兩個檔案有相同的記憶體位址範圍，合併後可能造成韌體異常，請使用者自行確認。
  *(Address overlap is not detected. If both files target the same memory range, the merged output may be invalid — verify address maps manually.)*

- 輸出行結尾為 **CRLF (`\r\n`)**，符合 Intel HEX 標準交換格式。
  *(Output uses CRLF line endings per Intel HEX convention.)*

- 僅接受副檔名為 `.hex` 的檔案。
  *(Only files with the `.hex` extension are accepted.)*

---

## 技術細節 Technical Details

| 項目 Item | 說明 Detail |
|---|---|
| 執行環境 Runtime | 純瀏覽器 JavaScript，無後端 / Pure browser JS, no backend |
| 檔案讀取 File Reading | `FileReader` API |
| 拖放支援 Drag & Drop | `DataTransfer` API |
| 下載產生 Download | `Blob` + `URL.createObjectURL()` |
| 行結尾 Line Endings | CRLF (`\r\n`) |
| 校驗碼遮罩 Checksum Mask | `0xFFFF` (16-bit) |
| 字型 Fonts | JetBrains Mono, DM Sans (Google Fonts) |

---

*文件版本 Doc version: 1.0 — 對應工具版本 Tool v0.02*
