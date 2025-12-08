## 0X10同樣概念但不同題型:
<br/>

- 在嵌入式開發中，我們經常需要定義時間相關的參數。假設我們現在正在開發一個基於 16-bit 架構 的微控制器系統。

  請你使用 #define 宣告一個名為 TICKS_PER_DAY 的巨集常數，用來表示一天內總共有多少個系統 tick。

  假設我們的系統時鐘頻率是 1 kHz（也就是每秒 1000 個 tick）。

  請寫出這行程式碼，並簡短說明你的寫法理由。」
<br/>
<br/>

我的答案:

```c
#define TICKS_PER_DAY(1000*60*60*24)UL

//U=unsign;代表不帶負號的整數，因此可獲得更大的定義範圍，如32bit的系統可獲得 2^32= 4294967296的整數表示。
//L=long;在某些編譯器裡，他和int一樣代表4byte，但在有些如16bit系統裡，可能int=2bytes ,他會等於4bytes。
//當我們加入UL可以有更好的裝置移植性。
```
正確答案:
  ```c
#define TICKS_PER_DAY  (1000UL * 60UL * 60UL * 24UL)
  ```
#錯誤解析

1.巨集名稱要和左括號之間留有空白

2.UL要接在常數後面

---
<br/>

- 在嵌入式系統中，為了節省函式呼叫（Function Call）的效能損耗，我們有時會用巨集來寫一些簡單的數學運算，類似 Inline Function 的概念。

  請你寫一個名為 MAX 的巨集，它接收兩個參數 A 和 B，並回傳其中較大的那個數值。

  寫完後，請告訴我：如果我這樣呼叫你的巨集：MAX(*ptr++, 5)，會發生什麼潛在問題？」

  ```c
  #define MAX(a,b)= (a>b) ? a : b  

  MAX(*ptr++, 5);
  // =(*ptr++>5) ? *ptr++ : 5
  //假設*ptr++完等於8，然後判斷式為正確所以回傳*ptr++ ，但是這裡會變成多加一次，所以變成9。
  ```
<br/>
<br/>

正確答案:
  ```c
#define MAX(A, B)   ((A) > (B) ? (A) : (B))
  ```
#錯誤解析
1.括弧要括好，假設A= x&2 ，判斷式變成 x & 2 > 3，因為 > 優先權高於 & ，所以會錯亂。

2.要永遠記住#define 這種是巨集替換 所以不該加 "=" ，只要空格就好。 

---
<br/>

- 接著我們來談談變數的生命週期與可視範圍（Scope），這是模組化設計的基礎。

  請告訴我，關鍵字 static 在 C 語言中有哪三種主要的用途？ 為了讓我知道你真的理解，請分別舉例說明：

  1.在函式內部（Function body）**宣告 static 變數。

  2.在模組內（Module/File level，但在函式外）**宣告 static 變數。

  3.在模組內宣告 static 函式。

  它們分別會有什麼行為？」
<br/>
<br/>

我的答案:

  1.在函式內部宣告static變數，該變數的存在週期只會在該函數內。
  
  2.在檔案內函數外宣告static變數，等於可以在檔案內宣告同名的變數。
  
  5.等於隱藏該變數無法讓其他檔案儲存。

正確答案:
  1."函數"內的static"變數"：變數的值在函式呼叫後不會消失。
  
  2."區域"內的static"變數"：變數無法在其他檔案透過extern存取他。
  
  3."區域"內的static"函式"：該函式只能被同一檔案的其他函式呼叫。

#錯誤解析
  1.正常的變數在函式內呼叫本來就會消失，所以加入static才不會消失阿! --> 那函式內的呼叫，外部檔案可以extern存取他嗎?
  
  2.應該要強調"區域外"可以用同名，例如: 
  ```C
int a = 50;       // 全域變數
void foo(void) {
static int a = 3; // 區域靜態變數
  }
  ```
  但是這也不重要，因為正常情況下的變數都可以。
  
  所以他的特點就會是無法讓其他檔案透過extern存取他，也就是隱藏起來。
  
  3.可能當下不太清楚static在"函式"的用法，下面是例子:
  ```C
#include <stdio.h>

static void secret() {
    printf("This is a static function, only visible in this file.\n");
}

void call_secret() {
    secret();  // OK
}
  ```

---
- 請解釋 volatile 這個關鍵字的含義。 並且，請列舉 三種 務必要使用 volatile 的具體場景（Scenario）。」
<br/>
<br/>

我的答案:
  volatile 提醒編譯器，不要對該變數做最佳化，每次都要重新讀取他的值。
  
  例子:
  
  1.周邊裝置的硬體暫存器，透過高低電位決定狀態，如果該程式的變數指向該暫存器的記憶體位置，那就會影響該程式變數。
  
  2.中斷程式會改動到的變數。
  
  3.多執行緒下，多個程式共享的變數。
  
#補充:
volatile 告訴編譯器，該變數的值可能會被其他因素改變，因此不該對其進行最佳化，每次存取都須要直接讀取記憶體資料。 

```c
///可能會最佳會為while(1)
int flag = 0; //正確為:volatile int flag = 0;

int main() {
    while (flag == 0) { 
        // 等中斷改變 flag
    }
}
```
當flag應為中斷程式改變數值時，如果沒有加入volatile，while迴圈可能被最佳化成無限迴圈。

例2:

ISR 中會存取到的"非自動變數"（Non-automatic variables）」或是「全域變數（Global variables）」。

自動變數如下:
```c
void foo() {
    int x = 10;   // automatic variable
}
```
函數結束就會被回收。

非自動變數如下:

- 全域變數

- static 變數（無論在函數內或函數外）

- 記憶體映射暫存器

- heap 分配的變數（比較少用在 ISR）

這些變數的特性是：

  在 ISR 執行時仍然存在，可以被 ISR 修改或讀取。

---
<br/>
-請問：一個變數可以同時是 const 也是 volatile 嗎？ （例如：const volatile int x;）

如果是，請舉一個實際的例子說明為什麼會有這種怪異的組合。

如果否，請解釋為什麼這兩個關鍵字互斥。」
<br/>
<br/>

我的答案:
  可以，當我要定義某個只能唯讀但是數值可能會變動的變數時，我可以用const volatile int 變數。 
  
  例1: #define ADC_DR   (*(const volatile uint32_t*) 0x4001244C )
       這個數值會被"硬體"更新，不能用最佳化，且不想被寫入。

#補充:
*(const volatile uint32_t*)0x4001244C 這在 記憶體映射 I/O（MMIO） 中是 標準、通用、幾乎所有 MCU 都用 的方法。
它是一種 「先把位址強制轉型成指標 → 再解參考」

疑問:那括弧位置為什麼不是這樣 *(const volatile uint32_t)*0x4001244C

解答:因為它可能會認定為*是乘法而不是指標，所以加上括弧區隔開來，所以必須寫成(type*) address

---
<br/>

假設我有一個變數 unsigned int a;，以及一個位元位置 int bit = 3;（代表第 3 個 bit，從 0 開始算）。

請利用位元運算子（Bitwise Operators），分別寫出以下三個操作的 C 語言程式碼（如果不確定可以用 #define 輔助）：

1.設定（Set） a 的第 3 個 bit 為 1。

2.清除（Clear） a 的第 3 個 bit 為 0。

3.反轉（Toggle/Flip） a 的第 3 個 bit（若是 0 變 1，1 變 0）。」

<br/>
<br/>

我的答案:
1. a |= (1<<bit);
2. a &= ~(1<<bit);
3. a ^= (1<<bit);

#補充:
" ^ "是XOR的邏輯運算式，XOR中文是邏輯互斥或，當兩個值相同時為0，不同時為1，

真值表如下:

| A | B | Output |
|---|---|:-:|
| 0 | 0 | 0 |
| 1 | 0 | 1 |
| 0 | 1 | 1 |
| 1 | 1 | 0 |

一般來說會封裝成巨集(Macro)

#define BIT_SET(var, bit) ((var) |= (1UL << (bit)))

#define BIT_CLEAR(var, bit) ((var) &= ~(1UL << (bit)))

#define BIT_TAGGLE(var, bit) ((var) ^= (1UL << (bit)))

BIT_SET(a,3);

---
<br/>

- 我們公司剛來了一位實習生，他寫了一個針對計時器（Timer）的中斷服務程式（ISR）。這段程式碼編譯可能會過（取決於編譯器擴充），但在實際系統運作上是完全不及格的。

請幫我進行 Code Review，指出這段程式碼犯了哪 4 個 致命錯誤：

```C
__interrupt double timer_isr(int initial_val)
{
    double result = initial_val * 3.14159;
    printf("ISR triggered! Result: %f\n", result);
    return result;
}
```」
```
我的答案:
<br/>
<br/>

 1.ISR不能傳遞常數

 ```c
 void timerisr(void)
 ```

2.ISR內不能return變數

3.不能用來計算精浮點數之類的數學算式，內部的程式應該要精簡短小。

4.使用printf會發生不可重入等相關問題 同3.一樣。  

#補充

```c
// 定義一個 volatile 全域變數作為旗標
volatile int timer_flag = 0;

__interrupt void timer_isr(void)
{
    // 1. 清除中斷旗標 (硬體相關，通常必須做)
    // Clear_HW_Interrupt_Flag(); 
    
    // 2. 設定軟體旗標通知主程式
    timer_flag = 1;
}

// 在 Main Loop 中處理
void main(void) {
    while(1) {
        if (timer_flag) {
            timer_flag = 0; // 清除旗標
            // 原本在 ISR 想做的繁重工作移到這裡做
            printf("Timer Event Processed!\n");
            do_complex_math();
        }
    }
}
```
---

<br/>

- 假設我們的硬體規格書（Datasheet）上寫著： 有一個控制暫存器位於絕對位址 0x67A9。 我們需要將數值 0xAA55 寫入這個位址。

  請寫出一段 C 語言程式碼來完成這個動作。 （提示：編譯器是標準 ANSI C，你需要自行處理型別轉換）」
<br/>
<br/>
我的答案:

```c
int *ptr = (int*)0x67A9 //ptr 指向int等於0x67A9指向int ，所以ptr = 0x6A79，所以指標*ptr 的地址為0x67A9。
*ptr = 0xAA55 ;
```

#補充(內容我還沒真正搞清楚，之後要讀)

1.缺少 Volatile（重要）：

還記得我們剛才聊過的 volatile 嗎？存取絕對位址通常是為了寫入硬體暫存器。如果你沒加 volatile，編譯器發現你寫入 *ptr 後再也沒讀取過它，很可能會把這行程式碼當作「無用程式碼」直接優化刪除。

2.記憶體對齊（Alignment）陷阱：

0x67A9 是一個奇數位址。在許多 32-bit 架構（如某些 ARM Cortex-M）上，使用 int*（通常是 4 bytes）去存取非對齊位址（Unaligned Address）會直接導致硬體錯誤（Usage Fault）。雖然這題主要考語法，但如果你能提到「這可能是個 byte 或 short 存取」，我會覺得你非常有經驗。

```c
// 將整數轉型為「指向 volatile int 的指標」，然後取值寫入
*(volatile int *)(0x67A9) = 0xAA55;
```

或者為了避免對齊問題，如果這是一個 8-bit 的暫存器，我們應該用 char：

```c
*(volatile unsigned char *)(0x67A9) = 0x55;
```
---
<br/>

- 請問：

  這段程式碼執行後會印出什麼？

  為什麼？（請解釋編譯器是怎麼處理 a + b 的）」

  ```c
  void test_logic(void)
  {
      unsigned int a = 6;
      int b = -20;
      
      if (a + b > 6)
      {
          printf("Result is > 6");
      }
      else
      {
          printf("Result is <= 6");
      }
  }
  ```
  
<br/>
<br/>

我的答案:

當unsigned int和 int 一起運算的時候，會被電腦歸類成兩個都是unsigned，所以第一個判斷式為a + b > 6

等同於 6 + 20 > 6 所以會印出"Result is > 6"

#錯誤解析:

-20 轉成 unsigned 不會變成 20 而是1x0000014 --> 0xFFFFFFEC(假設32bit系統) 等於2^32 - 20

<更好的回答版本> 在C語言中，表達式包含signed 和 unsigned 型別時，signed 會被自動提升為 unsigned，-20在二補數表示法中是一個最高位元（MSB）為 1 的數值，當它被強制視為無號數時，會變成一個接近 UINT_MAX 的巨大正整數。

---

<br/>

- 在嵌入式系統（Embedded Systems），特別是高可靠度（如車用、醫療）的專案中，我們通常嚴格禁止使用 malloc 和 free。

  請告訴我，為什麼嵌入式系統要避免使用動態記憶體配置？請列舉 兩個 主要的技術原因。」

<br/>
<br/>

答案: 

melloc 可能會發生記憶體碎片化的問題，如果沒有成功free掉，會導致一直占用記憶體，需要斷電重新開。

#補充:

「嵌入式系統避免使用動態記憶體配置主要有三個原因：

1.記憶體破碎化（Fragmentation）：長時間頻繁分配與釋放，會導致記憶體變成零碎的小區塊，最終即使總剩餘空間足夠，也無法分配出一塊連續的大記憶體。

2.記憶體洩漏（Memory Leak）：若程式邏輯錯誤導致沒有釋放，系統最終會當機（OOM）。

3.不可預測的執行時間（Non-deterministic execution time）：malloc 尋找空閒區塊的時間是不固定的，這違反了即時系統（Hard Real-Time）對時間確定性的要求。」

#錯誤解析:
1.記憶體碎片化 --> 記憶體破碎化 

2.如果沒有成功free掉 --> 記憶體洩漏 

在RTOS中 我們要求程式執行時間可預測，但是MELLOC在找尋空間記憶體區塊時，需要遍歷Heap的linklist，如果記憶體很亂，它可能找很久，這會導致你的系統反應時間忽快忽慢。

---

<br/>

```c
void obscure_syntax(void)
{
    int a = 5;
    int b = 7;
    int c;
    
    c = a+++b;
}
```

請問：

1.這行 c = a+++b; 合法嗎？編譯器會報錯嗎？

2.如果不報錯，執行完後，a, b, c 的值分別是多少？ （提示：編譯器是怎麼解讀這一串 +++ 的？）」

<br/>
<br/>

我的答案:

1.合法，不會報錯

2.a=6,b=7,c=13

c = a=a+1 +b = 6+7 = 13

正確答案:

2.a=6 b=7 c=12

#錯誤解析

a++ 是 後置遞增（Post-increment）。它的定義是：「先回傳變數當前的值給表達式使用，"等該行敘述結束後"，再把變數加 1」。

所以

c = a++ + b =12

a = a+1 = 6

---
<br/>
- 假設我們有一個結構 struct s，我們想定義一個新的型別名稱，用來表示『指向 struct s 的指標』。 這裡有兩種寫法：

```c
#define dPS struct s *
dPS p1, p2;
```

```c
typedef struct s * tPS;
tPS p3, p4;
```

請問：

1.這兩種寫法有什麼關鍵的差異？

2.在 p1, p2 和 p3, p4 這四個變數中，哪一個變數的型別其實不是指標？」

<br/>
<br/>

我的答案:

#define只是單純的文字變換，typedef是型別變換，所以

```c
#define dPS struct s *
dPS p1, p2; //等同於struct s * p1, p2; 
```

這時候p1會指向結構指標，p2只是實體結構變數 (Structure Instance)。

```c
typedef struct s * tPS;  
tPS p3, p4; //等同於struct s * p3 ; struct s * p4;
```

定義指標型別時，永遠優先選擇 typedef 。

--- 
<br/>
- 請看這段程式碼，原作者想要取得一個 『位元全為 1』 的變數（也就是 0 的一補數，1's complement of zero）：

```c
unsigned int zero = 0;
unsigned int compzero = 0xFFFF; /* 預期結果是 ...1111 */
```

<br/>
<br/>

我的答案:

1.在32bit上會變成0x0000FFFF

2.unsigned int compzero = ~0 ; 

---

<br/>

- 請用變數名稱 a，分別寫出以下 3 種 宣告：

  一個 指向整數的指標 (A pointer to an integer)。
  
  一個 包含 10 個指標的陣列，每個指標都指向一個整數 (An array of 10 pointers to integers)。
  
  一個 函式指標 (Function Pointer)，該函式接收一個整數參數，並回傳一個整數 (A pointer to a function that takes an integer as an    argument and returns an integer)。
  
 （請直接寫出三行 C 程式碼）」

<br/>
<br/>


我的答案:

1.int *a;

2.int * a[10];

3.int (*a)(int); #函式指標 (這題不熟)

---
<br/>

- 請告訴我，以下這兩個宣告有什麼具體差異？ 也就是說，在這兩種情況下，誰是唯讀的（Read-only）？誰是可以修改的？

  1.const int *ptr;

  2.int * const ptr;」

<br/>
<br/>

我的答案:

1. int唯讀,但可以更改*ptr的值
  
2. ptr唯讀,但可以更改int的值

#錯誤解析:
1. *ptr指向的值不能更改，但是可以改變*ptr指向的地址。

---

<br/>

- 在嵌入式系統的主程式（Main Loop）中，我們通常需要一個跑不停的無窮迴圈。

  請寫出你慣用的無窮迴圈寫法。 （提示：通常有 while 和 for 兩種流派，請選一種寫出來，並告訴我為什麼有些資深工程師或工具（如 Lint）會偏

  好 for 的寫法？） 」

<br/>
<br/>

我的答案:

while(1) {} 或 for(;;){} 

#補充

- 關鍵字：Lint（靜態分析工具）：

  許多靜態分析工具（如 PC-Lint）在檢查 while(1) 時，會跳出一個 Warning，叫作「Condition is always true（條件永遠為真）」。這雖然沒錯，但在追求 Zero Warning 的高品質專案中，這是個雜訊。

而 for(;;) 被 C 語言標準和 K&R 定義為正規的無窮迴圈寫法，工具通常會將其視為「Intended (故意的)」，不會報錯 。

---
  





   



