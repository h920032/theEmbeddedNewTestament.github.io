# 嵌入式系統的 C 語言基礎

> **嵌入式軟體開發的必備 C 程式設計概念**

## 📋 **目錄**
- [概述](#概述)
- [什麼是 C 程式設計？](#什麼是-c-程式設計)
- [為什麼嵌入式系統選擇 C？](#為什麼嵌入式系統選擇-c)
- [C 語言概念](#c-語言概念)
- [變數與資料型別](#變數與資料型別)
- [函式](#函式)
- [控制結構](#控制結構)
- [記憶體管理](#記憶體管理)
- [指標](#指標)
- [陣列與字串](#陣列與字串)
- [結構體與聯合體](#結構體與聯合體)
- [前處理器指令](#前處理器指令)
- [實作](#實作)
- [常見陷阱](#常見陷阱)
- [最佳實踐](#最佳實踐)
- [面試題目](#面試題目)

---

## 🎯 **概述**

C 是嵌入式系統的主要程式設計語言，原因如下：
- **直接硬體存取** - 能夠操作記憶體位址和暫存器
- **高效率** - 最小的執行時期額外負擔和可預測的效能
- **可攜性** - 可在不同的微控制器和架構上運作
- **成熟的生態系統** - 豐富的工具鏈、函式庫和社群支援

### **嵌入式開發的關鍵特徵**
- **靜態型別** - 編譯期型別檢查
- **手動記憶體管理** - 直接控制記憶體配置
- **低階存取** - 指標運算和位元操作
- **程序式程式設計** - 基於函式的程式碼組織

### **面試官意圖（他們在探查什麼）**
- 你能解釋為什麼 C 在嵌入式領域仍然占主導地位嗎？
- 你是否理解安全性 vs 控制性 vs 效能之間的取捨？
- 你能將語言特性與硬體行為連結起來嗎？

## 🤔 **什麼是 C 程式設計？**

C 是一種通用程式設計語言，由 Dennis Ritchie 於 1970 年代在貝爾實驗室開發。它被設計為一種簡潔、高效的語言，在保持跨不同電腦架構可攜性的同時，提供低階記憶體存取能力。

### **核心理念**

1. **簡潔性**：C 提供一組最小的功能，易於理解
2. **高效率**：C 程式碼可以被編譯為高效的機器碼，額外負擔極小
3. **可攜性**：C 程式可以在不同架構上編譯，只需最少的修改
4. **低階存取**：C 提供對記憶體和硬體功能的直接存取

### **語言特徵**

**優勢：**
- **效能**：接近組合語言的效率
- **控制力**：直接的記憶體和硬體存取
- **可攜性**：可在不同平台上運作
- **成熟度**：歷史悠久的語言，擁有豐富的工具支援

**限制：**
- **安全性**：沒有內建的記憶體安全或邊界檢查
- **複雜性**：手動記憶體管理容易出錯
- **抽象化**：有限的高階抽象
- **除錯**：執行時期錯誤可能難以除錯

### **C 與其他語言的比較**

```
嵌入式系統語言比較：

┌─────────────────┬─────────────┬─────────────┬─────────────────┐
│     語言        │    效能     │    安全性   │    學習曲線      │
├─────────────────┼─────────────┼─────────────┼─────────────────┤
│   C             │     高      │     低      │     中等        │
│   C++           │     高      │     中等    │      高         │
│   Rust          │     高      │     高      │      高         │
│   Python        │     低      │     高      │      低         │
│   Assembly      │    最高     │     低      │      高         │
└─────────────────┴─────────────┴─────────────┴─────────────────┘
```

## 🎯 **為什麼嵌入式系統選擇 C？**

### **歷史原因**

C 成為嵌入式系統的主導語言有以下幾個歷史因素：

1. **Unix 傳承**：C 與 Unix 一同開發，影響了早期的嵌入式系統
2. **編譯器技術**：C 編譯器是最早能產生高效程式碼的編譯器之一
3. **硬體存取**：C 的指標運算提供了直接的硬體存取能力
4. **標準化**：ANSI C 標準化提供了穩定性和可攜性

### **技術優勢**

**效能優點：**
- **最小的執行時期環境**：沒有垃圾回收或複雜的執行時期系統
- **可預測的效能**：確定性的執行時間
- **小型程式碼大小**：高效編譯為機器碼
- **直接硬體存取**：能夠操作暫存器和記憶體

**資源效率：**
- **記憶體使用**：最小的記憶體額外負擔
- **CPU 週期**：高效的指令產生
- **功耗**：因效率而降低功耗
- **即時能力**：可預測的時序特性

### **嵌入式特定優點**

**硬體整合：**
- **暫存器存取**：直接操作硬體暫存器
- **記憶體映射 I/O**：存取周邊暫存器
- **中斷處理**：低階中斷服務常式
- **DMA 程式設計**：直接記憶體存取程式設計

**系統控制：**
- **啟動程式碼**：系統初始化和啟動程式碼
- **設備驅動程式**：硬體抽象層實作
- **即時系統**：時間關鍵的應用程式開發
- **安全關鍵系統**：確定性行為需求

### **何時使用 C**

**使用 C 的時機：**
- **資源受限**：有限的記憶體或處理能力
- **即時需求**：可預測的時序至關重要
- **硬體存取**：需要直接控制硬體
- **舊有系統**：維護現有的 C 程式碼庫
- **效能關鍵**：需要最大效能

**考慮替代方案的時機：**
- **快速原型開發**：快速開發比效能更重要
- **安全關鍵**：需要內建的安全功能
- **複雜抽象**：需要高階抽象
- **團隊生產力**：開發者生產力比效能更重要

## 🧠 **C 語言概念**

### **程式設計範式**

C 主要是一種**程序式程式設計語言**，這意味著：

1. **基於函式**：程式碼被組織成函式
2. **由上而下設計**：程式從高階設計到低階
3. **資料與程式碼分離**：資料結構與函式分離
4. **逐步執行**：程式按順序執行指令

### **記憶體模型**

C 定義了一個抽象機器；實際的記憶體佈局是由實作定義的。嵌入式目標通常將程式碼放在 Flash/ROM 中，資料放在 RAM 中，連結器腳本控制區段的放置。

```
典型的嵌入式記憶體佈局（因目標/工具鏈而異）：
┌─────────────────────────────────────────────────────────────┐
│                    堆疊（區域變數）                          │
│                    ↓ 通常向下成長                            │
├─────────────────────────────────────────────────────────────┤
│                    堆積（動態記憶體）                        │
│                    ↑ 通常向上成長                            │
├─────────────────────────────────────────────────────────────┤
│                    .bss（零初始化資料）                      │
├─────────────────────────────────────────────────────────────┤
│                    .data（已初始化資料）                     │
├─────────────────────────────────────────────────────────────┤
│                    .text/.rodata（程式碼/常數）              │
└─────────────────────────────────────────────────────────────┘
```

### **編譯流程**

嵌入式目標上的 C 編譯成多個目的檔（翻譯單元），連結器將它們放置到連結器腳本定義的記憶體區域中。理解這個流程有助於閱讀映射檔和控制資料/程式碼的放置位置。

### 概念：翻譯單元、連結性和連結器腳本
- 一個原始檔 + 其標頭檔 → 一個翻譯單元 → 一個目的檔。
- 連結器合併目的檔和函式庫，解析外部符號。
- 連結器腳本將區段（`.text`、`.rodata`、`.data`、`.bss`）映射到 Flash/RAM。

### 試試看
1. 使用 `-Wl,-Map=out.map` 建置並開啟映射檔。找到一個 `static const` 表格與一個非 const 全域變數的位置。
2. 將一個符號從 `static` 改為非 `static`，觀察其可見性（外部連結性 vs 內部連結性）。

### 重點摘要
- 映射檔是你了解程式碼大小和放置位置的真實依據。
- 檔案作用域的 `static` 提供內部連結性；預設保持符號為區域性。
- 只有在必須控制放置位置時才使用區段/屬性；優先使用預設值。

C 程式在執行前經歷多個階段：

```
編譯流程：
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   原始碼    │ →  │   前處理    │ →  │    編譯     │ →  │    連結     │
│             │    │  （巨集）   │    │ （組合語言）│    │（可執行檔） │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
```

### **型別系統**

C 使用**靜態型別系統**搭配**弱型別**：

1. **靜態型別**：型別在編譯期檢查
2. **弱型別**：允許隱式型別轉換
3. **顯式轉型**：需要時手動型別轉換
4. **型別安全**：與現代語言相比，型別安全性有限

### **作用域與生命週期**

**作用域規則：**
- **檔案作用域**：在任何函式外宣告的識別字
- **區塊作用域**：在 `{}` 區塊內宣告的識別字（包括函式本體）
- **原型作用域**：函式原型中的參數名稱
- **函式作用域**：僅限標籤（用於 `goto`）

**生命週期規則：**
- **自動**：區域變數（基於堆疊）
- **靜態**：在函式呼叫之間持續存在的變數
- **動態**：手動配置的記憶體（基於堆積）
- **全域**：在整個程式期間存在的變數

## 🔢 **變數與資料型別**

### 概念：物件存放在哪裡？何時消亡？

與其死記型別，不如從儲存期間和生命週期的角度思考：誰擁有該物件、它被放在記憶體的哪裡、何時被初始化/銷毀。在 MCU 上，這些選擇會影響 RAM 使用量、啟動成本、確定性和安全性。

### 在嵌入式中為什麼重要
- 靜態物件可能由啟動程式碼零初始化，如果是 `const` 則可以存放在 Flash 中，減少 RAM 使用。
- 自動（堆疊）物件配置快速且具確定性，但預設未初始化。
- 動態（堆積）物件增加靈活性，但可能影響可預測性並造成記憶體碎片。

### 最小範例
```c
int g1;                 // 零初始化（`.bss`）
static int g2 = 42;     // 預先初始化（`.data`）

void f(void) {
  int a;                // 未初始化（堆疊，值不確定）
  static int b;         // 零初始化，跨呼叫保留值
  static const int lut[] = {1,2,3}; // 通常放在 Flash/ROM
  (void)a; (void)b; (void)lut;
}
```

### 試試看
1. 印出 `g1`、`g2`、一個區域變數和 `b` 的位址。檢查連結器映射檔以查看區段放置（`.text`、`.data`、`.bss`、堆疊）。
2. 將 `lut` 加大並觀察在有 `const` 和移除 `const` 時映射檔中 Flash 和 RAM 的使用情況。

### 重點摘要
- 只有靜態儲存期間保證零初始化。堆疊區域變數在賦值前為不確定值。
- 函式內的 `static` 的行為類似於具有函式作用域的全域變數。
- `const` 資料可能存放在非揮發性記憶體中；不要透過轉型移除 `const` 來寫入它。

> 平台備註：在 Cortex-M 上，大型零初始化物件會增加 `.bss` 並延長啟動清除時間；大型已初始化物件會增加從 Flash 到 RAM 的 `.data` 複製時間。

### **什麼是變數？**

變數是記憶體中有命名的儲存位置，可以存放資料。在 C 中，變數必須在使用前宣告，指定其型別並可選擇性地用一個值初始化。

### **變數概念**

**宣告 vs. 定義：**
- **宣告**：告知編譯器變數的型別和名稱
- **定義**：實際為變數配置記憶體
- **初始化**：為變數賦予初始值

**變數屬性：**
- **型別**：決定資料的大小和解釋方式
- **名稱**：用來存取變數的識別字
- **值**：儲存在變數中的資料
- **位址**：變數儲存的記憶體位置

### **資料型別分類**

**整數型別：**
- **有號**：可以表示正值和負值
- **無號**：只能表示正值
- **大小變體**：不同的位元寬度對應不同的範圍

**浮點型別：**
- **單精度**：32 位元浮點數
- **雙精度**：64 位元浮點數
- **IEEE 754**：浮點數表示的標準格式

**字元型別：**
- **char**：通常為 8 位元字元或小整數
- **字元編碼**：ASCII、UTF-8 或其他編碼
- **字串表示**：字元陣列

### **基本資料型別**

#### **整數型別**
```c
// 有號整數
int8_t   small_int;    // 8 位元有號（-128 到 127）
int16_t  medium_int;   // 16 位元有號（-32768 到 32767）
int32_t  large_int;    // 32 位元有號（-2^31 到 2^31-1）
int64_t  huge_int;     // 64 位元有號

// 無號整數
uint8_t  small_uint;   // 8 位元無號（0 到 255）
uint16_t medium_uint;  // 16 位元無號（0 到 65535）
uint32_t large_uint;   // 32 位元無號（0 到 2^32-1）
uint64_t huge_uint;    // 64 位元無號

// 傳統 C 型別（嵌入式中避免使用）
int      platform_dependent;  // 大小因平台而異
long     also_variable;       // 大小因平台而異
```

#### **浮點型別**
```c
float    single_precision;    // 通常為 32 位元 IEEE 754
double   double_precision;    // 由實作定義（32 或 64 位元）
```

#### **字元型別**
```c
char     character;           // 通常為 8 位元
uint8_t  byte_data;          // 明確的 8 位元無號
```

### **變數宣告與初始化**

#### **最佳實踐**
```c
// ✅ 好的做法：明確初始化
uint32_t counter = 0;
uint8_t status = 0xFF;
float temperature = 25.5f;

// ❌ 不好的做法：未初始化的變數
uint32_t counter;  // 包含垃圾資料
```

#### **常數**
```c
// 編譯期常數
#define MAX_BUFFER_SIZE 1024
#define PI 3.14159f

// 執行期常數
const uint32_t TIMEOUT_MS = 5000;
const float VOLTAGE_REFERENCE = 3.3f;

// 列舉常數
typedef enum {
    LED_OFF = 0,
    LED_ON = 1,
    LED_BLINK = 2
} led_state_t;
```

## 🔧 **函式**

### 概念：保持工作小型化，盡可能純粹，且可觀察

在嵌入式中，函式設計驅動可預測性和可測試性。優先使用具有明確輸入/輸出的小型、單一用途函式。除了定義明確的硬體介面（透過抽象層），避免隱藏的依賴性（全域變數）。

### 在嵌入式中為什麼重要
- 較小的函式改善堆疊使用估算和內聯機會。
- 純函式更容易在脫離目標的環境中進行單元測試。
- 明確的介面減少與硬體和時序的耦合。

### 最小範例：重構副作用
```c
// 之前：混合了 IO、計算和策略
void control_loop(void) {
  int raw = adc_read();
  float temp = convert_to_celsius(raw);
  if (temp > 30.0f) fan_on(); else fan_off();
}

// 之後：將 IO 與策略分離
float read_temperature_c(void) { return convert_to_celsius(adc_read()); }
bool fan_required(float temp_c) { return temp_c > 30.0f; }
void apply_fan(bool on) { if (on) fan_on(); else fan_off(); }
```

### 試試看
1. 為 `fan_required` 撰寫脫離目標（無硬體）的單元測試，以驗證閾值和遲滯。
2. 檢查呼叫點以確保高頻路徑仍然足夠小以便內聯。

### 重點摘要
- 將策略與機制分離以提高可測試性和可重用性。
- 最小化全域狀態；透過參數和回傳值傳遞資料。
- 對於熱路徑上非常小的輔助函式，考慮使用 `static inline`。

### **什麼是函式？**

函式是執行特定任務的可重複使用的程式碼區塊。它們是 C 程式設計中程式碼組織和重用的主要機制。

### **函式概念**

**函式組成：**
- **宣告**：函式原型（簽名）
- **定義**：函式實作（本體）
- **參數**：傳遞給函式的輸入資料
- **回傳值**：從函式回傳的輸出資料
- **作用域**：函式內可存取的變數和程式碼

**函式型別：**
- **標準函式庫函式**：內建函式（printf、malloc 等）
- **使用者自定義函式**：由程式設計師撰寫的函式
- **主函式**：程式的進入點
- **回呼函式**：作為參數傳遞的函式

### **函式設計原則**

**單一職責：**
- 每個函式應該做好一件事
- 函式應該聚焦且具有內聚性
- 避免函式做太多事情

**參數設計：**
- 使用參數作為輸入資料
- 使用回傳值作為輸出資料
- 避免使用全域變數進行函式間通訊

**錯誤處理：**
- 回傳錯誤碼以表示失敗狀況
- 使用一致的錯誤處理模式
- 記錄錯誤條件和回傳值

### **函式實作**

#### **基本函式結構**
```c
// 函式宣告（原型）
return_type function_name(parameter_list);

// 函式定義
return_type function_name(parameter_list) {
    // 函式本體
    // 區域變數
    // 陳述式
    return value;  // 選擇性的
}
```

#### **函式範例**
```c
// 無參數的簡單函式
void initialize_system(void) {
    // 系統初始化程式碼
    configure_clocks();
    setup_peripherals();
    enable_interrupts();
}

// 有參數和回傳值的函式
uint32_t calculate_average(uint32_t* values, size_t count) {
    if (count == 0) return 0;
    
    uint32_t sum = 0;
    for (size_t i = 0; i < count; i++) {
        sum += values[i];
    }
    return sum / count;
}

// 有多個回傳點的函式
bool validate_sensor_data(uint16_t value, uint16_t min, uint16_t max) {
    if (value < min) return false;
    if (value > max) return false;
    return true;
}
```

## 🔄 **控制結構**

### 概念：優先使用提前回傳和淺層巢狀

深層巢狀的分支會增加循環複雜度和 MCU 上的程式碼大小。使用守衛子句的提前回傳可以使關鍵路徑更明顯，並減少錯誤路徑中的堆疊壓力。

### 最小範例
```c
// 巢狀式
bool handle_packet(const pkt_t* p) {
  if (p) {
    if (valid_crc(p)) {
      if (!seq_replay(p)) { process(p); return true; }
    }
  }
  return false;
}

// 守衛式
bool handle_packet(const pkt_t* p) {
  if (!p) return false;
  if (!valid_crc(p)) return false;
  if (seq_replay(p)) return false;
  process(p);
  return true;
}
```

### 重點摘要
- 淺層巢狀改善可讀性和時序分析。
- 對密集分派使用 switch；避免貫穿（fall-through），除非是刻意的且有文件記錄。
- 在 ISR 相關程式碼中，保持分支簡短並避免沒有明確界限的迴圈。

### **什麼是控制結構？**

控制結構決定程式執行的流程。它們允許程式做出決策、重複操作並組織程式碼執行。

### **控制結構概念**

**決策制定：**
- **條件執行**：根據條件執行程式碼
- **布林邏輯**：用於決策的真/假條件
- **巢狀條件**：複雜的決策樹
- **預設動作**：條件不滿足時的備援行為

**迴圈：**
- **迭代**：多次重複操作
- **迴圈控制**：啟動、繼續和停止迴圈
- **迴圈變數**：控制迴圈執行的變數
- **無窮迴圈**：無限運行的迴圈（通常是錯誤）

**流程控制：**
- **循序執行**：程式碼按順序執行
- **分支**：跳轉到不同的程式碼區段
- **提前離開**：提前退出函式或迴圈
- **例外處理**：管理錯誤狀況

### **決策結構**

#### **if-else 陳述式**
```c
// 簡單的 if 陳述式
if (temperature > 30.0f) {
    turn_on_fan();
}

// if-else 陳述式
if (battery_level > 20) {
    normal_operation();
} else {
    low_power_mode();
}

// 巢狀 if-else
if (sensor_status == SENSOR_OK) {
    if (temperature > threshold) {
        activate_cooling();
    } else {
        deactivate_cooling();
    }
} else {
    handle_sensor_error();
}
```

#### **switch 陳述式**
```c
// 多條件的 switch 陳述式
switch (button_pressed) {
    case BUTTON_UP:
        increase_volume();
        break;
    case BUTTON_DOWN:
        decrease_volume();
        break;
    case BUTTON_SELECT:
        select_option();
        break;
    default:
        // 忽略未知按鍵
        break;
}
```

### **迴圈結構**

#### **for 迴圈**
```c
// 傳統的 for 迴圈
for (int i = 0; i < 10; i++) {
    process_data(i);
}

// 嵌入式風格的 for 迴圈
for (uint32_t i = 0; i < BUFFER_SIZE; i++) {
    buffer[i] = 0;  // 初始化緩衝區
}

// 無窮迴圈（嵌入式系統中常見）
for (;;) {
    process_events();
    update_display();
    delay_ms(100);
}
```

#### **while 迴圈**
```c
// 條件檢查迴圈
while (data_available()) {
    process_data();
}

// 帶 break 的無窮迴圈
while (1) {
    if (shutdown_requested()) {
        break;
    }
    main_loop();
}
```

#### **do-while 迴圈**
```c
// 至少執行一次
do {
    read_sensor();
} while (sensor_error());
```

## 💾 **記憶體管理**

### **什麼是記憶體管理？**

C 中的記憶體管理涉及配置、使用和釋放記憶體資源。與高階語言不同，C 需要手動記憶體管理，給予程式設計師直接控制權，但也帶來記憶體安全的責任。

### **記憶體管理概念**

**記憶體類型：**
- **堆疊記憶體**：區域變數的自動配置
- **堆積記憶體**：使用 malloc/free 的動態配置（如果啟用）
- **靜態儲存**：全域和靜態變數
- **Flash/ROM**：唯讀程式碼和 const 資料（平台概念）

**記憶體生命週期：**
- **配置**：向系統請求記憶體
- **使用**：讀取和寫入已配置的記憶體
- **釋放**：將記憶體歸還給系統
- **重用**：記憶體釋放後可以重新配置

**記憶體安全：**
- **邊界檢查**：確保記憶體存取在已配置的區域內
- **釋放後使用**：在記憶體被釋放後存取它
- **記憶體洩漏**：未能釋放已配置的記憶體
- **重複釋放**：對同一記憶體釋放兩次

### **堆疊 vs. 堆積**

**堆疊記憶體：**
- **自動配置**：變數自動配置
- **LIFO 順序**：後進先出配置模式
- **快速存取**：直接記憶體存取
- **有限大小**：堆疊大小通常很小
- **基於作用域**：作用域結束時釋放記憶體

**堆積記憶體：**
- **手動配置**：使用 malloc/calloc 明確配置
- **彈性大小**：可以配置大量記憶體
- **較慢存取**：間接記憶體存取
- **手動釋放**：必須明確釋放記憶體
- **碎片化**：隨時間可能變得碎片化

### **記憶體管理實作**

#### **堆疊記憶體**
```c
void stack_example(void) {
    // 堆疊配置的變數
    uint32_t local_var = 42;
    uint8_t buffer[256];
    struct sensor_data data;
    
    // 函式回傳時記憶體自動釋放
}
```

#### **堆積記憶體**
```c
void heap_example(void) {
    // 配置記憶體
    uint8_t* buffer = malloc(1024);
    if (buffer == NULL) {
        // 處理配置失敗
        return;
    }
    
    // 使用記憶體
    memset(buffer, 0, 1024);
    
    // 釋放記憶體
    free(buffer);
    buffer = NULL;  // 防止釋放後使用
}
```

#### **實務情境：堆疊 vs 堆積比較**

```c
/*
 * 堆疊 vs 堆積：何時使用哪個
 * 
 * 堆疊：小型、固定大小、短生命週期的資料
 * 堆積：大型、可變大小、或超出函式生命週期的資料
 */

// ✅ 堆疊：用於本地處理的小型固定緩衝區
void process_sensor(void) {
    uint8_t raw[8];           // 堆疊上 8 位元組 - 快速、自動
    read_sensor(raw, 8);
    uint16_t value = (raw[0] << 8) | raw[1];
    send_value(value);
}   // raw 在此自動釋放

// ✅ 堆積：大型緩衝區或回傳給呼叫者的資料
uint8_t* allocate_frame_buffer(size_t width, size_t height) {
    size_t size = width * height * 3;  // RGB
    uint8_t* fb = malloc(size);
    if (fb) memset(fb, 0, size);
    return fb;  // 呼叫者必須釋放
}
```

#### **常見陷阱與程式碼範例**

**陷阱 1：回傳堆疊位址（懸空指標）**
```c
// ❌ 錯誤：回傳堆疊記憶體的位址
uint8_t* bad_get_buffer(void) {
    uint8_t tmp[64];
    fill_buffer(tmp);
    return tmp;  // 未定義行為 - tmp 在回傳後已消失
}

// ✅ 修正：使用呼叫者提供的緩衝區或堆積
void good_get_buffer(uint8_t* out, size_t len) {
    fill_buffer(out);  // 呼叫者擁有記憶體
}

uint8_t* good_get_buffer_heap(size_t len) {
    uint8_t* buf = malloc(len);
    if (buf) fill_buffer(buf);
    return buf;  // 呼叫者必須釋放
}
```

**陷阱 2：錯誤路徑中的記憶體洩漏**
```c
// ❌ 錯誤：如果第二次配置失敗則記憶體洩漏
int bad_init(void) {
    ctx->buf1 = malloc(1024);
    if (!ctx->buf1) return -1;
    
    ctx->buf2 = malloc(2048);
    if (!ctx->buf2) return -1;  // 洩漏：buf1 未被釋放！
    
    return 0;
}

// ✅ 修正：錯誤時清理
int good_init(void) {
    ctx->buf1 = malloc(1024);
    if (!ctx->buf1) return -1;
    
    ctx->buf2 = malloc(2048);
    if (!ctx->buf2) {
        free(ctx->buf1);        // 清理第一次配置
        ctx->buf1 = NULL;
        return -1;
    }
    return 0;
}
```

**陷阱 3：釋放後使用**
```c
// ❌ 錯誤：存取已釋放的記憶體
void bad_cleanup(msg_t* msg) {
    free(msg->payload);
    log("Freed %zu bytes", msg->payload_len);  // 到目前為止還好
    
    // ... 稍後的程式碼 ...
    if (msg->payload[0] == 0xAA) { }  // 釋放後使用！payload 已被釋放
}

// ✅ 修正：釋放後設為 NULL，使用前檢查
void good_cleanup(msg_t* msg) {
    free(msg->payload);
    msg->payload = NULL;    // 防止意外重用
    msg->payload_len = 0;
}
```

**陷阱 4：重複釋放**
```c
// ❌ 錯誤：對同一記憶體釋放兩次
void bad_reset(void) {
    free(global_buf);
    // ... 其他程式碼 ...
    free(global_buf);  // 重複釋放 - 未定義行為
}

// ✅ 修正：釋放後設為 NULL
void good_reset(void) {
    free(global_buf);
    global_buf = NULL;
    // ... 其他程式碼 ...
    free(global_buf);  // 安全：free(NULL) 是無操作
}
```

**陷阱 5：堆疊溢位（大型區域陣列）**
```c
// ❌ 風險：堆疊上的大型陣列（可能溢出小型嵌入式堆疊）
void bad_process_image(void) {
    uint8_t frame[320 * 240];  // 堆疊上 76KB！
    capture_frame(frame);
}

// ✅ 更安全：對大型緩衝區使用靜態或堆積
static uint8_t frame_buffer[320 * 240];  // 在 .bss 中，不在堆疊

void good_process_image(void) {
    capture_frame(frame_buffer);
}
```

## 🎯 **指標**

### **什麼是指標？**

指標是儲存記憶體位址的變數。它們提供對資料的間接存取，是 C 程式設計的基礎，尤其在需要直接記憶體操作的嵌入式系統中。

### **指標概念**

**位址與值：**
- **位址**：資料儲存的記憶體位置
- **值**：儲存在記憶體位置的資料
- **指標變數**：儲存位址的變數
- **解參考**：存取位址處的值

**指標型別：**
- **資料指標**：指向變數和資料結構
- **函式指標**：指向函式
- **泛型指標（void 指標）**：可以指向任何型別的通用指標
- **空指標**：表示「無位址」的特殊指標值

**指標運算：**
- **遞增/遞減**：移動到下一個/上一個記憶體位置
- **加法/減法**：移動多個記憶體位置
- **比較**：比較記憶體位址
- **陣列關係**：陣列和指標密切相關

### **指標實作**

#### **基本指標操作**
```c
// 指標宣告和初始化
uint32_t value = 42;
uint32_t* ptr = &value;  // 取址運算子

// 解參考
uint32_t retrieved = *ptr;  // 解參考運算子

// 指標運算
uint32_t array[5] = {1, 2, 3, 4, 5};
uint32_t* array_ptr = array;

// 存取元素
uint32_t first = *array_ptr;      // array[0]
uint32_t second = *(array_ptr + 1); // array[1]
uint32_t third = array_ptr[2];    // array[2]
```

#### **實務嵌入式範例**

**記憶體映射暫存器存取**
```c
// 透過指標直接操作硬體暫存器
#define GPIO_BASE   0x40020000u
#define GPIO_MODER  (*(volatile uint32_t*)(GPIO_BASE + 0x00))
#define GPIO_ODR    (*(volatile uint32_t*)(GPIO_BASE + 0x14))
#define GPIO_IDR    (*(volatile uint32_t*)(GPIO_BASE + 0x10))

void gpio_set_output(uint8_t pin) {
    GPIO_MODER &= ~(3u << (pin * 2));   // 清除模式位元
    GPIO_MODER |= (1u << (pin * 2));    // 設定輸出模式
}

void gpio_write(uint8_t pin, uint8_t val) {
    if (val) GPIO_ODR |= (1u << pin);
    else     GPIO_ODR &= ~(1u << pin);
}
```

**指標運算：型別很重要**
```c
/*
 * 關鍵見解：ptr + 1 會前進 sizeof(*ptr) 個位元組
 * 
 * uint8_t*  + 1 = +1 位元組
 * uint16_t* + 1 = +2 位元組
 * uint32_t* + 1 = +4 位元組
 */
void demonstrate_pointer_arithmetic(void) {
    uint8_t  buf[16];
    
    uint8_t*  p8  = buf;
    uint16_t* p16 = (uint16_t*)buf;
    uint32_t* p32 = (uint32_t*)buf;
    
    // 全部從相同位址開始
    // p8  = 0x2000
    // p16 = 0x2000
    // p32 = 0x2000
    
    p8++;   // p8  = 0x2001（+1 位元組）
    p16++;  // p16 = 0x2002（+2 位元組）
    p32++;  // p32 = 0x2004（+4 位元組）
}
```

**實務：解析協定封包**
```c
/*
 * 解析：[SYNC:1][LEN:2][CMD:1][PAYLOAD:n][CRC:2]
 * 這是嵌入式協定（如 UART 框架）的解析方式
 */
typedef struct {
    uint8_t  cmd;
    uint16_t len;
    uint8_t* payload;
    uint16_t crc;
} packet_t;

bool parse_packet(uint8_t* raw, size_t raw_len, packet_t* pkt) {
    uint8_t* p = raw;
    uint8_t* end = raw + raw_len;
    
    // 檢查最小大小
    if (raw_len < 6) return false;
    
    // 解析 SYNC
    if (*p++ != 0xAA) return false;
    
    // 解析 LEN（小端序 16 位元）
    pkt->len = p[0] | (p[1] << 8);
    p += 2;
    
    // 存取 payload 前的邊界檢查
    if (p + pkt->len + 3 > end) return false;
    
    // 解析 CMD
    pkt->cmd = *p++;
    
    // Payload 指標（不複製，只引用）
    pkt->payload = p;
    p += pkt->len;
    
    // 解析 CRC
    pkt->crc = p[0] | (p[1] << 8);
    
    return true;
}
```

**指標比較與邊界檢查**
```c
/*
 * 帶邊界檢查的安全緩衝區迭代
 * 環形緩衝區和 DMA 的常見模式
 */
void safe_buffer_copy(uint8_t* dst, const uint8_t* src, 
                      size_t len, size_t dst_size) {
    const uint8_t* src_end = src + len;
    uint8_t* dst_end = dst + dst_size;
    
    while (src < src_end && dst < dst_end) {
        *dst++ = *src++;
    }
}

// 帶環繞的環形緩衝區讀取
size_t ring_read(ring_t* r, uint8_t* out, size_t max) {
    size_t count = 0;
    uint8_t* end = r->buf + r->size;  // 最後一個有效位置的下一個
    
    while (count < max && r->head != r->tail) {
        *out++ = *r->tail++;
        if (r->tail >= end) {
            r->tail = r->buf;  // 環繞到開頭
        }
        count++;
    }
    return count;
}
```

**多位元組存取模式（位元組序感知）**
```c
/*
 * 協定緩衝區的可移植多位元組讀取/寫入
 * 避免對齊問題，無論 CPU 位元組序都能正常運作
 */

// 從位元組緩衝區讀取 16 位元小端序
static inline uint16_t read_le16(const uint8_t* p) {
    return (uint16_t)p[0] | ((uint16_t)p[1] << 8);
}

// 從位元組緩衝區讀取 32 位元大端序（網路序）
static inline uint32_t read_be32(const uint8_t* p) {
    return ((uint32_t)p[0] << 24) | ((uint32_t)p[1] << 16) |
           ((uint32_t)p[2] << 8)  | (uint32_t)p[3];
}

// 將 16 位元小端序寫入位元組緩衝區
static inline void write_le16(uint8_t* p, uint16_t v) {
    p[0] = (uint8_t)(v & 0xFF);
    p[1] = (uint8_t)(v >> 8);
}

// 使用方式：建構回應封包
void build_response(uint8_t* buf, uint16_t seq, uint32_t value) {
    buf[0] = 0xAA;                  // 同步位元組
    write_le16(buf + 1, seq);       // 序列號
    buf[3] = 0x02;                  // 命令
    write_le16(buf + 4, (uint16_t)value);  // 酬載
}
```

**泛型指標與型別轉換**
```c
/*
 * void* 是「泛型」指標 - 可以指向任何型別
 * 使用前必須轉型才能解參考
 * 常見於回呼、記憶體配置器和 HAL API
 */

// 泛型比較回呼（類似 qsort）
typedef int (*compare_fn)(const void*, const void*);

int compare_uint16(const void* a, const void* b) {
    uint16_t va = *(const uint16_t*)a;
    uint16_t vb = *(const uint16_t*)b;
    return (va > vb) - (va < vb);
}

// 泛型記憶體池
void* pool_alloc(pool_t* pool, size_t size) {
    if (pool->free + size > pool->end) return NULL;
    void* ptr = pool->free;
    pool->free += size;
    return ptr;
}

// 使用方式
sensor_t* s = (sensor_t*)pool_alloc(&pool, sizeof(sensor_t));
```

#### **函式指標**
```c
// 函式指標型別
typedef void (*callback_t)(uint32_t);

// 接受回呼的函式
void process_data(uint32_t data, callback_t callback) {
    // 處理資料
    if (callback != NULL) {
        callback(data);
    }
}

// 回呼函式
void data_handler(uint32_t data) {
    printf("Received: %u\n", data);
}

// 使用方式
process_data(42, data_handler);
```

## 📊 **陣列與字串**

### 心智模型：陣列是區塊；指標是帶有意圖的位址

陣列名稱在表達式中會退化為指向其第一個元素的指標。陣列本身具有固定大小，存放在定義它的位置（堆疊、`.bss`、`.data`）。指標只是一個可以指向任何地方且可以重新指向的位址。

### 在嵌入式中為什麼重要
- 了解退化何時發生可以防止 `sizeof` 和參數傳遞的錯誤。
- 以 `static const` 放置在 Flash 中的陣列看起來像普通陣列但是唯讀的；只有對記憶體映射暫存器或由硬體/ISR 修改的資料才使用 `volatile`。

### 最小範例：退化與 sizeof
```c
static uint8_t table[16];

size_t size_in_caller = sizeof table;      // 16

void use_array(uint8_t *p) {
  size_t size_in_callee = sizeof p;        // 指標大小，不是陣列大小
  (void)size_in_callee;
}

void demo(void) {
  use_array(table);                        // 陣列退化為 uint8_t*
}
```

### 試試看
1. 在定義作用域和被呼叫函式參數內印出 `sizeof table`。
2. 將參數改為 `uint8_t a[16]` 並觀察在被呼叫函式中它仍然是指標。
3. 建立 `static const uint16_t lut[] = { ... }` 並透過映射檔驗證它是否存放在 Flash/ROM 中。

### 重點摘要
- 陣列不是指標；它們在大多數表達式邊界退化為指標。
- `sizeof(param)` 在函式內（其中 `param` 被宣告為 `type param[]`）會得到指標大小。
- 優先傳遞 `(ptr, length)` 配對，或包裝在 `struct` 中以保留大小資訊。

> 交叉參閱：請參閱 `Type_Qualifiers.md` 了解記憶體映射區域的 `const/volatile`，以及 `Structure_Alignment.md` 了解佈局影響。

### **什麼是陣列？**

陣列是相同型別元素的集合，儲存在連續的記憶體位置。它們提供對多個相關資料項目的高效存取。

### **陣列概念**

**陣列特徵：**
- **連續記憶體**：元素儲存在相鄰的記憶體位置
- **索引存取**：透過數字索引存取元素
- **固定大小**：大小在宣告時決定（在 C 中）
- **型別同質性**：所有元素必須是相同型別

**陣列操作：**
- **走訪**：按順序存取所有元素
- **搜尋**：尋找特定元素
- **排序**：按順序排列元素
- **修改**：更改元素值

**陣列限制：**
- **固定大小**：宣告後不能更改大小
- **無邊界檢查**：C 不檢查陣列邊界
- **記憶體浪費**：可能配置比需要更多的記憶體
- **無內建操作**：沒有內建的搜尋、排序等

### **字串概念**

**字串表示：**
- **空字元結尾**：字串以 '\0' 字元結尾
- **字元陣列**：字串是字元的陣列
- **長度計算**：必須計算字元數以找到長度
- **記憶體管理**：必須配置足夠的空間

**字串操作：**
- **串接**：組合字串
- **比較**：比較字串內容
- **搜尋**：尋找子字串
- **複製**：複製字串

### **陣列與字串實作**

#### **陣列操作**
```c
// 陣列宣告和初始化
uint32_t numbers[5] = {1, 2, 3, 4, 5};

// 陣列走訪
for (size_t i = 0; i < 5; i++) {
    printf("Element %zu: %u\n", i, numbers[i]);
}

// 陣列作為函式參數
void process_array(uint32_t* array, size_t size) {
    for (size_t i = 0; i < size; i++) {
        array[i] *= 2;  // 每個元素加倍
    }
}
```

#### **字串操作**
```c
// 字串宣告
char message[] = "Hello, World!";

// 字串長度計算
size_t length = 0;
while (message[length] != '\0') {
    length++;
}

// 字串複製
char destination[20];
size_t i = 0;
while (message[i] != '\0') {
    destination[i] = message[i];
    i++;
}
destination[i] = '\0';  // 加上空字元結尾
```

## 🏗️ **結構體與聯合體**

### **什麼是結構體？**

結構體是使用者自定義的資料型別，將不同型別的相關資料項目組合成一個單元。它們提供了在嵌入式系統中組織複雜資料的方式。

### **結構體概念**

**結構體組成：**
- **成員**：結構體內的個別資料項目
- **佈局**：成員在記憶體中的排列方式
- **對齊**：記憶體對齊需求
- **大小**：結構體佔用的總記憶體

**結構體用途：**
- **資料組織**：將相關資料組合在一起
- **函式參數**：傳遞多個相關的值
- **回傳值**：從函式回傳多個值
- **記憶體映射**：將結構體映射到硬體暫存器

**結構體設計：**
- **內聚性**：成員應該在邏輯上相關
- **大小最佳化**：最小化記憶體使用
- **對齊**：考慮記憶體對齊需求
- **存取模式**：為高效存取而設計

### **聯合體概念**

**聯合體特徵：**
- **共享記憶體**：所有成員共享相同的記憶體位置
- **單一值**：同一時間只能有一個成員是活動的
- **記憶體效率**：僅使用最大成員的大小
- **型別彈性**：可以表示不同的資料型別

**聯合體應用：**
- **型別轉換**：在不同資料型別之間轉換
- **記憶體效率**：只需要一種型別時節省記憶體
- **硬體存取**：以不同方式存取硬體暫存器
- **協定實作**：處理不同的訊息型別

### **結構體與聯合體實作**

#### **結構體範例**
```c
// 基本結構體
typedef struct {
    uint32_t id;
    float temperature;
    uint8_t status;
} sensor_data_t;

// 帶位元欄位的結構體
typedef struct {
    uint8_t red : 3;    // 紅色 3 位元
    uint8_t green : 3;  // 綠色 3 位元
    uint8_t blue : 2;   // 藍色 2 位元
} rgb_color_t;

// 帶函式指標的結構體
typedef struct {
    uint32_t (*read)(void);
    void (*write)(uint32_t value);
    uint32_t address;
} hardware_register_t;
```

#### **聯合體範例**
```c
// 用於型別轉換的聯合體
typedef union {
    uint32_t as_uint32;
    uint8_t as_bytes[4];
    float as_float;
} data_converter_t;

// 用於協定訊息的聯合體
typedef union {
    struct {
        uint8_t type;
        uint8_t length;
        uint8_t data[32];
    } message;
    uint8_t raw[34];
} protocol_message_t;
```
> 注意：透過聯合體進行型別雙關在 C 中是由實作定義的。為了嚴格的可攜性，使用 `memcpy` 在物件表示之間移動。

## 🔧 **前處理器指令**

> 準則：保持巨集最小化和區域化。為了型別安全、可除錯性和更好的編譯器分析，優先使用 `static inline` 函式，除非你真正需要語彙黏接/字串化或編譯期分支。

### **什麼是前處理器指令？**

前處理器指令是給 C 前處理器的指示，在編譯之前處理。它們提供文字替換、條件編譯和檔案引入功能。

### **前處理器概念**

**文字替換：**
- **巨集**：編譯前的文字替換
- **常數**：定義編譯期常數
- **類函式巨集**：接受參數的巨集
- **字串化**：將參數轉換為字串

**條件編譯：**
- **平台特定程式碼**：不同平台的不同程式碼
- **除錯程式碼**：僅在除錯建置中包含除錯程式碼
- **功能旗標**：在編譯期啟用/停用功能
- **標頭檔保護**：防止標頭檔被多次引入

**檔案管理：**
- **標頭檔引入**：引入其他檔案的宣告
- **檔案組織**：將介面與實作分離
- **相依性管理**：管理檔案相依性
- **模組化設計**：將程式碼組織成模組

### **前處理器實作**

#### **巨集定義**
```c
// 簡單巨集
#define MAX_BUFFER_SIZE 1024
#define PI 3.14159f

// 類函式巨集
#define MIN(a, b) ((a) < (b) ? (a) : (b))
#define ABS(x) ((x) < 0 ? -(x) : (x))

// 多行巨集
#define INIT_SENSOR(sensor, id, threshold) \
    do { \
        sensor.id = id; \
        sensor.threshold = threshold; \
        sensor.status = SENSOR_INACTIVE; \
    } while(0)
```

#### **條件編譯**
```c
// 平台特定程式碼
#ifdef ARM_CORTEX_M4
    #define CPU_FREQUENCY 168000000
#elif defined(ARM_CORTEX_M3)
    #define CPU_FREQUENCY 72000000
#else
    #define CPU_FREQUENCY 16000000
#endif

// 除錯程式碼
#ifdef DEBUG
    #define DEBUG_PRINT(msg) printf("DEBUG: %s\n", msg)
#else
    #define DEBUG_PRINT(msg) ((void)0)
#endif
```

## 🔧 **實作**

### **完整程式範例**

```c
#include <stdint.h>
#include <stdbool.h>

// 常數
#define MAX_SENSORS 8
#define TEMPERATURE_THRESHOLD 30.0f

// 資料結構
typedef struct {
    uint32_t id;
    float temperature;
    bool active;
} sensor_t;

typedef struct {
    sensor_t sensors[MAX_SENSORS];
    uint8_t sensor_count;
    bool system_active;
} system_state_t;

// 函式原型
void initialize_system(system_state_t* state);
void read_sensors(system_state_t* state);
void process_data(system_state_t* state);
void update_outputs(system_state_t* state);

// 主函式
int main(void) {
    system_state_t system;
    
    // 初始化系統
    initialize_system(&system);
    
    // 主迴圈
    while (system.system_active) {
        read_sensors(&system);
        process_data(&system);
        update_outputs(&system);
    }
    
    return 0;
}

// 函式實作
void initialize_system(system_state_t* state) {
    state->sensor_count = 0;
    state->system_active = true;
    
    // 初始化感測器
    for (uint8_t i = 0; i < MAX_SENSORS; i++) {
        state->sensors[i].id = i;
        state->sensors[i].temperature = 0.0f;
        state->sensors[i].active = false;
    }
}

void read_sensors(system_state_t* state) {
    for (uint8_t i = 0; i < state->sensor_count; i++) {
        if (state->sensors[i].active) {
            // 模擬感測器讀取
            state->sensors[i].temperature = 25.0f + (i * 2.0f);
        }
    }
}

void process_data(system_state_t* state) {
    for (uint8_t i = 0; i < state->sensor_count; i++) {
        if (state->sensors[i].active && 
            state->sensors[i].temperature > TEMPERATURE_THRESHOLD) {
            // 處理高溫
            activate_cooling();
        }
    }
}

void update_outputs(system_state_t* state) {
    // 根據處理後的資料更新系統輸出
    update_display();
    send_status_report();
}
```

## ⚠️ **常見陷阱**

### **1. 未初始化的變數**

**問題**：在變數初始化前使用它們
**解決方案**：始終初始化變數

```c
// ❌ 不好：未初始化的變數
uint32_t counter;
printf("Counter: %u\n", counter);  // 未定義行為

// ✅ 好的：已初始化的變數
uint32_t counter = 0;
printf("Counter: %u\n", counter);
```

### **2. 緩衝區溢位**

**問題**：寫入超出陣列邊界
**解決方案**：始終檢查陣列邊界

```c
// ❌ 不好：緩衝區溢位
uint8_t buffer[10];
for (int i = 0; i < 20; i++) {
    buffer[i] = 0;  // 緩衝區溢位！
}

// ✅ 好的：邊界檢查
uint8_t buffer[10];
for (int i = 0; i < 10; i++) {
    buffer[i] = 0;
}
```

### **3. 記憶體洩漏**

**問題**：不釋放已配置的記憶體
**解決方案**：始終釋放已配置的記憶體

```c
// ❌ 不好：記憶體洩漏
void bad_function(void) {
    uint8_t* buffer = malloc(1024);
    // 使用 buffer...
    // 忘記釋放了！
}

// ✅ 好的：正確清理
void good_function(void) {
    uint8_t* buffer = malloc(1024);
    if (buffer != NULL) {
        // 使用 buffer...
        free(buffer);
    }
}
```

### **4. 懸空指標**

**問題**：在記憶體釋放後使用指標
**解決方案**：釋放後將指標設為 NULL

```c
// ❌ 不好：懸空指標
uint8_t* ptr = malloc(100);
free(ptr);
*ptr = 42;  // 釋放後使用！

// ✅ 好的：釋放後設為空指標
uint8_t* ptr = malloc(100);
free(ptr);
ptr = NULL;  // 防止釋放後使用
```

## ✅ **最佳實踐**

### **1. 程式碼組織**

- **模組化設計**：將程式碼分解為邏輯模組
- **函式大小**：保持函式小型且聚焦
- **命名慣例**：使用一致的命名
- **文件**：為複雜的程式碼區段撰寫文件

### **2. 記憶體管理**

- **初始化**：始終初始化變數
- **邊界檢查**：檢查陣列邊界
- **記憶體清理**：釋放已配置的記憶體
- **空指標**：在解參考前檢查 NULL

### **3. 錯誤處理**

- **回傳值**：使用回傳值表示錯誤
- **錯誤碼**：定義一致的錯誤碼
- **優雅降級**：優雅地處理錯誤
- **記錄**：記錄錯誤以便除錯

### **4. 效能**

- **高效演算法**：選擇適當的演算法
- **記憶體使用**：最小化記憶體使用
- **迴圈最佳化**：最佳化關鍵迴圈
- **編譯器旗標**：使用適當的編譯器旗標

### **5. 安全性**

- **邊界檢查**：始終檢查陣列邊界
- **型別安全**：使用適當的資料型別
- **空值檢查**：使用前檢查指標
- **初始化**：初始化所有變數

## 🎯 **面試題目**

### **基礎題目**

1. **C 程式設計的關鍵特徵是什麼？**
   - 靜態型別、手動記憶體管理、低階存取
   - 程序式程式設計、直接硬體存取
   - 高效率、可攜性、成熟的生態系統

2. **堆疊記憶體和堆積記憶體有什麼區別？**
   - 堆疊：自動配置、LIFO、有限大小、基於作用域
   - 堆積：手動配置、彈性大小、較慢存取、手動釋放

3. **什麼是指標，為什麼它們在 C 中很重要？**
   - 指標儲存記憶體位址
   - 提供對資料的間接存取
   - 對動態記憶體配置至關重要
   - 實現高效的陣列和函式操作

### **進階題目**

1. **你如何在 C 中實作記憶體池？**
   - 預先配置固定大小區塊的記憶體
   - 維護可用區塊的空閒清單
   - 實作 O(1) 的配置和釋放
   - 優雅地處理池耗盡

2. **你如何設計回呼的函式指標系統？**
   - 定義函式指標型別
   - 將函式指標作為參數傳遞
   - 實作回呼註冊
   - 處理 NULL 函式指標

3. **你如何為嵌入式系統最佳化 C 程式？**
   - 使用適當的資料型別
   - 最小化記憶體使用
   - 最佳化關鍵迴圈
   - 使用編譯器最佳化

### **實作題目**

1. **撰寫一個原地反轉字串的函式**
2. **實作一個簡單的記憶體配置器**
3. **撰寫一個函式來找到第 n 個費波那契數**
4. **設計一個鏈結串列節點的結構體**

## 📚 **額外資源**

### **書籍**
- 《C 程式設計語言》Brian W. Kernighan 與 Dennis M. Ritchie 著
- 《C Programming: A Modern Approach》K.N. King 著
- 《Embedded C Coding Standard》Michael Barr 著

### **線上資源**
- [C 語言教學](https://www.tutorialspoint.com/cprogramming/)
- [C 標準函式庫參考](https://en.cppreference.com/w/c)
- [嵌入式 C 最佳實踐](https://www.embedded.com/)

### **工具**
- **GCC**：GNU 編譯器套件
- **Clang**：基於 LLVM 的編譯器
- **Valgrind**：記憶體分析工具
- **GDB**：GNU 除錯器

### **標準**
- **C11**：嵌入式工具鏈中廣泛使用
- **C17/C18**：C11 的錯誤修正修訂版
- **C23**：最新的 ISO C 標準（工具鏈支援程度不一）
- **MISRA C**：安全關鍵編碼標準

---

**下一步**：探索[記憶體管理](./Memory_Management.md)以了解記憶體配置策略，或深入[指標與記憶體位址](./Pointers_Memory_Addresses.md)了解低階記憶體操作。
