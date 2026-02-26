# 即時系統中的效能監控

> **在嵌入式即時系統中實作 CPU 使用率、記憶體使用量及時序分析效能監控系統的完整指南，包含 FreeRTOS 範例**

## 🎯 **概念 → 為何重要 → 最小範例 → 動手試試 → 重點整理**

### **概念**
效能監控就像汽車儀表板，能清楚顯示車速、油耗，以及引擎是否運行順暢。在嵌入式系統中，它是你觀察系統內部運作情形的窗口，幫助你在問題變得嚴重之前及早發現。

### **為何重要**
在即時系統中，效能問題可能意味著錯過截止時間、系統崩潰，甚至安全故障。若沒有監控，你等於在盲目飛行——無法得知系統是否高效運行還是即將失效。良好的監控能給你做出明智決策和及早發現問題所需的數據。

### **最小範例**
```c
// 簡單的效能監控掛鉤
void vApplicationIdleHook(void) {
    static uint32_t idle_count = 0;
    idle_count++;
    
    // 每 1000 個 tick 計算一次 CPU 使用率
    if (idle_count % 1000 == 0) {
        uint32_t total_ticks = xTaskGetTickCount();
        uint32_t cpu_util = 100 - ((idle_count * 100) / total_ticks);
        
        // 記錄或傳送 CPU 使用率數據
        log_performance_data("CPU_UTIL", cpu_util);
    }
}

// 記憶體監控掛鉤
void vApplicationMallocFailedHook(void) {
    // 記錄記憶體配置失敗
    log_performance_data("MEMORY_FAIL", 1);
    
    // 採取修正措施
    perform_memory_cleanup();
}
```

### **動手試試**
- **實驗**：在你的 FreeRTOS 系統中加入效能掛鉤並監控 CPU 使用率
- **挑戰**：實作一個追蹤配置模式的記憶體洩漏偵測器
- **除錯**：使用 GPIO 來測量任務執行時間並找出瓶頸

### **重點整理**
效能監控將被動式除錯轉化為主動式最佳化，為你提供建構可靠且高效嵌入式系統所需的數據。

---

## 📋 **目錄**
- [概述](#overview)
- [效能監控基礎](#performance-monitoring-fundamentals)
- [CPU 效能監控](#cpu-performance-monitoring)
- [記憶體效能監控](#memory-performance-monitoring)
- [時序效能監控](#timing-performance-monitoring)
- [效能分析工具](#performance-analysis-tools)
- [實作範例](#implementation-examples)
- [最佳實踐](#best-practices)
- [面試問題](#interview-questions)

---

## 🎯 **概述**

效能監控在即時系統中至關重要，可確保最佳運行、識別瓶頸並維持系統可靠性。有效的監控系統能即時掌握 CPU 使用率、記憶體消耗和時序效能，實現主動式最佳化和除錯。

### **核心概念**
- **效能監控** - 即時系統效能分析
- **CPU 使用率** - 處理器使用量與負載分析
- **記憶體監控** - RAM 使用量、碎片化及配置追蹤
- **時序分析** - 回應時間、截止時間及抖動測量
- **效能指標** - 可量化的效能指示器

---

## 📊 **效能監控基礎**

### **為何要監控效能？**

**1. 系統健康狀態：**
- 識別效能瓶頸
- 防止系統過載
- 維持即時性保證
- 最佳化資源使用

**2. 除錯支援：**
- 根本原因分析
- 效能退化偵測
- 資源競爭識別
- 時序違規分析

**3. 最佳化：**
- 建立效能基線
- 改善成效量測
- 資源配置最佳化
- 功耗分析

### **效能監控架構**

**核心元件：**
- **資料收集**：蒐集效能指標
- **資料儲存**：儲存歷史效能數據
- **分析引擎**：處理和分析指標
- **報告介面**：呈現效能資訊
- **警報系統**：通知效能問題

**監控流程：**
```
系統事件 → 資料收集 → 分析 → 儲存 → 報告 → 警報
```

### **效能指標分類**

**1. 資源指標：**
- CPU 使用率與負載
- 記憶體使用量與配置
- I/O 操作與頻寬
- 功耗

**2. 時序指標：**
- 任務執行時間
- 回應時間
- 截止時間遵守率
- 抖動與延遲

**3. 品質指標：**
- 錯誤率
- 系統穩定性
- 資源效率
- 使用者體驗

---

## 🖥️ **CPU 效能監控**

### **CPU 使用率測量**

**1. 閒置時間監控：**
- 追蹤系統閒置期間
- 計算 CPU 忙碌百分比
- 監控閒置任務執行
- 識別 CPU 瓶頸

**2. 任務層級監控：**
- 個別任務執行時間
- 任務切換頻率
- 優先權分佈分析
- 每任務 CPU 時間

**3. 負載平均計算：**
- 短期負載（1 分鐘）
- 中期負載（5 分鐘）
- 長期負載（15 分鐘）
- 負載趨勢分析

### **FreeRTOS CPU 監控實作**

**閒置時間追蹤：**
```c
typedef struct {
    uint32_t total_ticks;
    uint32_t idle_ticks;
    uint32_t busy_ticks;
    float cpu_utilization;
    uint32_t update_counter;
} cpu_monitor_t;

cpu_monitor_t g_cpu_monitor = {0};

void vUpdateCPUMonitoring(void) {
    static uint32_t last_idle_time = 0;
    uint32_t current_idle_time = xTaskGetIdleRunTimeCounter();
    
    // 計算閒置時間差值
    uint32_t idle_delta = current_idle_time - last_idle_time;
    uint32_t total_delta = pdMS_TO_TICKS(1000); // 1 秒窗口
    
    g_cpu_monitor.total_ticks += total_delta;
    g_cpu_monitor.idle_ticks += idle_delta;
    g_cpu_monitor.busy_ticks += (total_delta - idle_delta);
    g_cpu_monitor.update_counter++;
    
    // 每秒更新 CPU 使用率
    if (g_cpu_monitor.update_counter >= 1000) {
        g_cpu_monitor.cpu_utilization = (float)g_cpu_monitor.busy_ticks / 
                                       (float)g_cpu_monitor.total_ticks * 100.0f;
        
        // 重置計數器
        g_cpu_monitor.total_ticks = 0;
        g_cpu_monitor.idle_ticks = 0;
        g_cpu_monitor.busy_ticks = 0;
        g_cpu_monitor.update_counter = 0;
        
        printf("CPU 使用率: %.2f%%\n", g_cpu_monitor.cpu_utilization);
    }
    
    last_idle_time = current_idle_time;
}
```

**任務層級 CPU 監控：**
```c
typedef struct {
    TaskHandle_t task_handle;
    char task_name[16];
    uint32_t total_execution_time;
    uint32_t execution_count;
    uint32_t last_start_time;
    uint32_t max_execution_time;
    uint32_t min_execution_time;
    float average_execution_time;
} task_performance_t;

task_performance_t task_performance[10];
uint8_t task_count = 0;

void vStartTaskTiming(TaskHandle_t task_handle) {
    // 尋找或建立任務效能項目
    int task_index = vFindTaskIndex(task_handle);
    if (task_index >= 0) {
        task_performance[task_index].last_start_time = xTaskGetTickCount();
    }
}

void vEndTaskTiming(TaskHandle_t task_handle) {
    int task_index = vFindTaskIndex(task_handle);
    if (task_index >= 0) {
        uint32_t end_time = xTaskGetTickCount();
        uint32_t execution_time = end_time - task_performance[task_index].last_start_time;
        
        // 更新統計數據
        task_performance[task_index].total_execution_time += execution_time;
        task_performance[task_index].execution_count++;
        
        if (execution_time > task_performance[task_index].max_execution_time) {
            task_performance[task_index].max_execution_time = execution_time;
        }
        
        if (execution_time < task_performance[task_index].min_execution_time || 
            task_performance[task_index].min_execution_time == 0) {
            task_performance[task_index].min_execution_time = execution_time;
        }
        
        // 計算平均值
        task_performance[task_index].average_execution_time = 
            (float)task_performance[task_index].total_execution_time / 
            (float)task_performance[task_index].execution_count;
    }
}
```

### **負載平均計算**

**指數移動平均：**
```c
typedef struct {
    float load_1min;
    float load_5min;
    float load_15min;
    uint32_t update_counter;
} load_average_t;

load_average_t g_load_average = {0};

void vUpdateLoadAverage(float current_load) {
    // 指數移動平均計算
    float alpha_1min = 1.0f - exp(-1.0f / 60.0f);  // 1 分鐘
    float alpha_5min = 1.0f - exp(-1.0f / 300.0f); // 5 分鐘
    float alpha_15min = 1.0f - exp(-1.0f / 900.0f); // 15 分鐘
    
    g_load_average.load_1min = alpha_1min * current_load + (1.0f - alpha_1min) * g_load_average.load_1min;
    g_load_average.load_5min = alpha_5min * current_load + (1.0f - alpha_5min) * g_load_average.load_5min;
    g_load_average.load_15min = alpha_15min * current_load + (1.0f - alpha_15min) * g_load_average.load_15min;
    
    printf("負載平均: 1分鐘=%.2f, 5分鐘=%.2f, 15分鐘=%.2f\n", 
           g_load_average.load_1min, g_load_average.load_5min, g_load_average.load_15min);
}
```

---

## 💾 **記憶體效能監控**

### **記憶體使用追蹤**

**1. 堆疊使用監控：**
- 追蹤任務堆疊高水位標記
- 監控堆疊溢位條件
- 分析堆疊使用模式
- 最佳化堆疊配置

**2. 堆積使用監控：**
- 監控動態記憶體配置
- 追蹤記憶體碎片化
- 識別記憶體洩漏
- 最佳化記憶體池

**3. 靜態記憶體分析：**
- 全域變數使用
- 靜態配置追蹤
- 記憶體佈局最佳化
- ROM/RAM 使用平衡

### **FreeRTOS 記憶體監控**

**堆疊使用監控：**
```c
typedef struct {
    TaskHandle_t task_handle;
    char task_name[16];
    uint32_t stack_size;
    uint32_t stack_high_water_mark;
    uint32_t stack_usage_percentage;
    uint32_t min_stack_high_water;
    uint32_t max_stack_high_water;
} stack_monitor_t;

stack_monitor_t stack_monitors[10];
uint8_t stack_monitor_count = 0;

void vUpdateStackMonitoring(void) {
    for (int i = 0; i < stack_monitor_count; i++) {
        if (stack_monitors[i].task_handle != NULL) {
            // 取得目前堆疊高水位標記
            uint32_t current_high_water = uxTaskGetStackHighWaterMark(stack_monitors[i].task_handle);
            
            // 更新統計數據
            if (current_high_water < stack_monitors[i].min_stack_high_water || 
                stack_monitors[i].min_stack_high_water == 0) {
                stack_monitors[i].min_stack_high_water = current_high_water;
            }
            
            if (current_high_water > stack_monitors[i].max_stack_high_water) {
                stack_monitors[i].max_stack_high_water = current_high_water;
            }
            
            stack_monitors[i].stack_high_water_mark = current_high_water;
            stack_monitors[i].stack_usage_percentage = 
                ((stack_monitors[i].stack_size - current_high_water) * 100) / stack_monitors[i].stack_size;
            
            // 堆疊使用過高時發出警告
            if (stack_monitors[i].stack_usage_percentage > 80) {
                printf("警告: 任務 %s 堆疊使用過高: %.1f%%\n", 
                       stack_monitors[i].task_name, 
                       (float)stack_monitors[i].stack_usage_percentage);
            }
        }
    }
}
```

**堆積使用監控：**
```c
typedef struct {
    size_t total_heap_size;
    size_t free_heap_size;
    size_t minimum_free_heap;
    size_t largest_free_block;
    uint32_t allocation_count;
    uint32_t free_count;
    float fragmentation_percentage;
} heap_monitor_t;

heap_monitor_t g_heap_monitor = {0};

void vUpdateHeapMonitoring(void) {
    // 取得目前堆積統計數據
    g_heap_monitor.total_heap_size = configTOTAL_HEAP_SIZE;
    g_heap_monitor.free_heap_size = xPortGetFreeHeapSize();
    g_heap_monitor.minimum_free_heap = xPortGetMinimumEverFreeHeapSize();
    g_heap_monitor.largest_free_block = xPortGetLargestFreeBlock();
    
    // 計算碎片化程度
    if (g_heap_monitor.free_heap_size > 0) {
        g_heap_monitor.fragmentation_percentage = 
            (float)(g_heap_monitor.free_heap_size - g_heap_monitor.largest_free_block) / 
            (float)g_heap_monitor.free_heap_size * 100.0f;
    }
    
    // 堆積使用量達到危急時發出警報
    if (g_heap_monitor.free_heap_size < (g_heap_monitor.total_heap_size / 10)) {
        printf("危急: 堆積記憶體不足！可用: %zu, 總量: %zu\n", 
               g_heap_monitor.free_heap_size, g_heap_monitor.total_heap_size);
    }
    
    if (g_heap_monitor.fragmentation_percentage > 50) {
        printf("警告: 堆積碎片化程度過高: %.1f%%\n", 
               g_heap_monitor.fragmentation_percentage);
    }
}
```

**記憶體洩漏偵測：**
```c
typedef struct {
    void *address;
    size_t size;
    uint32_t allocation_time;
    char allocation_file[32];
    uint32_t allocation_line;
    bool is_freed;
} memory_allocation_t;

memory_allocation_t memory_allocations[100];
uint32_t allocation_count = 0;

void* vTrackedMalloc(size_t size, const char *file, uint32_t line) {
    void *ptr = pvPortMalloc(size);
    
    if (ptr != NULL && allocation_count < 100) {
        memory_allocations[allocation_count].address = ptr;
        memory_allocations[allocation_count].size = size;
        memory_allocations[allocation_count].allocation_time = xTaskGetTickCount();
        strncpy(memory_allocations[allocation_count].allocation_file, file, 31);
        memory_allocations[allocation_count].allocation_line = line;
        memory_allocations[allocation_count].is_freed = false;
        allocation_count++;
    }
    
    return ptr;
}

void vTrackedFree(void *ptr) {
    // 將配置標記為已釋放
    for (int i = 0; i < allocation_count; i++) {
        if (memory_allocations[i].address == ptr && !memory_allocations[i].is_freed) {
            memory_allocations[i].is_freed = true;
            break;
        }
    }
    
    vPortFree(ptr);
}

void vCheckMemoryLeaks(void) {
    uint32_t leak_count = 0;
    size_t total_leaked_size = 0;
    
    for (int i = 0; i < allocation_count; i++) {
        if (!memory_allocations[i].is_freed) {
            leak_count++;
            total_leaked_size += memory_allocations[i].size;
            printf("偵測到記憶體洩漏: %p, 大小: %zu, 檔案: %s, 行號: %lu\n",
                   memory_allocations[i].address,
                   memory_allocations[i].size,
                   memory_allocations[i].allocation_file,
                   memory_allocations[i].allocation_line);
        }
    }
    
    if (leak_count > 0) {
        printf("記憶體洩漏總數: %lu, 洩漏總大小: %zu 位元組\n", 
               leak_count, total_leaked_size);
    } else {
        printf("未偵測到記憶體洩漏\n");
    }
}
```

---

## ⏱️ **時序效能監控**

### **回應時間測量**

**1. 任務回應時間：**
- 測量從任務啟動到完成的時間
- 追蹤截止時間遵守情況
- 監控回應時間變化
- 識別時序瓶頸

**2. 中斷回應時間：**
- 測量中斷延遲
- 追蹤 ISR 執行時間
- 監控中斷頻率
- 分析中斷模式

**3. 系統回應時間：**
- 端到端回應時間
- 系統事件時序
- 通訊時序
- 整體系統效能

### **時序監控實作**

**任務回應時間監控：**
```c
typedef struct {
    TaskHandle_t task_handle;
    char task_name[16];
    uint32_t activation_count;
    uint32_t deadline_misses;
    uint32_t total_response_time;
    uint32_t min_response_time;
    uint32_t max_response_time;
    float average_response_time;
    uint32_t deadline_ticks;
} task_timing_t;

task_timing_t task_timing[10];
uint8_t timing_task_count = 0;

void vStartTaskTiming(TaskHandle_t task_handle, uint32_t deadline_ticks) {
    int task_index = vFindTimingTaskIndex(task_handle);
    if (task_index >= 0) {
        task_timing[task_index].last_activation_time = xTaskGetTickCount();
        task_timing[task_index].deadline_ticks = deadline_ticks;
    }
}

void vEndTaskTiming(TaskHandle_t task_handle) {
    int task_index = vFindTimingTaskIndex(task_handle);
    if (task_index >= 0) {
        uint32_t end_time = xTaskGetTickCount();
        uint32_t response_time = end_time - task_timing[task_index].last_activation_time;
        
        // 更新統計數據
        task_timing[task_index].activation_count++;
        task_timing[task_index].total_response_time += response_time;
        
        if (response_time < task_timing[task_index].min_response_time || 
            task_timing[task_index].min_response_time == 0) {
            task_timing[task_index].min_response_time = response_time;
        }
        
        if (response_time > task_timing[task_index].max_response_time) {
            task_timing[task_index].max_response_time = response_time;
        }
        
        // 檢查截止時間遵守情況
        if (response_time > task_timing[task_index].deadline_ticks) {
            task_timing[task_index].deadline_misses++;
            printf("截止時間未達成: 任務 %s, 回應時間: %lu, 截止時間: %lu\n",
                   task_timing[task_index].task_name,
                   response_time,
                   task_timing[task_index].deadline_ticks);
        }
        
        // 計算平均值
        task_timing[task_index].average_response_time = 
            (float)task_timing[task_index].total_response_time / 
            (float)task_timing[task_index].activation_count;
    }
}
```

**中斷時序監控：**
```c
typedef struct {
    uint32_t interrupt_number;
    uint32_t interrupt_count;
    uint32_t total_execution_time;
    uint32_t min_execution_time;
    uint32_t max_execution_time;
    float average_execution_time;
    uint32_t last_interrupt_time;
    uint32_t min_interrupt_interval;
    uint32_t max_interrupt_interval;
} interrupt_timing_t;

interrupt_timing_t interrupt_timing[32];
uint8_t interrupt_count = 0;

void vStartInterruptTiming(uint32_t interrupt_num) {
    int int_index = vFindInterruptIndex(interrupt_num);
    if (int_index >= 0) {
        uint32_t current_time = DWT->CYCCNT;
        interrupt_timing[int_index].last_interrupt_time = current_time;
    }
}

void vEndInterruptTiming(uint32_t interrupt_num) {
    int int_index = vFindInterruptIndex(interrupt_num);
    if (int_index >= 0) {
        uint32_t current_time = DWT->CYCCNT;
        uint32_t execution_time = current_time - interrupt_timing[int_index].last_interrupt_time;
        
        // 更新執行時間統計數據
        interrupt_timing[int_index].interrupt_count++;
        interrupt_timing[int_index].total_execution_time += execution_time;
        
        if (execution_time < interrupt_timing[int_index].min_execution_time || 
            interrupt_timing[int_index].min_execution_time == 0) {
            interrupt_timing[int_index].min_execution_time = execution_time;
        }
        
        if (execution_time > interrupt_timing[int_index].max_execution_time) {
            interrupt_timing[int_index].max_execution_time = execution_time;
        }
        
        // 計算平均值
        interrupt_timing[int_index].average_execution_time = 
            (float)interrupt_timing[int_index].total_execution_time / 
            (float)interrupt_timing[int_index].interrupt_count;
    }
}
```

### **抖動分析**

**抖動計算：**
```c
typedef struct {
    uint32_t sample_count;
    uint32_t total_jitter;
    uint32_t max_jitter;
    uint32_t min_jitter;
    float average_jitter;
    uint32_t jitter_threshold;
    uint32_t jitter_violations;
} jitter_analysis_t;

jitter_analysis_t g_jitter_analysis = {0};

void vUpdateJitterAnalysis(uint32_t expected_interval, uint32_t actual_interval) {
    int32_t jitter = (int32_t)actual_interval - (int32_t)expected_interval;
    uint32_t absolute_jitter = abs(jitter);
    
    g_jitter_analysis.sample_count++;
    g_jitter_analysis.total_jitter += absolute_jitter;
    
    if (absolute_jitter > g_jitter_analysis.max_jitter) {
        g_jitter_analysis.max_jitter = absolute_jitter;
    }
    
    if (absolute_jitter < g_jitter_analysis.min_jitter || 
        g_jitter_analysis.min_jitter == 0) {
        g_jitter_analysis.min_jitter = absolute_jitter;
    }
    
    // 檢查抖動閾值違規
    if (absolute_jitter > g_jitter_analysis.jitter_threshold) {
        g_jitter_analysis.jitter_violations++;
        printf("抖動違規: 預期: %lu, 實際: %lu, 抖動: %ld\n",
               expected_interval, actual_interval, jitter);
    }
    
    // 計算平均值
    g_jitter_analysis.average_jitter = 
        (float)g_jitter_analysis.total_jitter / (float)g_jitter_analysis.sample_count;
}
```

---

## 🛠️ **效能分析工具**

### **即時效能儀表板**

**效能資料結構：**
```c
typedef struct {
    cpu_monitor_t cpu;
    heap_monitor_t heap;
    load_average_t load;
    jitter_analysis_t jitter;
    uint32_t system_uptime;
    uint32_t last_update_time;
    bool monitoring_enabled;
} performance_dashboard_t;

performance_dashboard_t g_performance_dashboard = {0};

void vUpdatePerformanceDashboard(void) {
    if (!g_performance_dashboard.monitoring_enabled) {
        return;
    }
    
    // 更新所有監控元件
    vUpdateCPUMonitoring();
    vUpdateHeapMonitoring();
    vUpdateStackMonitoring();
    vUpdateLoadAverage(g_cpu_monitor.cpu_utilization / 100.0f);
    
    // 更新系統運行時間
    g_performance_dashboard.system_uptime = xTaskGetTickCount();
    g_performance_dashboard.last_update_time = xTaskGetTickCount();
    
    // 列印效能摘要
    vPrintPerformanceSummary();
}
```

**效能摘要顯示：**
```c
void vPrintPerformanceSummary(void) {
    printf("\n=== 效能儀表板 ===\n");
    printf("系統運行時間: %lu ticks\n", g_performance_dashboard.system_uptime);
    printf("CPU 使用率: %.2f%%\n", g_cpu_monitor.cpu_utilization);
    printf("負載平均: 1分鐘=%.2f, 5分鐘=%.2f, 15分鐘=%.2f\n",
           g_load_average.load_1min, g_load_average.load_5min, g_load_average.load_15min);
    printf("堆積使用量: %zu/%zu 位元組 (%.1f%%)\n",
           g_heap_monitor.total_heap_size - g_heap_monitor.free_heap_size,
           g_heap_monitor.total_heap_size,
           ((float)(g_heap_monitor.total_heap_size - g_heap_monitor.free_heap_size) / 
            (float)g_heap_monitor.total_heap_size) * 100.0f);
    printf("堆積碎片化: %.1f%%\n", g_heap_monitor.fragmentation_percentage);
    printf("平均抖動: %.2f ticks\n", g_jitter_analysis.average_jitter);
    printf("抖動違規次數: %lu\n", g_jitter_analysis.jitter_violations);
    printf("============================\n\n");
}
```

### **效能警報系統**

**警報設定：**
```c
typedef enum {
    ALERT_LEVEL_INFO,
    ALERT_LEVEL_WARNING,
    ALERT_LEVEL_CRITICAL
} alert_level_t;

typedef struct {
    alert_level_t level;
    char message[128];
    uint32_t timestamp;
    bool acknowledged;
} performance_alert_t;

performance_alert_t performance_alerts[50];
uint8_t alert_count = 0;

void vAddPerformanceAlert(alert_level_t level, const char *message) {
    if (alert_count < 50) {
        performance_alerts[alert_count].level = level;
        strncpy(performance_alerts[alert_count].message, message, 127);
        performance_alerts[alert_count].timestamp = xTaskGetTickCount();
        performance_alerts[alert_count].acknowledged = false;
        alert_count++;
        
        // 根據等級列印警報
        switch (level) {
            case ALERT_LEVEL_INFO:
                printf("[資訊] %s\n", message);
                break;
            case ALERT_LEVEL_WARNING:
                printf("[警告] %s\n", message);
                break;
            case ALERT_LEVEL_CRITICAL:
                printf("[危急] %s\n", message);
                break;
        }
    }
}

void vCheckPerformanceAlerts(void) {
    // 檢查 CPU 使用率
    if (g_cpu_monitor.cpu_utilization > 90) {
        vAddPerformanceAlert(ALERT_LEVEL_CRITICAL, "CPU 使用率達到危急水準");
    } else if (g_cpu_monitor.cpu_utilization > 80) {
        vAddPerformanceAlert(ALERT_LEVEL_WARNING, "CPU 使用率偏高");
    }
    
    // 檢查堆積使用量
    if (g_heap_monitor.free_heap_size < (g_heap_monitor.total_heap_size / 20)) {
        vAddPerformanceAlert(ALERT_LEVEL_CRITICAL, "堆積記憶體達到危急低水準");
    } else if (g_heap_monitor.free_heap_size < (g_heap_monitor.total_heap_size / 10)) {
        vAddPerformanceAlert(ALERT_LEVEL_WARNING, "堆積記憶體偏低");
    }
    
    // 檢查抖動違規
    if (g_jitter_analysis.jitter_violations > 10) {
        vAddPerformanceAlert(ALERT_LEVEL_WARNING, "抖動違規次數過多");
    }
}
```

---

## 💻 **實作範例**

### **完整效能監控系統**

**系統初始化：**
```c
void vInitializePerformanceMonitoring(void) {
    // 初始化所有監控元件
    memset(&g_cpu_monitor, 0, sizeof(cpu_monitor_t));
    memset(&g_heap_monitor, 0, sizeof(heap_monitor_t));
    memset(&g_load_average, 0, sizeof(load_average_t));
    memset(&g_jitter_analysis, 0, sizeof(jitter_analysis_t));
    memset(&g_performance_dashboard, 0, sizeof(performance_dashboard_t));
    
    // 設定初始值
    g_heap_monitor.minimum_free_heap = SIZE_MAX;
    g_jitter_analysis.min_jitter = UINT32_MAX;
    g_performance_dashboard.monitoring_enabled = true;
    g_jitter_analysis.jitter_threshold = 10; // 10 ticks 閾值
    
    // 啟動監控任務
    xTaskCreate(vPerformanceMonitoringTask, "PerfMon", 512, NULL, 1, NULL);
    
    printf("效能監控系統已初始化\n");
}
```

**效能監控任務：**
```c
void vPerformanceMonitoringTask(void *pvParameters) {
    TickType_t last_wake_time = xTaskGetTickCount();
    
    while (1) {
        // 更新所有監控元件
        vUpdatePerformanceDashboard();
        
        // 檢查效能警報
        vCheckPerformanceAlerts();
        
        // 定期檢查記憶體洩漏
        static uint32_t leak_check_counter = 0;
        leak_check_counter++;
        if (leak_check_counter >= 60) { // 每 60 秒
            vCheckMemoryLeaks();
            leak_check_counter = 0;
        }
        
        // 等待下一個監控週期
        vTaskDelayUntil(&last_wake_time, pdMS_TO_TICKS(1000));
    }
}
```

---

## ✅ **最佳實踐**

### **設計原則**

1. **最小額外負擔**
   - 保持監控輕量化
   - 使用高效資料結構
   - 最小化中斷影響
   - 最佳化更新頻率

2. **全面覆蓋**
   - 監控所有關鍵資源
   - 追蹤關鍵效能指標
   - 實作警報系統
   - 提供歷史數據

3. **即時回應性**
   - 確保監控不影響時序
   - 使用適當的任務優先權
   - 實作非阻塞操作
   - 優雅地處理監控失效

### **實作指南**

1. **資料收集**
   - 使用高效取樣方法
   - 實作資料過濾
   - 處理資料溢位
   - 確保資料準確性

2. **儲存與分析**
   - 實作環形緩衝區
   - 使用高效演算法
   - 提供資料匯出功能
   - 支援遠端監控

3. **警報與報告**
   - 定義明確的閾值
   - 實作升級機制
   - 提供可行動的警報
   - 支援多種通知方式

---

## 🔬 **引導式實驗**

### **實驗 1：CPU 使用率監控**
**目標**：在 FreeRTOS 中實作基本的 CPU 使用率監控
**步驟**：
1. 啟用 FreeRTOS 掛鉤（閒置掛鉤、tick 掛鉤）
2. 實作 CPU 使用率計算
3. 記錄或顯示使用率數據
4. 在不同負載條件下測試

**預期結果**：以最小額外負擔取得即時 CPU 使用率數據

### **實驗 2：記憶體使用追蹤**
**目標**：監控記憶體配置和使用模式
**步驟**：
1. 實作記憶體配置追蹤
2. 使用高水位標記監控堆疊使用
3. 追蹤堆積碎片化
4. 偵測記憶體洩漏

**預期結果**：完整掌握記憶體使用模式

### **實驗 3：效能數據視覺化**
**目標**：建立簡單的效能儀表板
**步驟**：
1. 即時收集效能指標
2. 實作資料儲存（環形緩衝區）
3. 建立簡單視覺化介面（UART 輸出、LCD）
4. 加入閾值違規警報

**預期結果**：具備警報功能的即時效能儀表板

---

## ✅ **自我檢核**

### **理解檢核**
- [ ] 你能解釋為何效能監控在即時系統中至關重要嗎？
- [ ] 你理解 CPU 使用率和記憶體使用量之間的差異嗎？
- [ ] 你能識別哪些效能指標對你的系統最重要嗎？
- [ ] 你知道如何實作基本的效能掛鉤嗎？

### **實務技能檢核**
- [ ] 你能在 FreeRTOS 中設定效能監控嗎？
- [ ] 你知道如何準確測量 CPU 使用率嗎？
- [ ] 你能實作記憶體洩漏偵測嗎？
- [ ] 你理解如何最小化監控額外負擔嗎？

### **進階概念檢核**
- [ ] 你能解釋效能監控設計中的取捨嗎？
- [ ] 你理解如何關聯不同的效能指標嗎？
- [ ] 你能實作基於系統負載的自適應監控嗎？
- [ ] 你知道如何優雅地處理監控失效嗎？

---

## 🔗 **相關連結**

### **相關主題**
- **[FreeRTOS 基礎](./FreeRTOS_Basics.md)** - 理解 RTOS 上下文
- **[任務建立與管理](./Task_Creation_Management.md)** - 監控任務效能
- **[排程演算法](./Scheduling_Algorithms.md)** - 理解排程效能
- **[效能剖析](../Debugging/Performance_Profiling.md)** - 詳細效能分析

### **先修知識**
- **[C 語言基礎](../Embedded_C/C_Language_Fundamentals.md)** - 基本程式設計概念
- **[記憶體管理](../Embedded_C/Memory_Management.md)** - 理解記憶體概念
- **[GPIO 設定](../Hardware_Fundamentals/GPIO_Configuration.md)** - 基本 I/O 設定

### **後續步驟**
- **[即時除錯](./Real_Time_Debugging.md)** - 使用效能數據進行除錯
- **[電源管理](./Power_Management.md)** - 監控功耗
- **[回應時間分析](./Response_Time_Analysis.md)** - 分析時序效能

---

## 📋 **快速參考：關鍵事實**

### **效能監控基礎**
- **目的**：即時掌握系統效能與資源使用狀況
- **類型**：CPU 使用率、記憶體使用量、時序分析、功耗
- **特性**：即時、非侵入式、全面、可行動
- **優點**：早期問題偵測、最佳化指引、可靠性保證

### **CPU 效能監控**
- **閒置時間監控**：追蹤系統閒置期間並計算 CPU 忙碌百分比
- **任務層級監控**：個別任務執行時間與 CPU 使用量
- **上下文切換監控**：追蹤任務切換頻率與額外負擔
- **效能計數器**：使用硬體計數器進行精確時序量測

### **記憶體效能監控**
- **堆疊使用**：監控堆疊高水位標記與溢位偵測
- **堆積使用**：追蹤動態記憶體配置與釋放模式
- **碎片化**：監控記憶體碎片化與配置效率
- **記憶體洩漏**：偵測和追蹤已配置但從未釋放的記憶體

### **時序效能監控**
- **回應時間**：測量從事件到回應完成的時間
- **抖動分析**：追蹤時序變化與截止時間遵守情況
- **截止時間監控**：確保任務滿足時序需求
- **延遲測量**：追蹤端到端系統回應時間

---

## ❓ **面試問題**

### **基本概念**

1. **什麼是效能監控，為什麼它很重要？**
   - 即時系統效能分析
   - 識別瓶頸和問題
   - 實現最佳化和除錯
   - 確保系統可靠性

2. **如何在 FreeRTOS 中測量 CPU 使用率？**
   - 監控閒置任務執行時間
   - 計算忙碌與閒置百分比
   - 追蹤任務執行模式
   - 使用效能計數器

3. **關鍵的記憶體監控指標有哪些？**
   - 堆疊使用與高水位標記
   - 堆積使用與碎片化
   - 記憶體配置模式
   - 記憶體洩漏偵測

### **進階主題**

1. **如何實作抖動分析？**
   - 測量時序變化
   - 計算統計量
   - 設定適當閾值
   - 追蹤違規模式

2. **說明效能監控中的取捨。**
   - 監控額外負擔 vs 洞察力
   - 資料準確性 vs 儲存空間
   - 即時分析 vs 歷史分析
   - 本地監控 vs 遠端監控

3. **如何在安全關鍵系統中處理效能監控？**
   - 確保監控可靠性
   - 實作故障安全機制
   - 驗證監控數據
   - 處理監控失效

### **實務場景**

1. **為即時控制應用設計效能監控系統。**
   - 識別關鍵指標
   - 實作監控元件
   - 設計警報系統
   - 針對即時運作進行最佳化

2. **如何使用監控數據除錯效能問題？**
   - 分析效能趨勢
   - 關聯不同指標
   - 識別根本原因
   - 實作解決方案

3. **說明電池供電設備的效能監控。**
   - 監控功耗
   - 追蹤效能與功耗的關係
   - 實作自適應監控
   - 針對電池壽命進行最佳化

本篇完整的效能監控文件為嵌入式工程師提供了在即時環境中實作有效效能監控系統所需的理論基礎、實務實作範例和最佳實踐。
