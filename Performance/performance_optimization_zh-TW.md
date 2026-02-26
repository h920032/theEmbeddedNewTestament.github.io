# 效能最佳化指南

## 概述
在嵌入式系統中，效能最佳化至關重要，因為資源有限、有即時約束條件以及功耗要求。本指南涵蓋必要的最佳化技術、效能分析方法，以及嵌入式軟體開發的最佳實務。

---

## 概念 → 為什麼重要 → 最小範例 → 動手試試 → 重點整理

**概念**：嵌入式系統中的效能最佳化是在有限資源下充分利用系統能力，同時維持可靠性並滿足即時約束條件。這不僅僅是讓程式碼跑得更快，更是理解速度、記憶體使用、功耗與程式碼可維護性之間的權衡取捨。

**為什麼重要**：在嵌入式系統中，效能直接影響電池壽命、回應速度以及是否能滿足即時截止期限。效能不佳可能導致錯過截止期限、過度耗電，甚至系統故障。良好的最佳化可以實現新功能、延長電池壽命並改善使用者體驗。

**最小範例**：一個簡單的迴圈最佳化範例，展示迴圈展開如何透過減少迴圈額外開銷並啟用更好的編譯器最佳化來提升效能。

**動手試試**：對一個簡單的嵌入式應用程式進行效能分析，找出最大的效能瓶頸，然後針對性地套用最佳化並測量實際的改善效果。

**重點整理**：效能最佳化需要測量、了解目標硬體，以及仔細考量各種權衡取捨。最好的最佳化通常來自演算法的改進，而非微觀最佳化。

---

## 目錄
1. [程式碼最佳化技術](#程式碼最佳化技術)
2. [記憶體最佳化策略](#記憶體最佳化策略)
3. [功耗最佳化](#功耗最佳化)
4. [即時效能分析](#即時效能分析)
5. [效能剖析與基準測試](#效能剖析與基準測試)
6. [最佳化工具](#最佳化工具)

---

## 程式碼最佳化技術

### 編譯器最佳化

#### 編譯器旗標
```bash
# 針對嵌入式系統的 GCC 最佳化旗標
gcc -O2 -march=armv7-a -mtune=cortex-a7 -mfpu=neon -mfloat-abi=hard \
    -ffast-math -funroll-loops -fomit-frame-pointer \
    -fno-stack-protector -fno-common -fno-builtin \
    -o optimized_program program.c

# ARM 專用最佳化
gcc -O3 -march=armv8-a -mtune=cortex-a53 \
    -fno-stack-protector -fomit-frame-pointer \
    -ffunction-sections -fdata-sections \
    -Wl,--gc-sections -o program program.c
```

#### 最佳化等級
```c
// 範例：最佳化感知的程式碼結構
// 使用 __attribute__ 為編譯器提供提示
__attribute__((hot)) void performance_critical_function(void) {
    // 此函式被標記為經常呼叫
    for (int i = 0; i < 1000; i++) {
        process_data(i);
    }
}

__attribute__((cold)) void rarely_called_function(void) {
    // 此函式被標記為很少呼叫
    log_debug_info();
}

// 對小型函式強制內聯
static inline uint32_t fast_multiply(uint32_t a, uint32_t b) {
    return a * b;
}
```

### 演算法最佳化

#### 迴圈最佳化
```c
// 未最佳化的迴圈
void unoptimized_loop(int *array, int size) {
    for (int i = 0; i < size; i++) {
        array[i] = array[i] * 2 + 1;
    }
}

// 使用迴圈展開的最佳化迴圈
void optimized_loop(int *array, int size) {
    int i;
    // 展開 4 次
    for (i = 0; i < size - 3; i += 4) {
        array[i] = array[i] * 2 + 1;
        array[i+1] = array[i+1] * 2 + 1;
        array[i+2] = array[i+2] * 2 + 1;
        array[i+3] = array[i+3] * 2 + 1;
    }
    // 處理剩餘元素
    for (; i < size; i++) {
        array[i] = array[i] * 2 + 1;
    }
}

// SIMD 最佳化範例
void simd_optimized_loop(int *array, int size) {
    // 使用 ARM NEON 指令進行向量化
    #ifdef __ARM_NEON
    int32x4_t *vec_array = (int32x4_t*)array;
    int32x4_t multiplier = vdupq_n_s32(2);
    int32x4_t adder = vdupq_n_s32(1);
    
    for (int i = 0; i < size/4; i++) {
        int32x4_t data = vld1q_s32(&array[i*4]);
        data = vmulq_s32(data, multiplier);
        data = vaddq_s32(data, adder);
        vst1q_s32(&array[i*4], data);
    }
    #endif
}
```

#### 資料結構最佳化
```c
// 為快取局部性最佳化的資料結構
typedef struct {
    uint32_t id;
    uint32_t value;
    uint8_t flags;
    uint8_t padding[3];  // 對齊到 8 位元組邊界
} __attribute__((packed)) optimized_struct_t;

// 快取友善的陣列存取
void cache_friendly_access(int *array, int rows, int cols) {
    // 以列優先順序存取資料以獲得更好的快取效能
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            array[i * cols + j] = process_element(i, j);
        }
    }
}

// 避免快取不友善的存取模式
void cache_unfriendly_access(int *array, int rows, int cols) {
    // 此模式會導致快取未命中
    for (int j = 0; j < cols; j++) {
        for (int i = 0; i < rows; i++) {
            array[i * cols + j] = process_element(i, j);
        }
    }
}
```

### 函式最佳化

#### 內聯函式
```c
// 小型函式應該被內聯
static inline uint32_t bit_count(uint32_t x) {
    x = x - ((x >> 1) & 0x55555555);
    x = (x & 0x33333333) + ((x >> 2) & 0x33333333);
    x = (x + (x >> 4)) & 0x0f0f0f0f;
    x = x + (x >> 8);
    x = x + (x >> 16);
    return x & 0x3f;
}

// 對關鍵函式強制內聯
static inline __attribute__((always_inline)) 
uint32_t critical_function(uint32_t input) {
    return input * 7 + 13;
}
```

#### 函式指標最佳化
```c
// 使用函式指標實現多型行為
typedef void (*process_func_t)(int *data, int size);

void process_data_optimized(int *data, int size, process_func_t func) {
    // 避免虛擬函式的額外開銷
    func(data, size);
}

// 使用範例
void process_fast(int *data, int size) {
    // 快速處理實作
    for (int i = 0; i < size; i++) {
        data[i] *= 2;
    }
}

void process_accurate(int *data, int size) {
    // 高精度處理實作
    for (int i = 0; i < size; i++) {
        data[i] = complex_calculation(data[i]);
    }
}
```

---

## 記憶體最佳化策略

### 記憶體配置最佳化

#### 結構體封裝
```c
// 未最佳化的結構體
typedef struct {
    uint8_t a;
    uint32_t b;
    uint8_t c;
    uint16_t d;
} unoptimized_struct_t;  // 大小：12 位元組（4 位元組對齊）

// 最佳化的結構體
typedef struct {
    uint32_t b;    // 4 位元組
    uint16_t d;    // 2 位元組
    uint8_t a;     // 1 位元組
    uint8_t c;     // 1 位元組
} __attribute__((packed)) optimized_struct_t;  // 大小：8 位元組
```

#### 記憶體池配置
```c
// 固定大小配置的記憶體池
typedef struct {
    uint8_t *pool;
    uint32_t pool_size;
    uint32_t block_size;
    uint32_t free_blocks;
    uint32_t *free_list;
} memory_pool_t;

// 初始化記憶體池
int memory_pool_init(memory_pool_t *pool, uint32_t block_size, uint32_t num_blocks) {
    pool->block_size = block_size;
    pool->pool_size = block_size * num_blocks;
    pool->free_blocks = num_blocks;
    
    // 配置池記憶體
    pool->pool = malloc(pool->pool_size);
    if (!pool->pool) return -1;
    
    // 初始化可用清單
    pool->free_list = malloc(num_blocks * sizeof(uint32_t));
    if (!pool->free_list) {
        free(pool->pool);
        return -1;
    }
    
    // 建立可用清單
    for (uint32_t i = 0; i < num_blocks; i++) {
        pool->free_list[i] = i;
    }
    
    return 0;
}

// 從池中配置
void* memory_pool_alloc(memory_pool_t *pool) {
    if (pool->free_blocks == 0) return NULL;
    
    uint32_t block_index = pool->free_list[--pool->free_blocks];
    return pool->pool + (block_index * pool->block_size);
}

// 釋放回池中
void memory_pool_free(memory_pool_t *pool, void *ptr) {
    if (!ptr) return;
    
    uint32_t block_index = ((uint8_t*)ptr - pool->pool) / pool->block_size;
    pool->free_list[pool->free_blocks++] = block_index;
}
```

### 堆疊最佳化

#### 堆疊使用量分析
```c
// 監控堆疊使用量
typedef struct {
    uint32_t max_stack_usage;
    uint32_t current_stack_usage;
    uint8_t *stack_start;
    uint8_t *stack_end;
} stack_monitor_t;

// 初始化堆疊監控器
void stack_monitor_init(stack_monitor_t *monitor, uint8_t *stack_start, uint32_t stack_size) {
    monitor->stack_start = stack_start;
    monitor->stack_end = stack_start + stack_size;
    monitor->max_stack_usage = 0;
    monitor->current_stack_usage = 0;
    
    // 以特定模式填充堆疊以偵測使用量
    for (uint32_t i = 0; i < stack_size; i++) {
        stack_start[i] = 0xAA;
    }
}

// 檢查堆疊使用量
uint32_t stack_monitor_check_usage(stack_monitor_t *monitor) {
    uint8_t *current_stack = (uint8_t*)&current_stack;
    uint32_t usage = monitor->stack_end - current_stack;
    
    if (usage > monitor->max_stack_usage) {
        monitor->max_stack_usage = usage;
    }
    
    return usage;
}
```

#### 遞迴最佳化
```c
// 將遞迴函式轉換為迭代
// 遞迴版本（堆疊使用量大）
int recursive_factorial(int n) {
    if (n <= 1) return 1;
    return n * recursive_factorial(n - 1);
}

// 迭代版本（堆疊使用效率高）
int iterative_factorial(int n) {
    int result = 1;
    for (int i = 2; i <= n; i++) {
        result *= i;
    }
    return result;
}

// 尾遞迴最佳化
int tail_recursive_factorial(int n, int acc) {
    if (n <= 1) return acc;
    return tail_recursive_factorial(n - 1, n * acc);
}
```

### 堆積最佳化

#### 自訂配置器
```c
// 適用於嵌入式系統的簡易自訂配置器
typedef struct {
    uint8_t *heap_start;
    uint8_t *heap_end;
    uint8_t *current_ptr;
    uint32_t total_allocated;
} simple_allocator_t;

// 初始化配置器
void simple_allocator_init(simple_allocator_t *alloc, uint8_t *heap_start, uint32_t heap_size) {
    alloc->heap_start = heap_start;
    alloc->heap_end = heap_start + heap_size;
    alloc->current_ptr = heap_start;
    alloc->total_allocated = 0;
}

// 配置記憶體
void* simple_allocator_alloc(simple_allocator_t *alloc, uint32_t size) {
    // 對齊到 4 位元組邊界
    size = (size + 3) & ~3;
    
    if (alloc->current_ptr + size > alloc->heap_end) {
        return NULL;  // 記憶體不足
    }
    
    void *ptr = alloc->current_ptr;
    alloc->current_ptr += size;
    alloc->total_allocated += size;
    
    return ptr;
}

// 重設配置器（適用於不需要個別釋放的嵌入式系統）
void simple_allocator_reset(simple_allocator_t *alloc) {
    alloc->current_ptr = alloc->heap_start;
    alloc->total_allocated = 0;
}
```

---

## 功耗最佳化

### CPU 電源管理

#### 動態頻率調整
```c
// 用於功耗最佳化的 CPU 頻率調整
typedef enum {
    CPU_FREQ_LOW = 0,    // 100 MHz
    CPU_FREQ_MEDIUM,     // 400 MHz
    CPU_FREQ_HIGH        // 800 MHz
} cpu_freq_t;

// 設定 CPU 頻率
int set_cpu_frequency(cpu_freq_t freq) {
    switch (freq) {
        case CPU_FREQ_LOW:
            // 設定鎖相迴路為低頻
            configure_pll(100000000);
            break;
        case CPU_FREQ_MEDIUM:
            configure_pll(400000000);
            break;
        case CPU_FREQ_HIGH:
            configure_pll(800000000);
            break;
        default:
            return -1;
    }
    
    // 更新系統時脈
    update_system_clock();
    return 0;
}

// 自適應頻率調整
void adaptive_frequency_scaling(void) {
    uint32_t cpu_usage = get_cpu_usage();
    
    if (cpu_usage < 25) {
        set_cpu_frequency(CPU_FREQ_LOW);
    } else if (cpu_usage < 75) {
        set_cpu_frequency(CPU_FREQ_MEDIUM);
    } else {
        set_cpu_frequency(CPU_FREQ_HIGH);
    }
}
```

#### 睡眠模式最佳化
```c
// 電源管理狀態
typedef enum {
    POWER_STATE_ACTIVE,
    POWER_STATE_IDLE,
    POWER_STATE_SLEEP,
    POWER_STATE_DEEP_SLEEP
} power_state_t;

// 進入睡眠模式
void enter_sleep_mode(power_state_t state) {
    switch (state) {
        case POWER_STATE_IDLE:
            // 停用 CPU 時脈，保持周邊裝置運作
            disable_cpu_clock();
            break;
            
        case POWER_STATE_SLEEP:
            // 停用大部分周邊裝置，保留 RAM
            disable_peripherals();
            enter_sleep_mode();
            break;
            
        case POWER_STATE_DEEP_SLEEP:
            // 停用所有裝置，僅保留喚醒來源
            disable_all_peripherals();
            save_context();
            enter_deep_sleep();
            break;
            
        default:
            break;
    }
}

// 喚醒處理函式
void wake_up_handler(void) {
    // 如需要則恢復上下文
    restore_context();
    
    // 重新啟用必要的周邊裝置
    enable_peripherals();
    
    // 恢復正常運作
    resume_operation();
}
```

### 周邊裝置電源管理

#### 周邊裝置時脈控制
```c
// 周邊裝置時脈管理
typedef struct {
    uint32_t peripheral_mask;
    uint32_t clock_enabled;
} peripheral_power_t;

// 啟用周邊裝置時脈
void enable_peripheral_clock(uint32_t peripheral) {
    // 設定時脈啟用位元
    PERIPHERAL_CLOCK_REG |= (1 << peripheral);
}

// 停用周邊裝置時脈
void disable_peripheral_clock(uint32_t peripheral) {
    // 清除時脈啟用位元
    PERIPHERAL_CLOCK_REG &= ~(1 << peripheral);
}

// 功耗感知的周邊裝置使用
void power_aware_peripheral_usage(void) {
    // 僅在需要時啟用
    enable_peripheral_clock(UART_CLOCK);
    uart_transmit(data, size);
    disable_peripheral_clock(UART_CLOCK);
    
    // 使用 DMA 進行高效資料傳輸
    enable_peripheral_clock(DMA_CLOCK);
    dma_transfer(source, destination, size);
    // DMA 完成後會自動停用
}
```

#### 中斷驅動的電源管理
```c
// 省電的中斷處理
typedef struct {
    uint32_t wake_up_sources;
    uint32_t sleep_duration;
} power_config_t;

// 配置電源管理
void configure_power_management(power_config_t *config) {
    // 啟用喚醒來源
    enable_wake_up_source(config->wake_up_sources);
    
    // 設定睡眠持續時間
    configure_sleep_timer(config->sleep_duration);
}

// 省電的主迴圈
void power_efficient_main_loop(void) {
    while (1) {
        // 處理待處理的任務
        process_pending_tasks();
        
        // 檢查是否可以進入睡眠
        if (can_enter_sleep()) {
            // 進入睡眠模式
            enter_sleep_mode(POWER_STATE_SLEEP);
            
            // 等待中斷
            __WFI();  // 等待中斷
            
            // 喚醒後恢復
            resume_after_wake_up();
        }
    }
}
```

---

## 即時效能分析

### 時序分析

#### 高解析度計時器
```c
// 用於效能測量的高解析度計時器
typedef struct {
    uint32_t start_time;
    uint32_t end_time;
    uint32_t duration;
} performance_timer_t;

// 開始計時
void timer_start(performance_timer_t *timer) {
    timer->start_time = get_high_resolution_time();
}

// 停止計時
void timer_stop(performance_timer_t *timer) {
    timer->end_time = get_high_resolution_time();
    timer->duration = timer->end_time - timer->start_time;
}

// 取得微秒為單位的時間
uint32_t timer_get_microseconds(performance_timer_t *timer) {
    return timer->duration / (get_cpu_frequency() / 1000000);
}

// 使用範例
void measure_performance(void) {
    performance_timer_t timer;
    
    timer_start(&timer);
    performance_critical_function();
    timer_stop(&timer);
    
    printf("函式執行耗時 %u 微秒\n", timer_get_microseconds(&timer));
}
```

#### 即時約束條件
```c
// 即時約束條件檢查
typedef struct {
    uint32_t deadline;
    uint32_t worst_case_time;
    uint32_t actual_time;
} real_time_constraint_t;

// 檢查即時約束條件
int check_real_time_constraint(real_time_constraint_t *constraint) {
    if (constraint->actual_time > constraint->deadline) {
        // 即時約束條件被違反
        return -1;
    }
    
    // 檢查是否有足夠的裕度
    uint32_t margin = constraint->deadline - constraint->actual_time;
    if (margin < constraint->worst_case_time * 0.1) {
        // 警告：裕度不足
        return 1;
    }
    
    return 0;  // 正常
}

// 即時任務執行
void real_time_task_execute(real_time_constraint_t *constraint) {
    performance_timer_t timer;
    
    timer_start(&timer);
    
    // 執行即時任務
    execute_real_time_task();
    
    timer_stop(&timer);
    constraint->actual_time = timer_get_microseconds(&timer);
    
    // 檢查約束條件
    int result = check_real_time_constraint(constraint);
    if (result < 0) {
        // 處理約束條件違反
        handle_constraint_violation();
    }
}
```

### 排程器分析

#### 任務排程分析
```c
// 任務排程資訊
typedef struct {
    uint32_t task_id;
    uint32_t priority;
    uint32_t execution_time;
    uint32_t period;
    uint32_t deadline;
    uint32_t missed_deadlines;
} task_info_t;

// 分析任務排程
void analyze_task_scheduling(task_info_t *tasks, int num_tasks) {
    uint32_t total_utilization = 0;
    
    for (int i = 0; i < num_tasks; i++) {
        // 計算使用率
        uint32_t utilization = (tasks[i].execution_time * 100) / tasks[i].period;
        total_utilization += utilization;
        
        // 檢查是否有錯過的截止期限
        if (tasks[i].missed_deadlines > 0) {
            printf("任務 %u 錯過了 %u 個截止期限\n", 
                   tasks[i].task_id, tasks[i].missed_deadlines);
        }
    }
    
    // 檢查總使用率
    if (total_utilization > 100) {
        printf("警告：總使用率超過 100%% (%u%%)\n", total_utilization);
    }
}
```

---

## 效能剖析與基準測試

### 效能剖析

#### 函式剖析
```c
// 函式剖析結構
typedef struct {
    char function_name[64];
    uint32_t call_count;
    uint32_t total_time;
    uint32_t min_time;
    uint32_t max_time;
    uint32_t average_time;
} function_profile_t;

// 剖析巨集
#define PROFILE_START(name) \
    performance_timer_t _timer_##name; \
    timer_start(&_timer_##name)

#define PROFILE_END(name) \
    timer_stop(&_timer_##name); \
    update_function_profile(#name, _timer_##name.duration)

// 更新函式剖析資料
void update_function_profile(const char *name, uint32_t duration) {
    function_profile_t *profile = find_or_create_profile(name);
    if (profile) {
        profile->call_count++;
        profile->total_time += duration;
        
        if (duration < profile->min_time || profile->min_time == 0) {
            profile->min_time = duration;
        }
        
        if (duration > profile->max_time) {
            profile->max_time = duration;
        }
        
        profile->average_time = profile->total_time / profile->call_count;
    }
}

// 使用範例
void profiled_function(void) {
    PROFILE_START(profiled_function);
    
    // 函式實作
    for (int i = 0; i < 1000; i++) {
        process_data(i);
    }
    
    PROFILE_END(profiled_function);
}
```

#### 記憶體剖析
```c
// 記憶體剖析
typedef struct {
    uint32_t total_allocations;
    uint32_t total_frees;
    uint32_t current_usage;
    uint32_t peak_usage;
    uint32_t allocation_count;
} memory_profile_t;

static memory_profile_t memory_profile = {0};

// 記憶體配置追蹤
void* tracked_malloc(size_t size) {
    void *ptr = malloc(size);
    if (ptr) {
        memory_profile.total_allocations += size;
        memory_profile.current_usage += size;
        memory_profile.allocation_count++;
        
        if (memory_profile.current_usage > memory_profile.peak_usage) {
            memory_profile.peak_usage = memory_profile.current_usage;
        }
    }
    return ptr;
}

// 記憶體釋放追蹤
void tracked_free(void *ptr) {
    if (ptr) {
        // 注意：這是簡化版——實務中需要追蹤配置大小
        memory_profile.total_frees++;
        memory_profile.allocation_count--;
    }
    free(ptr);
}

// 列印記憶體剖析資料
void print_memory_profile(void) {
    printf("記憶體剖析：\n");
    printf("  總配置量：%u 位元組\n", memory_profile.total_allocations);
    printf("  目前使用量：%u 位元組\n", memory_profile.current_usage);
    printf("  尖峰使用量：%u 位元組\n", memory_profile.peak_usage);
    printf("  配置次數：%u\n", memory_profile.allocation_count);
}
```

### 基準測試工具

#### 基準測試框架
```c
// 基準測試框架
typedef struct {
    char benchmark_name[64];
    uint32_t iterations;
    uint32_t total_time;
    uint32_t min_time;
    uint32_t max_time;
    uint32_t average_time;
} benchmark_t;

// 執行基準測試
void run_benchmark(const char *name, void (*function)(void), uint32_t iterations) {
    benchmark_t benchmark = {0};
    strncpy(benchmark.benchmark_name, name, sizeof(benchmark.benchmark_name) - 1);
    benchmark.iterations = iterations;
    
    performance_timer_t timer;
    
    for (uint32_t i = 0; i < iterations; i++) {
        timer_start(&timer);
        function();
        timer_stop(&timer);
        
        uint32_t duration = timer_get_microseconds(&timer);
        benchmark.total_time += duration;
        
        if (duration < benchmark.min_time || benchmark.min_time == 0) {
            benchmark.min_time = duration;
        }
        
        if (duration > benchmark.max_time) {
            benchmark.max_time = duration;
        }
    }
    
    benchmark.average_time = benchmark.total_time / iterations;
    
    // 列印結果
    printf("基準測試：%s\n", benchmark.benchmark_name);
    printf("  迭代次數：%u\n", benchmark.iterations);
    printf("  平均時間：%u 微秒\n", benchmark.average_time);
    printf("  最小時間：%u 微秒\n", benchmark.min_time);
    printf("  最大時間：%u 微秒\n", benchmark.max_time);
    printf("  總時間：%u 微秒\n", benchmark.total_time);
}
```

---

## 最佳化工具

### 靜態分析工具

#### 程式碼分析
```bash
# 使用 cppcheck 進行靜態分析
cppcheck --enable=all --xml --xml-version=2 . 2> static_analysis.xml

# 使用 clang-tidy 進行額外檢查
clang-tidy --checks=performance-* source_file.c

# 使用 gcc 警告
gcc -Wall -Wextra -Werror -O2 -o program program.c
```

#### 記憶體分析
```bash
# 使用 Valgrind 進行記憶體分析
valgrind --leak-check=full --show-leak-kinds=all ./program

# 使用 AddressSanitizer
gcc -fsanitize=address -g -o program program.c
```

### 動態分析工具

#### 效能監控
```bash
# 使用 perf 進行效能分析
perf record ./program
perf report

# 使用 gprof 進行函式剖析
gcc -pg -o program program.c
./program
gprof program gmon.out > profile.txt
```

#### 即時分析
```bash
# 使用 ftrace 進行核心追蹤
echo 1 > /sys/kernel/debug/tracing/tracing_on
./program
echo 0 > /sys/kernel/debug/tracing/tracing_on
cat /sys/kernel/debug/tracing/trace
```

---

## 最佳化最佳實務

### 一般準則
1. **先進行效能剖析** ——在最佳化之前先找出瓶頸
2. **衡量影響** ——驗證最佳化是否確實有改善效果
3. **考量權衡取捨** ——最佳化通常涉及妥協
4. **記錄變更** ——追蹤最佳化決策
5. **徹底測試** ——確保最佳化不會破壞功能

### 常見最佳化錯誤
1. **過早最佳化** ——在效能剖析之前就進行最佳化
2. **忽略記憶體使用量** ——只關注 CPU 效能
3. **不考慮功耗** ——忽略功耗影響
4. **過度最佳化** ——為了微小的效能提升而使程式碼變得難以閱讀
5. **平台特定的假設** ——假設最佳化在所有平台上都有效

### 最佳化檢查清單
- [ ] 對應用程式進行效能剖析以找出瓶頸
- [ ] 使用適當的編譯器最佳化
- [ ] 最佳化演算法與資料結構
- [ ] 減少記憶體配置與複製
- [ ] 使用高效的 I/O 操作
- [ ] 實施功耗感知的最佳化
- [ ] 測試效能改善
- [ ] 記錄最佳化決策

---

## 實作練習

### 實驗 1：迴圈展開
**目標**：學習迴圈展開如何透過減少迴圈額外開銷並啟用更好的編譯器最佳化來顯著提升效能。

**步驟**：
1. 對一個簡單的嵌入式應用程式（例如一個小迴圈）進行效能剖析以測量其效能。
2. 對效能最關鍵的迴圈套用迴圈展開。
3. 重新對應用程式進行效能剖析以觀察改善效果。

**預期結果**：效能關鍵迴圈的執行時間大幅減少。

### 實驗 2：資料結構最佳化
**目標**：了解資料結構如何影響效能和快取局部性。

**步驟**：
1. 對大量使用動態記憶體配置的程式進行效能剖析。
2. 實作自訂記憶體池並取代 malloc/free。
3. 重新對應用程式進行效能剖析以觀察改善效果。

**預期結果**：減少記憶體碎片化、改善快取效能，並可能降低功耗。

### 實驗 3：電源管理
**目標**：學習如何管理嵌入式系統的功耗。

**步驟**：
1. 對耗電量大的程式進行效能剖析。
2. 實作動態頻率調整和睡眠模式最佳化。
3. 重新對應用程式進行效能剖析以觀察改善效果。

**預期結果**：降低功耗，並可能延長電池壽命。

---

## 自我檢測

1. **什麼是效能最佳化？**
   - 效能最佳化是在有限資源下充分利用系統能力，同時維持可靠性並滿足即時約束條件。

2. **為什麼效能最佳化在嵌入式系統中很重要？**
   - 效能直接影響電池壽命、回應速度以及是否能滿足即時截止期限。效能不佳可能導致錯過截止期限、過度耗電，甚至系統故障。

3. **效能最佳化中有哪些關鍵的權衡取捨？**
   - 速度 vs. 記憶體使用量、功耗、程式碼可維護性、即時約束條件。

4. **如何在嵌入式系統中測量效能？**
   - 高解析度計時器、即時約束條件檢查、效能剖析、基準測試。

5. **什麼是迴圈展開？**
   - 迴圈展開是一種透過減少迭代次數並啟用更好的編譯器最佳化來降低迴圈額外開銷的技術。

---

## 延伸連結

1. **了解效能**
   - [Understanding Performance](https://www.embedded.com/understanding-performance/)
   - [Performance Metrics in Embedded Systems](https://www.embedded.com/performance-metrics-in-embedded-systems/)

2. **編譯器最佳化**
   - [GCC Optimization Options](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html)
   - [ARM Compiler Optimization](https://developer.arm.com/documentation/101754/0612/armclang-Reference/armclang-Command-line-Options)

3. **記憶體管理**
   - [Memory Management in Embedded Systems](https://www.embedded.com/memory-management-in-embedded-systems/)
   - [Custom Allocators](https://www.embedded.com/custom-allocators/)

4. **電源管理**
   - [Power Management in Embedded Systems](https://www.embedded.com/power-management-in-embedded-systems/)
   - [Dynamic Frequency Scaling](https://www.embedded.com/dynamic-frequency-scaling/)

5. **即時系統**
   - [Real-time Systems](https://www.embedded.com/real-time-systems/)
   - [Real-time Constraints](https://www.embedded.com/real-time-constraints/)

6. **效能剖析與基準測試**
   - [Profiling and Benchmarking](https://www.embedded.com/profiling-and-benchmarking/)
   - [Performance Profiling](https://www.embedded.com/performance-profiling/)

7. **最佳化工具**
   - [Static Analysis Tools](https://www.embedded.com/static-analysis-tools/)
   - [Dynamic Analysis Tools](https://www.embedded.com/dynamic-analysis-tools/)

8. **最佳實務**
   - [Optimization Best Practices](https://www.embedded.com/optimization-best-practices/)
   - [Common Mistakes](https://www.embedded.com/common-mistakes/)

---

## 資源

### 工具與軟體
- [GCC Optimization Options](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html)
- [ARM Compiler Optimization](https://developer.arm.com/documentation/101754/0612/armclang-Reference/armclang-Command-line-Options)
- [Valgrind](http://valgrind.org/) - 記憶體分析工具
- [perf](https://perf.wiki.kernel.org/) - Linux 效能分析工具

### 書籍與參考資料
- "Optimizing Software in C++" by Agner Fog
- "Computer Systems: A Programmer's Perspective" by Bryant and O'Hallaron
- "The Art of Computer Programming" by Donald Knuth

### 線上資源
- [Performance Calendar](http://calendar.perfplanet.com/)
- [Stack Overflow Performance Tag](https://stackoverflow.com/questions/tagged/performance)
- [ARM Developer Documentation](https://developer.arm.com/documentation)
