# 嵌入式系統的行內函式與巨集

> **在嵌入式 C 程式設計中使用行內函式與巨集進行效能最佳化技巧**

## 📋 **目錄**
- [概述](#概述)
- [什麼是行內函式與巨集？](#什麼是行內函式與巨集)
- [為什麼它們很重要？](#為什麼它們很重要)
- [最佳化概念](#最佳化概念)
- [行內函式](#行內函式)
- [類函式巨集](#類函式巨集)
- [條件編譯](#條件編譯)
- [效能考量](#效能考量)
- [除錯與安全性](#除錯與安全性)
- [實作](#實作)
- [常見陷阱](#常見陷阱)
- [最佳實踐](#最佳實踐)
- [面試問題](#面試問題)

---

## 🎯 **概述**

行內函式與巨集在嵌入式系統中不可或缺，用於：
- **效能最佳化** - 消除函式呼叫開銷
- **程式碼大小縮減** - 行內化小型、頻繁使用的函式
- **硬體抽象化** - 建立高效率的硬體存取函式
- **編譯時期最佳化** - 啟用編譯器最佳化
- **除錯控制** - 針對不同建置類型的條件編譯

### **核心概念**
- **行內函式** - 編譯器建議的函式行內化
- **類函式巨集** - 帶參數的文字替換
- **條件編譯** - 建置時期的程式碼選擇
- **型別安全** - 行內函式 vs 巨集的安全性
- **除錯考量** - 對除錯與效能分析的影響

## 🤔 **什麼是行內函式與巨集？**

行內函式與巨集是透過在呼叫端直接展開程式碼來消除函式呼叫開銷的程式碼最佳化技術。它們在效能與程式碼大小至關重要的嵌入式系統中特別重要。

### **核心概念**

**函式呼叫開銷：**
- **堆疊操作**：推入/彈出參數與返回地址
- **暫存器保存**：保留呼叫者保存的暫存器
- **分支指令**：跳轉至函式並返回
- **上下文切換**：在呼叫者與被呼叫者上下文之間切換

**程式碼展開：**
- **行內函式**：編譯器在呼叫端展開函式程式碼
- **巨集**：前處理器在編譯前執行文字替換
- **消除開銷**：不涉及函式呼叫機制
- **潛在大小增加**：程式碼可能在多個呼叫端重複

**最佳化策略：**
- **效能關鍵程式碼**：消除函式呼叫開銷
- **小型函式**：行內化頻繁使用的小型函式
- **硬體存取**：高效率的硬體暫存器操作
- **除錯控制**：針對不同建置的條件編譯

### **函式呼叫 vs. 行內展開**

**傳統函式呼叫：**
```
呼叫端：
    push parameter1
    push parameter2
    call function_name
    add esp, 8          ; 清理堆疊
    mov result, eax     ; 取得返回值

函式：
    push ebp
    mov ebp, esp
    ; 函式主體
    mov eax, result
    pop ebp
    ret
```

**行內展開：**
```
呼叫端：
    ; 函式主體直接插入此處
    mov eax, parameter1
    add eax, parameter2
    mov result, eax
```

## 🎯 **為什麼它們很重要？**

### **嵌入式系統需求**

**效能關鍵應用：**
- **即時系統**：可預測的時序需求
- **中斷處理器**：快速回應時間
- **硬體存取**：高效率的暫存器操作
- **信號處理**：高頻率操作

**資源限制：**
- **有限記憶體**：程式碼大小最佳化
- **電源效率**：減少 CPU 週期
- **快取效能**：更好的快取利用率
- **匯流排利用率**：高效率的記憶體存取

**硬體互動：**
- **暫存器存取**：直接硬體操作
- **位元操作**：高效率的位元操作
- **I/O 操作**：快速輸入/輸出操作
- **中斷控制**：快速中斷處理

### **實際影響**

**效能改善：**
```c
// 傳統函式呼叫（較慢）
uint32_t add_numbers(uint32_t a, uint32_t b) {
    return a + b;
}

// 行內函式（較快）
inline uint32_t add_numbers_inline(uint32_t a, uint32_t b) {
    return a + b;
}

// 在效能關鍵迴圈中使用
for (int i = 0; i < 1000000; i++) {
    result += add_numbers_inline(i, 1);  // 無函式呼叫開銷
}
```

**程式碼大小最佳化：**
```c
// 小型頻繁使用的函式
inline uint8_t get_lower_byte(uint32_t value) {
    return (uint8_t)(value & 0xFF);
}

// 多個呼叫端 - 在每個位置展開程式碼
uint8_t byte1 = get_lower_byte(data1);
uint8_t byte2 = get_lower_byte(data2);
uint8_t byte3 = get_lower_byte(data3);
```

**硬體抽象化：**
```c
// 高效率的硬體存取
inline void led_on(void) {
    *((volatile uint32_t*)0x40020014) |= (1 << 13);
}

inline void led_off(void) {
    *((volatile uint32_t*)0x40020014) &= ~(1 << 13);
}

// 使用 - 無函式呼叫開銷的直接硬體存取
led_on();   // 展開為直接暫存器操作
led_off();  // 展開為直接暫存器操作
```

### **何時使用行內函式與巨集**

**使用行內函式的時機：**
- **小型函式**：只有幾行程式碼的函式
- **頻繁呼叫**：被多次呼叫的函式
- **效能關鍵**：開銷很重要的程式碼
- **型別安全**：需要型別檢查和除錯支援

**使用巨集的時機：**
- **文字替換**：需要字面文字替換
- **條件編譯**：建置時期的程式碼選擇
- **硬體存取**：直接暫存器操作
- **跨平台**：需要針對不同平台的不同程式碼

**避免使用的時機：**
- **大型函式**：有很多行程式碼的函式
- **很少呼叫**：不常被呼叫的函式
- **複雜邏輯**：具有複雜控制流程的函式
- **除錯關鍵**：需要大量除錯的程式碼

## 🧠 **最佳化概念**

### **行內化如何運作**

**編譯器決策過程：**
1. **函式分析**：編譯器分析函式大小和複雜度
2. **呼叫端分析**：編譯器檢查函式如何被呼叫
3. **成本效益分析**：編譯器權衡行內化的利弊
4. **最佳化決策**：編譯器決定是否行內化

**行內化準則：**
- **函式大小**：小型函式更可能被行內化
- **呼叫頻率**：頻繁呼叫的函式是好的候選者
- **程式碼大小影響**：編譯器考慮整體程式碼大小的增加
- **效能影響**：編譯器估計效能改善

**編譯器最佳化：**
- **常量折疊**：編譯時期常量表達式的求值
- **死碼消除**：移除不可達的程式碼
- **暫存器分配**：行內化程式碼有更好的暫存器使用
- **指令排程**：改善指令排序

### **巨集展開過程**

**前處理器階段：**
1. **文字替換**：前處理器用文字替換巨集
2. **參數替換**：巨集參數被替換
3. **字串化**：參數可以轉換為字串
4. **符號黏貼**：符號可以被串接

**巨集 vs. 函式：**
- **巨集**：文字替換，無函式呼叫開銷
- **函式**：實際的函式呼叫帶有開銷
- **型別安全**：函式提供型別檢查，巨集不提供
- **除錯**：函式比巨集更容易除錯

### **效能特性**

**函式呼叫開銷：**
- **堆疊操作**：約 5-10 個週期
- **暫存器保存**：約 2-5 個週期
- **分支指令**：約 1-3 個週期
- **上下文切換**：約 2-5 個週期
- **總開銷**：每次呼叫約 10-23 個週期

**行內展開的好處：**
- **無堆疊操作**：消除堆疊開銷
- **無暫存器保存**：消除暫存器保存/恢復
- **無分支指令**：消除跳轉指令
- **更好的最佳化**：啟用更多編譯器最佳化

## ⚡ **行內函式**

### **什麼是行內函式？**

行內函式是編譯器可能在呼叫端展開而非產生函式呼叫的函式。它們提供巨集的好處（無函式呼叫開銷），同時保持型別安全和除錯支援。

### **行內函式概念**

**編譯器提示：**
- **inline 關鍵字**：建議編譯器應該行內化函式
- **always_inline 屬性**：強制編譯器行內化函式
- **編譯器分析**：編譯器根據最佳化準則做最終決定
- **大小限制**：編譯器可能不會行內化大型函式

**型別安全：**
- **型別檢查**：完整的 C 型別檢查和轉換
- **除錯支援**：函式出現在除錯器和堆疊追蹤中
- **錯誤訊息**：型別不匹配時有清楚的錯誤訊息
- **IDE 支援**：完整的 IDE 導航和重構支援

### **基本行內函式**

#### **簡單行內函式**
```c
// 基本行內函式
inline uint32_t square(uint32_t x) {
    return x * x;
}

// 多參數的行內函式
inline uint32_t multiply_add(uint32_t a, uint32_t b, uint32_t c) {
    return a * b + c;
}

// 使用
uint32_t result1 = square(5);           // 25
uint32_t result2 = multiply_add(2, 3, 4); // 10
```

#### **硬體存取函式**
```c
// 行內硬體暫存器存取
inline void gpio_set_pin(uint8_t pin) {
    volatile uint32_t* const GPIO_SET = (uint32_t*)0x40020008;
    *GPIO_SET = (1 << pin);
}

inline void gpio_clear_pin(uint8_t pin) {
    volatile uint32_t* const GPIO_CLEAR = (uint32_t*)0x4002000C;
    *GPIO_CLEAR = (1 << (pin + 16));
}

inline bool gpio_read_pin(uint8_t pin) {
    volatile uint32_t* const GPIO_DATA = (uint32_t*)0x40020000;
    return (*GPIO_DATA & (1 << pin)) != 0;
}

// 使用
gpio_set_pin(13);      // 設定 LED 腳位
bool state = gpio_read_pin(12);  // 讀取按鈕狀態
```

### **行內函式屬性**

#### **強制行內化**
```c
// 強制行內化（GCC/Clang）
inline __attribute__((always_inline)) uint32_t fast_multiply(uint32_t a, uint32_t b) {
    return a * b;
}

// 強制行內化（MSVC）
inline __forceinline uint32_t fast_multiply_msvc(uint32_t a, uint32_t b) {
    return a * b;
}

// 跨平台強制行內化
#ifdef __GNUC__
    #define FORCE_INLINE inline __attribute__((always_inline))
#elif defined(_MSC_VER)
    #define FORCE_INLINE __forceinline
#else
    #define FORCE_INLINE inline
#endif

FORCE_INLINE uint32_t cross_platform_multiply(uint32_t a, uint32_t b) {
    return a * b;
}
```

#### **帶最佳化的行內化**
```c
// 帶特定最佳化的行內化
inline __attribute__((always_inline, optimize("O3"))) 
uint32_t optimized_function(uint32_t x) {
    return x * x + x + 1;
}

// 無最佳化的行內化（用於除錯）
inline __attribute__((always_inline, optimize("O0"))) 
uint32_t debug_function(uint32_t x) {
    return x * x + x + 1;
}
```

### **行內函式最佳實踐**

#### **適當的使用案例**
```c
// 好的行內化候選者 - 小型、頻繁使用
inline uint8_t get_upper_byte(uint32_t value) {
    return (uint8_t)((value >> 8) & 0xFF);
}

// 好的候選者 - 硬體存取
inline void enable_interrupts(void) {
    __asm__ volatile("cpsie i" : : : "memory");
}

// 好的候選者 - 簡單數學
inline uint32_t min(uint32_t a, uint32_t b) {
    return (a < b) ? a : b;
}

// 不好的候選者 - 太大
inline void complex_algorithm(uint32_t* data, size_t size) {
    // 有很多行程式碼的複雜演算法
    // 不應該被行內化
}
```

## 🔧 **類函式巨集**

### **什麼是類函式巨集？**

類函式巨集是帶有參數的前處理器指令，執行文字替換。它們在呼叫端展開程式碼，消除函式呼叫開銷，但沒有型別安全性。

### **巨集概念**

**文字替換：**
- **前處理器階段**：巨集在編譯前展開
- **參數替換**：巨集參數被實際值替換
- **無型別檢查**：巨集不執行型別檢查
- **直接展開**：程式碼按字面替換到呼叫端

**巨集 vs. 函式：**
- **巨集**：文字替換，無函式呼叫開銷
- **函式**：實際的函式呼叫帶有開銷
- **型別安全**：函式提供型別檢查，巨集不提供
- **除錯**：函式比巨集更容易除錯

### **基本類函式巨集**

#### **簡單巨集**
```c
// 基本類函式巨集
#define SQUARE(x) ((x) * (x))

#define MAX(a, b) ((a) > (b) ? (a) : (b))

#define MIN(a, b) ((a) < (b) ? (a) : (b))

// 使用
uint32_t result1 = SQUARE(5);    // 展開為：((5) * (5))
uint32_t result2 = MAX(10, 20);  // 展開為：((10) > (20) ? (10) : (20))
```

#### **硬體存取巨集**
```c
// 硬體暫存器存取巨集
#define GPIO_SET_PIN(pin) \
    (*((volatile uint32_t*)0x40020008) |= (1 << (pin)))

#define GPIO_CLEAR_PIN(pin) \
    (*((volatile uint32_t*)0x4002000C) |= (1 << ((pin) + 16)))

#define GPIO_READ_PIN(pin) \
    ((*((volatile uint32_t*)0x40020000) & (1 << (pin))) != 0)

// 使用
GPIO_SET_PIN(13);      // 設定 LED 腳位
bool state = GPIO_READ_PIN(12);  // 讀取按鈕狀態
```

### **進階巨集技巧**

#### **多行巨集**
```c
// 使用 do-while(0) 的多行巨集
#define INIT_DEVICE(device, id, config) \
    do { \
        (device)->id = (id); \
        (device)->config = (config); \
        (device)->status = DEVICE_INACTIVE; \
    } while(0)

// 使用
device_t my_device;
INIT_DEVICE(&my_device, 1, 0x0F);
```

#### **條件巨集**
```c
// 條件編譯巨集
#ifdef DEBUG
    #define DEBUG_PRINT(msg) printf("DEBUG: %s\n", (msg))
#else
    #define DEBUG_PRINT(msg) ((void)0)
#endif

// 平台特定巨集
#ifdef ARM_CORTEX_M4
    #define CPU_FREQUENCY 168000000
#elif defined(ARM_CORTEX_M3)
    #define CPU_FREQUENCY 72000000
#else
    #define CPU_FREQUENCY 16000000
#endif
```

#### **字串化與符號黏貼**
```c
// 字串化 - 將參數轉換為字串
#define STRINGIFY(x) #x
#define TOSTRING(x) STRINGIFY(x)

// 符號黏貼 - 串接符號
#define CONCAT(a, b) a##b

// 使用
char* filename = TOSTRING(config.h);  // 展開為："config.h"
int var12 = CONCAT(var, 12);          // 展開為：var12
```

### **巨集安全考量**

#### **括號與副作用**
```c
// 帶括號的安全巨集
#define SQUARE(x) ((x) * (x))

// 不帶括號的不安全巨集
#define SQUARE_UNSAFE(x) x * x

// 使用範例
uint32_t a = 2, b = 3;
uint32_t result1 = SQUARE(a + b);      // 展開為：((a + b) * (a + b)) = 25
uint32_t result2 = SQUARE_UNSAFE(a + b); // 展開為：a + b * a + b = 11（錯誤！）
```

#### **多次求值**
```c
// 有多次求值的巨集（不安全）
#define MAX_UNSAFE(a, b) ((a) > (b) ? (a) : (b))

// 只有單次求值的函式（安全）
inline uint32_t max_safe(uint32_t a, uint32_t b) {
    return (a > b) ? a : b;
}

// 帶有副作用的使用
uint32_t counter = 0;
uint32_t result1 = MAX_UNSAFE(++counter, 5);  // counter 被遞增兩次！
uint32_t result2 = max_safe(++counter, 5);    // counter 只被遞增一次
```

## 🔄 **條件編譯**

### **什麼是條件編譯？**

條件編譯允許根據建置時期的條件編譯不同的程式碼。它對於建立可攜式程式碼和針對不同平台或建置配置進行最佳化不可或缺。

### **條件編譯概念**

**建置時期選擇：**
- **前處理器指令**：#ifdef、#ifndef、#if、#else、#elif、#endif
- **巨集定義**：定義巨集以控制編譯
- **平台偵測**：偵測目標平台和架構
- **功能旗標**：根據需求啟用/停用功能

**常見使用案例：**
- **除錯 vs. 發行**：除錯和發行建置使用不同程式碼
- **平台特定程式碼**：不同平台使用不同程式碼
- **功能選擇**：啟用/停用可選功能
- **最佳化等級**：不同建置使用不同最佳化

### **條件編譯實作**

#### **除錯 vs. 發行建置**
```c
// 除錯配置
#ifdef DEBUG
    #define DEBUG_PRINT(msg) printf("DEBUG: %s\n", (msg))
    #define ASSERT(condition) \
        do { \
            if (!(condition)) { \
                printf("ASSERTION FAILED: %s, line %d\n", __FILE__, __LINE__); \
                while(1); \
            } \
        } while(0)
#else
    #define DEBUG_PRINT(msg) ((void)0)
    #define ASSERT(condition) ((void)0)
#endif

// 使用
DEBUG_PRINT("Starting initialization");
ASSERT(device != NULL);
```

#### **平台特定程式碼**
```c
// 平台偵測
#ifdef __arm__
    #ifdef __ARM_ARCH_7M__
        #define PLATFORM "ARM Cortex-M7"
        #define CPU_FREQUENCY 216000000
    #elif defined(__ARM_ARCH_7EM__)
        #define PLATFORM "ARM Cortex-M7"
        #define CPU_FREQUENCY 180000000
    #elif defined(__ARM_ARCH_7M__)
        #define PLATFORM "ARM Cortex-M3"
        #define CPU_FREQUENCY 72000000
    #else
        #define PLATFORM "ARM (Unknown)"
        #define CPU_FREQUENCY 16000000
    #endif
#elif defined(__x86_64__)
    #define PLATFORM "x86_64"
    #define CPU_FREQUENCY 2400000000
#else
    #define PLATFORM "Unknown"
    #define CPU_FREQUENCY 16000000
#endif
```

#### **功能旗標**
```c
// 功能配置
#define FEATURE_UART    1
#define FEATURE_SPI     1
#define FEATURE_I2C     0
#define FEATURE_CAN     1

// 基於功能的條件編譯
#if FEATURE_UART
    void uart_init(void);
    void uart_send_byte(uint8_t byte);
    uint8_t uart_receive_byte(void);
#endif

#if FEATURE_SPI
    void spi_init(void);
    uint8_t spi_transfer(uint8_t data);
#endif

#if FEATURE_I2C
    void i2c_init(void);
    bool i2c_write(uint8_t address, uint8_t* data, uint8_t length);
#endif
```

## ⚡ **效能考量**

### **什麼影響效能？**

行內函式與巨集的效能取決於多個因素，包括編譯器最佳化、程式碼大小和使用模式。

### **效能因素**

**編譯器最佳化：**
- **行內化決策**：編譯器可能選擇不行內化
- **程式碼大小**：大型函式可能不被行內化
- **呼叫頻率**：頻繁呼叫的函式是更好的候選者
- **最佳化等級**：更高的最佳化等級可能行內化更多

**程式碼大小影響：**
- **重複**：行內化的程式碼在每個呼叫端重複
- **記憶體使用**：增加的程式碼大小可能影響快取效能
- **ROM 使用**：更多程式碼儲存在程式記憶體中
- **快取行為**：更大的程式碼可能導致更多快取未命中

**使用模式：**
- **呼叫頻率**：函式被呼叫的頻率
- **函式大小**：被行內化的函式大小
- **參數複雜度**：參數傳遞的複雜度
- **返回值**：返回值處理的複雜度

### **效能最佳化**

#### **行內函式最佳化**
```c
// 針對效能最佳化
inline __attribute__((always_inline)) 
uint32_t fast_bit_count(uint32_t value) {
    uint32_t count = 0;
    while (value) {
        count += value & 1;
        value >>= 1;
    }
    return count;
}

// 針對大小最佳化
inline __attribute__((always_inline)) 
uint32_t compact_bit_count(uint32_t value) {
    return __builtin_popcount(value);  // 使用內建函式
}
```

#### **巨集最佳化**
```c
// 位元操作的最佳化巨集
#define SET_BIT(reg, bit) ((reg) |= (1 << (bit)))
#define CLEAR_BIT(reg, bit) ((reg) &= ~(1 << (bit)))
#define TOGGLE_BIT(reg, bit) ((reg) ^= (1 << (bit)))
#define READ_BIT(reg, bit) (((reg) >> (bit)) & 1)

// 在效能關鍵程式碼中使用
volatile uint32_t* const gpio_odr = (uint32_t*)0x40020014;
SET_BIT(*gpio_odr, 13);    // 設定 LED 腳位
CLEAR_BIT(*gpio_odr, 13);  // 清除 LED 腳位
```

#### **條件最佳化**
```c
// 基於建置類型的條件最佳化
#ifdef DEBUG
    // 除錯版本 - 無最佳化
    inline uint32_t debug_multiply(uint32_t a, uint32_t b) {
        printf("Multiplying %u by %u\n", a, b);
        return a * b;
    }
#else
    // 發行版本 - 最佳化
    inline __attribute__((always_inline)) 
    uint32_t debug_multiply(uint32_t a, uint32_t b) {
        return a * b;
    }
#endif
```

## 🔍 **除錯與安全性**

### **除錯考量是什麼？**

除錯行內函式與巨集需要特別考量，因為它們被編譯器和前處理器處理的方式不同。

### **除錯概念**

**行內函式：**
- **除錯器支援**：行內函式出現在除錯器中
- **堆疊追蹤**：行內函式可能不出現在堆疊追蹤中
- **中斷點**：可以在行內函式中設定中斷點
- **變數檢查**：可以在行內函式中檢查變數

**巨集：**
- **無除錯器支援**：巨集在前處理後不存在
- **無堆疊追蹤**：巨集不出現在堆疊追蹤中
- **無中斷點**：無法在巨集中設定中斷點
- **文字替換**：巨集只是文字替換

### **除錯實作**

#### **除錯行內函式**
```c
// 帶除錯支援的行內函式
inline uint32_t debug_multiply(uint32_t a, uint32_t b) {
    #ifdef DEBUG
        printf("DEBUG: multiply(%u, %u)\n", a, b);
    #endif
    return a * b;
}

// 帶除錯使用
uint32_t result = debug_multiply(5, 3);  // 可以在此設定中斷點
```

#### **除錯巨集**
```c
// 帶除錯的巨集（有限制）
#define DEBUG_MULTIPLY(a, b) \
    ({ \
        uint32_t _a = (a); \
        uint32_t _b = (b); \
        uint32_t _result = _a * _b; \
        printf("DEBUG: multiply(%u, %u) = %u\n", _a, _b, _result); \
        _result; \
    })

// 使用（無法在巨集中設定中斷點）
uint32_t result = DEBUG_MULTIPLY(5, 3);
```

#### **安全性考量**
```c
// 帶型別檢查的安全巨集（有限制）
#define SAFE_MULTIPLY(a, b) \
    ({ \
        typeof(a) _a = (a); \
        typeof(b) _b = (b); \
        _a * _b; \
    })

// 帶完整型別檢查的更安全行內函式
inline uint32_t safe_multiply(uint32_t a, uint32_t b) {
    return a * b;
}
```

## 🔧 **實作**

### **完整行內函式與巨集範例**

```c
#include <stdint.h>
#include <stdbool.h>

// 平台偵測
#ifdef __arm__
    #define PLATFORM_ARM 1
#else
    #define PLATFORM_ARM 0
#endif

// 除錯配置
#ifdef DEBUG
    #define DEBUG_PRINT(msg) printf("DEBUG: %s\n", (msg))
    #define ASSERT(condition) \
        do { \
            if (!(condition)) { \
                printf("ASSERTION FAILED: %s, line %d\n", __FILE__, __LINE__); \
                while(1); \
            } \
        } while(0)
#else
    #define DEBUG_PRINT(msg) ((void)0)
    #define ASSERT(condition) ((void)0)
#endif

// 硬體暫存器定義
#define GPIOA_BASE    0x40020000
#define GPIOA_ODR     (GPIOA_BASE + 0x14)
#define GPIOA_IDR     (GPIOA_BASE + 0x10)

// 硬體存取巨集
#define GPIO_SET_PIN(pin) \
    (*((volatile uint32_t*)GPIOA_ODR) |= (1 << (pin)))

#define GPIO_CLEAR_PIN(pin) \
    (*((volatile uint32_t*)GPIOA_ODR) &= ~(1 << (pin)))

#define GPIO_READ_PIN(pin) \
    ((*((volatile uint32_t*)GPIOA_IDR) & (1 << (pin))) != 0)

// 硬體存取的行內函式
inline void gpio_set_pin_inline(uint8_t pin) {
    volatile uint32_t* const gpio_odr = (uint32_t*)GPIOA_ODR;
    *gpio_odr |= (1 << pin);
}

inline void gpio_clear_pin_inline(uint8_t pin) {
    volatile uint32_t* const gpio_odr = (uint32_t*)GPIOA_ODR;
    *gpio_odr &= ~(1 << pin);
}

inline bool gpio_read_pin_inline(uint8_t pin) {
    volatile uint32_t* const gpio_idr = (uint32_t*)GPIOA_IDR;
    return (*gpio_idr & (1 << pin)) != 0;
}

// 效能關鍵的行內函式
inline __attribute__((always_inline)) 
uint32_t fast_multiply(uint32_t a, uint32_t b) {
    return a * b;
}

inline __attribute__((always_inline)) 
uint32_t fast_add(uint32_t a, uint32_t b) {
    return a + b;
}

// 基於平台的條件編譯
#if PLATFORM_ARM
    inline void enable_interrupts(void) {
        __asm__ volatile("cpsie i" : : : "memory");
    }
    
    inline void disable_interrupts(void) {
        __asm__ volatile("cpsid i" : : : "memory");
    }
#else
    inline void enable_interrupts(void) {
        // 平台特定實作
    }
    
    inline void disable_interrupts(void) {
        // 平台特定實作
    }
#endif

// 除錯支援
inline uint32_t debug_multiply(uint32_t a, uint32_t b) {
    DEBUG_PRINT("Performing multiplication");
    uint32_t result = a * b;
    DEBUG_PRINT("Multiplication complete");
    return result;
}

// 主函式
int main(void) {
    DEBUG_PRINT("Starting application");
    
    // 使用巨集進行硬體存取
    GPIO_SET_PIN(13);      // 設定 LED 腳位
    bool button_state = GPIO_READ_PIN(12);  // 讀取按鈕狀態
    
    // 使用行內函式進行效能關鍵操作
    uint32_t result1 = fast_multiply(5, 3);
    uint32_t result2 = fast_add(10, 20);
    
    // 使用條件編譯
    enable_interrupts();
    
    // 使用除錯支援
    uint32_t debug_result = debug_multiply(4, 6);
    
    ASSERT(result1 == 15);
    ASSERT(result2 == 30);
    
    DEBUG_PRINT("Application complete");
    
    return 0;
}
```

## ⚠️ **常見陷阱**

### **1. 巨集副作用**

**問題**：巨集可能導致意外的副作用
**解決方案**：使用括號並避免多次求值

```c
// ❌ 不好：有副作用的巨集
#define SQUARE(x) x * x
#define MAX(a, b) a > b ? a : b

// 使用
uint32_t result1 = SQUARE(2 + 3);  // 展開為：2 + 3 * 2 + 3 = 11（錯誤！）
uint32_t counter = 0;
uint32_t result2 = MAX(++counter, 5);  // counter 被遞增兩次！

// ✅ 好：帶括號的安全巨集
#define SQUARE(x) ((x) * (x))
#define MAX(a, b) ((a) > (b) ? (a) : (b))

// ✅ 更好：使用行內函式
inline uint32_t square(uint32_t x) {
    return x * x;
}

inline uint32_t max(uint32_t a, uint32_t b) {
    return (a > b) ? a : b;
}
```

### **2. 行內函式大小**

**問題**：大型函式被行內化
**解決方案**：只行內化小型、頻繁使用的函式

```c
// ❌ 不好：大型行內函式
inline void complex_algorithm(uint32_t* data, size_t size) {
    // 50+ 行的複雜程式碼
    // 不應該被行內化
}

// ✅ 好：小型行內函式
inline uint32_t get_upper_byte(uint32_t value) {
    return (uint32_t)((value >> 8) & 0xFF);
}
```

### **3. 除錯問題**

**問題**：行內函式和巨集可能難以除錯
**解決方案**：使用適當的除錯策略

```c
// ❌ 不好：無除錯支援
#define HARDWARE_ACCESS(addr, value) (*((volatile uint32_t*)(addr)) = (value))

// ✅ 好：除錯支援
inline void hardware_access(uint32_t addr, uint32_t value) {
    #ifdef DEBUG
        printf("Writing 0x%08X to address 0x%08X\n", value, addr);
    #endif
    *((volatile uint32_t*)addr) = value;
}
```

### **4. 平台依賴性**

**問題**：程式碼無法跨平台移植
**解決方案**：使用條件編譯

```c
// ❌ 不好：平台特定程式碼
inline void enable_interrupts(void) {
    __asm__ volatile("cpsie i" : : : "memory");  // ARM 特定
}

// ✅ 好：平台無關程式碼
#ifdef __arm__
    inline void enable_interrupts(void) {
        __asm__ volatile("cpsie i" : : : "memory");
    }
#elif defined(__x86_64__)
    inline void enable_interrupts(void) {
        __asm__ volatile("sti");
    }
#else
    inline void enable_interrupts(void) {
        // 平台特定實作
    }
#endif
```

## ✅ **最佳實踐**

### **1. 選擇正確的工具**

- **行內函式**：用於型別安全和除錯支援
- **巨集**：用於文字替換和條件編譯
- **一般函式**：用於大型或複雜的函式
- **考慮權衡**：平衡效能、大小和可維護性

### **2. 針對效能最佳化**

- **分析關鍵程式碼**：測量效能影響
- **使用適當的行內化**：只行內化小型、頻繁使用的函式
- **考慮程式碼大小**：平衡效能與程式碼大小
- **測試不同編譯器**：驗證不同編譯器的行為

### **3. 確保安全性**

- **使用括號**：在巨集中總是使用括號
- **避免副作用**：小心巨集參數
- **型別安全**：盡可能偏好行內函式而非巨集
- **錯誤處理**：包含適當的錯誤檢查

### **4. 支援除錯**

- **除錯支援**：在除錯建置中包含除錯資訊
- **條件編譯**：使用條件編譯進行除錯
- **錯誤訊息**：提供清楚的錯誤訊息
- **文件**：記錄複雜的巨集和行內函式

### **5. 維護可攜性**

- **平台偵測**：使用條件編譯處理平台特定程式碼
- **編譯器偵測**：處理不同的編譯器功能
- **標準合規**：遵循 C 語言標準
- **測試**：在多個平台和編譯器上測試

## 🎯 **面試問題**

### **基礎問題**

1. **行內函式與巨集有什麼區別？**
   - 行內函式：編譯器建議的行內化，具有型別安全性
   - 巨集：前處理器文字替換，沒有型別安全性
   - 行內函式：更好的除錯支援
   - 巨集：條件編譯更靈活

2. **你什麼時候會使用行內函式 vs. 巨集？**
   - 行內函式：當型別安全和除錯很重要時
   - 巨集：當需要文字替換或條件編譯時
   - 行內函式：用於小型、頻繁使用的函式
   - 巨集：用於硬體存取和平台特定程式碼

3. **行內化有什麼效能好處？**
   - 消除函式呼叫開銷
   - 啟用編譯器最佳化
   - 減少堆疊使用
   - 改善快取效能

### **進階問題**

1. **你如何最佳化效能關鍵函式？**
   - 對小型函式使用行內函式
   - 使用適當的編譯器屬性
   - 分析程式碼以識別瓶頸
   - 考慮平台特定最佳化

2. **你如何處理平台特定程式碼？**
   - 使用前處理器指令的條件編譯
   - 定義平台特定巨集
   - 對平台特定操作使用行內函式
   - 在多個平台上測試

3. **你如何除錯行內函式和巨集？**
   - 使用條件編譯的除錯版本
   - 在除錯建置中包含除錯資訊
   - 使用適當的除錯工具
   - 記錄除錯策略

### **實作問題**

1. **撰寫一個用於位元操作的行內函式**
2. **建立一個用於硬體暫存器存取的巨集**
3. **實作用於除錯/發行建置的條件編譯**
4. **設計一個平台無關的硬體抽象層**

## 📚 **其他資源**

### **書籍**
- "The C Programming Language" by Brian W. Kernighan and Dennis M. Ritchie
- "C Programming: A Modern Approach" by K.N. King
- "Embedded C Coding Standard" by Michael Barr

### **線上資源**
- [Inline Functions Tutorial](https://www.tutorialspoint.com/cprogramming/c_inline_functions.htm)
- [Macros in C](https://www.tutorialspoint.com/cprogramming/c_preprocessors.htm)
- [GCC Inline Assembly](https://gcc.gnu.org/onlinedocs/gcc/Inline-Assembly.html)

### **工具**
- **Compiler Explorer**：跨編譯器測試行內函式
- **靜態分析**：偵測行內函式問題的工具
- **效能分析器**：測量行內函式效能
- **除錯工具**：除錯行內函式和巨集

### **標準**
- **C11**：具有行內函式規範的 C 語言標準
- **MISRA C**：安全關鍵編碼標準
- **平台 ABI**：架構特定的呼叫慣例

---

**下一步**：探索[編譯器內建函式](./Compiler_Intrinsics.md)以了解硬體特定操作，或深入[組合語言整合](./Assembly_Integration.md)以學習底層程式設計技術。
