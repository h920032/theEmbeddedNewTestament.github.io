# 堆疊溢位預防

## 📋 目錄
- [概述](#-概述)
- [堆疊記憶體基礎](#-堆疊記憶體基礎)
- [堆疊溢位原因](#-堆疊溢位原因)
- [堆疊大小分析](#-堆疊大小分析)
- [靜態分析工具](#-靜態分析工具)
- [動態堆疊監控](#-動態堆疊監控)
- [預防技術](#-預防技術)
- [即時堆疊保護](#-即時堆疊保護)
- [常見陷阱](#-常見陷阱)
- [最佳實踐](#-最佳實踐)
- [面試問題](#-面試問題)
- [額外資源](#-額外資源)

## 🎯 概述

堆疊溢位發生在呼叫堆疊超出其配置的記憶體空間時。在嵌入式系統中，這可能導致不可預測的行為、系統崩潰或安全漏洞。本指南涵蓋了預防和偵測堆疊溢位條件的技術。

## 🔧 堆疊記憶體基礎

### 堆疊佈局
```c
// 嵌入式系統中的典型堆疊佈局
typedef struct {
    uint8_t stack[STACK_SIZE];     // 堆疊記憶體
    uint8_t guard_zone[GUARD_SIZE]; // 堆疊保護區
    size_t stack_pointer;           // 目前堆疊指標
    size_t max_usage;               // 最大堆疊使用量
} stack_monitor_t;

#define STACK_SIZE 2048
#define GUARD_SIZE 64
```

### 堆疊框架分析
```c
// 分析堆疊框架大小
void analyze_stack_frame() {
    int local_var1 = 10;           // 4 位元組
    char local_var2[100];          // 100 位元組
    double local_var3 = 3.14;      // 8 位元組
    // 合計：112 位元組 + 對齊填充
    
    printf("估計堆疊框架大小：約 120 位元組\n");
}

// 帶堆疊分析的遞迴函式
int recursive_function(int n) {
    int local_var = n * 2;         // 每次呼叫 4 位元組
    char buffer[50];               // 每次呼叫 50 位元組
    
    if (n <= 0) return 0;
    
    // 堆疊使用量：每次遞迴呼叫約 60 位元組
    // 最大深度：STACK_SIZE / 60 ≈ 34 次呼叫
    return recursive_function(n - 1) + local_var;
}
```

## 🚨 堆疊溢位原因

### 1. 深度遞迴
```c
// 錯誤：無限制的遞迴
int factorial_recursive(int n) {
    if (n <= 1) return 1;
    return n * factorial_recursive(n - 1);  // 大 n 值時堆疊溢位
}

// 正確：尾遞迴或迭代
int factorial_iterative(int n) {
    int result = 1;
    for (int i = 2; i <= n; i++) {
        result *= i;
    }
    return result;
}

// 正確：尾遞迴（編譯器可以最佳化）
int factorial_tail_recursive(int n, int acc) {
    if (n <= 1) return acc;
    return factorial_tail_recursive(n - 1, n * acc);
}
```

### 2. 大型區域陣列
```c
// 錯誤：大型堆疊配置
void large_stack_array() {
    char buffer[10000];  // 10KB 在堆疊上 - 可能溢位
    // 使用緩衝區...
}

// 正確：使用堆積或靜態配置
void safe_large_buffer() {
    char* buffer = malloc(10000);  // 堆積配置
    if (buffer) {
        // 使用緩衝區...
        free(buffer);
    }
}

// 正確：靜態配置
void static_large_buffer() {
    static char buffer[10000];  // 靜態配置
    // 使用緩衝區...
}
```

### 3. 函式呼叫鏈
```c
// 錯誤：深度呼叫鏈
void function_a() { function_b(); }
void function_b() { function_c(); }
void function_c() { function_d(); }
// ... 更多層級
void function_z() { 
    char buffer[1000];  // 在深層級使用大量堆疊
    // 使用緩衝區...
}

// 正確：扁平化呼叫鏈
void flattened_function() {
    // 合併邏輯以減少呼叫深度
    char buffer[1000];
    // 所有邏輯在一個函式中
}
```

## 📊 堆疊大小分析

### 靜態堆疊分析
```c
// 靜態計算堆疊使用量
typedef struct {
    const char* function_name;
    size_t stack_usage;
    size_t max_depth;
} stack_analysis_t;

stack_analysis_t analyze_function_stack(const char* func_name) {
    stack_analysis_t analysis = {0};
    analysis.function_name = func_name;
    
    // 這將由靜態分析工具完成
    // 為了示範，我們進行估計
    if (strcmp(func_name, "main") == 0) {
        analysis.stack_usage = 256;  // 估計的堆疊使用量
        analysis.max_depth = 1;
    } else if (strcmp(func_name, "recursive_function") == 0) {
        analysis.stack_usage = 60;   // 每次呼叫
        analysis.max_depth = 30;     // 最大安全深度
    }
    
    return analysis;
}
```

### 動態堆疊監控
```c
// 在執行時期監控堆疊使用量
typedef struct {
    void* stack_start;
    void* stack_end;
    size_t max_usage;
    size_t current_usage;
} stack_monitor_t;

stack_monitor_t* create_stack_monitor(void* stack_start, size_t stack_size) {
    stack_monitor_t* monitor = malloc(sizeof(stack_monitor_t));
    if (!monitor) return NULL;
    
    monitor->stack_start = stack_start;
    monitor->stack_end = (char*)stack_start + stack_size;
    monitor->max_usage = 0;
    monitor->current_usage = 0;
    
    return monitor;
}

size_t get_stack_usage(stack_monitor_t* monitor) {
    void* current_sp;
    asm volatile ("mov %0, sp" : "=r" (current_sp));
    
    size_t usage = (char*)monitor->stack_end - (char*)current_sp;
    
    if (usage > monitor->max_usage) {
        monitor->max_usage = usage;
    }
    
    monitor->current_usage = usage;
    return usage;
}

void check_stack_overflow(stack_monitor_t* monitor) {
    size_t usage = get_stack_usage(monitor);
    size_t total_size = (char*)monitor->stack_end - (char*)monitor->stack_start;
    
    if (usage > total_size * 0.8) {  // 80% 閾值
        printf("警告：堆疊使用量高：%zu/%zu 位元組 (%.1f%%)\n",
               usage, total_size, (float)usage / total_size * 100);
    }
    
    if (usage >= total_size) {
        printf("錯誤：偵測到堆疊溢位！\n");
        // 處理溢位 - 系統重設、錯誤處理等
    }
}
```

## 🔍 靜態分析工具

### 編譯器堆疊分析
```c
// GCC 堆疊使用量分析
// 編譯方式：gcc -fstack-usage -Wstack-usage=1024

void function_with_stack_analysis() {
    char buffer[500];  // 如果堆疊使用量 > 1024，編譯器會發出警告
    // 使用緩衝區...
}

// 編譯器的堆疊使用量報告：
// function_with_stack_analysis: 500 動態, 0 靜態
```

### 自訂堆疊分析器
```c
// 簡單的堆疊使用量靜態分析
typedef struct {
    const char* function;
    size_t estimated_usage;
    bool has_recursion;
    int max_depth;
} stack_analyzer_t;

stack_analyzer_t analyze_stack_pattern(const char* code) {
    stack_analyzer_t analysis = {0};
    
    // 這將解析程式碼並分析：
    // - 區域變數大小
    // - 陣列配置
    // - 遞迴呼叫
    // - 函式呼叫深度
    
    return analysis;
}
```

## 🔍 動態堆疊監控

### 堆疊金絲雀保護
```c
// 用於溢位偵測的堆疊金絲雀
typedef struct {
    uint32_t canary;
    char data[100];
    uint32_t canary_end;
} protected_stack_frame_t;

void function_with_canary() {
    protected_stack_frame_t frame = {
        .canary = 0xDEADBEEF,
        .canary_end = 0xDEADBEEF
    };
    
    // 使用 frame.data...
    
    // 檢查金絲雀值
    if (frame.canary != 0xDEADBEEF || frame.canary_end != 0xDEADBEEF) {
        printf("偵測到堆疊溢位！\n");
        // 處理溢位
    }
}
```

### 堆疊保護區
```c
// 堆疊保護區監控
#define STACK_GUARD_SIZE 64
#define STACK_GUARD_PATTERN 0xAA

typedef struct {
    uint8_t guard_zone[STACK_GUARD_SIZE];
    uint8_t stack_data[STACK_SIZE];
} guarded_stack_t;

guarded_stack_t* create_guarded_stack() {
    guarded_stack_t* stack = malloc(sizeof(guarded_stack_t));
    if (!stack) return NULL;
    
    // 初始化保護區
    for (int i = 0; i < STACK_GUARD_SIZE; i++) {
        stack->guard_zone[i] = STACK_GUARD_PATTERN;
    }
    
    return stack;
}

bool check_guard_zone(guarded_stack_t* stack) {
    for (int i = 0; i < STACK_GUARD_SIZE; i++) {
        if (stack->guard_zone[i] != STACK_GUARD_PATTERN) {
            printf("保護區損壞 - 堆疊溢位！\n");
            return false;
        }
    }
    return true;
}
```

## 🛡️ 預防技術

### 1. 堆疊大小設定
```c
// 根據分析設定堆疊大小
#define MAIN_STACK_SIZE 2048
#define TASK_STACK_SIZE 1024
#define ISR_STACK_SIZE 512

typedef struct {
    uint8_t stack[MAIN_STACK_SIZE];
    size_t max_usage;
} main_stack_t;

// 堆疊大小驗證
bool validate_stack_size(size_t required_size, size_t available_size) {
    if (required_size > available_size * 0.8) {  // 80% 安全餘量
        printf("警告：堆疊大小可能不足\n");
        printf("需要：%zu，可用：%zu\n", required_size, available_size);
        return false;
    }
    return true;
}
```

### 2. 遞迴限制
```c
// 帶深度限制的安全遞迴函式
#define MAX_RECURSION_DEPTH 100

int safe_recursive_function(int n, int depth) {
    if (depth >= MAX_RECURSION_DEPTH) {
        printf("錯誤：超過最大遞迴深度\n");
        return -1;  // 錯誤條件
    }
    
    if (n <= 1) return 1;
    
    return n * safe_recursive_function(n - 1, depth + 1);
}

// 使用方式
int result = safe_recursive_function(10, 0);
```

### 3. 大型資料使用動態記憶體
```c
// 大型資料結構使用堆積
typedef struct {
    size_t size;
    void* data;
} dynamic_buffer_t;

dynamic_buffer_t* create_dynamic_buffer(size_t size) {
    dynamic_buffer_t* buffer = malloc(sizeof(dynamic_buffer_t));
    if (!buffer) return NULL;
    
    buffer->size = size;
    buffer->data = malloc(size);
    
    if (!buffer->data) {
        free(buffer);
        return NULL;
    }
    
    return buffer;
}

void destroy_dynamic_buffer(dynamic_buffer_t* buffer) {
    if (buffer) {
        free(buffer->data);
        free(buffer);
    }
}
```

### 4. 函式內聯
```c
// 內聯小型函式以減少呼叫堆疊
static inline int small_function(int a, int b) {
    return a + b;  // 內聯，無堆疊框架
}

// 對頻繁呼叫的小型函式使用 inline
static inline void update_counter(uint32_t* counter) {
    (*counter)++;
}
```

## ⏱️ 即時堆疊保護

### 堆疊溢位處理常式
```c
// 堆疊溢位例外處理常式
void stack_overflow_handler(void) {
    printf("堆疊溢位例外！\n");
    
    // 儲存關鍵資料
    save_critical_data();
    
    // 重設系統或重啟任務
    system_reset();
}

// 安裝溢位處理常式
void install_stack_overflow_handler(void) {
    // 為堆疊溢位設定例外處理常式
    // 實作取決於架構
}
```

### 任務堆疊監控
```c
// 監控 RTOS 任務中的堆疊使用量
typedef struct {
    void* stack_start;
    size_t stack_size;
    size_t min_free_space;
    const char* task_name;
} task_stack_monitor_t;

void monitor_task_stack(task_stack_monitor_t* monitor) {
    void* current_sp;
    asm volatile ("mov %0, sp" : "=r" (current_sp));
    
    size_t used_space = (char*)monitor->stack_start + monitor->stack_size - (char*)current_sp;
    size_t free_space = monitor->stack_size - used_space;
    
    if (free_space < monitor->min_free_space) {
        printf("警告：任務 '%s' 堆疊空間不足：剩餘 %zu 位元組\n",
               monitor->task_name, free_space);
    }
}
```

### 堆疊使用量剖析
```c
// 隨時間剖析堆疊使用量
typedef struct {
    size_t peak_usage;
    size_t current_usage;
    uint32_t sample_count;
    float average_usage;
} stack_profile_t;

void update_stack_profile(stack_profile_t* profile, size_t current_usage) {
    profile->current_usage = current_usage;
    
    if (current_usage > profile->peak_usage) {
        profile->peak_usage = current_usage;
    }
    
    profile->sample_count++;
    profile->average_usage = ((profile->average_usage * (profile->sample_count - 1)) + 
                             current_usage) / profile->sample_count;
}
```

## ⚠️ 常見陷阱

### 1. 低估堆疊使用量
```c
// 錯誤：低估堆疊使用量
void underestimated_function() {
    char buffer[100];      // 100 位元組
    int array[50];         // 200 位元組
    double values[10];     // 80 位元組
    // 合計：約 400 位元組，但編譯器可能使用更多
    
    // 函式呼叫會增加更多堆疊使用量
    recursive_function(5);  // 額外的堆疊使用量
}

// 正確：保守估計
void conservative_function() {
    // 保守估計堆疊使用量
    // 包含填充、對齊、函式呼叫
    // 加入安全餘量
}
```

### 2. 忽略編譯器最佳化
```c
// 錯誤：假設堆疊使用量可預測
void optimized_function() {
    char buffer[100];
    // 編譯器可能最佳化掉未使用的變數
    // 堆疊使用量可能比預期少
}

// 正確：使用堆疊分析工具
void analyzed_function() {
    char buffer[100];
    // 使用工具驗證實際堆疊使用量
    // 不要依賴手動計算
}
```

### 3. 未考慮中斷上下文
```c
// 錯誤：在 ISR 中使用大量堆疊
void large_isr_function(void) {
    char buffer[1000];  // ISR 中大量堆疊使用
    // ISR 的堆疊空間有限
}

// 正確：ISR 中最小化堆疊使用
void minimal_isr_function(void) {
    // 使用靜態變數或全域緩衝區
    static char buffer[1000];  // 靜態配置
    // 或如有必要使用堆積配置
}
```

## ✅ 最佳實踐

### 1. 堆疊大小規劃
```c
// 根據分析規劃堆疊大小
#define STACK_SIZE_PLANNING

#ifdef STACK_SIZE_PLANNING
    #define MAIN_STACK_SIZE 2048    // 基於分析
    #define TASK1_STACK_SIZE 1024   // 任務 1 需求
    #define TASK2_STACK_SIZE 512    // 任務 2 需求
    #define ISR_STACK_SIZE 256      // ISR 需求
#endif

// 在編譯時驗證堆疊大小
static_assert(MAIN_STACK_SIZE >= 2048, "主堆疊太小");
static_assert(TASK1_STACK_SIZE >= 1024, "任務 1 堆疊太小");
```

### 2. 堆疊使用量監控
```c
// 在開發階段監控堆疊使用量
#ifdef DEBUG
    #define MONITOR_STACK_USAGE() \
        do { \
            size_t usage = get_current_stack_usage(); \
            if (usage > STACK_SIZE * 0.8) { \
                printf("堆疊使用量高：%zu 位元組\n", usage); \
            } \
        } while(0)
#else
    #define MONITOR_STACK_USAGE() ((void)0)
#endif

void function_with_monitoring() {
    MONITOR_STACK_USAGE();
    // 函式程式碼...
    MONITOR_STACK_USAGE();
}
```

### 3. 堆疊溢位偵測
```c
// 實作堆疊溢位偵測
#define STACK_OVERFLOW_DETECTION

#ifdef STACK_OVERFLOW_DETECTION
    #define CHECK_STACK_OVERFLOW() \
        do { \
            if (is_stack_overflow()) { \
                handle_stack_overflow(); \
            } \
        } while(0)
#else
    #define CHECK_STACK_OVERFLOW() ((void)0)
#endif

bool is_stack_overflow(void) {
    // 實作取決於架構
    // 檢查堆疊指標邊界
    return false;  // 佔位符
}

void handle_stack_overflow(void) {
    printf("偵測到堆疊溢位！\n");
    // 實作恢復機制
}
```

### 4. 堆疊使用量文件
```c
// 為函式記錄堆疊使用量
/**
 * @brief 處理已知堆疊使用量的資料
 * @param data 輸入資料
 * @return 處理結果
 * 
 * 堆疊使用量：約 200 位元組
 * - 區域變數：150 位元組
 * - 函式呼叫：50 位元組
 * - 安全餘量：100 位元組
 * 建議總計：300 位元組
 */
int process_data_with_documentation(int data) {
    char buffer[100];      // 100 位元組
    int temp[10];          // 40 位元組
    double result;         // 8 位元組
    // 額外開銷：約 50 位元組
    
    // 處理資料...
    return (int)result;
}
```

## 🎯 面試問題

### 基礎問題
1. **什麼是堆疊溢位，為什麼在嵌入式系統中很危險？**
   - 堆疊溢位：呼叫堆疊超出配置的記憶體
   - 危險：不可預測的行為、系統崩潰、安全漏洞

2. **如何預防堆疊溢位？**
   - 限制遞迴深度
   - 對大型配置使用堆積
   - 監控堆疊使用量
   - 設定適當的堆疊大小

3. **哪些工具可以幫助偵測堆疊溢位？**
   - 靜態分析工具
   - 堆疊金絲雀
   - 執行時期監控
   - 編譯器警告

### 進階問題
1. **如何在嵌入式系統中實作堆疊溢位偵測？**
   - 堆疊金絲雀
   - 保護區
   - 堆疊指標邊界檢查
   - 例外處理常式

2. **堆疊配置和堆積配置之間的權衡是什麼？**
   - 堆疊：更快、自動清理、大小有限
   - 堆積：更大的空間、手動管理、碎片化風險

3. **如何在即時系統中分析堆疊使用量？**
   - 靜態分析最壞情況的堆疊使用量
   - 執行時期監控實際使用量
   - 堆疊剖析工具

## 📚 額外資源

### 標準和文件
- **C 標準**：堆疊行為規格
- **ARM 架構參考**：堆疊管理
- **RTOS 文件**：任務堆疊管理

### 相關主題
- **[記憶體保護](./Memory_Protection.md)** - 記憶體安全機制
- **[記憶體池配置](./Memory_Pool_Allocation.md)** - 高效記憶體管理
- **[快取感知程式設計](./Cache_Aware_Programming.md)** - 記憶體存取模式
- **[即時系統](./Real_Time_Systems.md)** - RTOS 堆疊管理

### 工具和函式庫
- **GCC 堆疊使用量分析**：`-fstack-usage`
- **Valgrind**：堆疊溢位偵測
- **自訂堆疊監控器**：嵌入式專用解決方案

---

**下一個主題：** [記憶體保護](./Memory_Protection.md) → [快取感知程式設計](./Cache_Aware_Programming.md)
