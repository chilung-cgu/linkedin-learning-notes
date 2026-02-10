# 進階 C 語言程式設計筆記 (深度完整版)
## Advanced C Programming: Optimize Performance and Efficiency - Go Beyond the Basics

這份筆記是根據 **LinkedIn Learning** 課程 "Advanced C Programming: Optimize Performance and Efficiency" 的 **完整 Transcript 逐字稿** 與 **Exercise Files 原始碼** 所整理的「終極完整版」。
經過完整的課程內容稽核 (Audit)，確認本課程共包含 5 個章節以及 Introduction 與 Conclusion。本筆記已將所有細節、程式碼範例、挑戰題目 (Challenges) 與解答 (Solutions) 整合，目標是讓讀者**完全無需觀看影片**，即可掌握 100% 的課程精髓。

---

## Introduction: Course Overview (課程簡介)

講師 **Dan Gookin** 設計本課程的目的是幫助具備基礎 C 語言能力的開發者，進一步提升技能 (Hone your programming Kung Fu)。

*   **先備知識**：需要具備 C 語言基礎。如果你是完全的新手，建議先修習基礎課程。
*   **開發環境**：課程中使用 **Code::Blocks** IDE，但你可以使用任何你習慣的 IDE (VS Code, Xcode, CLion 等)。
*   **Exercise Files 使用指南**：
    *   檔案命名規則：`Chapter-Movie-Topic.c` (例如 `01-01_assignments.c`)。
    *   講師強烈建議：不要只是看，要親自動手修改這些檔案 (Mess with the files) 來驗證觀念。

---

## Chapter 1: Programming: Weird-Symbol Roundup (程式設計：特殊符號總覽)

程式語言為了簡潔與效率，常使用許多特殊的符號來代表運算。本章節深入剖析 C 語言中常見但容易被誤解的賦值運算子與三元運算子。

### 1-1. Assignment Operators (賦值運算子)

**核心概念 (Transcript 重點)**：
*   所有賦值運算子的目的都是**將一個值填入變數中** (fill a variable with a value)。
*   最基本的賦值運算子是等號 (`=`)。它將右側的值（Literal、變數、運算結果或函數回傳值）賦予左側的變數。
*   **先決條件**：使用賦值運算子（特別是複合運算子）之前，變數**必須已經初始化**。如果對未初始化的變數進行運算，結果將是不可預測的 (Garbage in, garbage out)。

**複合賦值運算子 (Shortcut/Compound Assignment Operators)**：
當我們需要「基於變數當前的值進行修改」時（例如 `height = height * 2`），C 語言提供了更高效的縮寫寫法。

| 運算子 | 意義 | 範例 | 等同於 |
| :--- | :--- | :--- | :--- |
| `+=` | 加後賦值 | `v += 20` | `v = v + 20` |
| `-=` | 減後賦值 | `v -= 2` | `v = v - 2` |
| `*=` | 乘後賦值 | `v *= 4` | `v = v * 4` |
| `/=` | 除後賦值 | `v /= 3` | `v = v / 3` |
| `%=` | 取餘數後賦值 | `v %= 7` | `v = v % 7` |

**⚠️ 重要細節 (講師特別強調)**：
運算子的順序非常重要，必須是**數學運算子在前，等號在後**。
*   `v += 20`：將 `v` 的值加上 20。
*   `v = +20`：**危險！** 這只是將 `+20`（正 20）賦值給 `v`，完全覆蓋了原本的值。這是初學者常犯的錯誤。

**程式碼分析 (Exercise File: `01-01_assignments.c` 完整版)**：
```c
#include <stdio.h>

int main()
{
    int v = 0;  // 初始化變數為 0

    printf("v equals %d\n", v);           // 輸出: v equals 0

    /* Addition: 加法 */
    v = v + 20;  // 可改寫為 v += 20;
    printf("v + 20 equals %d\n", v);      // 輸出: v + 20 equals 20

    /* Subtraction: 減法 */
    v = v - 2;   // 可改寫為 v -= 2;
    printf("v - 2 equals %d\n", v);       // 輸出: v - 2 equals 18

    /* Division: 除法 (注意整數除法會截斷) */
    v = v / 3;   // 可改寫為 v /= 3;  (18/3 = 6)
    printf("v / 3 equals %d\n", v);       // 輸出: v / 3 equals 6

    /* Multiplication: 乘法 */
    v = v * 4;   // 可改寫為 v *= 4;  (6*4 = 24)
    printf("v * 4 equals %d\n", v);       // 輸出: v * 4 equals 24

    /* Modulus: 取餘數 */
    v = v % 7;   // 可改寫為 v %= 7;  (24 % 7 = 3)
    printf("v %% 7 equals %d\n", v);      // 輸出: v % 7 equals 3
    // 注意: printf 中的 %% 才會印出 % 符號

    return(0);
}
```
**執行結果**：
```
v equals 0
v + 20 equals 20
v - 2 equals 18
v / 3 equals 6
v * 4 equals 24
v % 7 equals 3
```

### 1-2. The Ternary Operator (三元運算子)

**核心概念 (Transcript 重點)**：
*   C 語言中的運算子依據操作對象的數量分類：
    *   **Unary (一元)**：操作一個變數 (e.g., `sizeof`, `(cast)`, `++`)。
    *   **Binary (二元)**：操作兩個變數 (e.g., `+`, `-`, `==`)。
    *   **Ternary (三元)**：操作三個變數。C 語言中只有一個三元運算子：`? :`。
*   它是一種**簡寫的 If-True 決策結構** (shorthand if-true decision)。
*   有些工程師因為它「令人愉快地隱晦 (delightfully cryptic)」而喜愛它，也有人因為可讀性差而避免使用。

**語法與邏輯**：
```c
Condition ? Expression1 : Expression2;
```
1.  **評估** `Condition`。
2.  若結果為 **True** (非零)，執行並回傳 `Expression1`。
3.  若結果為 **False** (零)，執行並回傳 `Expression2`。

**講師建議**：
> "Know what it is and how it works, recognize it in the code, but if you don't have to use it, you don't have to if you don't want to. If you find a place for it, great. Otherwise, if-else works just fine."
> (知道它是什麼、能看懂它，但不需要強迫自己使用。如果覺得適合就用，不然 `if-else` 也很好。)

**程式碼分析 - 基礎應用 (`01-04_ternary1.c`)**：
傳統 `if-else` 解決 "Max Problem"：
```c
if( a > b )
    larger = a;
else
    larger = b;
```
轉換為三元運算子：
```c
larger = (a > b) ? a : b;
```
這樣的轉換讓程式碼更精簡，直接表達「`larger` 的值取決於 `a` 與 `b` 的大小關係」。

**程式碼分析 - 函數內應用 (`01-04_ternary2.c`)**：
三元運算子不僅僅是用來賦值，它是一個**表達式 (Expression)**，會有回傳值。因此可以直接嵌入在函數呼叫中。

```c
// 根據這一行決定印出 'A' 還是 'B'
printf("In this case, variable %c is greater.\n",
        (a > b) ? 'A' : 'B'
      );
```
這避免了為了顯示不同字元而寫兩個 `printf` 的冗餘。

**程式碼分析 - 巢狀應用 (`01-04_ternary3.c`)**：
雖然可以巢狀使用 (Nested ternary operators)，但容易變得非常複雜 (get real hairy)。

```c
classification = ( ( age < 19 ) ? "kid" :
    ( age < 65 ? "adult" :
      "geezer" ));
```
**邏輯解析**：
1.  `(age < 19)` ? 是 -> "kid"。
2.  否 -> 進入第二層判斷：`(age < 65)` ? 是 -> "adult"。
3.  否 -> "geezer"。
**注意**：為了避免編譯器誤判優先權或人類閱讀困難，**務必使用括號**將每一層邏輯包起來 (make ample use of parentheses)。

### 1-3. Challenges & Solutions (本章挑戰與解答)

**Challenge 1: Use an Assignment Operator**
*   **目標**：寫一個程式，要求使用者輸入數值，將該數值乘以 5，再除以 3，並顯示每次運算的結果。必須使用複合賦值運算子。
*   **解答分析**：
    *   **變數選擇**：講師建議使用 floating point (`float` 或 `double`)，因為除法運算 (`/3`) 很容易產生小數。如果用 `int` 會導致截斷誤差。
    *   **格式化輸出**：使用 `%.1f` 來將輸出限制為小數點後一位，增加可讀性。
    *   **運算子應用**：
        ```c
        f *= 5.0; // 不要寫 f = f * 5;
        f /= 3.0; // 不要寫 f = f / 3;
        ```

**Challenge 2: A Ternary-Operator Decision**
*   **目標**：要求使用者輸入一個大於 0 的整數，並回報該數字是奇數 (Odd) 還是偶數 (Even)。限定使用三元運算子。
*   **解答分析**：
    *   **邏輯**：判斷奇偶數最快的方法是取餘數運算 `v % 2`。結果是 1 為奇數，0 為偶數。
    *   **三元運算子應用**：
        ```c
        // 不需要用 if-else，直接在 printf 中使用三元運算子回傳對應字串
        printf("%s", (v % 2) ? "odd" : "even");
        ```
    *   **輸入驗證 (Optional)**：講師在解答中額外加了 `while(v < 1)` 迴圈來確保使用者輸入正整數，這是好的防呆習慣。

---

## Chapter 2: Main Function Arguments (主函數參數)

本章節介紹如何透過命令列 (Command Line) 與程式互動，這是所有 C 程式（無論是在圖形介面還是文字介面）啟動的基礎機制。

### 2-1. Work with arguments in the main() function

**核心概念 (Transcript 重點)**：
*   **啟動機制**：無論作業系統多麼圖形化，內部程式啟動都是透過命令列形式：`關鍵字` (Command Name) + `一系列選項/參數` (Options/Switches)。
*   這些文字資訊會作為 `main()` 函數的參數傳入 C 程式。程式可以存取、評估並利用這些資訊。
*   **參數定義**：
    *   如果程式不需要參數，`main()` 可以留空或寫 `void`。
    *   如果需要，必須同時指定兩個參數：`int argc` 和 `char *argv[]`。

**參數詳解**：
1.  **`argc` (Argument Count)**:
    *   整數類型。
    *   代表命令列上輸入的項目**總數**。
2.  **`argv` (Argument Vector)**:
    *   字串陣列 (Array of Strings)。
    *   每個字串代表一個命令列輸入的項目。
    *   寫法：`char *argv[]` (陣列表示法) 或 `char **argv` (指標表示法)。講師提到大多數人習慣用 `argv[]`。

**重要行為模式**：
*   **程式名稱總是存在**：`argv[0]` 永遠是程式本身的名稱（或是包含路徑的名稱）。因此 `argc` 至少為 1。
*   **雙引號的作用**：
    *   如果你輸入 `myprogram this is a test`，`argc` 是 5。
    *   如果你輸入 `myprogram "this is a test"`，`argc` 是 2。
    *   雙引號會將內部的文字視為**單一參數** (Single Argument)，且雙引號本身**不會**包含在字串中。這是所有作業系統的通則。

**IDE 設定 (Code::Blocks)**：
在 IDE 中執行時，因為沒有真正的命令列，你需要去 Project 設定中尋找 "Set programs' arguments" 來模擬輸入參數。

**程式碼分析 (`02-01_arguments1.c`)**：
```c
#include <stdio.h>

// main 函數接收 argc 和 argv
int main(int argc, char *argv[])
{
    // 顯示參數數量 (包含程式名稱)
    printf("There were %d command line arguments\n", argc);

    // 遍歷並顯示所有參數 (包含 argv[0])
    int x;
    for(x=0; x < argc; x++)
    {
        printf("Argument %d: %s\n", x, argv[x]);
    }

    return(0);
}
```
**應用場景**：
一旦獲得 `argv` 中的字串，你可以像處理任何 C 字串一樣處理它們：
*   使用 `strcmp` 比較（例如檢查是否輸入了 `-help`）。
*   使用 `atoi` 轉換為數字。
*   作為 `fopen` 的檔名參數。

### 2-2. Challenges & Solutions (本章挑戰與解答)

**Challenge: Reading Command-Line Arguments**
*   **目標**：寫一個程式讀取 `main` 函數原本的參數。判斷使用者是否輸入了至少一個參數（例如檔名）。
    *   如果沒有：顯示錯誤訊息並解釋需要參數。
    *   如果有：顯示該參數內容。
*   **解答分析**：
    *   **檢查點**：`if (argc < 2)`。
        *   因為 `argc` 至少是 1 (程式本身)，所以要檢查是否有額外參數，必須看是否小於 2。
    *   **錯誤處理**：
        ```c
        if(argc < 2) {
            puts("Error: You must provide a filename.");
            return 1; // 回傳非零值表示執行失敗
        }
        ```
    *   **顯示與執行**：講師提到，大部分程式不會去檢查檔名是否合法 (Valid filename)，通常就是直接嘗試開啟 (`fopen`)，如果失敗就算了。這是 C 語言常見的設計哲學。

---

## Chapter 3: Beyond Basic Variables (超越基礎變數)

本章節介紹外部變數、型別轉換以及靜態變數的特性與最佳實踐。

### 3-1. Set up an External Variable (外部變數)

**核心概念 (Transcript 重點)**：
*   **Local (區域)**：預設情況下，變數僅在定義它的函數內有效。離開該函數後，變數與其值就沒有意義了。
*   **Global/External (全域/外部)**：定義在任何函數之外的變數。它對程式中**所有**函數都是可見的。
*   **跨檔案共享**：
    *   如果你有兩個原始碼檔案（模組），想要共享變數，必須使用 `extern` 關鍵字。
    *   `extern int data;` 告訴編譯器：「有一個叫做 `data` 的整數變數存在於別處（另一個模組），請相信我，連結器 (Linker) 稍後會找到它。」
    *   這不會配置新的記憶體，只是宣告。

**講師警告 (Best Practices)**：
> "Why not just make all variables external? The answer is that doing so would simply be sloppy programming practice."
> (為什麼不把所有變數都設為全域？因為那是草率的程式設計習慣。)
*   C 語言的效率來自於區域變數，它們在函數結束後會釋放記憶體。
*   全域變數會讓程式變得難以閱讀與維護（不知道誰修改了它）。
*   **正常做法**：透過函數參數傳遞數值，或使用指標。
*   **例外**：當需要在不同模組間共享複雜的結構 (Structures) 時，外部變數有時是必要的。

**程式碼分析 - 跨檔案變數 (`03-01_extern3_main.c` & `03-01_extern3_manipulate.c`)**：

*   `03-01_extern3_main.c`: **定義**變數，配置記憶體。
    ```c
    int data[5] = { 2, 3, 5, 7, 9 }; // Global definition
    ```
*   `03-01_extern3_manipulate.c`: **宣告**變數，不配置記憶體。
    ```c
    extern int data[]; // Global declaration (告知編譯器)
    
    void manipulateData(void)
    {
        // 直接存取 main.c 中的 data
        for(int x=0;x<5;x++)
            data[x] *= 2;
    }
    ```

### 3-2. Typecast Variables (變數型別轉換)

**核心概念 (Transcript 重點)**：
*   Typecasting 是一種「欺騙編譯器」的方式 (fooling the compiler)，讓它暫時將某個變數視為另一種型別。
*   **語法**：`(type) variable`。
*   **自動轉換**：許多函數（如 `sqrt()`）會自動將整數參數轉換為 `double`，這時不需要顯式轉型。
*   **限制**：轉型並不改變數值的底層位元呈現，只是改變解讀方式。
    *   例如將負數整數強制轉型為 `unsigned`，不會得到絕對值，而會得到一個巨大的正整數。要取絕對值應使用 `abs()` 函數。

**程式碼分析 - 整數除法陷阱 (`03-02_cast1.c`)**：
這是初學者最常遇到的問題。
```c
#include <stdio.h>
#include <math.h>

int main()
{
    int a, b;
    float c;

    a = 10; b = 3;
    c = a/b;  // 整數除法！結果是 3，不是 3.333...
    printf("%d/%d = %.2f\n", a, b, c);  // 輸出: 10/3 = 3.00 (不正確!)

    // 正確寫法：先將其中一個運算元轉型
    c = (float)a / b;  // 輸出: 10/3 = 3.33

    return(0);
}
```
**重點**：`10/3` 在 C 語言中是整數除法，結果是 `3`。即使存入 `float` 變數，結果也只是 `3.00`。必須先將運算元轉型。

**程式碼分析 - 函數自動轉換 (`03-02_cast2.c`)**：
```c
#include <stdio.h>
#include <math.h>

int main()
{
    int a;
    float aroot;

    printf("Type an integer: ");
    scanf("%d", &a);
    aroot = sqrt(a);  // sqrt() 自動將 int 轉為 double，無需手動轉型
    printf("The square root of %d is %f\n", a, aroot);

    return(0);
}
```
**重點**：`sqrt()` 預期 `double` 參數，C 編譯器會自動隱式轉換 (Implicit Cast)。

**程式碼分析 - 隨機數種子 (`03-02_cast3.c`)**：
```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

int main()
{
    int r;

    srand((unsigned)time(NULL));  // time() 回傳 time_t，需轉型為 unsigned
    r = rand();
    printf("%d is a random number.\n", r);

    return(0);
}
```
**重點**：`srand()` 預期 `unsigned int` 作為種子。`time(NULL)` 回傳 `time_t` (通常是 `long`)，需要顯式轉型。

**關鍵應用場景 - 記憶體配置 (`03-02_cast4.c`)**：
這是最常見需要轉型的地方。
`malloc()` 函數回傳 `void *` (通用指標)，代表一個記憶體位址，但沒有型別資訊。為了使用這塊記憶體，我們必須告訴編譯器我們要存什麼資料。

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    int *m;

    // 配置 16 個 int 的記憶體空間
    m = (int *)malloc(16 * sizeof(int));
    if(m == NULL) {
        puts("Memory allocation failed!");
        return(1);
    }

    // 使用記憶體...
    *m = 100;
    printf("Value: %d\n", *m);

    // 方法1：使用指標算術運算
    *m = 100;           // 第1個int (索引0)
    *(m + 1) = 200;     // 第2個int (索引1)
    *(m + 2) = 300;     // 第3個int (索引2)
    
    // 方法2：使用陣列語法（更直觀）
    m[0] = 100;         // 第1個int
    m[1] = 200;         // 第2個int
    m[2] = 300;         // 第3個int
    m[15] = 999;        // 最後一個int (索引15)

    free(m);  // 別忘記釋放記憶體！
    return(0);
}
```
講師指出："This type of statement is where you'll most often see a cast used pretty much without exception."

### 3-3. Take Advantage of Static Variables (靜態變數的優勢)

**核心概念 (Transcript 重點)**：
*   **問題**：區域變數在函數結束後數值會遺失 (lost)。
*   **解決方案**：
    1.  使用 Global variable（不推薦，如前所述）。
    2.  使用 `static` 關鍵字。
*   **Static 行為**：`static` 區域變數雖定義在函數內，但**在函數結束後仍保留其值**。它只在第一次呼叫時初始化一次。

**初始化陷阱 (Exercise File: `03-05_static1.c`)**：
```c
void f(void)
{
    static int x = 0; // 只執行一次
    printf("x = %d\n",x);
    x++;
}
```
如果將程式碼寫成：
```c
static int x;
x = 0; // 錯誤！這行每次函數執行都會跑，導致 x 永遠被重置為 0
```
**重要**：若需要初始化 `static` 變數，必須在宣告的那一行同時進行。

**進階應用 - 回傳複雜資料 (`03-05_static2.c`)**：
這是 `static` 最常見的用途之一。當函數需要回傳一個陣列或字串時，不能回傳一般區域變數的指標（因為記憶體會被回收）。

**錯誤示範**：
```c
char *repeat(char r)
{
    char string[32]; // Local array
    ...
    return string; // 危險！回傳指向已釋放記憶體的指標
}
```
編譯器會發出警告：`function returns address of local variable`。執行結果是未定義的 (unpredictable/garbage)。

**修正版**：
```c
char *repeat(char r)
{
    static char string[32]; // Static array (存在於 Global memory區段)
    ...
    return string; // 安全，因為記憶體在整個程式執行期間都有效
}
```
透過加上 `static`，我們解決了指標懸空 (Dangling Pointer) 的問題。

### 3-4. Challenges & Solutions (本章挑戰與解答)

**Challenge 1: Specifying a Cast**
*   **目標**：輸入一個整數，使用 `printf` 的 `%f` 格式將其顯示為小數點後 1 位的浮點數。
*   **解答分析**：
    *   **核心問題**：`printf` 的 `%f` 預期接收一個 `double` (或 float)。如果你傳入一個 `int`，記憶體讀取會錯誤，導致顯示 garbage (例如 0.0 或巨大數字)。
    *   **解決方案**：
        ```c
        printf("%.1f", (float)a);
        ```
    *   講師強調：這是顯示整數為浮點格式的**唯一**正確方法 (The only way)。

**Challenge 2: Setting up a Static Variable**
*   **目標**：寫一個函數，回傳包含前 5 個質數 (2, 3, 5, 7, 11) 的陣列。`main` 函數負責顯示。
*   **解答分析**：
    *   **關鍵字 `static`**：這是此題的靈魂。
        ```c
        int *primes() {
            static int p[5] = {2, 3, 5, 7, 11};
            return p;
        }
        ```
    *   如果不加 `static`，陣列 `p` 在函數返回時就會被銷毀，回傳的指標將指向無效記憶體。
    *   `main` 函數中使用 `int *array = primes();` 來接收位址。


<!-- 註記開始 -->
> 📌 **重要提醒**：關於 static 變數的完整使用時機和原理解釋，
> 請參考 [/Users/chilung/Downloads/basic_c/C語言_static_變數完整指南.md](file:///Users/chilung/Downloads/basic_c/C語言_static_變數完整指南.md)
> 
> 🔍 **重點章節對應**：
> - 靜態變數使用時機：L81~95（基本型別 vs 陣列回傳差異）
> - C/C++ 參數傳遞機制：L178~182（Call by Value vs Call by Reference）
> 
> 💡 **核心概念**：
> - C 語言只有 Call by Value（包含指標副本傳遞）
> - C++ 才有真正的 Call by Reference (Reference 是變數的別名（Alias）)
> - Static 變數主要用於：陣列回傳、狀態保持、單例模式
<!-- 註記結束 -->

---

## Chapter 4: Arrays and Structures (陣列與結構)

本章節深入探討 C 語言中兩個最重要的複合資料型別：陣列與結構，以及它們如何與函數互動、如何進行排序與記憶體操作。

### 4-1. Sort an Array (陣列排序)

**核心概念 (Transcript 重點)**：
*   **Bubble Sort (氣泡排序)**：
    *   最基本但效率較差的排序法。
    *   原理：透過巢狀迴圈 (Nested Loops) 不斷比較相鄰元素，將最大（或最小）的數值「浮」到陣列尾端。
    *   **Transcript 描述**：它讓數字在陣列中「跳來跳去」(bounce around)，直到就定位。對於 10 個元素的陣列，可能需要 27 次交換。若是大型陣列，效率極低。

**程式碼分析 - Bubble Sort 完整實作 (`04-01_arraysort1.c`)**：
```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define SIZE 10

int main()
{
    int numbers[SIZE];
    int x, outer, inner, temp;

    /* 用隨機數填充陣列 */
    srand((unsigned)time(NULL));
    for(x = 0; x < SIZE; x++)
        numbers[x] = rand() % 100 + 1;  // 1~100 的隨機數

    /* 顯示未排序的陣列 */
    puts("Unsorted array:");
    for(x = 0; x < SIZE; x++)
        printf(" %3d", numbers[x]);
    putchar('\n');

    /* selection sort核心邏輯 */
    /* 每一次選出最小的放最前面 */
    for(outer = 0; outer < SIZE; outer++)
        for(inner = outer + 1; inner < SIZE; inner++)
        {
            if(numbers[outer] > numbers[inner])  // 如果前者大於後者
            {
                temp = numbers[inner];           // 交換！
                numbers[inner] = numbers[outer];
                numbers[outer] = temp;
            }
        }

    /* 顯示已排序的陣列 */
    puts("Sorted array:");
    for(x = 0; x < SIZE; x++)
        printf(" %3d", numbers[x]);
    putchar('\n');

    return(0);
}
```

*   **Quick Sort (快速排序, `qsort`)**：
    *   更高效的排序方式，標準函式庫 (`stdlib.h`) 提供 `qsort()` 函數。
    *   **參數需求**：
        1.  Base address (陣列名稱)。
        2.  Number of elements (陣列大小)。
        3.  Size of each element (使用 `sizeof(int)` 等)。
        4.  Comparison function (比較函數)。

**比較函數詳解 (The Comparison Function)**：
這是 `qsort` 最「奇怪」但也最強大的部分。你必須自己寫一個函數來決定排序邏輯。
*   **格式**：`int compare(const void *a, const void *b)`
*   **參數**：接收兩個 `const void *` 指標 (通用指標)。
*   **回傳值邏輯**：
    *   **負數 (< 0)**：`a` 排在 `b` 前面 (Swap b for a)。
    *   **正數 (> 0)**：`b` 排在 `a` 前面 (Swap a for b)。
    *   **零 (0)**：相等，不交換。
*   **實作細節**：因為接收的是 `void *`，使用前必須**強制轉型** (Typecast) 並**解參考** (Dereference)。

**程式碼分析 (`04-01_arraysort3.c`)**：

```c
int compare(const void *a, const void *b)
{
    // 1. (int *)a : 將 void 指標轉型為 int 指標
    // 2. *(int *)a : 取出該位址的整數值
    // 3. 相減 : 回傳正負值
    return( *(int *)a - *(int *)b );
}

int main()
{
    // ...
    qsort(numbers, SIZE, sizeof(int), compare);
    // ...
}
```
講師建議："It's a mess, but to be honest, most programmers simply copy this skeleton when they do a Q sort." (這段程式碼看起來很亂，但大多數工程師都是直接複製貼上這個樣板。)

### 4-2. Work with Arrays and Functions (陣列與函數)

**核心概念 (Transcript 重點)**：
*   **傳遞陣列即傳遞指標**：
    *   當你把陣列傳給函數時，你傳的其實是**記憶體中的基底位址** (Base Address)。
    *   `void func(int a[])` 和 `void func(int *a)` 在編譯器眼中是**完全一樣**的。
*   **呼叫方式**：只需使用陣列名稱，**不要**加方括號。
    *   正確：`sortArray(numbers);`
    *   錯誤：`sortArray(numbers[]);` (Compiler Error)
*   **不需要回傳**：因為傳遞的是位址，函數內的修改會直接反映在原始陣列上，因此不需要 `return` 陣列。

**回傳陣列的唯一場景 (`04-04_arrayfunct2.c`)**：
如果函數負責**創造**陣列並回傳，該陣列必須是 `static` 的。
```c
int *generate()
{
    static int a[10]; // 必須是 static，否則函數結束後記憶體會被回收
    // fill array...
    return a; // 回傳位址
}
```
但講師認為更好的做法是：在 `main` 宣告陣列，傳給函數去填充 (`04-04_arrayfunct3.c`)。

### 4-3. Send a Structure to a Function (結構與函數)

**核心概念 (Transcript 重點)**：
*   傳遞結構常讓工程師感到焦慮 (angst)，但掌握規則後並不難。
*   **兩大前提**：
    1.  結構必須定義 (Define structure type)。
    2.  變數必須宣告 (Declare variable)。
*   **常見錯誤** (`04-07_structfunct1.c`)：
    *   如果結構定義在 `main` 裡面，外部的函數原型 (Prototype) 會看不懂這個結構型別。
    *   **解決方案**：將 `struct` 定義移到全域 (Global)，且必須放在所有函數原型**之前**。編譯器是 Top-down 讀取的。

**效能最佳化：使用指標 (`04-07_structfunct3.c`)**：
*   直接傳遞結構 (`void func(struct person p)`) 會複製整個結構的內容，造成 **Overhead**。
*   **推薦做法**：傳遞結構的指標 (`void func(struct person *p)`)。
*   **語法改變**：
    *   傳值：使用點運算子 (Dot Operator) -> `p.name`
    *   傳指標：使用箭頭運算子 (Arrow Operator) -> `p->name`

### 4-4. Build an Array of Structures (結構陣列)

**核心概念 (Transcript 重點)**：
*   就像整數陣列一樣，我們也可以建立結構的陣列。
*   **宣告**：`struct weather week[7];`
*   **初始化**：
    *   逐一賦值 (Redundant but works)。
    *   **預先賦值 (Pre-assigned)**：
        ```c
        struct weather week[7] = {
            { "Sunday", 72.5 },
            { "Monday", 68.4 },
            // ...
        };
        ```
*   **結構交換的強大特性 (`04-10_structarray4.c`)**：
    講師特別展示了一個強大的功能：**結構可以直接賦值**。
    ```c
    week[1] = week[5];
    ```
    這行程式碼會將 `week[5]` 的所有成員（包括內部的字串陣列）完整複製到 `week[1]`。
    *   "You don't need to manually copy each member... Just set one element equal to another." (你不需要手動複製每個成員，直接賦值即可。)

### 4-5. Challenges & Solutions (本章挑戰與解答)

**Challenge 1: Sort a String**
*   **目標**：將使用者輸入的字串進行字母排序 (Alphabetical sort)。
*   **解答分析**：
    *   字串本質上就是 `char` 陣列。
    *   **演算法**：使用 Bubble Sort 進行雙層迴圈比較。
    *   **比較**：`if(buffer[a] > buffer[b])`
    *   **交換**：`temp = buffer[a]; buffer[a] = buffer[b]; buffer[b] = temp;`
    *   講師也展示了第二種解法：排序字串陣列 (Array of Strings)，這時需要用 `strcpy` 來交換字串。

**Challenge 2: An Array Modification Function**
*   **目標**：寫一個函數，將傳入字串的所有字母轉為大寫，並將空白取代為底線 (`_`)。
*   **解答分析**：
    *   **傳遞方式**：傳遞字串的指標 `void modify(char *s)`。
    *   **指標操作效率高**：
        ```c
        while(*s) {
            *s = toupper(*s); // 轉大寫 (需 include <ctype.h>)
            if(*s == ' ') *s = '_'; // 取代空白
            s++; // 移動指標
        }
        ```
    *   使用指標 (`*s`) 直接操作記憶體，比使用陣列索引 (`s[i]`) 稍微高效且語法簡潔。

**Challenge 3: Create a Structure Function**
*   **目標**：創建一個結構（包含姓名與年齡），並用函數填入資料與顯示資料。
*   **解答分析**：
    *   **填入資料函數 (Fill Structure)**：
        *   **關鍵**：必須傳遞指標！如果傳值，函數內的修改只會發生在副本上，回到 main 後結構還是空的。
        *   `void fill(struct person *p)`
        *   `scanf("%s", p->name);` (陣列本身就是位址，不需要 `&`)
        *   `scanf("%d", &p->age);` (整數需要 `&` 取址)
    *   **顯示資料函數**：可以傳值，但傳指標效率較好（避免複製）。

**Challenge 4: Sort an Array of Structures**
*   **目標**：將一週的天氣結構陣列依據溫度由低到高排序。
*   **解答分析**：
    *   這是本章節最精華的挑戰。
    *   **關鍵 Insight**：當我們發現 `week[a].temp > week[b].temp` 需要交換時，**不需要**手動交換 name 和 temp。
    *   直接交換整個結構！
        ```c
        struct weather temp;
        // ... inside loop ...
        temp = week[a];    // 備份整個結構
        week[a] = week[b]; // 複製整個結構
        week[b] = temp;    // 還原整個結構
        ```
    *   這充分利用了 C 語言結構賦值的深拷貝特性，程式碼非常乾淨。

---

## Chapter 5: Pointer Tips (指標技巧)

本章節將深入探討 C 語言中最惡名昭彰但也最強大的功能：指標。我們將會釐清取址運算子、解參考運算子，以及它們與陣列的關係，最後解開複雜的運算子優先權之謎。

### 5-1. When to use the Ampersand Operator (何時使用 `&`)

**核心概念 (Transcript 重點)**：
*   **角色**：`&` 是「取址運算子」(Address-of Operator)。
*   **功能**：回傳變數在記憶體中的位址。這是它唯一的用途。
*   **用法**：必須放在變數的前面 (Pre-fixed)，就像火車頭一樣。
*   **不需要初始化**：變數只要被宣告 (Declared) 就擁有了記憶體位址，不需要等到初始化 (Initialized) 才能取址。

**陣列與 `&` 的特殊關係 (`05-01_ampersand3.c`)**：
這是一個常見的盲點。
*   `int a[10];`
*   `a` (不加方括號)：本身就代表陣列的基底位址 (Base Address)，等同於指標。
*   `&a`：也是陣列的基底位址。
*   `&a[0]`：也是陣列的基底位址。
*   **結論**：`a`, `&a`, `&a[0]` 三者印出來的位址是一模一樣的。
*   **例外**：當存取陣列中的個別元素時 (e.g., `a[3]`)，它就只是一個普通的整數變數，這時如果需要位址，就**必須**加上 `&` (e.g., `&a[3]`)。

### 5-2. How to bind the Asterisk Operator (如何綁定 `*`)

**核心概念 (Transcript 重點)**：
*   **角色**：`*` 是「解參考運算子」(Dereference Operator)，但它也有「宣告指標」的功能，這種雙重人格 (Split Personality) 造成了混淆。
*   **作為宣告**：`int *p;` (產生一個指標變數)。
*   **作為運算**：`*p` (取出 `p` 指向的記憶體位址中的**值**)。此時我們稱之為 "Pointer references the value at the location"。

**綁定問題 (Binding Confusion) - `05-02_pointer1.c`**：
考慮以下程式碼：
```c
putchar( ++*s );
```
這到底是什麼意思？
1.  將 `s` (位址) 加 1，然後印出新位址的值？
2.  將 `*s` (值) 加 1，然後印出新值？

答案取決於**優先權 (Precedence)**。
在某些複雜寫法中，例如 `*s++`：
*   講師解釋：`*` 綁定的是字元 (Value)，而後綴 `++` 作用於指標 (Address)。結果是：先印出當前的字元，然後將指標移動到下一個位置。

### 5-3. Understanding Arrays and Pointers (理解陣列與指標)

**核心概念 (Transcript 重點)**：
*   陣列與指標在大部分情況下行為相似，但在存取元素時語法不同。
    *   **Array Notation**: `f[x]`
    *   **Pointer Notation**: `*(f + x)`
*   **Pointer Math (指標運算)**：
    *   當你寫 `pf + 1` 時，電腦並不是只把記憶體位址加 1。
    *   它會加上「該變數型別的大小」(Size of the variable type)。
    *   如果是 `int` (通常 4 bytes)，`pf + 1` 實際上是位址 `+4`。如果是 `double`，可能是 `+8`。
    *   **講師名言**："All this overhead is managed by the compiler." (這些細節都由編譯器幫你處理好了。)
*   **最佳實踐**：
    講師建議使用 **Pointer Notation with Offsets** (指標加位移) 來操作陣列。
    保持指標變數 (如 `pf` 或 `f`) 指向基底位址，然後用 `*(pf+i)` 來存取，而不要一直移動指標本身 (如 `pf++`)，除非你真的很清楚自己在做什麼。

### 5-4. Obeying the Order of Precedence (遵守運算子優先權)

**核心概念 (Transcript 重點)**：
*   **Pecking Order (啄序/優先權)**：決定了複雜方程式中誰先執行。
*   **層級架構 (Hierarchy Overview)**：
    1.  **Parentheses `()`**：括號擁有最高權力，可以覆蓋一切。
    2.  **Unary Operators (一元運算子)**：`!`, `~`, `++`, `--`, `*` (pointer), `&` (address), `(type)`, `sizeof`。
        *   **Associativity (結合性)**：**Right-to-Left** (由右向左)。這點非常重要！
    3.  **Math (Multiplication/Division)**：`*`, `/`, `%`。 (Left-to-Right)
    4.  **Math (Addition/Subtraction)**：`+`, `-`。 (Left-to-Right)

**程式碼分析 - Postfix vs Prefix (`05-08_order2.c`)**：
這是理解 `++` 運算子的關鍵範例。

```c
#include <stdio.h>

int main()
{
    int a, b;

    a = b = 10;
    printf("a   = %d\tb   = %d\n", a, b);       // 輸出: a = 10, b = 10
    printf("a++ = %d\t++b = %d\n", a++, ++b);   // 輸出: a++ = 10, ++b = 11
    printf("a   = %d\tb   = %d\n", a, b);       // 輸出: a = 11, b = 11

    return(0);
}
```

**執行結果**：
```
a   = 10   b   = 10
a++ = 10   ++b = 11
a   = 11   b   = 11
```

**關鍵差異**：
*   `a++` (Postfix)：**先使用原值**，然後 a 加 1。所以 printf 印出 10，但之後 a 變成 11。
*   `++b` (Prefix)：**先加 1**，然後使用新值。所以 printf 印出 11，b 也是 11。

**終極謎題解析 (`05-08_order3.c`)**：

```c
putchar( ++*s++ );
```
這行程式碼充滿了「隱晦的優雅 (Cryptic Elegance)」但也「極度困惑 (Dreadfully Confusing)」。讓我們依據優先權拆解：

1.  **`s++` (Postfix Increment)**：
    *   優先權高於 `*` 嗎？其實 Postfix 優先權通常最高。
    *   但它的效果是：回傳 `s` **原本的位址**，但在語句結束後將 `s` 加 1。
2.  **`*` (Dereference)**：
    *   作用於 `s++` 回傳的**舊位址**。取出該位址的字元 (Value)。
3.  **`++` (Prefix Increment)**：
    *   作用於 `*` 取出的字元值。將該字元值加 1。

**結果**：取出當前字元 -> 字元值加 1 -> 印出 -> 指標移到下一格。
(Transform "abc" to "bcd").

**講師建議**：
> "My advice... handle each of operations separately. But thanks to precedence, this type of obfuscation proliferates in the C language."
> (建議將這些操作分開寫，以提高可讀性。但在 C 語言中這類混淆寫法很常見，你必須看得懂。)
> "Remember, that liberal use of parentheses can always ensure that the equation is evaluated as you intended."
> (善用括號，確保程式依照你的想法執行。)

### 5-5. Challenges & Solutions (本章挑戰與解答)

**Challenge 1: Incrementing a Pointer**
*   **目標**：透過指標來設定整數變數的值，然後透過指標將其值加 1。
*   **解答分析**：
    *   **優先權陷阱再確認**：
        ```c
        int f;
        int *fptr = &f;
        *fptr = 89;
        printf("%d", ++*fptr); 
        ```
    *   這裡使用 `++*fptr` (Prefix)。因為 `++` (Prefix) 和 `*` (Dereference) 都是 Right-to-Left，但因為位置關係，`*fptr` 先被求值（取出 89），然後 `++` 作用於這個值（變 90）。

**Challenge 2: Displaying an Array (Pointer Notation)**
*   **目標**：創建一個字串，計算長度，然後使用「指標表示法」遍歷顯示，但**禁止**移動指標 (No `s++`)。
*   **解答分析**：
    *   **限制**：不能改變基底位址。
    *   **解法**：使用 Offset。
        ```c
        for(x=0; x<length; x++) {
            putchar( *(text + x) );
        }
        ```
    *   必須把 `*(text + x)` 括號起來。如果不加括號寫成 `*text + x`，會變成「取出 text 第一個字元，然後加上整數 x」，這完全是錯誤的邏輯。

---

## Conclusion: Next Steps (結語：下一步)

恭喜完成本課程！這只是 C 語言深水區的起點。
講師建議的後續學習路徑：

1.  **應用領域**：
    *   網路程式設計 (Network Programming)
    *   作業系統開發 (OS Development)
    *   微控制器/嵌入式系統 (Microcontrollers / Embedded Systems)
    *   遊戲開發 (Game Development)

2.  **學習資源**：
    *   **Manual Pages**: 在 Linux/Mac 終端機輸入 `man <function_name>` (如 `man printf`) 是最權威的參考。
    *   **Online Reference**: University of Illinois 的 C Library Reference Guide。
    *   **Books**: 講師網站 `c-for-dummies.com`。

3.  **持續練習**：
    *   講師強調：「動手做 (Mess with the files)」是掌握 C 語言的唯一方法。
    *   嘗試修改 Exercise Files，觀察結果變化。
    *   挑戰自己寫更複雜的程式，例如結合本課程學到的指標、結構與陣列。

---
**[End of Notes]**
*本筆記已經過完整審核，涵蓋 Introduction, Chapters 1-5, Conclusion 以及所有 Exercise Files 的關鍵細節。*
