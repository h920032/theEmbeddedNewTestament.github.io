# 對齊記憶體配置

## 📋 目錄
- [概述](#-概述)
- [記憶體對齊概念](#-記憶體對齊概念)
- [對齊要求](#-對齊要求)
- [對齊配置技術](#-對齊配置技術)
- [硬體特定對齊](#-硬體特定對齊)
- [效能考量](#-效能考量)
- [常見陷阱](#-常見陷阱)
- [最佳實踐](#-最佳實踐)
- [面試問題](#-面試問題)
- [額外資源](#-額外資源)

## 🎯 概述

對齊記憶體配置在嵌入式系統中至關重要，因為硬體對最佳效能和正確運作有特定的對齊要求。本指南涵蓋了以特定對齊約束條件配置記憶體的技術。

## 🔧 記憶體對齊概念

### 什麼是記憶體對齊？
記憶體對齊是指將資料放置在特定值的倍數（對齊邊界）的記憶體位址上。

```c
// 範例：4 位元組對齊
struct aligned_data {
    uint32_t value;  // 需要 4 位元組對齊
    uint8_t flag;    // 可以在任何位址
} __attribute__((aligned(4)));
```

### 為什麼對齊在嵌入式系統中很重要
- **效能**：未對齊的存取可能導致多次記憶體週期
- **硬體要求**：某些周邊裝置需要特定的對齊
- **快取效率**：對齊的資料可改善快取效能
- **DMA 要求**：DMA 傳輸通常需要特定的對齊

## 📏 對齊要求

### 常見的對齊要求
```c
// 不同的對齊要求
#define ALIGN_1   1   // 無特殊對齊
#define ALIGN_2   2   // 2 位元組對齊
#define ALIGN_4   4   // 4 位元組對齊
#define ALIGN_8   8   // 8 位元組對齊
#define ALIGN_16  16  // 16 位元組對齊（快取行）
#define ALIGN_32  32  // 32 位元組對齊（AVX）
#define ALIGN_64  64  // 64 位元組對齊（AVX-512）
```

### ARM 架構對齊
```c
// ARM 特定的對齊要求
#ifdef __ARM_ARCH_7A__
    #define ARM_ALIGN 8   // ARMv7-A 通常 8 位元組對齊
#elif defined(__ARM_ARCH_8A__)
    #define ARM_ALIGN 16  // ARMv8-A 通常 16 位元組對齊
#endif
```

## 🛠️ 對齊配置技術

### 1. 使用 `aligned_alloc()`（C11）
```c
#include <stdlib.h>

void* aligned_malloc_example() {
    // 配置 1024 位元組並以 16 位元組對齊
    void* ptr = aligned_alloc(16, 1024);
    if (ptr == NULL) {
        // 處理配置失敗
        return NULL;
    }
    return ptr;
}
```

### 2. 使用 `posix_memalign()`
```c
#include <stdlib.h>

int posix_aligned_alloc_example() {
    void* ptr;
    int result = posix_memalign(&ptr, 16, 1024);
    if (result != 0) {
        // 處理錯誤
        return -1;
    }
    // 使用 ptr...
    free(ptr);
    return 0;
}
```

### 3. 手動對齊計算
```c
#include <stdint.h>

void* manual_aligned_alloc(size_t alignment, size_t size) {
    // 計算所需的填充
    size_t padding = alignment - 1;
    size_t total_size = size + padding;
    
    // 配置額外空間
    void* raw_ptr = malloc(total_size);
    if (raw_ptr == NULL) {
        return NULL;
    }
    
    // 計算對齊後的位址
    uintptr_t raw_addr = (uintptr_t)raw_ptr;
    uintptr_t aligned_addr = (raw_addr + padding) & ~padding;
    
    return (void*)aligned_addr;
}
```

### 4. 使用編譯器屬性
```c
// GCC/Clang 對齊屬性
struct __attribute__((aligned(16))) aligned_struct {
    uint32_t data[4];
    uint8_t flags;
};

// 配置對齊的結構體
aligned_struct* create_aligned_struct() {
    return (aligned_struct*)malloc(sizeof(aligned_struct));
}
```

## 🔧 硬體特定對齊

### DMA 緩衝區對齊
```c
// 快取行對齊的 DMA 緩衝區
#define DMA_ALIGNMENT 64  // 快取行大小

typedef struct {
    uint8_t buffer[1024];
} __attribute__((aligned(DMA_ALIGNMENT))) dma_buffer_t;

dma_buffer_t* allocate_dma_buffer() {
    dma_buffer_t* buffer = (dma_buffer_t*)aligned_alloc(
        DMA_ALIGNMENT, 
        sizeof(dma_buffer_t)
    );
    
    if (buffer) {
        // 確保緩衝區為 DMA 快取行對齊
        // 必要時刷新快取
        __builtin___clear_cache((char*)buffer, 
                               (char*)buffer + sizeof(dma_buffer_t));
    }
    
    return buffer;
}
```

### SIMD 向量對齊
```c
// ARM NEON 的 SIMD 向量對齊
#ifdef __ARM_NEON
    #include <arm_neon.h>
    
    typedef struct {
        float32x4_t vector_data[4];  // 16 位元組對齊
    } __attribute__((aligned(16))) neon_buffer_t;
    
    neon_buffer_t* allocate_neon_buffer() {
        return (neon_buffer_t*)aligned_alloc(16, sizeof(neon_buffer_t));
    }
#endif
```

### 周邊暫存器對齊
```c
// 硬體暫存器結構對齊
typedef struct {
    volatile uint32_t control;    // 0x00
    volatile uint32_t status;     // 0x04
    volatile uint32_t data;       // 0x08
    volatile uint32_t reserved;   // 0x0C
} __attribute__((aligned(4))) peripheral_regs_t;

// 映射周邊暫存器
peripheral_regs_t* map_peripheral(uintptr_t base_addr) {
    // 確保基址為 4 位元組對齊
    if (base_addr & 0x3) {
        return NULL;  // 無效的對齊
    }
    return (peripheral_regs_t*)base_addr;
}
```

## ⚡ 效能考量

### 快取行對齊
```c
// 快取行對齊的資料結構
#define CACHE_LINE_SIZE 64

typedef struct {
    uint32_t data[16];  // 64 位元組
} __attribute__((aligned(CACHE_LINE_SIZE))) cache_aligned_data_t;

// 在多核心系統中避免偽共享
typedef struct {
    uint32_t core1_data[16];
    char padding[CACHE_LINE_SIZE - 64];  // 填充到下一快取行
    uint32_t core2_data[16];
} __attribute__((aligned(CACHE_LINE_SIZE))) multi_core_data_t;
```

### 記憶體存取模式
```c
// 帶對齊的優化記憶體複製
void aligned_memory_copy(void* dst, const void* src, size_t size) {
    // 確保兩個指標都是 8 位元組對齊
    if (((uintptr_t)dst & 0x7) == 0 && ((uintptr_t)src & 0x7) == 0) {
        // 使用 64 位元傳輸
        uint64_t* d64 = (uint64_t*)dst;
        const uint64_t* s64 = (const uint64_t*)src;
        size_t count = size / 8;
        
        for (size_t i = 0; i < count; i++) {
            d64[i] = s64[i];
        }
        
        // 處理剩餘位元組
        uint8_t* d8 = (uint8_t*)(d64 + count);
        const uint8_t* s8 = (const uint8_t*)(s64 + count);
        for (size_t i = 0; i < (size % 8); i++) {
            d8[i] = s8[i];
        }
    } else {
        // 回退到逐位元組複製
        memcpy(dst, src, size);
    }
}
```

## ⚠️ 常見陷阱

### 1. 不正確的對齊計算
```c
// 錯誤：這不能保證對齊
void* wrong_aligned_alloc(size_t alignment, size_t size) {
    return malloc(size + alignment);  // 錯誤的方法
}

// 正確：適當的對齊計算
void* correct_aligned_alloc(size_t alignment, size_t size) {
    size_t padding = alignment - 1;
    size_t total_size = size + padding;
    void* raw_ptr = malloc(total_size);
    if (!raw_ptr) return NULL;
    
    uintptr_t raw_addr = (uintptr_t)raw_ptr;
    uintptr_t aligned_addr = (raw_addr + padding) & ~padding;
    return (void*)aligned_addr;
}
```

### 2. 忘記釋放對齊的記憶體
```c
// 錯誤：對 aligned_alloc() 使用 free()
void* ptr = aligned_alloc(16, 1024);
// ... 使用 ptr ...
free(ptr);  // 可能有效但不保證

// 正確：使用適當的釋放函式
void* ptr = aligned_alloc(16, 1024);
// ... 使用 ptr ...
free(ptr);  // aligned_alloc 使用標準 free
```

### 3. 未對齊的結構體成員
```c
// 錯誤：帶對齊要求的壓縮結構體
struct __attribute__((packed)) misaligned_struct {
    uint8_t flag;
    uint32_t data;  // 可能未對齊
};

// 正確：在壓縮結構體中考慮對齊
struct __attribute__((packed)) correct_struct {
    uint8_t flag;
    uint8_t padding[3];  // 手動填充以對齊
    uint32_t data;
};
```

## ✅ 最佳實踐

### 1. 使用標準函式庫函式
```c
// 在可用時優先使用標準函式
void* allocate_aligned(size_t alignment, size_t size) {
    #if __STDC_VERSION__ >= 201112L
        return aligned_alloc(alignment, size);
    #else
        // 回退實作
        return manual_aligned_alloc(alignment, size);
    #endif
}
```

### 2. 驗證對齊要求
```c
bool is_valid_alignment(size_t alignment) {
    // 對齊值必須是 2 的冪次方
    return (alignment != 0) && ((alignment & (alignment - 1)) == 0);
}

void* safe_aligned_alloc(size_t alignment, size_t size) {
    if (!is_valid_alignment(alignment)) {
        return NULL;
    }
    return aligned_alloc(alignment, size);
}
```

### 3. 為頻繁配置考慮使用記憶體池
```c
typedef struct {
    void* pool;
    size_t alignment;
    size_t block_size;
    size_t total_blocks;
    size_t used_blocks;
} aligned_memory_pool_t;

aligned_memory_pool_t* create_aligned_pool(size_t alignment, 
                                          size_t block_size, 
                                          size_t num_blocks) {
    aligned_memory_pool_t* pool = malloc(sizeof(aligned_memory_pool_t));
    if (!pool) return NULL;
    
    pool->alignment = alignment;
    pool->block_size = block_size;
    pool->total_blocks = num_blocks;
    pool->used_blocks = 0;
    
    pool->pool = aligned_alloc(alignment, block_size * num_blocks);
    if (!pool->pool) {
        free(pool);
        return NULL;
    }
    
    return pool;
}
```

### 4. 除錯對齊問題
```c
#include <assert.h>

void debug_alignment(void* ptr, size_t alignment) {
    uintptr_t addr = (uintptr_t)ptr;
    assert((addr % alignment) == 0);
    printf("指標 %p 為 %zu 位元組對齊\n", ptr, alignment);
}

// 使用方式
void* ptr = aligned_alloc(16, 1024);
debug_alignment(ptr, 16);
```

## 🎯 面試問題

### 基礎問題
1. **什麼是記憶體對齊，為什麼在嵌入式系統中很重要？**
   - 記憶體對齊將資料放置在特定值的倍數的位址上
   - 對效能、硬體要求和快取效率至關重要

2. **如何配置 16 位元組對齊的記憶體？**
   ```c
   void* ptr = aligned_alloc(16, size);
   // 或
   void* ptr;
   posix_memalign(&ptr, 16, size);
   ```

3. **在 ARM 上存取未對齊資料會發生什麼？**
   - 可能導致對齊錯誤
   - 因多次記憶體存取而導致效能下降
   - 在嚴格對齊架構上產生硬體例外

### 進階問題
1. **如何實作具有特定對齊的記憶體池？**
   - 預先配置對齊的記憶體區塊
   - 追蹤可用/已用區塊
   - 確保所有配置維持對齊

2. **不同對齊大小之間的權衡是什麼？**
   - 較大的對齊：較佳效能，較多記憶體浪費
   - 較小的對齊：較少浪費，潛在的效能影響

3. **如何在跨平台嵌入式系統中處理對齊？**
   - 針對不同架構使用條件編譯
   - 在執行時期實作對齊偵測
   - 使用可移植的對齊巨集

## 📚 額外資源

### 標準與文件
- **C11 標準**：`aligned_alloc()` 規格
- **POSIX**：`posix_memalign()` 文件
- **ARM 架構參考手冊**：對齊要求
- **GCC 文件**：`__attribute__((aligned))`

### 相關主題
- **[記憶體池配置](./Memory_Pool_Allocation.md)** - 高效記憶體管理
- **[結構體對齊](./Structure_Alignment.md)** - 資料結構對齊
- **[DMA 緩衝區管理](./DMA_Buffer_Management.md)** - DMA 特定對齊
- **[快取感知程式設計](./Cache_Aware_Programming.md)** - 快取行對齊

### 工具與函式庫
- **Valgrind**：記憶體對齊檢查
- **AddressSanitizer**：對齊違規偵測
- **GCC/Clang**：對齊屬性和內建函式

---

**下一主題：** [記憶體碎片化](./Memory_Fragmentation.md) → [記憶體洩漏偵測](./Memory_Leak_Detection.md)
