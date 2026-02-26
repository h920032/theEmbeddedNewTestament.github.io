# 嵌入式系統的位元操作

> **嵌入式 C 程式設計中必備的位元操作與操控技術**

## 📋 **目錄**
- [概述](#概述)
- [什麼是位元操作？](#什麼是位元操作)
- [為什麼位元操作很重要？](#為什麼位元操作很重要)
- [位元操作概念](#位元操作概念)
- [位元運算子](#位元運算子)
- [位元欄位](#位元欄位)
- [位元遮罩](#位元遮罩)
- [位元移位](#位元移位)
- [硬體暫存器操作](#硬體暫存器操作)
- [常見位元技巧](#常見位元技巧)
- [效能考量](#效能考量)
- [實作](#實作)
- [常見陷阱](#常見陷阱)
- [最佳實踐](#最佳實踐)
- [面試題目](#面試題目)

---

## 🎯 **概述**

### 概念：位元是與硬體和協定的契約

位元操作必須精確匹配資料手冊定義的欄位；正確性和可攜性取決於使用在 C 語言中有良好定義的寬度、遮罩和移位。

### 為什麼它在嵌入式中很重要
- 暫存器欄位需要精確的遮罩/移位而不產生未定義行為（UB）。
- 協定打包/解包必須遵守位元組順序和寬度。
- 移位負數或過大是未定義的；依賴固定寬度型別。

### 最小範例：安全的欄位更新
```c
#define REG (*(volatile uint32_t*)0x40000000u)
#define MODE_Pos  4u
#define MODE_Msk  (0x7u << MODE_Pos) // 3 位元欄位

static inline void reg_set_mode(uint32_t mode) {
  mode &= 0x7u;                // 限制範圍
  uint32_t v = REG;
  v = (v & ~MODE_Msk) | (mode << MODE_Pos);
  REG = v;
}
```

### 試試看
1. 實作 `get_mode()` 並驗證所有值 0..7 的往返正確性。
2. 將結構體打包到 `uint32_t` 中；透過 UART 發送；解包並驗證。

### 重點
- 使用 `uint*_t`，永遠不要移位有號值。
- 定義 `*_Pos` 和 `*_Msk` 巨集或列舉；避免魔術數字。
- 在線上與記憶體中順序不同的地方記錄位元組順序。

### 面試官意圖（他們在探測什麼）
- 你能在不破壞鄰近位元的情況下更新位元欄位嗎？
- 你理解關於移位和有號型別的未定義行為嗎？
- 你能推理位元組順序和線上佈局嗎？

---

## 🧪 引導式實驗
1) 暫存器欄位往返
- 實作 3 位元欄位的 set/get，並模糊測試值 0..7 以驗證無交叉位元污染。

2) 協定打包/解包
```c
typedef struct { uint8_t type; uint16_t value; } msg_t;
uint32_t pack(const msg_t* m){ return ((uint32_t)m->type<<24)|((uint32_t)m->value<<8); }
void unpack(uint32_t w, msg_t* m){ m->type=(w>>24)&0xFF; m->value=(w>>8)&0xFFFF; }
```
- 在小端序和大端序主機上驗證；討論網路位元組順序。

## ✅ 自我檢查
- 為什麼對有號 `int` 做 `(x << 31)` 有問題？
- 當多個寫入者存在時，如何在不影響其他位元的情況下切換一個位元？

## 🔗 交叉連結
- `Embedded_C/Structure_Alignment.md` 佈局相關
- `Embedded_C/Memory_Mapped_IO.md` 暫存器覆蓋相關

位元操作在嵌入式系統中至關重要，用於：
- **硬體暫存器存取** - 設定/清除個別位元
- **記憶體效率** - 將多個值打包到單一變數中
- **效能最佳化** - 快速的位元級操作
- **協定實作** - 解析二進位資料格式
- **旗標管理** - 高效的布林狀態追蹤

### **關鍵概念**
- **位元運算子** - AND、OR、XOR、NOT、移位操作
- **位元欄位** - 結構體中的命名位元位置
- **位元遮罩** - 用於設定/清除特定位元的模式
- **位元組順序** - 位元組順序考量
- **位元計數** - 高效計算已設定的位元

## 🤔 **什麼是位元操作？**

位元操作是處理二進位資料中個別位元的過程。它涉及在位元層級直接操控資料的二進位表示，而不是處理更大的資料單位如位元組或字組。

### **核心概念**

**二進位表示：**
- 電腦中所有資料都以二進位（0 和 1）儲存
- 每個位元代表 2 的冪次
- 位元位置從右到左編號（從 0 開始）
- 多個位元組合代表更大的值

**位元級操作：**
- **個別位元存取**：讀取或寫入特定位元
- **位元模式**：處理位元群組
- **位元遮罩**：使用模式選擇特定位元
- **位元移位**：將位元向左或向右移動

**記憶體效率：**
- 將多個布林值打包到單一位元組中
- 在更大的變數中儲存多個小值
- 在受限系統中最佳化記憶體使用
- 減少資料結構大小

### **二進位數字系統**

**位置記數法：**
```
二進位數字：10110101
位元位置：  76543210
值：      128+32+16+4+1 = 181

位元位置 7：1 × 2^7 = 128
位元位置 6：0 × 2^6 = 0
位元位置 5：1 × 2^5 = 32
位元位置 4：1 × 2^4 = 16
位元位置 3：0 × 2^3 = 0
位元位置 2：1 × 2^2 = 4
位元位置 1：0 × 2^1 = 0
位元位置 0：1 × 2^0 = 1
```

**常見位元模式：**
```
2 的冪次：   1, 2, 4, 8, 16, 32, 64, 128
二進位：     00000001, 00000010, 00000100, 00001000, 等等

所有位元設定：11111111（8 位元為 255）
所有位元清除：00000000（0）
```

## 🎯 **為什麼位元操作很重要？**

### **嵌入式系統需求**

**硬體互動：**
- **暫存器存取**：硬體暫存器通常有個別位元控制
- **旗標管理**：狀態旗標和控制位元
- **配置**：透過位元設定進行裝置配置
- **中斷控制**：啟用/停用特定中斷來源

**記憶體限制：**
- **有限的 RAM**：將多個值打包到單一變數中
- **Flash 儲存**：最佳化儲存使用
- **快取效率**：減少記憶體佔用
- **頻寬**：最小化資料傳輸大小

**效能需求：**
- **即時系統**：時間關鍵程式碼的快速位元操作
- **中斷處理**：快速的旗標檢查和設定
- **協定處理**：高效的二進位資料解析
- **演算法最佳化**：快速的數學運算

### **實際應用**

**硬體暫存器控制：**
```c
// GPIO 控制
volatile uint32_t* const GPIO_ODR = (uint32_t*)0x40020014;
*GPIO_ODR |= (1 << 5);   // 設定位元 5（開啟 LED）
*GPIO_ODR &= ~(1 << 5);  // 清除位元 5（關閉 LED）
```

**旗標管理：**
```c
// 狀態旗標
uint8_t status_flags = 0;
#define FLAG_ERROR     (1 << 0)  // 位元 0
#define FLAG_BUSY      (1 << 1)  // 位元 1
#define FLAG_COMPLETE  (1 << 2)  // 位元 2

// 設定旗標
status_flags |= FLAG_BUSY;
status_flags |= FLAG_ERROR;

// 檢查旗標
if (status_flags & FLAG_ERROR) {
    // 處理錯誤
}

// 清除旗標
status_flags &= ~FLAG_BUSY;
```

**資料打包：**
```c
// 將多個值打包到單一變數中
uint32_t packed_data = 0;
uint8_t temperature = 25;    // 0-255 範圍
uint8_t humidity = 60;       // 0-255 範圍
uint16_t pressure = 1013;    // 0-65535 範圍

// 打包資料
packed_data = (pressure << 16) | (humidity << 8) | temperature;

// 解包資料
temperature = packed_data & 0xFF;
humidity = (packed_data >> 8) & 0xFF;
pressure = (packed_data >> 16) & 0xFFFF;
```

### **位元操作何時重要**

**高影響場景：**
- 硬體暫存器存取
- 記憶體受限系統
- 效能關鍵應用
- 二進位協定實作
- 旗標和狀態管理

**低影響場景：**
- 具有充足資源的高階應用
- 無硬體互動的簡單應用
- 清晰度比效能更重要的原型程式碼
- 效能需求最小的應用

## 🧠 **位元操作概念**

### **位元級思維**

**個別位元操作：**
- **設定位元**：使特定位元為 1
- **清除位元**：使特定位元為 0
- **切換位元**：反轉特定位元
- **測試位元**：檢查特定位元是否已設定

**位元模式：**
- **遮罩**：用於選擇特定位元的模式
- **旗標**：代表布林狀態的個別位元
- **欄位**：代表值的位元群組
- **信號**：代表硬體信號的位元

**位元位置慣例：**
- **LSB（最低有效位元）**：最右邊的位元（位置 0）
- **MSB（最高有效位元）**：最左邊的位元
- **從零開始的索引**：位元位置從 0 開始
- **2 的冪次**：每個位置代表 2^n

### **記憶體佈局**

**位元組組織：**
```
8 位元位元組：  [7][6][5][4][3][2][1][0]
16 位元字組：   [15][14][13][12][11][10][9][8][7][6][5][4][3][2][1][0]
32 位元雙字組：  [31][30]...[2][1][0]
```

**位元組順序考量：**
- **小端序**：最低有效位元組在前
- **大端序**：最高有效位元組在前
- **網路位元組順序**：大端序（標準）
- **主機位元組順序**：取決於架構

### **效能特性**

**位元操作速度：**
- **單週期操作**：大多數位元操作非常快
- **硬體支援**：許多處理器有專用位元指令
- **記憶體效率**：位元操作使用最少的記憶體
- **快取友好**：小型資料結構適合快取

**最佳化機會：**
- **位元級平行處理**：同時處理多個位元
- **查找表**：預先計算的位元模式
- **特殊指令**：CPU 特定的位元操作
- **編譯器最佳化**：自動位元操作最佳化

## 🔧 **位元運算子**

### **什麼是位元運算子？**

位元運算子在其運算元的個別位元上執行操作。它們在二進位層級工作，逐位比較或操作位元。

### **運算子分類**

**邏輯運算子：**
- **AND (&)**：只有當兩個運算元都是 1 時結果才為 1
- **OR (|)**：任一運算元為 1 時結果為 1
- **XOR (^)**：運算元不同時結果為 1
- **NOT (~)**：反轉所有位元

**移位運算子：**
- **左移 (<<)**：將位元向左移動，以零填充
- **右移 (>>)**：將位元向右移動，以零（邏輯移位）或符號位元（算術移位）填充

### **基本運算子**

#### **AND (&) 運算子**
```c
// 位元 AND - 只有當兩個運算元都是 1 時結果位元才為 1
uint8_t a = 0b10101010;  // 170
uint8_t b = 0b11001100;  // 204
uint8_t result = a & b;   // 0b10001000 = 136

// 常見用途
uint8_t mask_lower_nibble = 0x0F;  // 0b00001111
uint8_t value = 0xAB;              // 0b10101011
uint8_t lower_nibble = value & mask_lower_nibble;  // 0x0B
```

**AND 應用：**
- **遮罩**：擷取特定位元
- **清除**：將特定位元設為 0
- **測試**：檢查特定位元是否已設定
- **範圍限制**：將值保持在特定範圍內

#### **OR (|) 運算子**
```c
// 位元 OR - 任一運算元為 1 時結果位元為 1
uint8_t a = 0b10101010;  // 170
uint8_t b = 0b11001100;  // 204
uint8_t result = a | b;   // 0b11101110 = 238

// 常見用途
uint8_t set_bit_3 = 0x08;  // 0b00001000
uint8_t value = 0x01;      // 0b00000001
uint8_t result = value | set_bit_3;  // 0b00001001
```

**OR 應用：**
- **設定位元**：將特定位元設為 1
- **合併旗標**：合併多個旗標集合
- **預設值**：設定預設位元模式
- **聯集操作**：組合位元模式

#### **XOR (^) 運算子**
```c
// 位元 XOR - 運算元不同時結果位元為 1
uint8_t a = 0b10101010;  // 170
uint8_t b = 0b11001100;  // 204
uint8_t result = a ^ b;   // 0b01100110 = 102

// 常見用途
uint8_t toggle_bit_2 = 0x04;  // 0b00000100
uint8_t value = 0x01;         // 0b00000001
uint8_t result = value ^ toggle_bit_2;  // 0b00000101
```

**XOR 應用：**
- **切換位元**：反轉特定位元
- **加密**：簡單的位元級加密
- **同位檢查**：錯誤檢測
- **找出差異**：識別已更改的位元

#### **NOT (~) 運算子**
```c
// 位元 NOT - 反轉所有位元
uint8_t a = 0b10101010;  // 170
uint8_t result = ~a;      // 0b01010101 = 85

// 常見用途
uint8_t mask = 0x0F;      // 0b00001111
uint8_t inverted_mask = ~mask;  // 0b11110000
```

**NOT 應用：**
- **反轉遮罩**：建立互補的位元模式
- **清除位元**：與 AND 搭配使用以清除特定位元
- **位元計數**：計算未設定的位元
- **範圍操作**：建立排除遮罩

### **移位運算子**

#### **左移 (<<)**
```c
// 左移 - 將位元向左移動，以零填充
uint8_t value = 0b00000001;  // 1
uint8_t result = value << 3;  // 0b00001000 = 8

// 常見用途
uint8_t create_mask = 1 << 5;  // 0b00100000（位元 5）
uint32_t multiply_by_8 = value << 3;  // 乘以 8
```

**左移應用：**
- **建立遮罩**：產生位元模式
- **乘法**：乘以 2 的冪次
- **位元定位**：將位元放置在特定位置
- **旗標建立**：建立旗標常數

#### **右移 (>>)**
```c
// 右移 - 將位元向右移動，以零填充
uint8_t value = 0b10001000;  // 136
uint8_t result = value >> 3;  // 0b00010001 = 17

// 常見用途
uint8_t extract_upper_nibble = value >> 4;  // 取得高 4 位元
uint32_t divide_by_8 = value >> 3;  // 除以 8
```

**右移應用：**
- **擷取欄位**：取得特定位元範圍
- **除法**：除以 2 的冪次
- **位元計數**：計算尾隨零
- **範圍擷取**：擷取特定位元位置

## 📊 **位元欄位**

### **什麼是位元欄位？**

位元欄位是結構體中的命名位元位置，允許您透過名稱而非位置來存取個別位元或位元群組。

### **位元欄位概念**

**命名存取：**
- 個別位元或位元群組有名稱
- 使用欄位名稱存取位元，而非遮罩
- 編譯器自動處理位元定位
- 提供型別安全和可讀性

**記憶體效率：**
- 將多個小值打包到單一變數中
- 在受限系統中減少記憶體使用
- 最佳化資料結構大小
- 在節省空間的同時保持可讀性

**硬體映射：**
- 直接映射到硬體暫存器佈局
- 匹配硬體位元欄位定義
- 確保正確的位元定位
- 維護硬體相容性

### **位元欄位實作**

#### **基本位元欄位**
```c
// 簡單位元欄位結構體
typedef struct {
    uint8_t error : 1;      // 1 位元用於錯誤旗標
    uint8_t busy : 1;       // 1 位元用於忙碌旗標
    uint8_t complete : 1;   // 1 位元用於完成旗標
    uint8_t reserved : 5;   // 5 位元保留
} status_flags_t;

// 使用方式
status_flags_t flags = {0};
flags.error = 1;        // 設定錯誤旗標
flags.busy = 1;         // 設定忙碌旗標

if (flags.complete) {   // 檢查完成旗標
    // 處理完成
}
```

#### **硬體暫存器映射**
```c
// GPIO 配置暫存器
typedef struct {
    uint32_t mode : 2;      // GPIO 模式（00=輸入, 01=輸出, 10=替代, 11=類比）
    uint32_t pull_up : 1;   // 上拉電阻
    uint32_t pull_down : 1; // 下拉電阻
    uint32_t speed : 2;     // 輸出速度
    uint32_t reserved : 26; // 保留位元
} gpio_config_t;

// 使用方式
gpio_config_t config = {0};
config.mode = 1;        // 輸出模式
config.pull_up = 1;     // 啟用上拉
config.speed = 2;       // 高速
```

#### **資料打包**
```c
// 將多個值打包到單一變數中
typedef struct {
    uint32_t temperature : 8;   // 8 位元用於溫度（0-255）
    uint32_t humidity : 8;      // 8 位元用於濕度（0-255）
    uint32_t pressure : 16;     // 16 位元用於氣壓（0-65535）
} sensor_data_t;

// 使用方式
sensor_data_t data = {0};
data.temperature = 25;   // 設定溫度
data.humidity = 60;      // 設定濕度
data.pressure = 1013;    // 設定氣壓
```

## 🎭 **位元遮罩**

### **什麼是位元遮罩？**

位元遮罩是用於選擇、修改或測試更大值中特定位元的位元模式。它們提供一種系統化的方式來處理個別位元或位元群組。

### **遮罩概念**

**選擇模式：**
- **單位元遮罩**：選擇個別位元
- **多位元遮罩**：選擇位元群組
- **範圍遮罩**：選擇位元範圍
- **反轉遮罩**：選擇特定位元以外的所有位元

**遮罩操作：**
- **AND 遮罩**：擷取特定位元
- **OR 遮罩**：設定特定位元
- **XOR 遮罩**：切換特定位元
- **NOT 遮罩**：反轉選擇

### **遮罩實作**

#### **單位元遮罩**
```c
// 定義單位元遮罩
#define BIT_0  (1 << 0)   // 0b00000001
#define BIT_1  (1 << 1)   // 0b00000010
#define BIT_2  (1 << 2)   // 0b00000100
#define BIT_3  (1 << 3)   // 0b00001000
#define BIT_4  (1 << 4)   // 0b00010000
#define BIT_5  (1 << 5)   // 0b00100000
#define BIT_6  (1 << 6)   // 0b01000000
#define BIT_7  (1 << 7)   // 0b10000000

// 使用方式
uint8_t value = 0x42;    // 0b01000010
uint8_t bit_1 = value & BIT_1;  // 檢查位元 1 是否已設定
value |= BIT_3;          // 設定位元 3
value &= ~BIT_5;         // 清除位元 5
value ^= BIT_2;          // 切換位元 2
```

#### **多位元遮罩**
```c
// 定義多位元遮罩
#define NIBBLE_LOWER  0x0F    // 0b00001111（低 4 位元）
#define NIBBLE_UPPER  0xF0    // 0b11110000（高 4 位元）
#define BYTE_LOWER    0xFF    // 0b11111111（低 8 位元）
#define BYTE_UPPER    0xFF00  // 0b1111111100000000（高 8 位元）

// 使用方式
uint16_t value = 0x1234;
uint8_t lower_byte = value & BYTE_LOWER;   // 0x34
uint8_t upper_byte = (value & BYTE_UPPER) >> 8;  // 0x12
```

#### **範圍遮罩**
```c
// 為特定位元範圍建立遮罩
#define MASK_3_BITS(n)    ((1 << n) - 1)  // 從 0 開始的 n 個連續位元
#define MASK_RANGE(start, end) (((1 << (end - start + 1)) - 1) << start)

// 使用方式
#define MASK_BITS_2_4  MASK_RANGE(2, 4)   // 0b00011100
#define MASK_BITS_0_2  MASK_3_BITS(3)     // 0b00000111

uint8_t value = 0x5A;    // 0b01011010
uint8_t bits_2_4 = (value & MASK_BITS_2_4) >> 2;  // 擷取位元 2-4
```

## 🔄 **位元移位**

### **什麼是位元移位？**

位元移位在值內將位元向左或向右移動，有效地乘以或除以 2 的冪次。這是位元操作和算術最佳化的基本操作。

### **移位概念**

**方向：**
- **左移 (<<)**：將位元向左移動，乘以 2^n
- **右移 (>>)**：將位元向右移動，除以 2^n
- **零填充**：移位以零填充（邏輯移位）
- **符號填充**：右移可能以符號位元填充（算術移位）

**應用：**
- **乘法/除法**：快速算術操作
- **位元定位**：將位元放置在特定位置
- **欄位擷取**：擷取特定位元範圍
- **遮罩建立**：產生位元模式

### **移位實作**

#### **算術操作**
```c
// 以 2 的冪次進行快速乘法和除法
uint32_t value = 10;
uint32_t multiplied = value << 2;   // 10 * 4 = 40
uint32_t divided = value >> 1;      // 10 / 2 = 5

// 2 的冪次計算
uint32_t power_of_2 = 1 << n;       // 2^n
uint32_t mask_for_n_bits = (1 << n) - 1;  // n 個連續的 1
```

#### **欄位操作**
```c
// 擷取和插入位元欄位
uint32_t packed_value = 0x12345678;

// 從位元 8 開始擷取 8 位元欄位
uint8_t field = (packed_value >> 8) & 0xFF;

// 在位元 8 處插入 8 位元值
uint32_t new_value = (packed_value & ~(0xFF << 8)) | (field << 8);
```

#### **位元位置操作**
```c
// 設定位置 n 的位元
uint32_t set_bit(uint32_t value, uint8_t position) {
    return value | (1 << position);
}

// 清除位置 n 的位元
uint32_t clear_bit(uint32_t value, uint8_t position) {
    return value & ~(1 << position);
}

// 切換位置 n 的位元
uint32_t toggle_bit(uint32_t value, uint8_t position) {
    return value ^ (1 << position);
}

// 測試位置 n 的位元
bool test_bit(uint32_t value, uint8_t position) {
    return (value & (1 << position)) != 0;
}
```

## 🔧 **硬體暫存器操作**

### **什麼是硬體暫存器操作？**

硬體暫存器操作涉及操作硬體暫存器中的個別位元，以控制裝置行為、讀取狀態資訊或配置硬體周邊。

### **暫存器操作概念**

**記憶體映射暫存器：**
- 硬體暫存器以記憶體位址的形式出現
- 個別位元控制特定硬體功能
- 讀寫暫存器影響硬體行為
- 某些暫存器是唯讀或唯寫的

**位元級控制：**
- **控制位元**：配置硬體行為
- **狀態位元**：讀取硬體狀態
- **中斷位元**：控制中斷行為
- **配置位元**：設定裝置參數

### **硬體暫存器實作**

#### **GPIO 控制**
```c
// GPIO 暫存器定義
volatile uint32_t* const GPIOA_ODR = (uint32_t*)0x40020014;
volatile uint32_t* const GPIOA_IDR = (uint32_t*)0x40020010;
volatile uint32_t* const GPIOA_CRL = (uint32_t*)0x40020000;

// GPIO 控制函式
void gpio_set_pin(uint8_t pin) {
    *GPIOA_ODR |= (1 << pin);
}

void gpio_clear_pin(uint8_t pin) {
    *GPIOA_ODR &= ~(1 << pin);
}

void gpio_toggle_pin(uint8_t pin) {
    *GPIOA_ODR ^= (1 << pin);
}

bool gpio_read_pin(uint8_t pin) {
    return (*GPIOA_IDR & (1 << pin)) != 0;
}
```

#### **UART 配置**
```c
// UART 暫存器定義
volatile uint32_t* const UART_CR1 = (uint32_t*)0x40011000;
volatile uint32_t* const UART_CR2 = (uint32_t*)0x40011004;
volatile uint32_t* const UART_SR = (uint32_t*)0x40011000;

// UART 控制位元
#define UART_ENABLE      (1 << 13)
#define UART_TX_ENABLE   (1 << 3)
#define UART_RX_ENABLE   (1 << 2)
#define UART_TX_READY    (1 << 7)
#define UART_RX_READY    (1 << 5)

// UART 配置
void uart_enable(void) {
    *UART_CR1 |= UART_ENABLE | UART_TX_ENABLE | UART_RX_ENABLE;
}

void uart_disable(void) {
    *UART_CR1 &= ~(UART_ENABLE | UART_TX_ENABLE | UART_RX_ENABLE);
}

bool uart_tx_ready(void) {
    return (*UART_SR & UART_TX_READY) != 0;
}

bool uart_rx_ready(void) {
    return (*UART_SR & UART_RX_READY) != 0;
}
```

## 🎯 **常見位元技巧**

### **什麼是位元技巧？**

位元技巧是使用位元操作高效解決問題的巧妙技術。它們通常提供比傳統方法更快或更節省記憶體的解決方案。

### **基本位元技巧**

#### **計算已設定的位元**
```c
// 計算值中為 1 的位元數量
uint32_t count_set_bits(uint32_t value) {
    uint32_t count = 0;
    while (value) {
        count += value & 1;
        value >>= 1;
    }
    return count;
}

// 使用查找表的快速方法
uint32_t count_set_bits_fast(uint32_t value) {
    static const uint8_t lookup[256] = {
        0, 1, 1, 2, 1, 2, 2, 3, 1, 2, 2, 3, 2, 3, 3, 4,
        // ...（完整查找表）
    };
    
    return lookup[value & 0xFF] + 
           lookup[(value >> 8) & 0xFF] + 
           lookup[(value >> 16) & 0xFF] + 
           lookup[(value >> 24) & 0xFF];
}
```

#### **找到最低已設定位元**
```c
// 找到最低已設定位元的位置
int find_lowest_set_bit(uint32_t value) {
    if (value == 0) return -1;
    return __builtin_ctz(value);  // 計算尾隨零
}

// 替代方法
int find_lowest_set_bit_alt(uint32_t value) {
    if (value == 0) return -1;
    return value & -value;  // 隔離最低已設定位元
}
```

#### **2 的冪次檢查**
```c
// 檢查值是否為 2 的冪次
bool is_power_of_2(uint32_t value) {
    return value != 0 && (value & (value - 1)) == 0;
}

// 取得下一個 2 的冪次
uint32_t next_power_of_2(uint32_t value) {
    value--;
    value |= value >> 1;
    value |= value >> 2;
    value |= value >> 4;
    value |= value >> 8;
    value |= value >> 16;
    return value + 1;
}
```

#### **位元反轉**
```c
// 反轉位元組中的位元
uint8_t reverse_bits(uint8_t value) {
    value = ((value & 0x55) << 1) | ((value & 0xAA) >> 1);
    value = ((value & 0x33) << 2) | ((value & 0xCC) >> 2);
    value = ((value & 0x0F) << 4) | ((value & 0xF0) >> 4);
    return value;
}
```

## ⚡ **效能考量**

### **什麼影響位元操作效能？**

位元操作效能取決於多個因素，包括硬體支援、編譯器最佳化和操作複雜度。

### **效能因素**

**硬體支援：**
- **專用指令**：某些處理器有特殊的位元指令
- **指令延遲**：不同操作有不同的時序
- **管線效率**：操作如何適應 CPU 管線
- **快取行為**：記憶體存取模式影響效能

**編譯器最佳化：**
- **常數折疊**：編譯器可能預先計算常數表達式
- **指令選擇**：編譯器選擇最佳的機器指令
- **迴圈最佳化**：編譯器可能最佳化位元操作迴圈
- **內聯**：小型位元函式可能被內聯

**操作複雜度：**
- **單一操作**：個別位元操作非常快
- **複雜模式**：多重操作可能較慢
- **記憶體存取**：對記憶體的位元操作比暫存器慢
- **分支預測**：條件位元操作可能較慢

### **效能最佳化**

#### **使用內建函式**
```c
// 可用時使用編譯器內建函式
uint32_t count_bits = __builtin_popcount(value);
uint32_t leading_zeros = __builtin_clz(value);
uint32_t trailing_zeros = __builtin_ctz(value);
uint32_t bit_reverse = __builtin_bitreverse32(value);
```

#### **最佳化常見模式**
```c
// 最佳化位元欄位操作
#define SET_BIT(reg, bit)    ((reg) |= (1 << (bit)))
#define CLEAR_BIT(reg, bit)  ((reg) &= ~(1 << (bit)))
#define TOGGLE_BIT(reg, bit) ((reg) ^= (1 << (bit)))
#define READ_BIT(reg, bit)   (((reg) >> (bit)) & 1)

// 對複雜操作使用查找表
static const uint8_t bit_count_table[256] = {
    // 所有位元組值的預先計算位元計數
};
```

#### **記憶體存取最佳化**
```c
// 最小化記憶體存取
uint32_t read_modify_write(uint32_t* reg, uint32_t mask, uint32_t value) {
    uint32_t current = *reg;
    current = (current & ~mask) | (value & mask);
    *reg = current;
    return current;
}
```

## 🔧 **實作**

### **完整位元操作範例**

```c
#include <stdint.h>
#include <stdbool.h>

// 硬體暫存器定義
#define GPIOA_BASE    0x40020000
#define GPIOA_ODR     (GPIOA_BASE + 0x14)
#define GPIOA_IDR     (GPIOA_BASE + 0x10)
#define GPIOA_CRL     (GPIOA_BASE + 0x00)

// GPIO 暫存器指標
volatile uint32_t* const gpio_odr = (uint32_t*)GPIOA_ODR;
volatile uint32_t* const gpio_idr = (uint32_t*)GPIOA_IDR;
volatile uint32_t* const gpio_crl = (uint32_t*)GPIOA_CRL;

// 位元操作巨集
#define SET_BIT(reg, bit)    ((reg) |= (1 << (bit)))
#define CLEAR_BIT(reg, bit)  ((reg) &= ~(1 << (bit)))
#define TOGGLE_BIT(reg, bit) ((reg) ^= (1 << (bit)))
#define READ_BIT(reg, bit)   (((reg) >> (bit)) & 1)

// 使用位元欄位的狀態旗標
typedef struct {
    uint8_t error : 1;
    uint8_t busy : 1;
    uint8_t complete : 1;
    uint8_t timeout : 1;
    uint8_t reserved : 4;
} status_flags_t;

// GPIO 控制函式
void gpio_set_pin(uint8_t pin) {
    SET_BIT(*gpio_odr, pin);
}

void gpio_clear_pin(uint8_t pin) {
    CLEAR_BIT(*gpio_odr, pin);
}

void gpio_toggle_pin(uint8_t pin) {
    TOGGLE_BIT(*gpio_odr, pin);
}

bool gpio_read_pin(uint8_t pin) {
    return READ_BIT(*gpio_idr, pin);
}

// 位元計數函式
uint32_t count_set_bits(uint32_t value) {
    uint32_t count = 0;
    while (value) {
        count += value & 1;
        value >>= 1;
    }
    return count;
}

bool is_power_of_2(uint32_t value) {
    return value != 0 && (value & (value - 1)) == 0;
}

// 主函式
int main(void) {
    status_flags_t flags = {0};
    
    // 設定狀態旗標
    flags.error = 1;
    flags.busy = 1;
    
    // GPIO 操作
    gpio_set_pin(5);      // 開啟 LED
    gpio_clear_pin(6);    // 關閉 LED
    
    // 檢查 GPIO 狀態
    if (gpio_read_pin(0)) {
        // 按鈕被按下
        gpio_toggle_pin(5);  // 切換 LED
    }
    
    // 位元計數範例
    uint32_t test_value = 0x12345678;
    uint32_t bit_count = count_set_bits(test_value);
    
    // 2 的冪次檢查
    if (is_power_of_2(64)) {
        // 64 是 2 的冪次
    }
    
    return 0;
}
```

## ⚠️ **常見陷阱**

### **1. 符號擴展問題**

**問題**：右移有號值可能造成符號擴展
**解決方案**：對位元操作使用無號型別

```c
// ❌ 錯誤：有號型別的符號擴展
int32_t value = -1;
int32_t shifted = value >> 1;  // 可能不如預期

// ✅ 正確：使用無號型別
uint32_t value = 0xFFFFFFFF;
uint32_t shifted = value >> 1;  // 正確運作
```

### **2. 移位的未定義行為**

**問題**：移位負數量或超出位元寬度
**解決方案**：驗證移位量

```c
// ❌ 錯誤：未定義行為
uint32_t value = 1;
uint32_t result = value << 32;  // 未定義行為

// ✅ 正確：驗證移位量
uint32_t safe_shift(uint32_t value, uint8_t shift) {
    if (shift >= 32) return 0;
    return value << shift;
}
```

### **3. 位元欄位可攜性**

**問題**：位元欄位行為在不同編譯器之間可能不同
**解決方案**：改用明確的位元操作

```c
// ❌ 錯誤：不可攜的位元欄位
typedef struct {
    uint8_t field1 : 3;
    uint8_t field2 : 5;
} non_portable_t;

// ✅ 正確：明確的位元操作
#define FIELD1_MASK  0x07
#define FIELD2_MASK  0xF8
#define FIELD1_SHIFT 0
#define FIELD2_SHIFT 3
```

### **4. 位元組順序問題**

**問題**：位元操作在不同架構上可能有不同行為
**解決方案**：使用可攜的位元操作

```c
// ❌ 錯誤：依賴位元組順序的程式碼
uint32_t value = 0x12345678;
uint8_t byte = *(uint8_t*)&value;  // 取決於位元組順序

// ✅ 正確：可攜的位元擷取
uint8_t byte = (value >> 0) & 0xFF;  // 始終取得最低有效位元組
```

## ✅ **最佳實踐**

### **1. 使用適當的型別**

- **無號型別**：對位元操作使用以避免符號問題
- **明確大小**：使用 uint8_t、uint16_t、uint32_t 以提高清晰度
- **避免 char**：char 的有號/無號取決於平台
- **考慮對齊**：對硬體暫存器使用適當的型別

### **2. 驗證操作**

- **檢查移位量**：確保移位在有效範圍內
- **驗證位元位置**：檢查位元位置是否有效
- **測試邊界情況**：用零、最大值和邊界條件測試
- **使用斷言**：在除錯建置中加入執行時檢查

### **3. 記錄位元佈局**

- **清楚的文件**：記錄位元欄位佈局和含義
- **使用常數**：將位元位置和遮罩定義為常數
- **註釋複雜操作**：解釋不明顯的位元操作
- **維持一致性**：使用一致的命名慣例

### **4. 最佳化效能**

- **使用內建函式**：可用時使用編譯器內建函式
- **最小化記憶體存取**：盡可能快取暫存器值
- **使用查找表**：對經常呼叫的複雜操作
- **分析關鍵程式碼**：測量關鍵路徑中的位元操作效能

### **5. 確保可攜性**

- **避免未定義行為**：不要依賴未定義的位元操作
- **使用標準操作**：堅持使用有良好定義的位元操作
- **在多平台測試**：在不同架構上驗證行為
- **考慮位元組順序**：適當處理位元組順序問題

## 🎯 **面試題目**

### **基礎題目**

1. **C 語言中的基本位元運算子有哪些？**
   - AND (&)、OR (|)、XOR (^)、NOT (~)
   - 左移 (<<)、右移 (>>)
   - 每個運算子都有特定的使用情境和行為

2. **如何設定、清除和切換特定位元？**
   - 設定：value |= (1 << bit_position)
   - 清除：value &= ~(1 << bit_position)
   - 切換：value ^= (1 << bit_position)

3. **邏輯移位和算術移位有什麼區別？**
   - 邏輯移位：始終以零填充
   - 算術移位：右移可能以符號位元填充
   - C 標準未規定負值的行為

### **進階題目**

1. **如何計算值中已設定位元的數量？**
   - 迴圈方法：逐一檢查每個位元
   - 查找表：預先計算的位元計數
   - 內建函式：__builtin_popcount()
   - Brian Kernighan 演算法：value &= (value - 1)

2. **如何檢查數字是否為 2 的冪次？**
   - 使用：(value != 0) && ((value & (value - 1)) == 0)
   - 2 的冪次恰好有一個已設定的位元
   - 減 1 會清除最低已設定位元

3. **如何反轉值中的位元？**
   - 位元組反轉：使用查找表或位元操作
   - 字組反轉：使用內建函式或分治法
   - 考慮位元組順序和資料大小

### **實作題目**

1. **撰寫找到最低已設定位元的函式**
2. **實作檢查兩個數字是否符號相反的函式**
3. **撰寫不使用暫時變數交換兩個變數的程式碼**
4. **設計向上取整到下一個 2 的冪次的函式**

## 📚 **其他資源**

### **書籍**
- 《Hacker's Delight》Henry S. Warren Jr. 著
- 《C 程式設計語言》Brian W. Kernighan 與 Dennis M. Ritchie 著
- 《Bit Twiddling Hacks》Sean Eron Anderson 著

### **線上資源**
- [Bit Twiddling Hacks](https://graphics.stanford.edu/~seander/bithacks/)
- [位元操作教學](https://www.tutorialspoint.com/cprogramming/c_bitwise_operators.htm)
- [位元運算](https://en.wikipedia.org/wiki/Bitwise_operation)

### **工具**
- **位元計算器**：線上位元操作工具
- **Compiler Explorer**：跨編譯器測試位元操作
- **靜態分析**：檢測位元操作問題的工具
- **除錯工具**：在除錯器中檢查位元級資料

### **標準**
- **C11**：包含位元操作規範的 C 語言標準
- **MISRA C**：安全關鍵編碼標準
- **CERT C**：安全編碼標準

---

**下一步**：探索[結構體對齊](./Structure_Alignment.md)以了解記憶體佈局最佳化，或深入研究[內聯函式與巨集](./Inline_Functions_Macros.md)以了解效能最佳化技術。
