# C Programming for Embedded Applications - 練習檔案使用指南

這份指南旨在協助你理解與使用課程附帶的 `Exercise Files`。

## 1. 資料夾結構邏輯 (Folder Structure)

課程資料夾通常依照「章節 (`ChXX`)」->「影片單元 (`XX_YY`)」->「狀態 (`Begin`/`End`)」的邏輯編排。

```text
Exercise Files/
├── Ch02/                  <-- 章節 (Chapter)
│   ├── 02_03/             <-- 單元 (Unit)
│   │   ├── Begin/         <-- 初始狀態 (跟著影片一起做)
│   │   └── End/           <-- 完成狀態 (參考解答)
│   └── 02_04/
└── Ch03/
    └── ...
```

- **Begin**: 這是講師在影片開始時的程式碼狀態。如果你想跟著影片練習，請開啟這裡的專案。
- **End**: 這是影片結束時的完成品。如果你卡關了，或只想看最終結果，請參考這裡。

## 2. 專案類型與開發環境

本課程混用了兩種主要的開發環境，對應不同的硬體架構：

### A. Arduino (AVR 架構)
- **識別方式**：資料夾內含有 `.ino` 檔案。
- **目標硬體**：Arduino Uno (ATmega328P)。
- **如何開啟**：使用 **Arduino IDE** 開啟 `.ino` 檔。
- **學習重點**：
  - 適合快速驗證邏輯與週邊控制 (GPIO, UART)。
  - 注意：Arduino IDE 隱藏了很多底層細節 (如 `main()` 函式)，若要深入了解嵌入式 C，需看其底層 Library。

### B. Keil µVision (ARM Cortex-M 架構)
- **識別方式**：資料夾內含有 `.uvproj` 或 `.uvprojx` 檔案。
- **目標硬體**：ST STM32F3 Discovery (Cortex-M4F)。
- **如何開啟**：使用 **Keil µVision 5** (Windows Only) 開啟專案檔。
- **重點檔案**：
  - `main.c`：核心邏輯所在。
  - `startup_stm32f30x.s`：啟動程式碼 (Startup Code)，包含 Reset Handler 與 Vector Table (進階學習必看)。
- **學習重點**：
  - 這是業界標準的開發環境。
  - 可以觀察 Registers, Memory Map, Disassembly (組合語言)。

## 3. 如何閱讀與分析 (How to Analyze)

建議的學習流程：

1.  **先看 `main.c` 或 `.ino`**：
    -   尋找 `setup()` 或硬體初始化函式。
    -   尋找 `while(1)` 或 `loop()` 無窮迴圈，這是程式的主體。
2.  **關注關鍵字**：
    -   搜尋課程提到的重點，如 `volatile`, `const`, `static`, `#define`。
    -   注意對暫存器的操作 (如 `DDRB |= ...` 或 `REG->Bit = ...`)。
3.  **比較 Begin 與 End**：
    -   使用 Diff 工具比較 `Begin` 與 `End` 資料夾中的 `main.c`，觀察講師到底改了什麼。

## 4. 常見問題 (FAQ)

-   **Q: 我是 Mac 使用者，無法跑 Keil 怎麼辦？**
    -   A: 你可以直接用 VS Code 開啟 `main.c` 閱讀程式碼邏輯。雖然無法編譯或模擬 (除非裝 Cross-Compiler)，但對於理解程式碼結構已經足夠。
-   **Q: 為什麼有些資料夾只有一層，沒有 Begin/End？**
    -   A: 有些單元只是展示特定觀念 (如 Memory Layout)，不需要動手寫 Code，所以只有一份原始檔。
