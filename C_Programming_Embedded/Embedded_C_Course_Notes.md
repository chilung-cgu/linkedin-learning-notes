# C Programming for Embedded Applications 學習筆記

這份筆記整理自 LinkedIn Learning 課程《C Programming for Embedded Applications》，並結合外商韌體工程師面試常見考題進行深化。

## 目錄

1. [Embedded Systems (嵌入式系統基礎)](#chapter-1-embedded-systems)
2. [Data Types (資料型態深入解析)](#chapter-2-data-types)
3. [Bit Manipulation (位元操作技術)](#chapter-3-bit-manipulation)
4. [Qualifiers (關鍵修飾字)](#chapter-4-qualifiers)
5. [Function Alternatives (函式替代方案)](#chapter-5-function-alternatives)
6. [FPU Alternatives (浮點運算替代方案)](#chapter-6-floating-point-unit-alternatives)
7. [Final Exam (模擬期末考)](#final-exam)

---

## Introduction

### 課程目標
本課程旨在讓具備 C 語言基礎的開發者，跨越「一般 C 語言」與「嵌入式 C 語言」的鴻溝。重點不在於語法本身，而在於**如何寫出對硬體有意識 (Hardware Aware)、高效能且低功耗的程式碼**。

### 先備知識
- 基本 C 語言語法 (Pointers, Arrays, Structs)
- 基本計算機組織概念 (Binary/Hexadecimal, Memory addresses)

---

## Chapter 1: Embedded Systems

### 1.1 Applications: OS vs. Embedded (一般系統 vs. 嵌入式系統)

| 特性 | 一般系統 (Desktop/Server) | 嵌入式系統 (Embedded System) |
| :--- | :--- | :--- |
| **資源 (Resources)** | 豐富 (GB/TB 等級記憶體、多核 CPU) | **極度受限 (KB/MB 等級記憶體、單核 MCU)** |
| **即時性 (Real-time)** | 非強制 (Soft Real-time / Best Effort) | **強制 (Hard Real-time)**，必須在特定時間內回應 |
| **可靠性 (Reliability)** | 可容忍重開機或當機 | **通常不可停機 (24/7運作)**，Watchdog Timer 是標配 |
| **硬體互動** | 透過 OS Driver 層層抽象 | **直接讀寫暫存器 (Direct Register Access)** |
| **主要目標** | 通用計算、多工處理 | 特定功能 (Dedicated Function)、低功耗 |

> [!IMPORTANT]
> **面試觀點**：當被問到 "What makes embedded programming different?" 時，關鍵字是 <mark> **Constraints (限制)** </mark>。你的每一行程式碼都必須考慮到 Memory, Power, 和 Timing 的代價。

### 1.2 Resource Constraints (資源限制詳解)

在嵌入式開發中，我們必須對以下四種資源斤斤計較：

#### Memory (記憶體)
- **Code Space (Flash/ROM)**: 存放程式碼 (.text) 與唯讀常數 (.rodata)。
    - *實例*：
      - NXP Kinetis KL (ARM Cortex-M0+): **128 KB Flash**。
      - ST STM32F (ARM Cortex-M4): **64 KB Flash**。
      - ATMega328P (Arduino Uno): **32 KB Flash**。
      - *對比*：一個簡單的 Windows Notepad 程式就要 **240 KB**，比上述所有 MCU 的 Flash 還大。
    - *優化策略*：避免肥大的 Library (如 `printf` 完整版)，使用 `const` 關鍵字將變數放入 Flash。

- **Data Space (SRAM)**: 存放變數 (.data, .bss)、Heap 與 Stack。
    - *實例*：
      - NXP Kinetis KL: **16 KB RAM**。
      - ST STM32F: **12 KB RAM**。
      - ATMega328P: **2 KB RAM** (甚至存不下一個小的文字檔)。
    - *概念釐清*：
      - **Desktop RAM**: Main memory, 透過 Cache 存取，包含 Virtual Memory (Swap)。
      - **Embedded RAM**: Tightly Coupled Memory (TCM) 或直接透過 Data Bus 存取，**沒有 Virtual Memory**，使用固定位址 (Fixed Address Range)。
    - *優化策略*：精確控制 Stack 深度，避免遞迴 (Recursion)，盡量少用動態記憶體配置 (malloc/free) 以避免碎片化 (Fragmentation)。

#### Storage (儲存)
- Embedded System 通常沒有硬碟，只有 Flash Memory 或 SD 卡。寫入次數有限 (Write Cycles)，需考慮磨損平衡 (Wear Leveling)。

#### Energy Consumption (功耗)
- 手持裝置或 IoT Sensor 極度依賴電池壽命。散熱片 (Heat sinks) 在 MCU 上極為罕見。
- *實例應用*：
  - 微波爐：控制 Keypad, Door sensor, Display, Magnetron。
  - HVAC：控制 Keypad, Display, 讀取 Analog sensor (溫度)。
- *優化策略*：
    - 多使用 Sleep Mode (例如 NXP/ST 都有 `Sleep_ms()` 或類似的 Low Power Mode API)。
    - 降低 CPU 時脈 (Clock scaling)。
    - 使用 Interrupt 取代 Polling (輪詢)。
    - 選擇適當的資料型態 (例如 8-bit MCU 處理 `uint8_t` 比 `uint32_t` 省電)。

#### Processing Power (運算能力)
- **8-bit MCU** (如 NXP S08, TI MSP430, ATMega): 用於洗衣機、乾燥機、簡單控制。
- **32-bit MCU** (如 ARM Cortex-M): 雖然是 32-bit，但通常是 RISC 架構，指令集精簡。
- *優化策略*：使用定點數 (Fixed-point) 取代浮點數 (Floating-point)，使用查表法 (Lookup Table) 取代複雜數學運算。

### 1.3 C vs. Embedded C

從語言標準來看，**C 與 Embedded C 是完全一樣的語言** (ISO C Standard)。差別在於**使用方式**與**環境**。

- **標準誤區**: 雖然有 `ISO/IEC TR 18037` (Embedded C Standard) 定義了 Fixed-point arithmetic, Named address spaces 等擴展，但**大多數 IDE (Keil, IAR, GCC) 並沒有完全遵循此標準**，而是提供廠商自定義的 Library 或 Keyword (如 `__interrupt`, `__packed`)。

1. **Freestanding vs. Hosted Implementation**:
    - **Hosted (一般 C)**: 有完整的 OS 與 Standard Library (如 `stdio.h`, `stdlib.h` 的完整功能)。`main()` 是程式入口，結束後返回 OS。
    - **Freestanding (Embedded C)**: 無 OS 或僅有 RTOS，Standard Library 功能閹割。程式入口通常是 Reset Handler，`main()` **絕對不能 return** (通常是 `while(1)` 無窮迴圈)。

2. **Hardware Access**:
    - Embedded C 頻繁使用指標操作絕對位址 (Absolute Addresses) 來控制周邊硬體。
    - 關鍵字 `volatile` 是必備知識，但在一般 C 開發較少見。

3. **Bitwise Operations**:
    - 為了節省空間與控制硬體，Embedded C 充滿了位元操作 (Setting/Clearing bits)。

### 1.4 Chapter 1 Quiz (Official - 官方測驗)

1. **問：以下何者不是嵌入式系統節能的動機？**
   - **答：Embedded microcontrollers don't have energy saving features.** (錯誤選項)
   - *詳解*：事實上，節能功能是嵌入式 MCU 最常見的功能之一。

2. **問：為什麼微控制器 (MCU) 的 ROM 通常不大？**
   - **答：because the programs they are supposed to run are usually quite short**
   - *詳解*：許多嵌入式程式只會在一個無窮迴圈中控制少量的 Input/Output 裝置。

3. **問：以下何者是低階嵌入式微控制器常見的 RAM 大小？**
   - **答：2kB**
   - *詳解*：1kB 到 4kB 是 MCU 非常常見的 RAM 容量範圍。

4. **問：在不支援作業系統的低階微控制器中，RAM 和 ROM 通常如何組織？**
   - **答：RAM and ROM are both mapped in the same and only addressing space, so they are accessed by the same bus.**
   - *詳解*：該匯流排也是 CPU 與 I/O 裝置之間的主要通訊手段。

5. **問：許多微控制器的 IDE 僅支援 C 或 C++，而未完全遵循 Embedded C 標準。(True/False)**
   - **答：TRUE**
   - *詳解*：許多 MCU 使用標準 C/C++ 編譯器搭配特定目標的硬體抽象層 (HAL)，而非正式的 ISO Embedded C 擴充標準。

6. **問：傳統 OS 應用程式與嵌入式系統有何不同？**
   - **答：Embedded systems have much less memory, storage and CPU power.**
   - *詳解*：這是因為數位控制演算法通常非常簡單。

7. **問：微控制器通常如何與內部的 I/O 模組 (如 Timer, Serial ports) 通訊？**
   - **答：Through dedicated registers at fixed addresses in a so-called Memory-Mapped Input/Output scheme.**
   - *詳解*：Memory-mapped I/O 在這些系統中很方便，因為 RAM 和 ROM 很小，有足夠的空間分配給裝置暫存器。

8. **問：以下何者最不可能需要低階嵌入式微控制器？**
   - **答：a smartphone running the Android operating system**
   - *詳解*：這需要圖形單元 (GPU)、包含 Cache 的記憶體階層、數個網路裝置以及強大的 CPU。

9. **問：為什麼嵌入式系統使用低階微控制器？**
   - **答：because the usual CPU needs in embedded devices are quite modest**
   - *詳解*：許多嵌入式程式只會在一個無窮迴圈中控制少量的 Input/Output 裝置。

### 1.5 Chapter 1 Quiz (模擬面試題 / 補充)

1. **問：什麼是 Freestanding Implementation？與 Hosted Implementation 有何不同？**
   - 答：**Freestanding** 環境 (如 Embedded System) 沒有完整的作業系統支援，Standard Library 功能受限，且程式入口通常不是 `main` (而是 Reset_Handler)。**Hosted** 環境 (如 Linux/Windows) 則有完整 OS 與 Library 支援。

2. **問：在嵌入式系統中，為什麼 `main()` 函式通常是一個無窮迴圈 `while(1)`？**
   - 答：因為嵌入式韌體通常沒有 OS 可以「返回 (return)」。若 `main()` 結束，CPU 行為將未定義 (可能重置或執行到無效記憶體)。系統必須持續運行以處理任務。

3. **問：解釋 Memory-Mapped I/O 的概念。當不是 Memory-mapped I/O 的時候，有其他通訊方式嗎？**
   - 答：
     - **概念**：請想像記憶體像是這棟大樓的「門牌號碼」。
       - **一般記憶體**：`0x20000000` 住著變數 `x`。你寫入數據，它就存起來。
       - **Memory-Mapped I/O**：`0x40001000` 住著「控制 LED 的開關」。你寫入 `1`，它**不是存起來**，而是**直接拉高電壓點亮 LED**。
       - **重點**：CPU 不知道也不在乎這兩個地址的差別，它都用同一套指令 (`STR`, `MOV`) 去寫入。這就是「記憶體映射」的意思。
     - **替代方案 (Port-Mapped I/O)**：
       - 這是 x86 架構 (Intel/AMD) 較常見的「I/O 獨立定址」。
       - 硬體有自己的「專屬通道」，不跟記憶體共用門牌。
       - CPU 必須用**特殊指令** (如 `IN`, `OUT`) 才能跟硬體講話，C 語言的指標 (Pointer) 對它是無效的。
     - **總結**：ARM (手機、MCU) 和 RISC-V 幾乎全是用 Memory-Mapped I/O，這讓 C 語言操作硬體變得非常直覺。

---

---

### 1.6 Chapter 1 Deep Dive & FAQ (核心觀念釐清)

針對初學者常混淆的觀念進行深度解析：

#### Q.0: 到底什麼是 ROM？跟 Flash 一樣嗎？
- **定義**：ROM (Read-Only Memory) 是「唯讀記憶體」。在嵌入式系統中，它用來存放**程式碼 (Instructions)** 與 **常數 (Constants)**。
- **現代的 ROM**：早期 ROM 是真的寫死不可改的 (Mask ROM)。但現代 MCU (如 STM32, Arduino) 的 ROM 其實是 **Flash Memory** (快閃記憶體)。
- **特性**：斷電後資料**不會消失** (Non-volatile)。
- **對比**：
  - **RAM (Random Access Memory)**：存變數 (Data)。斷電後資料**消失**。像黑板，隨寫隨擦。
  - **ROM (Flash)**：存程式 (Code)。斷電後資料**還在**。像課本，印上去就固定了 (除非重新燒錄)。

#### Q1: 為什麼拿 Notepad (240KB) 來對比 MCU Flash？
- **量級差異**：在 PC 上，240KB 的程式小到微不足道。但在 MCU 世界 (如 Arduino Uno)，Flash (存程式的地方) 總共只有 **32KB**。
- **現實衝擊**：這意味著你如果在 PC 上習慣隨便 `include <stdio.h>` 然後呼叫 `printf("Hello")`，這一行 code 可能就引入了 10KB~20KB 的 Library 程式碼，直接吃掉你一半的 Flash 空間。
- **結論**：嵌入式工程師必須對 Code Size 極度敏感，不能像寫 App 一樣依賴肥大的 Standard Library。

#### Q1.1: 筆記提到「使用 const 關鍵字將變數放入 Flash」，這是什麼意思？如何使用？
- **問題背景**：MCU 的 **RAM (存變數)** 通常比 **Flash (存程式)** 小得多 (例如 RAM 2KB vs Flash 32KB)。
- **這招怎麼用？**
  - **錯誤寫法 (佔用 RAM)**：
    ```c
    // 雖然內容不變，但沒有 const，編譯器會把它放在 RAM 裡
    char welcome_msg[] = "Welcome to Embedded System Course..."; 
    ```
    這行字串如果很長，你的 2KB RAM 馬上就滿了 (Stack Overflow)。
  - **正確寫法 (放在 Flash)**：
    ```c
    // 加了 const，編譯器知道這是不變的，直接放在 Flash (ROM) 裡
    const char welcome_msg[] = "Welcome to Embedded System Course...";
    ```
    這樣 CPU 讀取時直接從 Flash 讀，**完全不佔用寶貴的 RAM**。
- **核心邏輯**：空間大挪移。既然 Flash 空間比較大且用不完，我們就把所有「唯讀資料 (Lookup Table, 字串, 設定參數)」都趕去 Flash，把珍貴的 RAM 留給真正需要變動的變數 (Variables)。

#### Q1.2: (讀者提問) OS 環境 vs Embedded 環境的變數存放位置差異？
- **你的理解**：「OS 下 const 在 memory (.rodata)；Embedded 下 const 在 ROM」 -> **完全正確 (Correct)**。
- **深度解析**：
  - **OS 環境 (Linux/Windows)**：
    - 程式執行時，**所有東西 (Code, Data, Const) 其實都載入到 RAM (Main Memory)** 了。
    - `const` 變數放在 `.rodata` (Read-only Data) 區域。雖然它在 RAM 裡，但 OS 會利用 **MMU (Memory Management Unit)** 把這塊記憶體標記為「唯讀」。如果你試圖寫入，硬體會攔截並觸發 Segmentation Fault。
    - 局部的非 `const` 變數放在 Stack (RAM)。
  - **Embedded 環境 (Bare-metal ARM)**：
    - 這裡的架構是 **Flash (ROM)** 與 **SRAM (RAM)** 實體分開。
    - `const` 全域變數 (Global/Static) 會被 Linker 直接安排在 **Flash** 地址上。CPU 讀取它時，是直接從 Flash 抓資料，**完全不佔用 RAM**。這就是為什麼我們一直強調要用 `const` 來省 RAM。
    - 非 `const` 變數自然就在 RAM 裡。

#### Q2: 問「為什麼 MCU ROM 通常很小？」，答案是「因為程式通常很短」。這邏輯是什麼？
- **直接關聯**：**ROM 就是用來裝程式的**。
  - 程式很短 (Short Program) = 程式碼行數少 = 編譯出來的 Binary 很小 = 佔用的 Bytes 很少。
  -既然程式只需要幾 KB，製造商就不會浪費成本去放幾 GB 的 ROM。
- **應用場景**：嵌入式裝置通常只做一件特定的事 (Dedicated Task)，例如「控制冷氣溫度」。這種邏輯比「跑一個 Windows 作業系統」簡單太多了，所以程式碼自然很短，需要的 ROM 自然就小。

#### Q3: Low-end (Bare-metal) vs. High-end (OS-based) 的記憶體架構差別？
| 特性 | Low-end MCU (e.g., Cortex-M, AVR) | High-end CPU (e.g., Cortex-A, x86) |
|:---|:---|:---|
| **Addressing** | **Physical Address (實體位址)** | **Virtual Address (虛擬位址)** |
| **MMU (Memory Management Unit)** | 無 (或僅有 MPU 保護) | 有 (負責 VA 轉 PA) |
| **View** | 程式設計師直接看到硬體 (RAM/ROM/IO 都在同一個 4GB 空間) | OS 透過 Page Tables 隔離硬體，程式看到的是假的連續記憶體 |
| **I/O Access** | 直接讀寫暫存器位址 (如 `0x40001000`) | 必須透過 Kernel Driver (ioremap) 映射後才能存取 |

#### Q4: Memory-Mapped I/O (MMIO) 到底是什麼？有別的方法嗎？
- **定義**：CPU 把「周邊硬體暫存器」假裝成「記憶體」。
   - 例如：位址 `0x20000000` 是 RAM，位址 `0x40001000` 是控制 LED 的暫存器。
   - 對 CPU 來說，讀寫這兩個位址 **使用完全一樣的指令** (如 `LDR`, `STR` / 指標操作)。
- **另一種方法 (Port-Mapped I/O)**：
   - 早期 x86 架構較常見。記憶體有記憶體的位址，I/O 有自己的位址 (不佔用記憶體空間)。
   - CPU 需要**特殊指令** (如 `IN`, `OUT`) 才能控制硬體。
   - **現代主流 (ARM, RISC-V)**：絕大多數採用 MMIO，因為這樣 C 語言可以用 `Pointer` 直接控制，非常方便。

#### Q5: Memory-Mapped I/O 的位址 (如 0x40001000) 是 Virtual Address 嗎？
- **情況 A: Bare-metal (無 OS，本課程範疇)**：**不是，它是 Physical Address (實體位址)**。
  - 你在 Datasheet 上看到 GPIO Base Address 是 `0x48000000`，你的 C 語言 Pointer 就直接設為 `0x48000000`。CPU 發出的訊號直接對應到硬體線路。
- **情況 B: Linux (有 MMU 的高階 OS)**：**是，它是 Virtual Address (虛擬位址)**。
  - 在 Linux Driver 中，你**不能**直接存取 `0x48000000`。你必須呼叫 `ioremap(0x48000000, size)`，OS 會幫你產生一個 Virtual Address (例如 `0xFFFF1234`) 對應到該實體位址。
  - **但在這個課程的 MCU 世界 (Cortex-M)，我們直接操作 Physical Address。**

#### Q6: Hosted vs. Freestanding (Return 的迷思)
- **Hosted (有 OS)**：
   - 程式是 OS 的一個 Process。
   - `main()` return 0; 代表「告訴 OS 我執行完了，請把 CPU 資源收回去分配給別人 (如 Chrome, Spotify)」。
- **Freestanding (無 OS)**：
   - 整個 CPU 只跑你這隻程式。
   - 若 `main()` return 了，CPU 要去哪？**無處可去**。
   - 通常 Startup Code (啟動碼) 會在呼叫 `main()` 的後面放一個 `while(1);` 死迴圈或 `SystemReset()`。
   - **如果 return，系統通常會當機或不斷重啟 (Watchdog reset)**。所以嵌入式程式的 `main` 必須是無窮迴圈 `while(1) { ... task ... }`。

---

---

### 1.7 OpenBMC 實戰案例分析 (Device Tree & Physical Address)

**情境**：OpenBMC Engineer 檢視 Yosemite 4 DTS (Device Tree Source) 時的疑問。
*Source: `aspeed-bmc-facebook-yosemite4.dts`*

#### Q1: DTS 裡的 `@80000000` 是 Physical Address 嗎？
- **YES (是)**。Device Tree 描述的是硬體視圖，所有的位址都是 **Physical Address (實體位址)**。
- Linux Kernel 啟動時會讀取這些數值，知道硬體在哪裡，然後自行建立 Virtual Address Mapping (Page Tables) 來存取它們。

#### Q2: 如何計算 `ramoops` 是否超出 `memory` 範圍？
讓我們解析相關片段：
```dts
memory@80000000 {
    device_type = "memory";
    reg = <0x80000000 0x80000000>; // 格式為 <Start_Address  Size>
};
```
- **Start**: `0x80000000` (2GB 處)
- **Size**: `0x80000000` (2GB)
- **Range (範圍)**: 從 `0x80000000` 開始，長度 2GB，結束於 `0x80000000 + 0x80000000 - 1` = **`0xFFFFFFFF`**。
- **結論**：這台機器裝了 2GB 的 DRAM，佔據了整個 32-bit Address Space 的後半段。

```dts
ramoops@b8dfa000 {
    compatible = "ramoops";
    reg = <0xb8dfa000 0x6000>; 
};
```
- **Address**: `0xb8dfa000`
- **Math Check**: 
  - `0x80000000` (Memory Start) < **`0xb8dfa000` (Ramoops)** < `0xFFFFFFFF` (Memory End)
- **結論**：`ramoops` **完全在 Memory 範圍內**。
  - 它位於 `0xb8dfa000`，大約是在 DRAM 開始後的 900MB 處。

#### Q3: 既然在 Memory 裡面，為什麼要特別寫出來？(Reserved Memory)
- **目的**：這是一種 **Reserved Memory (保留記憶體)** 技術。
- **流程**：
  1. Linux Kernel 看到 `memory` 節點，知道全系統有 2GB RAM 可用。
  2. 但它又看到 `ramoops` (或 `reserved-memory` node)，知道 `0xb8dfa000` 這塊區域**已被預訂**。
  3. OS 的記憶體管理系統 (Buddy System) **不會**把這塊 RAM 分配給一般程式 (如 `malloc`)。
  4. 這塊區域專門留給 Kernel 在當機 (Panic/Oops) 時寫入除錯訊息。因為它不被一般程式覆蓋，且 Warm Reboot 後 RAM 內容通常因為電容還有電而保留，Kernel 重啟後就可以把當機紀錄讀出來存檔。

#### Q4: 記憶體越大，位址越多？
- **位址空間 (Address Space)**：對於 32-bit CPU (如 AST2600)，這個「門牌號碼本」是固定的 **4GB** (0x00000000 ~ 0xFFFFFFFF)。
- **實體記憶體 (Physical Memory)**：是你實際蓋的「房子」。
  - 你的「門牌本」有 40 億個號碼，但你不一定每塊地都蓋房子。
  - ASPEED 規定 DRAM 必須蓋在 `0x80000000` 這條街上。
  - 如果你裝 1GB RAM -> 你的房子蓋在 `0x80000000 ~ 0xBFFFFFFF`。
  - 如果你裝 2GB RAM -> 你的房子蓋在 `0x80000000 ~ 0xFFFFFFFF`。
- **低階觀點**：位址就是一條條的電線 (Address Bus)。位址越多代表電線越多 (32條線 vs 64條線)，能索引的範圍就越大。

#### Q5: 那 `0x00000000` 到 `0x7FFFFFFF` (也就是小於 0x80000000 的部分) 是做什麼用的？
- 這是典型的 ARM SoC (System on Chip) Memory Map 設計。DRAM 只是其中一部分，其他空間早已被規劃好：
  - **Boot ROM (0x0... ~)**：晶片出廠時燒死的開機程式，Reset 後的第一條指令從這裡開始。
  - **Internal SRAM**：晶片內建的高速記憶體 (Usually Small, e.g., 64KB)，用於早期開機階段 (DRAM 還沒 Initialize 前)。
  - **MMIO (Peripherals) Area**：還記得我們剛談的 Memory-Mapped I/O 嗎？
    - AST2600 的控制暫存器 (Video Engine, GPIO, UART, I2C, SPI) 大多分布在這個區域。
    - 例如 `0x1E780000` 可能是 Watchdog 的控制暫存器。你對這個地址寫入，就是在設定 Watchdog。
  - **SPI Flash Mapping**：BMC 通常透過 SPI 介面連接 NOR Flash (存 BIOS/BMC Firmware)。這個 Flash 的內容也會被映射到某個記憶體區段 (如 `0x20000000`) 供 CPU 直接讀取 (XIP, eXecute In Place)。

#### Q6: OpenBMC (Linux) 也算是 Embedded 嗎？為何有 Virtual Address？
- **廣義的 Embedded Systems**：包含兩大類。
  1. **Bare-metal / RTOS (Deeply Embedded)**：
     - 通常用 MCU (如 STM32, Arduino)。
     - 資源極少 (KB 等級)。
     - **直接操作 Physical Address** (如本課程 Ch1 所述)。
     - 沒有 MMU，沒有複雜的 Process 隔離。
  2. **Embedded Linux (Rich Embedded)**：
     - 使用 MPU (Micro-Processor Unit，如 Cortex-A7, AST2600)。
     - 資源較多 (MB ~ GB 等級)。
     - **有 MMU，使用 Virtual Address**。
     - 跑完整的 Linux Kernel，支援多工、網路、複雜文件系統。
     - **OpenBMC 屬於這一類**。
- **為什麼課程只教 Bare-metal？**
  - 因為 **Bare-metal 是基礎**。即使在 Linux 環境下，寫 Driver (驅動程式) 的人還是要懂 Physical Address、MMIO、Bit Manipulation 這些底層邏輯。Kernel Driver 的職責就是把這些底層髒活 (Physical) 包裝成高層 (Virtual) 給 App 用。你的工作 (Firmware Engineer) 正是這兩者之間的橋樑。

---

---

---

## Chapter 2: Data Types

在嵌入式系統中，選擇正確的資料型態不僅關於「存不存得下」，更關乎「效能」與「正確性」。

### 2.1 Integral Types (整數型態)

#### 為什麼不該用 `int`, `short`, `long`？
標準 C 語言中，`int` 的大小取決於編譯器與處理器架構 (Implementation Defined)。
- **8-bit MCU** (如 AVR/8051): `int` 通常是 **16-bit** (-32768 ~ 32767)。
- **32-bit MCU** (如 ARM Cortex-M): `int` 通常是 **32-bit** (-21億 ~ 21億)。

這種不確定性在移植程式碼 (Porting) 時是災難。例如一個計數器若預期要數到 40,000，在 ARM 上沒問題，在 AVR 上就會 Overflow。

#### 解決方案：`<stdint.h>`
**始終使用固定寬度的整數型態 (Fixed-width Integers)**。這不僅是為了可移植性，更是為了**可讀性**與**記憶體優化**。

| 型態 | 大小 (Bits) | 範圍 (Unsigned) | 用途建議 |
| :--- | :--- | :--- | :--- |
| `uint8_t` | 8 | 0 ~ 255 | 狀態旗標、小型計數器、通訊 Byte |
| `int16_t` | 16 | -32768 ~ 32767 | ADC 讀值 (通常 10~12 bits)、中型迴圈 |
| `uint32_t` | 32 | 0 ~ 42億 | 記憶體位址、時間戳記 (Millis/Micros) |
| `uint64_t` | 64 | 很大 | 高精度計時器 (需注意 32-bit MCU 處理效能) |

> [!TIP]
> **最佳實踐**：
> 1. 除非你需要負數，否則**盡量使用 `unsigned` 型態**。
> 2. 定義自己的型別 (typedef) 或嚴格使用 `stdint.h`，讓閱讀程式碼的人一眼就能知道變數的範圍。

### 2.2 Floating-point Types (浮點數型態)

- **Double-precision (`double`)**: 64-bit IEEE 754。有效位數約 15 位。
- **Single-precision (`float`)**: 32-bit IEEE 754。有效位數約 6 位。

**嵌入式警語**：
1. **FPU 支援度**：許多 Cortex-M4F 只支援 **Single Precision FPU**。若你寫 `double x = 3.14;`，編譯器會自動呼叫軟體模擬函式 (Software Library)，導致效能暴跌 (100x slower)。
2. **常數後綴**：務必在浮點常數後加上 `f` (如 `3.14f`)，否則 C 語言預設會將 `3.14` 視為 `double`。

### 2.3 Lab Analysis: Memory Usage (實作分析)

#### 案例一：Keil MDK (ARM Cortex-M)
在 `Exercise Files/Ch02/02_03/End/memory/main.c` 中，我們看到：
```c
#define SIZE (4000)
uint32_t x;
uint8_t sensor_data[SIZE]; // 4000 bytes

int main(){
    for(x=0; x<SIZE; x++)
      sensor_data[x] = (uint8_t)x;
    while(1);
}
```
*分析*：
- 若將 `sensor_data` 改為 `uint32_t`，記憶體消耗將從 4KB 暴增為 16KB。
- 對於只有 16KB RAM 的 MCU (如 Kinetis KL25Z)，這一個陣列就佔滿了所有記憶體，導致 Stack Overflow 或 Linker Error。

#### 案例二：Arduino (AVR ATmega328P)
在 `Exercise Files/Ch02/02_04/End/types/types.ino` 中：
```c
#define SIZE (1000)
uint32_t x;
uint8_t sensor_data[SIZE]; // 1000 bytes
```
*分析*：
- ATmega328P 只有 2KB RAM。
- `sensor_data` 佔用了 50%。
- 若將 `SIZE` 改為 2000，Arduino IDE 編譯時會警告 "Low memory availability, stability problems may occur"。

### 2.4 Custom Data Types (自定義型態)

#### Structures (`struct`)
用於將相關變數打包。
```c
typedef struct {
    uint8_t  id;
    uint16_t x_axis;
    uint16_t y_axis;
} SensorData_t;
```

**Memory Alignment (記憶體對齊)**：
編譯器為了效能，會在 Struct 成員間插入 Padding bytes，使其對齊 Word boundary (通常是 4 bytes)。
- **危險**：若直接將 Struct `memcpy` 到通訊 Buffer 或 Flash，Padding 會破壞資料格式。
- **解法**：使用 `__attribute__((packed))` (GCC) 強制移除 Padding，但存取速度可能變慢 (Unaligned access)。

#### Unions (`union`)
共用同一塊記憶體空間。
*常用於暫存器存取*：同時以 `uint32_t` (整體) 與 `struct` (個別 Bit) 存取。

```c
typedef union {
    uint32_t R; // Register Content
    struct {
        uint32_t ENABLE : 1;
        uint32_t IRQ    : 1;
        uint32_t MODE   : 2;
        uint32_t _RES   : 28; // Reserved
    } B; // Bits
} ControlReg_t;
```

#### Enumerations (`enum`)
用於定義狀態機 (State Machine) 的狀態，增加程式可讀性。
```c
typedef enum {
    STATE_IDLE,
    STATE_READING,
    STATE_ERROR
} SystemState_t;
```

### 2.5 Chapter 2 Quiz (Official - 官方測驗)

1. **問：為什麼浮點數運算比整數運算花費更多時間？**
   - **答：because floating point numbers have 3 parts, each with its own encoding**
   - *詳解*：處理這三個不同部分來執行運算需要額外的處理工作。

2. **問：什麼是浮點運算單元 (FPU)？**
   - **答：an extra hardware unit that performs floating point operations**
   - *詳解*：這些運算通常是 CPU 指令集擴充的一部分。

3. **問：浮點數的編碼類似於什麼？**
   - **答：Scientific notation (科學記號)**
   - *詳解*：科學記號指定了一個有效數字 (significand) 和一個指數 (exponent)。

4. **問：C 標準函式庫中的哪個標頭檔定義了可攜式的固定大小整數型別？**
   - **答：stdint.h**
   - *詳解*：`stdint.h` 定義了整數型別與巨集。

5. **問：以下何者是 Unsigned 32-bit integer 的可攜式型別？**
   - **答：uint32_t**
   - *詳解*：這正是 `uint32_t` 的含義：Unsigned Integer, 32-bit type。

6. **問：若宣告的變數超過 RAM 大小，Arduino IDE 會停止編譯過程。(True/False)**
   - **答：TRUE**
   - *詳解*：編譯器實際上會停止並告知空間不足。

7. **問：為什麼選擇正確的變數型別很重要？**
   - **答：because we may easily run out of RAM in an embedded system**
   - *詳解*：因為 RAM 在嵌入式系統中是非常稀缺的資源，我們應該精確使用所需的型別。

8. **問：浮點數是如何編碼的？**
   - **答：with a sign, an exponent and a significand, each with its own encoding**
   - *詳解*：IEEE 754 標準定義了基於這三個部分來儲存浮點數的方法。

### 2.6 Chapter 2 Quiz (模擬面試題 / 補充)

1. **問：在 32-bit ARM 系統中，`sizeof(struct { uint8_t a; uint32_t b; })` 的結果是多少？為什麼？**
   - 答：**8 bytes**。`a` 佔 1 byte，但為了讓 `b` 對齊 4-byte boundary，編譯器在 `a` 後面插入 3 bytes padding (填充位元)。
   
2. **問：什麼是 Endianness (位元組順序)？它如何影響 `union` 的使用？**
   - 答：**Big-endian** (高位在低位址) vs. **Little-endian** (低位在低位址)。
   - 若使用 `union` 解析通訊協定的封包 (如將 `uint8_t bytes[4]` 轉為 `uint32_t`)，必須確認 MCU 與通訊協定的 Endianness 是否一致，否則數值會顛倒。

3. **問：若程式碼中大量使用 `double` 變數在 Cortex-M4F (只支援單精度 FPU) 上運算，會有什麼後果？**
   - 答：M4F 的 FPU 只能加速 `float`。`double` 運算仍會退化為軟體模擬，導致效能低落。應將 `double` 改為 `float` 並加上 `f` 後綴 (如 `3.14f`)。

---

---

## Chapter 3: Bit Manipulation

嵌入式開發的核心技能。因為硬體暫存器 (Registers) 通常是以 Bit 為單位控制的，我們必須精通如何「只改變特定 Bit 而不影響其他 Bit」。

### 3.1 Masking (遮罩技術)

**Mask** 是一個用來選擇特定位元的數值。

#### 基本操作公式
假設 `REG` 是 8-bit 暫存器，我們想操作第 3 個 Bit (Bit 2, 0-indexed)：

1. **Set Bit (設為 1)**: 使用 OR (`|`)
   ```c
   REG |= (1 << 2); // 0000 0100
   ```
   *原理*：任何數 OR 1 都是 1，OR 0 維持原值。

2. **Clear Bit (設為 0)**: 使用 AND (`&`) 與 NOT (`~`)
   ```c
   REG &= ~(1 << 2); // AND 1111 1011
   ```
   *原理*：任何數 AND 0 都是 0，AND 1 維持原值。

3. **Toggle Bit (反轉)**: 使用 XOR (`^`)
   ```c
   REG ^= (1 << 2);
   ```
   *原理*：任何數 XOR 1 會反轉，XOR 0 維持原值。

4. **Check Bit (檢查)**: 使用 AND (`&`)
   ```c
   if (REG & (1 << 2)) {
       // Bit is set
   }
   ```

### 3.2 Lab Analysis: Masking (實作分析)

#### 案例一：Arduino Port Manipulation (AVR)
在 `Exercise Files/Ch03/03_02/masking/masking.ino` 中，我們看到如何直接操作硬體暫存器來控制 LED，而不是使用效能較差的 `digitalWrite`。

```c
#define MASK(x) ((unsigned char) (1<<(x)))

void setup(){                
  // DDRB (Data Direction Register Port B)
  // Set Bit 5 to 1 (Output)
  DDRB |= MASK(5);
}
  
void loop(){
  // PORTB (Data Register Port B)
  PORTB |= MASK(5);  // Turn LED on (Set Bit)
  delay(500);
  PORTB &= ~MASK(5); // Turn LED off (Clear Bit)
  delay(500);
}
```
*分析*：
- **巨集 (Macro)**: `#define MASK(x) ((unsigned char) (1<<(x)))` 是一個標準且高效的寫法。編譯器會在 Compile-time 計算出常數 (如 `MASK(5)` 變成 `0x20`)，不會耗費 Runtime 資源。
- **Direct Register Access**: 直接操作 `DDRB` 與 `PORTB`，比 Arduino 的 `pinMode()` 和 `digitalWrite()` 快上數十倍。

### 3.3 Bit Fields (位元欄位)

#### 實戰案例：Complex Register Map
在 `Exercise Files/Ch03/03_04/bit fields/main.c` 中，展示了一個非常經典的 **Union + Struct** 設計模式，用於定義複雜的硬體暫存器。

```c
typedef union {
  uint8_t Byte; // 允許以 Byte 為單位存取 (例如: reg.Byte = 0xFF)
  struct {
    uint8_t PS0       :1;  
    uint8_t PS1       :1;  
    uint8_t PS2       :1;  
    // ...
    uint8_t TOF       :1;  // Timer Overflow Flag 
  } Bits; // 允許以 Bit 為單位存取 (例如: reg.Bits.TOF = 1)
  struct {
    uint8_t PS        :3;  // 3-bit field
    uint8_t CLKS      :2;  // 2-bit field
    uint8_t           :3;  // Padding/Reserved
  } MergedBits; // 允許存取多位元欄位
} MultiFieldByte;
```

**Memory Layout**:
- 所有的 `Byte`, `Bits`, `MergedBits` 共用同一個 8-bit 記憶體空間。
- 修改 `reg.Bits.PS0 = 1`，會同時改變 `reg.Byte` 的數值。

**Pros (優點)**：
- **可讀性極高**：`reg.Bits.TOF = 1` 比 `reg |= (1<<7)` 清楚得多。
- **自動 Masking**：編譯器會自動生成 Masking 程式碼。例如寫入 `reg.MergedBits.PS = 5`，編譯器會自動處理 Masking，確保不寫到其他 Bits。

**Cons (缺點/風險)**：
- **Implementation Dependent**: Bit 的順序 (LSB first vs MSB first) 未定義。
- **Concurrency**: 雖然看起來像操作單一欄位，但 CPU 讀寫時是 Read-Modify-Write 整個 Byte，若無保護 (Atomic/Critical Section)，在中斷中不安全。

### 3.4 Chapter 3 Quiz (Official - 官方測驗)

1. **問：為什麼 Bit Fields (位元欄位) 通常實作在 Union 中的 Struct 內？**
   - **答：because it's often useful to access the whole byte, each bit field, or each individual bit, in the same application**
   - *詳解*：使用 Union 允許同一個記憶體位置被存取為完整的 Byte (或 Word)，或是個別的位元欄位，提供了操作暫存器或資料結構的靈活性。

2. **問：Bit Fields 是如何定義的？**
   - **答：as structs, where each field specifies the number of bits it implements**
   - *詳解*：在 C 語言中，Bit Fields 定義為 `struct` 的成員，成員名稱後接冒號和佔用的位元數。

3. **問：哪一行程式碼可以清除 I/O 暫存器 PORTC 的第 3 個 Bit (Bit 3)？**
   - **答：PORTC &= ~MASK(3);**
   - *詳解*：要清除特定 Bit 而不影響其他 Bit，使用 Bitwise AND (`&`) 搭配 Mask (目標 Bit 為 0)。將單一位元 Mask 取反 (`~`) 即可產生此模式 (例如 `~00001000` 變為 `11110111`)。

4. **問：使用巨集函式 `MASK(x)`，下列哪個運算式能產生正確的 Mask 來設定 Bits 0, 3 和 4？**
   - **答：MASK(0) | MASK(3) | MASK(4)**
   - *詳解*：要建立多個 Bits 的 Mask，使用 Bitwise OR (`|`) 來組合個別的 Bit Mask。

5. **問：什麼是 Binary Mask (二進位遮罩)？**
   - **答：a binary number used to modify or read one or more bits of a variable of the same bit-length**
   - *詳解*：Mask 是一個數值，搭配位元運算子使用，用來隔離、反轉、清除或設定暫存器或變數中的特定 Bits。

### 3.5 Chapter 3 Quiz (模擬面試題 / 補充)

1. **問：寫一行程式碼，將整數 `x` 的第 `n` 個 Bit 清除 (Clear to 0)。**
   - 答：`x &= ~(1U << n);`

2. **問：解釋 Read-Modify-Write (RMW) 的風險。**
   - 答：當你在主程式做 `REG |= 0x01` 時，CPU 其實是做「讀出 REG -> 修改數值 -> 寫回 REG」三個動作。若在「修改數值」時發生中斷 (ISR)，且 ISR 也修改了同一個 REG 的其他 Bit，等 ISR 結束返回，主程式會把舊的 REG 值(加上修改後的bit)寫回，導致 ISR 的修改被覆蓋 (Lost Update)。
   - *解法*：使用 Atomic Instruction 或暫時關閉中斷 (Critical Section)。

3. **問：使用 Union 定義暫存器時，為什麼常看到 `volatile` 關鍵字？**
   - 答：因為暫存器的值可能由硬體改變 (例如 Timer 數值增加、UART 收到資料)。若變數定義為 `volatile MultiFieldByte reg;`，編譯器就會知道每次存取都要真的去讀寫記憶體，而不能優化成快取值。

---

---

## Chapter 4: Qualifiers

### 4.1 The `const` Qualifier (唯讀修飾字)

在嵌入式系統中，`const` 不僅是「不可修改」的承諾，更是**記憶體配置**的重要指示。

#### `const` vs. `#define` (面試必考)
影片中特別比較了這兩者。雖然 `#define` (Macro) 也可以定義常數，但 `const` 變數通常是更好的選擇。

| 特性 | `#define` (Macro) | `const` Variable |
| :--- | :--- | :--- |
| **處理階段** | Preprocessor (文字替換) | Compiler (編譯階段) |
| **Type Checking** | 無 (容易出錯) | 有 (安全) |
| **Scope** | 無 (Global) | 有 (Follow scope rules) |
| **Debugging** | 難 (Symbol table 無資訊) | 易 (Symbol table 有資訊) |
| **Memory** | 每次使用都展開 (Code bloating) | 通常只存一份 (Reference) |

#### Flash vs. RAM Storage
- **標準行為**: 宣告 `const` 通常會讓 Linker 把變數放在 **Read-only Memory (Flash)**，節省寶貴的 RAM。
- **Arduino/AVR 特例**: 影片中提到，在 AVR 架構下，單純用 `const` 仍可能佔用 RAM。若要強制放在 Flash，需使用特殊修飾字 (如 `PROGMEM`)。

**指標與 const 的四種變化 (面試必考)**：
1. `int * ptr;` : 普通指標。
2. `const int * ptr;` : 指標指向的**資料**不可變 (Read-only data)。
3. `int * const ptr;` : **指標本身**不可變 (Fixed pointer)。
4. `const int * const ptr;` : 兩者都不可變。

### 4.2 The `volatile` Qualifier (易變修飾字)

這是嵌入式 C 最重要的關鍵字。它告訴編譯器：「**這個變數可能會被程式以外的因素 (Hardware, ISR) 改變，請不要對它做任何優化**」。

#### 什麼時候需要 `volatile`？
1. **Memory-mapped Peripheral Registers**: 硬體暫存器。
2. **Global variables modified by an ISR**: 中斷服務常式修改的變數。
3. **Global variables shared by multiple tasks**: 多執行緒共享變數。

### 4.3 Lab Analysis: Volatile & Optimization
以 `Exercise Files/Ch04/04_04/volatile/volatile.ino` 為例：

#### The Problem (消失的迴圈)
```c
void wait(){
  uint32_t x=100000;
  while(x)
    x--;
}
```
當編譯器開啟優化 (Optimization) 時，它會發現：
1. `x` 是一個 Local Variable。
2. `x` 的值在迴圈後沒有被使用。
3. `x--` 沒有任何 Side Effect (如寫入硬體)。
**結論**：這個迴圈是廢話，**直接刪除**。

#### The Fix (使用 volatile)
```c
void wait(){
  // 告訴編譯器：x 可能被外部修改，或存取 x 有副作用
  volatile uint32_t x=100000; 
  while(x)
    x--; // 編譯器被迫保留這個迴圈
}
```
> [!NOTE]
> 雖然用 `volatile` 可以解決被優化掉的問題，但**這不是實現延遲的好方法** (時間不精確，且佔用 CPU)。正規作法是使用 Hardware Timer。

### 4.4 Chapter 4 Quiz (Official - 官方測驗)

1. **問：為什麼編譯器會移除一個耗時的迴圈 (Time-consuming loop)？**
   - **答：Because the end result may be that the iterating variable is changed to one final value that is never used later. This can be optimized out for the sake of speed.**
   - *詳解*：編譯器會透過移除 "Dead Code" (對輸出無影響的程式碼) 來優化。一個只會增加變數但沒人使用的迴圈是此類優化的首選目標。

2. **問：為什麼編譯器只產生 If-else 語句中其中一條路徑 (True/False) 的 Assembly code？**
   - **答：Because the compiler is certain that it's only possible to execute one path. This optimization saves space by not implementing the impossible path.**
   - *詳解*：若 If 的條件在編譯時期已知，編譯器會 "Prune" (修剪) 永遠不會執行的分支，以節省記憶體並提升效能。

3. **問：以下宣告中 `const` 的意義為何？ `const uint8_t a = 25;`**
   - **答：The variable a will not be changed by any part of the code in this scope.**
   - *詳解*：`const` 修飾字告訴編譯器，該變數在初始化後不應被改變。任何嘗試修改它的行為都會導致編譯錯誤。

4. **問：使用 `#define` 定義常數有什麼缺點？**
   - **答：The compiler can't enforce correctness of data types for macros.**
   - *詳解*：Macro 只是 Preprocessor 進行的簡單文字替換。因為它們不是真正的變數，沒有型別，所以編譯器無法檢查其使用是否正確。

5. **問：變數怎麼可能被「程式碼以外的東西」改變？**
   - **答：An input/output device may modify the variable.**
   - *詳解*：在嵌入式系統中，變數常映射到硬體暫存器 (Memory-Mapped I/O)。外部硬體事件可以獨立於 CPU 程式流程改變這些暫存器的值。

6. **問：以下宣告中 `volatile` 的意義為何？ `volatile uint8_t a;`**
   - **答：The variable a may be changed at any time by something other than this code.**
   - *詳解*：`volatile` 修飾字告訴編譯器，該變數的值可能被外部 (如硬體或中斷) 改變。這能防止編譯器優化掉對該變數的「看似多餘」的讀寫操作。

### 4.5 Chapter 4 Quiz (模擬面試題 / 補充)

1. **問：一個變數可以同時是 `const` 和 `volatile` 嗎？請舉例。**
   - 答：**可以**。
   - 例：**唯讀的硬體狀態暫存器 (Read-only Status Register)**。
   - `volatile`: 數值會隨硬體狀態改變，不可優化 (每次都要重讀)。
   - `const`: 程式碼不應該去寫入它 (編譯時期保護)。
   - 宣告：`volatile const uint32_t * port_status_reg;`

2. **問：Refactoring 時，為什麼建議用 `const` 取代 `#define`？**
   - 答：`const` 有型別檢查 (Type Safety)，且在 Debugger 中看得到符號與數值。`#define` 只是文字替換，容易產生難以除錯的副作用，且可能導致 Code Bloating。

---

---

## Chapter 5: Function Alternatives (函式替代方案)

在嵌入式系統中，「呼叫函式 (Function Call)」是有代價的：
1. **Push arguments to stack** (參數堆疊)
2. **Jump to address** (跳轉)
3. **Execute code**
4. **Pop return value**
5. **Jump back**
Context Switching 與 Branch Prediction 失誤都可能影響效能。本章介紹三種替代方案。

### 5.1 Lookup Tables (LUT, 查表法)

**核心概念**：用「空間 (Memory)」換取「時間 (Time)」。
原本需要複雜計算 (如 sin, cos, log) 的數值，預先算好存成 Constants Library (陣列)。

#### Lab Analysis: LUT vs Function
案例來自 `Exercise Files/Ch05/05_04/End/LUT/LUT.ino` 與 `05_05/LUT/main.c`。

**1. Arduino Implementation (AVR)**:
```c
// 定義在 Flash (PROGMEM) 的 LUT，避免佔用 RAM
const double log_LUT[256] PROGMEM = {-1.0E30, 0.0, 0.301, ...};
```
- **量測結果**：
  - 使用 `log10()` 函式：約 **4000 ms** (4秒)。
  - 使用 `log_LUT[]` 查表：約 **32 ms**。
  - **加速比 (Speedup)**：**~125 倍**！
- **代價**：增加了 18.8% 的 Program Memory (Flash) 使用量。

**2. Keil Implementation (ARM Cortex-M4)**:
```c
const double log_LUT[256] = {...}; // ARM compiler 預設 const 就在 Flash
```
- **量測結果**：加速比約 **130 倍**。

> [!TIP]
> **結論**：若你的 MCU Flash 空間足夠，且需要頻繁執行複雜運算 (Log, Trig, Sqrt)，**LUT 是最強大的優化手段**。

### 5.2 Macro Functions (`#define`)

利用 Preprocessor 進行文字替換，完全消除 Function overhead。

```c
#define MAX(a, b) ((a) > (b) ? (a) : (b))
```
- **Pros**: 速度快 (Inline)，無 Call overhead。
- **Cons**:
    - **No Type Checking**: 若傳入 pointer 可能編譯過但執行錯誤。
    - **Side Effects**: `MAX(x++, y)` 會導致 `x` 被加兩次。
    - **Debug**: 無法設斷點 (Breakpoint)。

### 5.3 Inline Functions (`inline`)

C99 標準引入的關鍵字，試圖結合 Function 的安全性與 Macro 的速度。

#### Lab Analysis: Keil Inline
案例來自 `Exercise Files/Ch05/05_06/inline/main.c`。

```c
// 強制編譯器 Inline (GCC/ARM style)
uint32_t wsum(uint32_t x, uint32_t y) __attribute__((always_inline));

uint32_t wsum(uint32_t x, uint32_t y){
  return (2 * x - y);
}
```

- **量測結果**：
  - Regular Function Call: 251 ms
  - Inline Function Call: 169 ms
  - **加速比**：約 **1.5 倍**。
  
> [!NOTE]
> **Inline 的現實**：
> 1. 加速效果遠不如 LUT (1.5x vs 125x)。
> 2. `inline` 只是給編譯器的「建議」。編譯器若覺得函式太長，會直接忽略你的建議 (除非用 `__attribute__((always_inline))`)。
> 3. 風險：Code Size 可能暴增 (Code Bloat)。

### 5.4 Chapter 5 Quiz (Official - 官方測驗)

1. **問：呼叫 `square(5-2)` 會評估出什麼結果？假設 Macro 定義為：`#define square(x) x*x`**
   - **答：-7**
   - *詳解*：問題在於 Macro 缺少括號。沒有括號的情況下，`5-2*5-2` 根據運算子優先順序被評估為 `5-(2*5)-2 = -7`。

2. **問：考慮部分定義的 Lookup Table (LUT) 與函式。如何實作該函式，使其能使用僅有 256 個點的 LUT，對 0 到 255 範圍內的浮點數回傳良好的 Log 近似值？**
   - **答：The function may take the argument, retrieve its two closest points from the lookup table, and use them to calculate the output by interpolation.**
   - *詳解*：內插法 (Interpolation) 允許函式估算儲存在 LUT 中離散點之間的數值，提升精確度。

3. **問：在下列 LUT 宣告中，為什麼 `const` 修飾字至關重要？ `const uint32_t squares[256] = {0,1,4,9,16,25,...};`**
   - **答：Because if it's missing, the array will be allocated twice: The actual array in RAM, and the initializing data in ROM.**
   - *詳解*：在許多嵌入式系統中，已初始化的陣列會在啟動時從 ROM 複製到 RAM。使用 `const` 允許編譯器將資料僅存在 ROM 中，節省寶貴的 RAM 空間。

4. **問：Inline functions 通常能改善嵌入式系統的執行速度。關於此加速，下列敘述何者正確？**
   - **答：The speedup is modest (less than twice as fast) but significant enough for time-critical applications.**
   - *詳解*：Inline 透過移除 Function Call 的開銷 (如 Push/Pop stack, Jumping) 來提升效能，但這種改善通常是漸進式的，而非戲劇性的。

5. **問：在 Keil MDK (STM32F303K8) 中，`const` 是否足以將 LUT 放進 ROM？**
   - **答：Yes.**
   - *詳解*：對於 ARM 編譯器 (如 Keil MDK)，將 `const` 應用於全域或靜態陣列足以將其放入 Flash (ROM)。

6. **問：在 Arduino IDE (Arduino UNO) 中，`const` 是否足以將 LUT 放進 ROM？**
   - **答：No, you need to add the PROGMEM modifier.**
   - *詳解*：在 AVR 架構 (如 Arduino UNO) 上，`const` 僅防止修改，仍會將資料放入 RAM。必須使用 `PROGMEM` 關鍵字強制將資料留在 Flash Memory。

7. **問：Inline functions 總是優於普通函式，因為 Inlining 會產生較小的程式。(True/False)**
   - **答：FALSE**
   - *詳解*：Inlining 通常會增加程式大小，因為函式的程式碼會在每個呼叫點被複製 (Duplicated)，而不是只有一個實體。

8. **問：撰寫 Macro functions 時，哪項檢查最重要？**
   - **答：making sure parentheses are correctly used**
   - *詳解*：因為 Macro 進行簡單的文字替換，在參數和整個表達式周圍使用括號至關重要，以確保運算子優先順序不會導致意外結果。

### 5.5 Chapter 5 Quiz (模擬面試題 / 補充)

1. **問：什麼情況下 `inline` 關鍵字會失效 (編譯器拒絕 inline)？**
   - 答：
     1. 函式太過龐大複雜。
     2. 包含遞迴呼叫 (Recursion)。
     3. 函式位址被取用 (Pointer to function)，導致必須產生一個實體函式。

2. **問：在 AVR (Arduino) 上使用 LUT 時，為什麼要加 `PROGMEM`？**
   - 答：AVR 是 Harvard Architecture，資料與程式空間分開。標準 C 的 `const` 變數仍會被 Copy 到 RAM。使用 `PROGMEM` 告訴編譯器：「將此資料留在 Flash，並生成特殊的 Flash 讀取指令 (LPM)」，從而節省寶貴的 SRAM。

3. **問：請比較 Macro 與 Inline Function。**
   - 答：
     - **Macro**: Preprocessor 處理，無 Type Check，有 Side Effect 風險，無法 Debug。
     - **Inline**: Compiler 處理，有 Type Check，參數只評估一次 (無 Side Effect)，可 Debug (視 IDE 而定)。
     - **結論**: 總是優先使用 `static inline`，除非該編譯器不支援 C99。

---

---

## Chapter 6: Floating-Point Unit Alternatives

處理小數運算時，初學者常直接用 `float`。但在沒有 FPU 的 MCU 上，這是效能殺手。

### 6.1 Identify the Problem (軟體浮點數的代價)

當你的 MCU (如 ARM Cortex-M0/M3) **沒有 FPU (Floating Point Unit)**，卻寫了 `float a = b * c;`：
- 編譯器會插入 **Runtime Library** (如 `__aeabi_fmul`) 來模擬浮點運算。
- **後果**：
  - **Memory Impact**: 需要額外的 Flash 存放 Library，額外的 RAM 進行運算。
  - **Performance Drop**: 硬體一個指令 (1 cycle) 可完成的事，軟體模擬需要 **100~500 cycles**。
  - **Energy**: CPU 長時間滿載，功耗大增。

### 6.2 Fixed-point Math (定點數運算)

定點數是**將小數放大成整數**來運算，最後輸出時再縮小回來。

#### 原理: Q-Format Encoding
在電腦中，我們通常用 **2 的冪次方**作為 Scale Factor (利用 Shift 運算取代乘除)。
例如 **Q7.8** 格式 (16-bit 整數)：
- **7 bits** Integer part.
- **8 bits** Fractional part.
- **Scale Factor** = $2^8 = 256$。

```c
// 範例：Q7.8
// 數值 1.5 -> 1.5 * 256 = 384 (0x0180)
```

#### Lab Analysis: Fixed Point Performance (Arduino)
案例來自 `Exercise Files/Ch06/06_03/fixed/fixed.ino`。使用 `FixedPoints.h` Library。

```c
#ifdef USE_FIXED_POINT
  SQ7x8 a, b, c; // Signed Q7.8 type
#else
  volatile float a, b, c;
#endif
```

- **量測結果**：
  - Software Float (`float`): **2.58 seconds**。
  - Fixed Point (`SQ7x8`): **0.34 seconds**。
  - **加速比 (Speedup)**：**~7.5 倍**！

### 6.3 Hardware FPU (硬體浮點運算單元)

若使用支援 FPU 的 MCU (如 STM32F3 / Cortex-M4F)：

#### Lab Analysis: FPU Enable vs Disable
案例來自 `Exercise Files/Ch06/06_04/fpu/main.c`。

1. **FPU Disable (Software Float)**:
   - 組合語言看到 `BL __aeabi_fmul` (Branch with Link to function)。
   - 執行時間：**1.2 seconds**。

2. **FPU Enable (Hardware Float)**:
   - 組合語言看到 `vmov.f32`, `vmul.f32` (FPU 專用指令，S0-S31 暫存器)。
   - 執行時間：**0.246 seconds**。
   - **加速比**：**4.86 倍**。

> [!WARNING]
> **陷阱**：Cortex-M4F 的 FPU 通常只支援 **Single Precision (`float`)**。若你不小心寫了 `double` (例如 `3.14` 沒加 `f`)，編譯器仍會退化成軟體模擬！

### 6.4 Chapter 6 Quiz (Official - 官方測驗)

1. **問：假設你在 STM32F303K8 (Keil MDK) 上開發，並啟用了 FPU。若程式中包含以下變數的密集浮點運算，會發生什麼事？**
   ```c
   float a,b;
   double x;
   ```
   - **答：Runtime functions for doubles would perform most of the calculations.**
   - *詳解*：STM32F303K8 配備單精度 FPU。雖然 `float` (32-bit) 可直接由 FPU 處理，但 `double` (64-bit) 需要透過 Runtime Library 進行軟體模擬。若計算混合了 float 和 double，float 會被提升為 double，強迫運算在軟體中執行，而非 FPU。

2. **問：在沒有 FPU 的嵌入式系統中，為什麼定點數 (Fixed-point) 運算比浮點數運算快？**
   - **答：because fixed-point arithmetic is almost the same as integer arithmetic**
   - *詳解*：定點數運算利用了微控制器的原生整數運算單元 (ALU)，該單元高度優化且快速。無 FPU 的浮點運算必須透過複雜的軟體程式來處理正負號、指數和有效位數，需要更多 CPU Cycles。

3. **問：使用 32-bit 單精度浮點數時，為什麼 `100,000,000 + 1` 的結果仍是 `100,000,000`？**
   - **答：because the precision of a 32-bit float covers 7 decimal places or less**
   - *詳解*：標準 32-bit float 提供約 7 位十進制有效數字。1億 ($10^8$) 有 9 位有效數字。當加上 1 時，第 9 位數值會因為 32-bit 浮點格式的精度限制與捨入而被捨棄。

4. **問：在無 FPU 的微控制器上使用浮點數時，程式設計師必須自己追蹤所有浮點變數的 Sign, Exponent 和 Significand。(True/False)**
   - **答：FALSE**
   - *詳解*：程式設計師無需手動管理浮點數的內部結構。編譯器與 Runtime Software Libraries 會根據 IEEE 754 標準自動處理這些細節，允許程式設計師使用標準的 float 型別與運算子。

### 6.5 Chapter 6 Quiz (模擬面試題 / 補充)

1. **問：為什麼銀行系統(Banking)不適合用 `float`，而要用定點數或整數？**
   - 答：因為 `float` (IEEE 754) 精度有限，會有 "Big number eating small number" 的問題 (例如 `100,000,000.0f + 1.0f` 可能仍等於 `100,000,000.0f`)。定點數擁有固定的解析度 (Resolution)。

2. **問：在 Cortex-M4F 上，如何確保你的浮點運算真的用到 FPU？**
   - 答：
     1. 編譯選項要開啟 FPU (如 `-mfloat-abi=hard -mfpu=fpv4-sp-d16`)。
     2. 程式碼中所有浮點常數都要加 `f` (如 `1.23f`)，確保是 `float` 而非 `double`。

3. **問：Q15.16 格式的定點數，解析度(最小單位)是多少？**
   - 答：$1 / 2^{16} \approx 0.0000152$。

---

## Final Exam (模擬期末考 - 綜合面試題)

以下題目旨在測試你是否具備挑戰外商韌體工程師的能力。

### 題目 1: 記憶體映射 (Memory Mapped I/O)
**Q**: 請定義一個巨集 (Macro)，存取位址 `0x40001000` 的 32-bit 硬體暫存器，並將其第 3 個 bit (Bit 2) 設為 1。

<details>
<summary>參考解答</summary>

```c
#define REG_ADDR  0x40001000
#define REG_PTR   ((volatile uint32_t *)REG_ADDR)

void set_bit_2(void) {
    *REG_PTR |= (1U << 2);
}
```
*Key Points*: `volatile`,Type casting, Pointer deference, Bitwise OR.
</details>

### 題目 2: 中斷安全性 (Interrupt Safety)
**Q**: 為什麼在主程式讀取一個由 ISR 更新的 16-bit 變數 (在 8-bit MCU 上) 是不安全的？

<details>
<summary>參考解答</summary>

因為在 8-bit MCU 上，讀取 16-bit 變數需要兩次 Memory Access (High byte & Low byte)。若在讀完 High byte 後發生中斷，且 ISR 修改了該變數，等 ISR 返回後再讀 Low byte，主程式就會讀到「新舊混合」的錯誤數值 (Tearing)。
*解法*：讀取前暫時關閉中斷 (Atomic Access)。
</details>

### 題目 3: 程式碼優化與 volatile
**Q**: 以下程式碼有什麼問題？如何修正？
```c
void delay(int count) {
    while (count > 0) {
        count--;
    }
}
```

<details>
<summary>參考解答</summary>

問題：編譯器若開啟優化，會發現這個迴圈沒有做任何有意義的事 (No side effect)，可能會直接把整個 `while` 迴圈刪除，導致延遲失效。
修正：
```c
void delay(volatile int count) { // count 宣告為 volatile，或在迴圈內加 assembly NOP
    while (count > 0) {
        count--;
    }
}
```
或使用硬體 Timer 才是正解。
</details>

### 題目 4: 指標算術
**Q**: `int *p = (int *)0x1000; p++;` 執行後，`p` 的值是多少？
- (A) 0x1001
- (B) 0x1002
- (C) 0x1004

<details>
<summary>參考解答</summary>

(C) 0x1004。
因為 `p` 是 `int*`，在 32-bit 系統中 `int` 通常是 4 bytes。指標加 1 代表前進「一個元素的大小」。
</details>

---
**Congratulations!** 你已完成 C Programming for Embedded Applications 課程的模擬學習。建議將上述程式碼範例在你的開發板或 Simulator 上實作一遍。
