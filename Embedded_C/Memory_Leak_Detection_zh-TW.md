# 記憶體洩漏偵測

## 📋 目錄
- [概述](#-概述)
- [記憶體洩漏類型](#-記憶體洩漏類型)
- [靜態分析工具](#-靜態分析工具)
- [動態分析工具](#-動態分析工具)
- [自訂記憶體追蹤](#-自訂記憶體追蹤)
- [嵌入式專屬偵測](#-嵌入式專屬偵測)
- [即時洩漏偵測](#-即時洩漏偵測)
- [常見洩漏模式](#-常見洩漏模式)
- [預防策略](#-預防策略)
- [最佳實踐](#-最佳實踐)
- [面試問題](#-面試問題)
- [額外資源](#-額外資源)

## 🎯 概述

記憶體洩漏發生在已配置的記憶體未被適當釋放時，導致記憶體逐漸耗盡。在嵌入式系統中，由於資源有限且運行時間長，記憶體洩漏可能是災難性的。

## 🔍 記憶體洩漏類型

### 經典記憶體洩漏
```c
// 經典記憶體洩漏 - 已配置但從未釋放
void classic_memory_leak() {
    void* ptr = malloc(1024);
    // 使用 ptr...
    // 忘記呼叫 free(ptr) - 洩漏！
}

// 錯誤路徑中的記憶體洩漏
void leak_in_error_path() {
    void* ptr1 = malloc(512);
    if (ptr1 == NULL) return;  // 未清理就提前返回
    
    void* ptr2 = malloc(256);
    if (ptr2 == NULL) {
        // 忘記釋放 ptr1 - 洩漏！
        return;
    }
    
    // 使用兩個指標...
    free(ptr1);
    free(ptr2);
}
```

### 資源洩漏
```c
// 檔案控制代碼洩漏
void file_handle_leak() {
    FILE* file = fopen("data.txt", "r");
    if (file == NULL) return;
    
    // 處理檔案...
    // 忘記呼叫 fclose(file) - 資源洩漏！
}

// 互斥鎖洩漏
#include <pthread.h>

void mutex_leak() {
    pthread_mutex_t* mutex = malloc(sizeof(pthread_mutex_t));
    pthread_mutex_init(mutex, NULL);
    
    // 使用互斥鎖...
    // 忘記呼叫 pthread_mutex_destroy() 和 free() - 洩漏！
}
```

### 循環參照洩漏
```c
typedef struct node {
    int data;
    struct node* next;
} node_t;

void circular_reference_leak() {
    node_t* head = malloc(sizeof(node_t));
    node_t* tail = malloc(sizeof(node_t));
    
    head->next = tail;
    tail->next = head;  // 循環參照
    
    // 如果我們只釋放 head，tail 就變得無法存取
    free(head);
    // tail 現在已洩漏！
}
```

## 🔧 靜態分析工具

### 編譯器警告
```c
// 啟用所有警告以偵測記憶體洩漏
// gcc -Wall -Wextra -Werror -fsanitize=address

void potential_leak_function() {
    void* ptr = malloc(100);
    // 編譯器警告：變數 'ptr' 已設值但未使用
    // 這可能表示洩漏
}
```

### 使用 Clang 的靜態分析
```c
// clang --analyze -Xanalyzer -analyzer-output=text

void analyzed_function() {
    void* ptr = malloc(100);
    if (some_condition()) {
        free(ptr);
        return;
    }
    // 靜態分析器偵測到：ptr 在此路徑未被釋放
}
```

### 自訂靜態分析
```c
// 常見模式的簡單靜態分析
#include <regex.h>

void check_for_malloc_without_free(const char* source_code) {
    regex_t malloc_regex, free_regex;
    regcomp(&malloc_regex, "malloc\\s*\\(", REG_EXTENDED);
    regcomp(&free_regex, "free\\s*\\(", REG_EXTENDED);
    
    // 計算 malloc 和 free 的呼叫次數
    // 如果 malloc_count > free_count，可能存在洩漏
}
```

## 🔍 動態分析工具

### AddressSanitizer (ASan)
```c
// 編譯方式：gcc -fsanitize=address -g

void asan_demo() {
    void* ptr = malloc(100);
    // 使用 ptr...
    // 如果我們忘記 free(ptr)，ASan 會回報洩漏
    
    // ASan 也能偵測：
    // - 緩衝區溢位
    // - 釋放後使用
    // - 重複釋放
}

// ASan 輸出範例：
// ==12345==ERROR: LeakSanitizer: detected memory leaks
// ==12345==SUMMARY: AddressSanitizer: 1 byte(s) leaked in 1 allocation(s)
```

### Valgrind
```c
// 執行方式：valgrind --leak-check=full --show-leak-kinds=all

void valgrind_demo() {
    void* ptr = malloc(100);
    // 使用 ptr...
    // Valgrind 會回報：
    // ==12345== 100 bytes in 1 blocks are definitely lost
    // ==12345==    at 0x4C2AB80: malloc (in /usr/lib/valgrind/vgpreload_memcheck.so)
    // ==12345==    by 0x400544: valgrind_demo (demo.c:5)
}
```

### 自訂記憶體追蹤器
```c
typedef struct {
    void* ptr;
    size_t size;
    const char* file;
    int line;
    bool freed;
} allocation_record_t;

typedef struct {
    allocation_record_t* records;
    size_t count;
    size_t capacity;
} memory_tracker_t;

memory_tracker_t* create_memory_tracker(size_t initial_capacity) {
    memory_tracker_t* tracker = malloc(sizeof(memory_tracker_t));
    if (!tracker) return NULL;
    
    tracker->records = calloc(initial_capacity, sizeof(allocation_record_t));
    tracker->count = 0;
    tracker->capacity = initial_capacity;
    
    return tracker;
}

void* tracked_malloc(size_t size, const char* file, int line) {
    void* ptr = malloc(size);
    if (ptr) {
        // 加入追蹤器
        allocation_record_t record = {
            .ptr = ptr,
            .size = size,
            .file = file,
            .line = line,
            .freed = false
        };
        add_allocation_record(&record);
    }
    return ptr;
}

void tracked_free(void* ptr) {
    if (ptr) {
        // 在追蹤器中標記為已釋放
        mark_as_freed(ptr);
        free(ptr);
    }
}
```

## 🔧 嵌入式專屬偵測

### 記憶體池洩漏偵測
```c
typedef struct {
    void* pool;
    size_t block_size;
    size_t total_blocks;
    bool* allocated_blocks;
    size_t allocated_count;
} leak_detecting_pool_t;

leak_detecting_pool_t* create_leak_detecting_pool(size_t block_size, 
                                                  size_t num_blocks) {
    leak_detecting_pool_t* pool = malloc(sizeof(leak_detecting_pool_t));
    if (!pool) return NULL;
    
    pool->block_size = block_size;
    pool->total_blocks = num_blocks;
    pool->allocated_count = 0;
    
    pool->pool = malloc(block_size * num_blocks);
    pool->allocated_blocks = calloc(num_blocks, sizeof(bool));
    
    return pool;
}

void* pool_allocate_with_tracking(leak_detecting_pool_t* pool) {
    for (size_t i = 0; i < pool->total_blocks; i++) {
        if (!pool->allocated_blocks[i]) {
            pool->allocated_blocks[i] = true;
            pool->allocated_count++;
            return (char*)pool->pool + (i * pool->block_size);
        }
    }
    return NULL;
}

void pool_free_with_tracking(leak_detecting_pool_t* pool, void* ptr) {
    size_t block_index = ((char*)ptr - (char*)pool->pool) / pool->block_size;
    if (block_index < pool->total_blocks && pool->allocated_blocks[block_index]) {
        pool->allocated_blocks[block_index] = false;
        pool->allocated_count--;
    }
}

void report_pool_leaks(leak_detecting_pool_t* pool) {
    if (pool->allocated_count > 0) {
        printf("記憶體洩漏：池中有 %zu 個區塊未被釋放\n", 
               pool->allocated_count);
        
        for (size_t i = 0; i < pool->total_blocks; i++) {
            if (pool->allocated_blocks[i]) {
                printf("  區塊 %zu 仍被配置中\n", i);
            }
        }
    }
}
```

### 基於堆疊的記憶體追蹤
```c
// 在嵌入式系統中追蹤堆疊上的配置
#define MAX_STACK_ALLOCATIONS 100

typedef struct {
    void* ptr;
    const char* function;
    int line;
} stack_allocation_t;

static stack_allocation_t allocation_stack[MAX_STACK_ALLOCATIONS];
static int stack_top = 0;

void* stack_tracked_malloc(size_t size, const char* function, int line) {
    void* ptr = malloc(size);
    if (ptr && stack_top < MAX_STACK_ALLOCATIONS) {
        allocation_stack[stack_top].ptr = ptr;
        allocation_stack[stack_top].function = function;
        allocation_stack[stack_top].line = line;
        stack_top++;
    }
    return ptr;
}

void stack_tracked_free(void* ptr) {
    if (ptr) {
        // 從堆疊中尋找並移除
        for (int i = 0; i < stack_top; i++) {
            if (allocation_stack[i].ptr == ptr) {
                // 透過移位來移除
                for (int j = i; j < stack_top - 1; j++) {
                    allocation_stack[j] = allocation_stack[j + 1];
                }
                stack_top--;
                break;
            }
        }
        free(ptr);
    }
}

void check_stack_leaks() {
    if (stack_top > 0) {
        printf("偵測到堆疊洩漏：%d 個配置未被釋放\n", stack_top);
        for (int i = 0; i < stack_top; i++) {
            printf("  %s:%d - %p\n", 
                   allocation_stack[i].function,
                   allocation_stack[i].line,
                   allocation_stack[i].ptr);
        }
    }
}
```

## ⏱️ 即時洩漏偵測

### 輕量級洩漏監控器
```c
typedef struct {
    uint32_t total_allocations;
    uint32_t total_frees;
    uint32_t current_allocated;
    uint32_t peak_allocated;
} rt_leak_monitor_t;

static rt_leak_monitor_t leak_monitor = {0};

void* rt_tracked_malloc(size_t size) {
    void* ptr = malloc(size);
    if (ptr) {
        leak_monitor.total_allocations++;
        leak_monitor.current_allocated++;
        
        if (leak_monitor.current_allocated > leak_monitor.peak_allocated) {
            leak_monitor.peak_allocated = leak_monitor.current_allocated;
        }
    }
    return ptr;
}

void rt_tracked_free(void* ptr) {
    if (ptr) {
        leak_monitor.total_frees++;
        leak_monitor.current_allocated--;
        free(ptr);
    }
}

void report_rt_leaks() {
    printf("配置數：%u，釋放數：%u，目前：%u，峰值：%u\n",
           leak_monitor.total_allocations,
           leak_monitor.total_frees,
           leak_monitor.current_allocated,
           leak_monitor.peak_allocated);
    
    if (leak_monitor.current_allocated > 0) {
        printf("潛在洩漏：%u 個區塊未被釋放\n", 
               leak_monitor.current_allocated);
    }
}
```

### 定期洩漏檢查
```c
// 在即時系統中定期檢查洩漏
void periodic_leak_check() {
    static uint32_t check_counter = 0;
    check_counter++;
    
    if (check_counter % 10000 == 0) {  // 每 10k 次配置檢查一次
        uint32_t leak_count = leak_monitor.total_allocations - leak_monitor.total_frees;
        
        if (leak_count > 100) {  // 洩漏偵測閾值
            printf("警告：偵測到潛在記憶體洩漏\n");
            report_rt_leaks();
        }
    }
}
```

## 🚨 常見洩漏模式

### 1. 例外/錯誤路徑洩漏
```c
// 錯誤：錯誤路徑中的洩漏
void* allocate_with_error_leak() {
    void* ptr1 = malloc(100);
    if (!ptr1) return NULL;
    
    void* ptr2 = malloc(200);
    if (!ptr2) {
        // 洩漏：ptr1 未被釋放
        return NULL;
    }
    
    // 使用兩個指標...
    free(ptr1);
    free(ptr2);
    return ptr2;
}

// 正確：適當的清理
void* allocate_with_cleanup() {
    void* ptr1 = malloc(100);
    if (!ptr1) return NULL;
    
    void* ptr2 = malloc(200);
    if (!ptr2) {
        free(ptr1);  // 返回前先清理
        return NULL;
    }
    
    // 使用兩個指標...
    free(ptr1);
    free(ptr2);
    return ptr2;
}
```

### 2. 迴圈洩漏
```c
// 錯誤：迴圈中的洩漏
void loop_leak() {
    for (int i = 0; i < 1000; i++) {
        void* ptr = malloc(100);
        // 使用 ptr...
        // 忘記 free(ptr) - 洩漏！
    }
}

// 正確：在迴圈中釋放
void loop_no_leak() {
    for (int i = 0; i < 1000; i++) {
        void* ptr = malloc(100);
        // 使用 ptr...
        free(ptr);  // 下次迭代前釋放
    }
}
```

### 3. 函式出口洩漏
```c
// 錯誤：多個出口點未進行清理
void* function_with_exit_leaks() {
    void* ptr = malloc(100);
    
    if (condition1()) {
        return ptr;  // 洩漏：ptr 未被釋放
    }
    
    if (condition2()) {
        return NULL;  // 洩漏：ptr 未被釋放
    }
    
    // 使用 ptr...
    free(ptr);
    return NULL;
}

// 正確：單一出口點或適當清理
void* function_with_cleanup() {
    void* ptr = malloc(100);
    if (!ptr) return NULL;
    
    void* result = NULL;
    
    if (condition1()) {
        result = ptr;
    } else if (condition2()) {
        free(ptr);
        result = NULL;
    } else {
        // 使用 ptr...
        free(ptr);
        result = NULL;
    }
    
    return result;
}
```

## 🛡️ 預防策略

### 1. C 語言中的 RAII 模式
```c
typedef struct {
    void* ptr;
    void (*cleanup)(void*);
} raii_wrapper_t;

raii_wrapper_t* create_raii_wrapper(void* ptr, void (*cleanup)(void*)) {
    raii_wrapper_t* wrapper = malloc(sizeof(raii_wrapper_t));
    if (wrapper) {
        wrapper->ptr = ptr;
        wrapper->cleanup = cleanup;
    }
    return wrapper;
}

void destroy_raii_wrapper(raii_wrapper_t* wrapper) {
    if (wrapper && wrapper->cleanup) {
        wrapper->cleanup(wrapper->ptr);
    }
    free(wrapper);
}

// 使用方式
void example_raii() {
    raii_wrapper_t* wrapper = create_raii_wrapper(
        malloc(100), 
        free
    );
    
    // 使用 wrapper->ptr...
    
    destroy_raii_wrapper(wrapper);  // 自動釋放 ptr
}
```

### 2. 帶自動清理的記憶體池
```c
typedef struct {
    void* pool;
    size_t block_size;
    size_t total_blocks;
    bool* allocated_blocks;
} auto_cleanup_pool_t;

auto_cleanup_pool_t* create_auto_cleanup_pool(size_t block_size, 
                                              size_t num_blocks) {
    auto_cleanup_pool_t* pool = malloc(sizeof(auto_cleanup_pool_t));
    if (!pool) return NULL;
    
    pool->block_size = block_size;
    pool->total_blocks = num_blocks;
    pool->pool = malloc(block_size * num_blocks);
    pool->allocated_blocks = calloc(num_blocks, sizeof(bool));
    
    return pool;
}

void destroy_auto_cleanup_pool(auto_cleanup_pool_t* pool) {
    if (pool) {
        // 銷毀前檢查洩漏
        size_t leak_count = 0;
        for (size_t i = 0; i < pool->total_blocks; i++) {
            if (pool->allocated_blocks[i]) {
                leak_count++;
            }
        }
        
        if (leak_count > 0) {
            printf("警告：池中有 %zu 個區塊洩漏\n", leak_count);
        }
        
        free(pool->pool);
        free(pool->allocated_blocks);
        free(pool);
    }
}
```

### 3. 智慧指標模式
```c
typedef struct {
    void* ptr;
    int ref_count;
} smart_ptr_t;

smart_ptr_t* create_smart_ptr(void* ptr) {
    smart_ptr_t* smart = malloc(sizeof(smart_ptr_t));
    if (smart) {
        smart->ptr = ptr;
        smart->ref_count = 1;
    }
    return smart;
}

smart_ptr_t* smart_ptr_ref(smart_ptr_t* smart) {
    if (smart) {
        smart->ref_count++;
    }
    return smart;
}

void smart_ptr_unref(smart_ptr_t* smart) {
    if (smart) {
        smart->ref_count--;
        if (smart->ref_count <= 0) {
            free(smart->ptr);
            free(smart);
        }
    }
}
```

## ✅ 最佳實踐

### 1. 使用一致的配置/釋放模式
```c
// 定義配置模式
#define ALLOCATE_AND_CHECK(ptr, size) \
    do { \
        ptr = malloc(size); \
        if (!ptr) { \
            fprintf(stderr, "配置失敗於 %s:%d\n", __FILE__, __LINE__); \
            return NULL; \
        } \
    } while(0)

#define FREE_AND_NULL(ptr) \
    do { \
        if (ptr) { \
            free(ptr); \
            ptr = NULL; \
        } \
    } while(0)

// 使用方式
void* safe_allocation_example() {
    void* ptr1 = NULL, *ptr2 = NULL;
    
    ALLOCATE_AND_CHECK(ptr1, 100);
    ALLOCATE_AND_CHECK(ptr2, 200);
    
    // 使用指標...
    
    FREE_AND_NULL(ptr2);
    FREE_AND_NULL(ptr1);
    return NULL;
}
```

### 2. 在除錯建置中實作記憶體洩漏偵測
```c
#ifdef DEBUG
    #define MALLOC(size) debug_malloc(size, __FILE__, __LINE__)
    #define FREE(ptr) debug_free(ptr, __FILE__, __LINE__)
#else
    #define MALLOC(size) malloc(size)
    #define FREE(ptr) free(ptr)
#endif

void* debug_malloc(size_t size, const char* file, int line) {
    void* ptr = malloc(size);
    if (ptr) {
        printf("DEBUG: malloc(%zu) 於 %s:%d -> %p\n", size, file, line, ptr);
        add_to_debug_tracker(ptr, size, file, line);
    }
    return ptr;
}

void debug_free(void* ptr, const char* file, int line) {
    if (ptr) {
        printf("DEBUG: free(%p) 於 %s:%d\n", ptr, file, line);
        remove_from_debug_tracker(ptr);
        free(ptr);
    }
}
```

### 3. 定期記憶體審計
```c
void perform_memory_audit() {
    struct mallinfo info = mallinfo();
    
    printf("記憶體審計：\n");
    printf("  已配置總量：%d 位元組\n", info.uordblks);
    printf("  可用總量：%d 位元組\n", info.fordblks);
    printf("  最大可用區塊：%d 位元組\n", info.mxordblk);
    printf("  可用區塊數：%d\n", info.ordblks);
    
    // 檢查潛在洩漏
    if (info.uordblks > 0 && info.fordblks < 1000) {
        printf("警告：可用記憶體不足 - 可能存在洩漏？\n");
    }
}
```

## 🎯 面試問題

### 基礎問題
1. **什麼是記憶體洩漏，為什麼在嵌入式系統中特別嚴重？**
   - 記憶體洩漏：已配置的記憶體未被適當釋放
   - 在嵌入式系統中的問題：資源有限、運行時間長、無虛擬記憶體

2. **如何偵測記憶體洩漏？**
   - 靜態分析工具（編譯器警告、靜態分析器）
   - 動態分析工具（Valgrind、AddressSanitizer）
   - 自訂記憶體追蹤

3. **記憶體洩漏的常見原因是什麼？**
   - 忘記釋放已配置的記憶體
   - 錯誤路徑未進行清理
   - 循環參照
   - 例外處理未進行清理

### 進階問題
1. **如何為嵌入式系統實作記憶體洩漏偵測器？**
   - 低負擔的輕量級追蹤
   - 基於堆疊的配置追蹤
   - 定期洩漏檢查

2. **哪些策略可以預防記憶體洩漏？**
   - RAII 模式
   - 智慧指標
   - 記憶體池
   - 一致的配置/釋放模式

3. **如何在即時系統中處理記憶體洩漏？**
   - 預先配置記憶體池
   - 盡可能使用靜態配置
   - 實作輕量級洩漏偵測

## 📚 額外資源

### 標準與文件
- **C 標準函式庫**：記憶體管理函式
- **Valgrind 文件**：記憶體錯誤偵測
- **AddressSanitizer**：執行時期記憶體錯誤偵測器

### 相關主題
- **[記憶體池配置](./Memory_Pool_Allocation.md)** - 高效記憶體管理
- **[記憶體碎片化](./Memory_Fragmentation.md)** - 記憶體碎片化問題
- **[堆疊溢位預防](./Stack_Overflow_Prevention.md)** - 堆疊管理
- **[記憶體保護](./Memory_Protection.md)** - 記憶體安全

### 工具與函式庫
- **Valgrind**：全面的記憶體分析
- **AddressSanitizer**：快速記憶體錯誤偵測
- **自訂記憶體追蹤器**：嵌入式專屬解決方案

---

**下一主題：** [堆疊溢位預防](./Stack_Overflow_Prevention.md) → [記憶體保護](./Memory_Protection.md)
