# 嵌入式系統中的記憶體管理

## 📋 目錄
- [概述](#-概述)
- [記憶體類型](#-記憶體類型)
- [堆疊與堆積](#-堆疊與堆積)
- [記憶體配置](#-記憶體配置)
- [記憶體釋放](#-記憶體釋放)
- [記憶體安全](#-記憶體安全)
- [常見陷阱](#-常見陷阱)
- [最佳實踐](#-最佳實踐)
- [面試題目](#-面試題目)
- [其他資源](#-其他資源)

## 🎯 概述

記憶體管理在資源有限且可靠性至關重要的嵌入式系統中是關鍵的。了解記憶體如何配置、使用和釋放，對於撰寫高效且安全的嵌入式程式碼至關重要。

### 嵌入式開發的關鍵概念
- **確定性配置** - 可預測的記憶體使用模式
- **記憶體安全** - 防止緩衝區溢位和記憶體洩漏
- **資源限制** - 在有限的 RAM 內工作
- **即時需求** - 避免在關鍵路徑中進行動態配置

### **面試官意圖（他們在探測什麼）**
- 你能論證堆疊、堆積和靜態配置的選擇嗎？
- 你理解確定性、碎片化和失敗模式嗎？
- 你能解釋即時系統中的記憶體權衡嗎？

## 🔢 記憶體類型

### 靜態記憶體
```c
// 全域變數 - 在編譯時期配置
static uint8_t global_buffer[1024];
static const char* const_string = "Hello World";

// 靜態區域變數 - 在函式呼叫之間持續存在
void function() {
    static int counter = 0;  // 只初始化一次，持續存在
    counter++;
}
```

### 堆疊記憶體
```c
void stack_example() {
    int local_var = 42;           // 堆疊配置
    uint8_t buffer[256];          // 堆疊陣列
    struct sensor_data data;       // 堆疊結構體
    
    // 函式返回時堆疊記憶體自動釋放
}
```

### 堆積記憶體
```c
#include <stdlib.h>

void heap_example() {
    // 動態配置
    uint8_t* buffer = malloc(1024);
    if (buffer == NULL) {
        // 處理配置失敗
        return;
    }
    
    // 使用 buffer...
    
    // 必須明確釋放
    free(buffer);
    buffer = NULL;  // 防止釋放後使用
}
```

## 🏗️ 堆疊與堆積

### 堆疊特性
```c
// 堆疊配置快速且確定
void stack_operations() {
    uint32_t stack_var = 0x12345678;
    uint8_t stack_array[64];
    
    // 注意：在 C 語言中，自動（堆疊）變數不會自動初始化。
    // 記憶體自動釋放
    // 沒有碎片化問題
}
```

### 堆積特性
```c
// 堆積配置靈活但需要管理
void heap_operations() {
    // 配置記憶體
    void* ptr1 = malloc(100);
    void* ptr2 = malloc(200);
    
    // 使用記憶體...
    
    // 釋放順序不需要相反；碎片化行為
    // 取決於配置器。優先使用固定大小的池以避免碎片化。
    free(ptr1);
    free(ptr2);
}
```

### 記憶體佈局
```c
// 典型嵌入式系統記憶體佈局
/*
高位址
    ┌─────────────────┐
    │     堆疊        │ ← 向下成長
    ├─────────────────┤
    │     堆積        │ ← 向上成長
    ├─────────────────┤
    │     .bss        │ ← 未初始化資料
    ├─────────────────┤
    │     .data       │ ← 已初始化資料
    ├─────────────────┤
    │     .text       │ ← 程式碼
    └─────────────────┘
低位址
*/
```

## 📊 記憶體配置

### 靜態配置
```c
// 嵌入式系統的預配置緩衝區
#define BUFFER_SIZE 1024
#define MAX_SENSORS 8

// 靜態配置 - 無執行時開銷
static uint8_t sensor_buffer[BUFFER_SIZE];
static struct sensor_data sensors[MAX_SENSORS];

// 固定大小配置的記憶體池
#define POOL_SIZE 16
#define POOL_COUNT 32

typedef struct {
    uint8_t data[POOL_SIZE];
    uint8_t used;
} memory_pool_t;

static memory_pool_t memory_pools[POOL_COUNT];
```

### 動態配置
```c
// 帶有錯誤檢查的安全動態配置
void* safe_malloc(size_t size) {
    void* ptr = malloc(size);
    if (ptr == NULL) {
        // 記錄錯誤或優雅處理
        return NULL;
    }
    return ptr;
}

// 對齊式配置
// 注意：posix_memalign 是 POSIX 專用的，在裸機上通常不可用。
// 僅在託管的 POSIX 目標上使用。對於裸機，優先使用靜態
// 對齊的儲存或自訂池。
#if defined(__unix__) || defined(__APPLE__)
void* aligned_malloc(size_t size, size_t alignment) {
    void* ptr = NULL;
    if (posix_memalign(&ptr, alignment, size) != 0) {
        return NULL;
    }
    return ptr;
}
#endif

// 靜態對齊緩衝區（裸機的可攜方法）
#if defined(__GNUC__) || defined(__clang__)
__attribute__((aligned(32))) static uint8_t dma_buffer[1024];
#elif defined(_MSC_VER)
__declspec(align(32)) static uint8_t dma_buffer[1024];
#endif
```

### 記憶體池實作
```c
typedef struct {
    uint8_t* pool;
    size_t pool_size;
    size_t block_size;
    uint8_t* free_list;
    size_t free_count;
} mem_pool_t;

// 初始化記憶體池
int mem_pool_init(mem_pool_t* pool, size_t block_size, size_t block_count) {
    pool->block_size = block_size;
    pool->pool_size = block_size * block_count;
    pool->pool = malloc(pool->pool_size);
    
    if (pool->pool == NULL) {
        return -1;
    }
    
    // 初始化空閒串列
    pool->free_list = pool->pool;
    pool->free_count = block_count;
    
    // 在空閒串列中連結區塊
    uint8_t* current = pool->pool;
    for (size_t i = 0; i < block_count - 1; i++) {
        *(uint8_t**)current = current + block_size;
        current += block_size;
    }
    *(uint8_t**)current = NULL;
    
    return 0;
}

// 從池中配置
void* mem_pool_alloc(mem_pool_t* pool) {
    if (pool->free_count == 0) {
        return NULL;  // 池已耗盡
    }
    
    uint8_t* block = pool->free_list;
    pool->free_list = *(uint8_t**)block;
    pool->free_count--;
    
    return block;
}

// 釋放回池
void mem_pool_free(mem_pool_t* pool, void* ptr) {
    if (ptr == NULL) return;
    
    // 加入空閒串列
    *(uint8_t**)ptr = pool->free_list;
    pool->free_list = ptr;
    pool->free_count++;
}
```

## 🗑️ 記憶體釋放

### 安全釋放模式
```c
// 釋放前始終檢查是否為 NULL
void safe_free(void** ptr) {
    if (ptr != NULL && *ptr != NULL) {
        free(*ptr);
        *ptr = NULL;  // 防止釋放後使用
    }
}

// 使用範例
void cleanup_example() {
    uint8_t* buffer = malloc(1024);
    // 使用 buffer...
    
    safe_free((void**)&buffer);
    // buffer 現在是 NULL
}
```

### 記憶體洩漏預防
```c
// 在除錯建置中追蹤配置
#ifdef DEBUG
static size_t total_allocated = 0;
static size_t peak_allocated = 0;

void* debug_malloc(size_t size) {
    void* ptr = malloc(size);
    if (ptr != NULL) {
        total_allocated += size;
        if (total_allocated > peak_allocated) {
            peak_allocated = total_allocated;
        }
    }
    return ptr;
}

void debug_free(void* ptr) {
    if (ptr != NULL) {
        // 注意：這是簡化版 - 實際實作需要追蹤大小
        free(ptr);
    }
}
#endif
```

## 🛡️ 記憶體安全

### 緩衝區溢位預防
```c
// 安全字串操作
void safe_strcpy(char* dest, const char* src, size_t dest_size) {
    if (dest == NULL || src == NULL || dest_size == 0) {
        return;
    }
    
    size_t i;
    for (i = 0; i < dest_size - 1 && src[i] != '\0'; i++) {
        dest[i] = src[i];
    }
    dest[i] = '\0';  // 始終以 null 結尾
}

// 安全陣列存取
#define SAFE_ARRAY_ACCESS(array, index, size) \
    ((index) < (size) ? &(array)[index] : NULL)

// 使用方式
void safe_array_example() {
    uint8_t buffer[64];
    uint8_t* element = SAFE_ARRAY_ACCESS(buffer, 32, 64);
    if (element != NULL) {
        *element = 42;
    }
}
```

### 記憶體初始化
```c
// 將記憶體初始化為已知狀態
void secure_memset(void* ptr, int value, size_t size) {
    volatile uint8_t* p = (volatile uint8_t*)ptr;
    for (size_t i = 0; i < size; i++) {
        p[i] = (uint8_t)value;
    }
}

// 清除敏感資料
void clear_sensitive_data(uint8_t* data, size_t size) {
    secure_memset(data, 0, size);
}

// 注意：使用 volatile 寫入迴圈是防止編譯器最佳化掉清除操作的
// 常見技術，但 C 標準並未完全保證此行為。若可用，優先使用
// 符合標準的 API，如 memset_s（C11 附錄 K，可選）或
// 編譯器特定的內在函式/pragma。
#if defined(__STDC_LIB_EXT1__)
void clear_sensitive_data_portable(uint8_t* data, rsize_t size) {
    memset_s(data, size, 0, size);
}
#endif

> 平台注意事項：在獨立式/裸機目標上，`malloc`/`free` 可能不可用或
> 在即時路徑中不可取。優先使用靜態配置和記憶體池以確保可預測性。
```

## ⚠️ 常見陷阱

### 記憶體洩漏
```c
// 錯誤：記憶體洩漏
void memory_leak_example() {
    uint8_t* buffer = malloc(1024);
    // 使用 buffer...
    // 忘記釋放 - 記憶體洩漏！
}

// 正確：適當的清理
void proper_cleanup_example() {
    uint8_t* buffer = malloc(1024);
    if (buffer == NULL) {
        return;
    }
    
    // 使用 buffer...
    
    free(buffer);
    buffer = NULL;
}
```

### 釋放後使用
```c
// 錯誤：釋放後使用
void use_after_free_example() {
    uint8_t* buffer = malloc(1024);
    free(buffer);
    buffer[0] = 42;  // 未定義行為！
}

// 正確：釋放後將指標設為 NULL
void safe_free_example() {
    uint8_t* buffer = malloc(1024);
    free(buffer);
    buffer = NULL;  // 防止意外使用
}
```

### 堆疊溢位
```c
// 錯誤：大型堆疊配置
void stack_overflow_example() {
    uint8_t large_buffer[8192];  // 可能造成堆疊溢位
    // 使用 buffer...
}

// 正確：對大型配置使用堆積
void safe_large_allocation() {
    uint8_t* buffer = malloc(8192);
    if (buffer != NULL) {
        // 使用 buffer...
        free(buffer);
    }
}
```

## ✅ 最佳實踐

### 記憶體管理指南
```c
// 1. 始終檢查配置是否成功
void* ptr = malloc(size);
if (ptr == NULL) {
    // 處理配置失敗
    return ERROR_CODE;
}

// 2. 對唯讀資料使用 const
const uint8_t* read_only_data = get_sensor_data();

// 3. 初始化變數
uint8_t buffer[64] = {0};  // 零初始化

// 4. 使用適當的資料型別
uint8_t small_value = 42;      // 0-255
uint16_t medium_value = 1000;  // 0-65535
uint32_t large_value = 1000000; // 0-4294967295

// 5. 存取前檢查邊界
if (index < array_size) {
    array[index] = value;
}
```

### 嵌入式特定模式
```c
// 固定大小配置的記憶體池
#define SENSOR_DATA_SIZE 32
#define MAX_SENSORS 16

static uint8_t sensor_pool[SENSOR_DATA_SIZE * MAX_SENSORS];
static bool pool_used[MAX_SENSORS] = {false};

uint8_t* allocate_sensor_buffer() {
    for (int i = 0; i < MAX_SENSORS; i++) {
        if (!pool_used[i]) {
            pool_used[i] = true;
            return &sensor_pool[i * SENSOR_DATA_SIZE];
        }
    }
    return NULL;  // 沒有可用的空間
}

void free_sensor_buffer(uint8_t* buffer) {
    if (buffer == NULL) return;
    
    // 從指標計算索引
    size_t index = (buffer - sensor_pool) / SENSOR_DATA_SIZE;
    if (index < MAX_SENSORS) {
        pool_used[index] = false;
    }
}
```

## 🎯 面試題目

### 基本概念
1. **堆疊記憶體和堆積記憶體有什麼區別？**
   - 堆疊：自動配置/釋放，後進先出（LIFO），大小有限
   - 堆積：手動配置/釋放，大小靈活，可能碎片化

2. **如何在嵌入式系統中防止記憶體洩漏？**
   - 盡可能使用靜態配置
   - 始終釋放已配置的記憶體
   - 對固定大小配置使用記憶體池
   - 釋放後將指標設為 NULL

3. **什麼是記憶體碎片化？如何預防？**
   - 碎片化發生在空閒記憶體分散在小塊中
   - 預防：使用記憶體池，將相似大小的區塊配置在一起

### 進階主題
1. **如何為嵌入式系統實作記憶體池？**
   - 預配置固定大小的區塊
   - 維護空閒串列
   - O(1) 配置和釋放
   - 無碎片化

2. **靜態配置和動態配置的權衡是什麼？**
   - 靜態：可預測，無執行時開銷，靈活性有限
   - 動態：靈活，有執行時開銷，可能碎片化

3. **如何在嵌入式系統中處理記憶體配置失敗？**
   - 檢查回傳值
   - 實作優雅降級
   - 對關鍵元件使用靜態配置
   - 監控記憶體使用

## 📚 其他資源

- **書籍**：《Making Embedded Systems》Elecia White 著
- **標準**：MISRA C 記憶體管理指南
- **工具**：Valgrind、AddressSanitizer 用於記憶體除錯
- **文件**：ARM Cortex-M 記憶體模型文件

**下一主題：** [指標與記憶體位址](./Pointers_Memory_Addresses.md) → [型別限定詞](./Type_Qualifiers.md)
