# RTOS 中的排程演算法

> **理解即時排程演算法、基於優先權的排程，以及嵌入式系統中的時序分析，重點介紹 FreeRTOS 實作與即時排程原理**

## 🎯 **概念 → 為什麼重要 → 最小範例 → 動手試試 → 重點整理**

### **概念**
排程演算法就像 CPU 的交通管制員。它不是讓任務爭搶誰可以執行，而是由排程器做出智慧決策，決定哪個任務何時應該執行，確保每個任務都能輪到，且關鍵任務不會被卡住。

### **為什麼重要**
在即時系統中，錯過截止期限可能意味著安全著陸與墜機之間的差距。良好的排程確保關鍵任務（如讀取感測器或控制致動器）在需要時總能獲得 CPU 時間，而較不重要的任務（如狀態更新）則等待輪到它們。

### **最小範例**
```c
// 任務優先權決定執行順序
void highPriorityTask(void *pvParameters) {
    while (1) {
        readCriticalSensor();  // 必須每 10ms 執行一次
        vTaskDelay(pdMS_TO_TICKS(10));
    }
}

void mediumPriorityTask(void *pvParameters) {
    while (1) {
        processData();         // 可以稍微等待
        vTaskDelay(pdMS_TO_TICKS(50));
    }
}

void lowPriorityTask(void *pvParameters) {
    while (1) {
        updateStatusLED();     // 非時間關鍵
        vTaskDelay(pdMS_TO_TICKS(100));
    }
}

// 以不同優先權建立任務
xTaskCreate(highPriorityTask, "High", 128, NULL, 3, NULL);
xTaskCreate(mediumPriorityTask, "Medium", 128, NULL, 2, NULL);
xTaskCreate(lowPriorityTask, "Low", 128, NULL, 1, NULL);
```

### **動手試試**
- **實驗**：建立不同優先權的任務並觀察執行順序
- **挑戰**：設計一個系統，讓三個任務必須滿足不同的截止期限
- **除錯**：使用 FreeRTOS 鉤子函式監控任務切換與時序

### **重點整理**
良好的排程在於對緊急性、重要性與資源效率之間做出智慧的取捨，確保您的系統滿足所有時序要求。

---

## 📋 **目錄**
- [概述](#概述)
- [什麼是排程演算法？](#什麼是排程演算法)
- [為什麼排程很重要？](#為什麼排程很重要)
- [排程概念](#排程概念)
- [基於優先權的排程](#基於優先權的排程)
- [速率單調排程](#速率單調排程)
- [最早截止期限優先](#最早截止期限優先)
- [循環排程](#循環排程)
- [排程分析](#排程分析)
- [FreeRTOS 排程器](#freertos-排程器)
- [實作](#實作)
- [常見陷阱](#常見陷阱)
- [最佳實踐](#最佳實踐)
- [面試問題](#面試問題)

---

## 🎯 **概述**

排程演算法是即時作業系統的核心，決定哪些任務何時執行以及執行多長時間。理解排程演算法對於設計能滿足即時需求、處理多個並行操作，並在各種條件下提供可預測效能的嵌入式系統至關重要。

### **關鍵概念**
- **排程演算法** - 決定任務執行順序的方法
- **優先權管理** - 指派與管理任務優先權
- **時序分析** - 分析系統時序與可排程性
- **即時約束** - 滿足截止期限與回應時間要求
- **資源利用** - 高效使用系統資源

---

## 🤔 **什麼是排程演算法？**

排程演算法是決定即時系統中任務執行順序與時序的數學方法。它們確保系統資源被高效使用，同時滿足截止期限和回應時間等即時約束。

### **核心概念**

**排程目的：**
- **資源分配**：決定哪些任務何時獲得 CPU 時間
- **時序保證**：確保任務滿足其時序要求
- **系統效率**：最佳化資源利用與效能
- **可預測性**：提供可預測的系統行為

**排程特性：**
- **搶占式 vs 非搶占式**：較高優先權的任務是否可以中斷較低優先權的任務
- **靜態 vs 動態**：優先權是固定的還是可以改變的
- **最佳 vs 啟發式**：演算法是否提供最佳解
- **複雜度**：排程演算法的計算複雜度

**即時需求：**
- **硬即時**：錯過截止期限會導致系統失效
- **軟即時**：錯過截止期限會降低效能
- **固即時**：錯過截止期限會導致資料遺失
- **混合即時**：不同即時需求的組合

### **排程系統架構**

**基本排程系統：**
```
┌─────────────────────────────────────────────────────────────┐
│                    任務佇列                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │   任務 1    │  │   任務 2    │  │   任務 3    │        │
│  │ (優先權 3)  │  │ (優先權 2)  │  │ (優先權 1)  │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    排程器                                    │
│  ┌─────────────────────────────────────────────────────┐   │
│  │           排程演算法                                 │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    CPU                                     │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              目前正在執行的任務                       │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

**排程決策流程：**
```
┌─────────────────────────────────────────────────────────────┐
│                    排程週期                                  │
├─────────────────────────────────────────────────────────────┤
│  1. 檢查是否有新任務或優先權變更                             │
│  2. 評估排程演算法準則                                       │
│  3. 選擇下一個要執行的任務                                   │
│  4. 如有需要則執行上下文切換                                  │
│  5. 執行所選任務                                             │
│  6. 監控任務執行與時序                                       │
└─────────────────────────────────────────────────────────────┘
```

---

## 🎯 **為什麼排程很重要？**

有效的排程對即時系統至關重要，因為它直接影響系統效能、可靠性以及滿足時序要求的能力。不良的排程可能導致錯過截止期限、系統失效和不可預測的行為。

### **即時系統需求**

**時序約束：**
- **截止期限遵守**：任務必須在指定的時間限制內完成
- **回應時間**：系統必須在要求的時間範圍內回應事件
- **抖動控制**：最小化任務執行時序的變異
- **可預測性**：系統行為在所有條件下都必須是可預測的

**資源管理：**
- **CPU 利用率**：高效使用可用的處理資源
- **記憶體管理**：在多個任務之間最佳化記憶體使用
- **功耗效率**：在任務執行期間管理功耗
- **資源共享**：協調對共享資源的存取

**系統可靠性：**
- **容錯能力**：儘管元件故障仍能繼續運作
- **錯誤復原**：為排程失效實作復原機制
- **系統穩定性**：在不同負載下維持穩定性
- **效能保證**：提供有保證的效能水準

### **排程對系統效能的影響**

**效能指標：**
- **吞吐量**：每單位時間完成的任務數量
- **延遲**：從任務就緒到任務完成的時間
- **抖動**：任務執行時序的變異
- **效率**：資源利用率與額外開銷

**服務品質：**
- **即時保證**：滿足時序要求
- **可預測性**：在不同條件下一致的效能
- **回應能力**：對外部事件的快速回應
- **穩定性**：長期維持效能

---

## 🔧 **排程概念**

### **任務特性**

**任務參數：**
- **週期**：連續任務啟動之間的時間
- **截止期限**：任務完成的最大允許時間
- **執行時間**：完成任務執行所需的時間
- **優先權**：任務的相對重要性
- **資源需求**：任務執行所需的資源

**任務分類：**
- **週期性任務**：按固定間隔執行的任務
- **非週期性任務**：回應事件而執行的任務
- **偶發性任務**：具有最小到達間隔時間的任務
- **關鍵任務**：必須滿足嚴格時序要求的任務

### **排程指標**

**時序指標：**
- **回應時間**：從任務到達到完成的時間
- **最差情況回應時間**：最大可能的回應時間
- **平均回應時間**：多次執行的平均回應時間
- **抖動**：回應時間的變異

**利用率指標：**
- **CPU 利用率**：任務使用的 CPU 時間百分比
- **可排程性**：所有任務是否都能滿足其截止期限
- **額外開銷**：花費在排程決策和上下文切換上的時間
- **效率**：有效工作與總時間的比率

### **排程約束**

**系統約束：**
- **資源限制**：有限的 CPU、記憶體和 I/O 資源
- **時序要求**：嚴格的截止期限和回應時間要求
- **先後約束**：任務之間的相依性
- **資源衝突**：共享資源存取需求

**演算法約束：**
- **計算複雜度**：排程決策所需的時間
- **記憶體需求**：排程資料結構所需的記憶體
- **實作複雜度**：實作演算法的難度
- **維護需求**：持續的維護和調整需求

---

## 🚀 **基於優先權的排程**

### **優先權排程基礎**

**基本原則：**
- **優先權指派**：每個任務都有一個數值優先權值
- **搶占式執行**：較高優先權的任務可以中斷較低優先權的任務
- **優先權反轉**：低優先權的任務可能阻塞高優先權的任務
- **優先權繼承**：任務繼承其存取資源的優先權

**優先權指派策略：**
- **速率單調**：較高頻率的任務獲得較高優先權
- **截止期限單調**：較短截止期限的任務獲得較高優先權
- **基於價值**：較高價值的任務獲得較高優先權
- **應用特定**：基於需求的自訂優先權指派

### **FreeRTOS 優先權實作**

**優先權設定：**
```c
// 優先權設定
#define configMAX_PRIORITIES 32
#define configUSE_PREEMPTION 1
#define configUSE_TIME_SLICING 1
#define configUSE_TICKLESS_IDLE 0

// 優先權等級
#define PRIORITY_CRITICAL    5    // 系統關鍵任務
#define PRIORITY_HIGH        4    // 高優先權使用者任務
#define PRIORITY_NORMAL      3    // 一般操作任務
#define PRIORITY_LOW         2    // 背景任務
#define PRIORITY_IDLE        1    // 閒置任務

// 以優先權建立任務
void vCreateTasks(void) {
    TaskHandle_t xTaskHandle;
    
    // 以最高優先權建立關鍵任務
    xTaskCreate(
        vCriticalTask,           // 任務函式
        "Critical",              // 任務名稱
        256,                     // 堆疊大小
        NULL,                    // 參數
        PRIORITY_CRITICAL,       // 優先權
        &xTaskHandle             // 任務控制代碼
    );
    
    // 建立高優先權任務
    xTaskCreate(
        vHighPriorityTask,       // 任務函式
        "High",                  // 任務名稱
        256,                     // 堆疊大小
        NULL,                    // 參數
        PRIORITY_HIGH,           // 優先權
        &xTaskHandle             // 任務控制代碼
    );
    
    // 建立一般優先權任務
    xTaskCreate(
        vNormalTask,             // 任務函式
        "Normal",                // 任務名稱
        256,                     // 堆疊大小
        NULL,                    // 參數
        PRIORITY_NORMAL,         // 優先權
        &xTaskHandle             // 任務控制代碼
    );
}
```

**優先權管理：**
```c
// 動態優先權變更
void vPriorityManager(void *pvParameters) {
    TaskHandle_t xManagedTask = (TaskHandle_t)pvParameters;
    UBaseType_t uxCurrentPriority;
    
    while (1) {
        // 取得目前優先權
        uxCurrentPriority = uxTaskPriorityGet(xManagedTask);
        
        // 根據系統條件調整優先權
        if (system_under_load()) {
            // 負載下提高優先權
            vTaskPrioritySet(xManagedTask, uxCurrentPriority + 1);
        } else {
            // 恢復一般優先權
            vTaskPrioritySet(xManagedTask, PRIORITY_NORMAL);
        }
        
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}

// 優先權繼承範例
void vResourceTask(void *pvParameters) {
    SemaphoreHandle_t xMutex = (SemaphoreHandle_t)pvParameters;
    
    while (1) {
        // 取得互斥鎖（將發生優先權繼承）
        if (xSemaphoreTake(xMutex, portMAX_DELAY) == pdTRUE) {
            // 使用共享資源
            vTaskDelay(pdMS_TO_TICKS(100));
            
            // 釋放互斥鎖
            xSemaphoreGive(xMutex);
        }
        
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}
```

### **優先權反轉預防**

**優先權繼承：**
```c
// 優先權繼承互斥鎖
SemaphoreHandle_t xPriorityInheritanceMutex;

void vHighPriorityTask(void *pvParameters) {
    while (1) {
        // 等待資源
        if (xSemaphoreTake(xPriorityInheritanceMutex, portMAX_DELAY) == pdTRUE) {
            // 臨界區段
            vTaskDelay(pdMS_TO_TICKS(50));
            
            // 釋放資源
            xSemaphoreGive(xPriorityInheritanceMutex);
        }
        
        vTaskDelay(pdMS_TO_TICKS(200));
    }
}

void vLowPriorityTask(void *pvParameters) {
    while (1) {
        // 取得資源
        if (xSemaphoreTake(xPriorityInheritanceMutex, portMAX_DELAY) == pdTRUE) {
            // 長臨界區段
            vTaskDelay(pdMS_TO_TICKS(1000));
            
            // 釋放資源
            xSemaphoreGive(xPriorityInheritanceMutex);
        }
        
        vTaskDelay(pdMS_TO_TICKS(5000));
    }
}

// 初始化優先權繼承互斥鎖
xPriorityInheritanceMutex = xSemaphoreCreateMutex();
```

---

## ⏰ **速率單調排程**

### **速率單調原理**

**基本概念：**
- **優先權指派**：較高頻率的任務獲得較高優先權
- **最佳性**：對於截止期限等於週期的週期性任務是最佳的
- **可排程性**：Liu-Layland 界限用於可排程性分析
- **實作**：基於任務頻率的簡單優先權指派

**速率單調分析：**
```c
// 速率單調可排程性測試
typedef struct {
    uint32_t period;        // 任務週期（以 tick 為單位）
    uint32_t execution;     // 最差情況執行時間
    uint8_t priority;       // 指派的優先權
} rms_task_t;

bool rms_schedulability_test(rms_task_t tasks[], uint8_t task_count) {
    double utilization = 0.0;
    
    // 計算總利用率
    for (uint8_t i = 0; i < task_count; i++) {
        utilization += (double)tasks[i].execution / tasks[i].period;
    }
    
    // 速率單調的 Liu-Layland 界限
    double bound = task_count * (pow(2.0, 1.0/task_count) - 1.0);
    
    return utilization <= bound;
}

// 範例：三個週期性任務
rms_task_t rms_tasks[] = {
    {100, 20, 3},   // 任務 1：100ms 週期，20ms 執行，優先權 3
    {200, 40, 2},   // 任務 2：200ms 週期，40ms 執行，優先權 2
    {400, 60, 1}    // 任務 3：400ms 週期，60ms 執行，優先權 1
};

void vRateMonotonicExample(void) {
    uint8_t task_count = sizeof(rms_tasks) / sizeof(rms_tasks[0]);
    
    if (rms_schedulability_test(rms_tasks, task_count)) {
        printf("系統在速率單調排程下是可排程的\n");
    } else {
        printf("系統在速率單調排程下是不可排程的\n");
    }
}
```

### **速率單調實作**

**使用 RMS 建立任務：**
```c
// 速率單調任務建立
void vCreateRMSTasks(void) {
    // 按週期排序任務（最高頻率 = 最高優先權）
    qsort(rms_tasks, sizeof(rms_tasks)/sizeof(rms_tasks[0]), 
          sizeof(rms_tasks[0]), compare_period);
    
    // 以 RMS 優先權建立任務
    for (uint8_t i = 0; i < sizeof(rms_tasks)/sizeof(rms_tasks[0]); i++) {
        xTaskCreate(
            vPeriodicTask,                    // 任務函式
            "RMS_Task",                       // 任務名稱
            256,                              // 堆疊大小
            &rms_tasks[i],                    // 參數
            rms_tasks[i].priority,            // RMS 優先權
            NULL                              // 任務控制代碼
        );
    }
}

// 週期性任務實作
void vPeriodicTask(void *pvParameters) {
    rms_task_t *task = (rms_task_t*)pvParameters;
    TickType_t xLastWakeTime;
    
    // 以目前時間初始化 xLastWakeTime 變數
    xLastWakeTime = xTaskGetTickCount();
    
    while (1) {
        // 執行任務工作
        vTaskWork(task);
        
        // 等待下一個週期
        vTaskDelayUntil(&xLastWakeTime, pdMS_TO_TICKS(task->period));
    }
}

// 任務工作函式
void vTaskWork(rms_task_t *task) {
    printf("週期 %lu 的任務執行 %lu ms\n", 
           task->period, task->execution);
    
    // 模擬工作
    vTaskDelay(pdMS_TO_TICKS(task->execution));
}
```

---

## ⏱️ **最早截止期限優先**

### **EDF 原理**

**基本概念：**
- **動態優先權**：任務優先權根據目前截止期限而改變
- **最佳性**：對於獨立任務的搶占式排程是最佳的
- **可排程性**：可達到 100% CPU 利用率
- **複雜度**：比固定優先權排程更複雜

**EDF 可排程性測試：**
```c
// EDF 可排程性測試
bool edf_schedulability_test(rms_task_t tasks[], uint8_t task_count) {
    double utilization = 0.0;
    
    // 計算總利用率
    for (uint8_t i = 0; i < task_count; i++) {
        utilization += (double)tasks[i].execution / tasks[i].period;
    }
    
    // 對於獨立任務，EDF 界限為 100%
    return utilization <= 1.0;
}

// 含截止期限的 EDF 任務結構
typedef struct {
    uint32_t period;        // 任務週期
    uint32_t execution;     // 最差情況執行時間
    uint32_t deadline;      // 任務截止期限
    uint32_t next_deadline; // 下一個截止期限時間
} edf_task_t;

// EDF 優先權計算
uint32_t edf_calculate_priority(edf_task_t *task) {
    // 較早的截止期限 = 較高的優先權
    return task->next_deadline;
}
```

### **EDF 實作**

**EDF 排程器：**
```c
// EDF 排程器實作
void vEDFScheduler(void *pvParameters) {
    edf_task_t *tasks = (edf_task_t*)pvParameters;
    uint8_t task_count = sizeof(tasks) / sizeof(tasks[0]);
    uint8_t highest_priority_task = 0;
    
    while (1) {
        // 找到截止期限最早的任務
        uint32_t earliest_deadline = UINT32_MAX;
        
        for (uint8_t i = 0; i < task_count; i++) {
            if (tasks[i].next_deadline < earliest_deadline) {
                earliest_deadline = tasks[i].next_deadline;
                highest_priority_task = i;
            }
        }
        
        // 執行最高優先權的任務
        vExecuteTask(&tasks[highest_priority_task]);
        
        // 更新已完成任務的截止期限
        tasks[highest_priority_task].next_deadline += tasks[highest_priority_task].period;
        
        vTaskDelay(pdMS_TO_TICKS(1));
    }
}

// 任務執行函式
void vExecuteTask(edf_task_t *task) {
    printf("正在執行截止期限為 %lu 的 EDF 任務\n", task->next_deadline);
    
    // 模擬任務執行
    vTaskDelay(pdMS_TO_TICKS(task->execution));
}
```

---

## 🔄 **循環排程**

### **循環排程原理**

**基本概念：**
- **時間片段**：每個任務獲得固定的時間配額
- **公平性**：在相同優先權的任務之間平均分配 CPU 時間
- **搶占**：當時間配額到期時任務被搶占
- **額外開銷**：上下文切換的額外開銷影響效能

**時間配額選擇：**
```c
// 時間配額設定
#define TIME_QUANTUM_MS 10    // 10ms 時間配額
#define TASK_SLICE_TICKS pdMS_TO_TICKS(TIME_QUANTUM_MS)

// 循環排程任務結構
typedef struct {
    uint8_t priority;         // 任務優先權
    uint32_t time_remaining;  // 目前配額中剩餘的時間
    bool is_running;          // 任務是否正在執行
} rr_task_t;

// 循環排程器
void vRoundRobinScheduler(void *pvParameters) {
    rr_task_t *tasks = (rr_task_t*)pvParameters;
    uint8_t task_count = sizeof(tasks) / sizeof(tasks[0]);
    uint8_t current_task = 0;
    
    while (1) {
        // 找到具有相同優先權的下一個就緒任務
        uint8_t next_task = (current_task + 1) % task_count;
        
        // 檢查下一個任務是否具有相同優先權且已就緒
        if (tasks[next_task].priority == tasks[current_task].priority &&
            tasks[next_task].is_running) {
            current_task = next_task;
        }
        
        // 在時間配額內執行目前任務
        vExecuteTaskRR(&tasks[current_task], TASK_SLICE_TICKS);
        
        vTaskDelay(pdMS_TO_TICKS(1));
    }
}
```

### **FreeRTOS 循環排程**

**時間片段設定：**
```c
// FreeRTOS 時間片段設定
#define configUSE_TIME_SLICING 1
#define configIDLE_SHOULD_YIELD 1

// 循環排程任務建立
void vCreateRoundRobinTasks(void) {
    // 建立具有相同優先權的任務以進行循環排程
    for (uint8_t i = 0; i < 3; i++) {
        xTaskCreate(
            vRoundRobinTask,              // 任務函式
            "RR_Task",                    // 任務名稱
            256,                          // 堆疊大小
            (void*)i,                     // 任務編號
            2,                           // 所有任務相同優先權
            NULL                          // 任務控制代碼
        );
    }
}

// 循環排程任務實作
void vRoundRobinTask(void *pvParameters) {
    uint8_t task_number = (uint8_t)pvParameters;
    
    while (1) {
        printf("循環排程任務 %d 正在執行\n", task_number);
        
        // 模擬工作
        vTaskDelay(pdMS_TO_TICKS(100));
        
        // 讓出 CPU 給其他任務（啟用時間片段時為選用）
        taskYIELD();
    }
}
```

---

## 📊 **排程分析**

### **回應時間分析**

**基本 RTA：**
```c
// 固定優先權排程的回應時間分析
typedef struct {
    uint32_t period;        // 任務週期
    uint32_t execution;     // 最差情況執行時間
    uint8_t priority;       // 任務優先權
    uint32_t response_time; // 計算出的回應時間
} rta_task_t;

uint32_t calculate_response_time(rta_task_t *task, rta_task_t tasks[], uint8_t task_count) {
    uint32_t response_time = task->execution;
    uint32_t interference = 0;
    bool converged = false;
    uint32_t iterations = 0;
    
    while (!converged && iterations < 100) {
        interference = 0;
        
        // 計算來自較高優先權任務的干擾
        for (uint8_t i = 0; i < task_count; i++) {
            if (tasks[i].priority > task->priority) {
                interference += ceil((double)response_time / tasks[i].period) * tasks[i].execution;
            }
        }
        
        uint32_t new_response_time = task->execution + interference;
        
        if (new_response_time == response_time) {
            converged = true;
        } else {
            response_time = new_response_time;
        }
        
        iterations++;
    }
    
    return response_time;
}

// RTA 範例
void vResponseTimeAnalysis(void) {
    rta_task_t tasks[] = {
        {100, 20, 3, 0},   // 高優先權
        {200, 40, 2, 0},   // 中優先權
        {400, 60, 1, 0}    // 低優先權
    };
    
    uint8_t task_count = sizeof(tasks) / sizeof(tasks[0]);
    
    // 計算回應時間
    for (uint8_t i = 0; i < task_count; i++) {
        tasks[i].response_time = calculate_response_time(&tasks[i], tasks, task_count);
        printf("任務 %d：回應時間 = %lu ms\n", i, tasks[i].response_time);
    }
}
```

### **可排程性測試**

**利用率界限測試：**
```c
// 利用率界限測試
bool test_utilization_bound(rta_task_t tasks[], uint8_t task_count) {
    double total_utilization = 0.0;
    
    // 計算總利用率
    for (uint8_t i = 0; i < task_count; i++) {
        total_utilization += (double)tasks[i].execution / tasks[i].period;
    }
    
    // 速率單調界限
    double rms_bound = task_count * (pow(2.0, 1.0/task_count) - 1.0);
    
    // EDF 界限
    double edf_bound = 1.0;
    
    printf("總利用率：%.3f\n", total_utilization);
    printf("RMS 界限：%.3f\n", rms_bound);
    printf("EDF 界限：%.3f\n", edf_bound);
    
    return total_utilization <= rms_bound;
}
```

---

## ⚙️ **FreeRTOS 排程器**

### **排程器設定**

**基本設定：**
```c
// FreeRTOS 排程器設定
#define configUSE_PREEMPTION           1
#define configUSE_TIME_SLICING         1
#define configUSE_TICKLESS_IDLE        0
#define configUSE_IDLE_HOOK            0
#define configUSE_TICK_HOOK            0
#define configCPU_CLOCK_HZ             16000000
#define configTICK_RATE_HZ             1000
#define configMAX_PRIORITIES           32
#define configMINIMAL_STACK_SIZE       128
#define configMAX_TASK_NAME_LEN        16
#define configUSE_16_BIT_TICKS         0
#define configIDLE_SHOULD_YIELD        1
#define configUSE_MUTEXES              1
#define configUSE_RECURSIVE_MUTEXES    0
#define configUSE_COUNTING_SEMAPHORES  1
#define configUSE_ALTERNATIVE_API      0
#define configCHECK_FOR_STACK_OVERFLOW 2
#define configUSE_MALLOC_FAILED_HOOK   1
#define configUSE_APPLICATION_TASK_TAG 0
#define configUSE_QUEUE_SETS           1
#define configUSE_TASK_NOTIFICATIONS   1
#define configSUPPORT_STATIC_ALLOCATION 1
#define configSUPPORT_DYNAMIC_ALLOCATION 1
```

**排程器鉤子函式：**
```c
// 排程器鉤子函式
void vApplicationIdleHook(void) {
    // 當閒置任務執行時呼叫
    // 可用於電源管理
    __WFI();  // 等待中斷
}

void vApplicationTickHook(void) {
    // 每個 tick 呼叫一次
    // 可用於週期性操作
    static uint32_t tick_count = 0;
    tick_count++;
    
    if (tick_count % 1000 == 0) {
        // 每 1000 個 tick
        printf("系統已運行 %lu 秒\n", tick_count / 1000);
    }
}

void vApplicationMallocFailedHook(void) {
    // 當 malloc 失敗時呼叫
    printf("記憶體配置失敗！\n");
    
    // 處理記憶體配置失敗
    // 可以重新啟動系統或釋放記憶體
}

void vApplicationStackOverflowHook(TaskHandle_t xTask, char *pcTaskName) {
    // 當偵測到堆疊溢位時呼叫
    printf("任務堆疊溢位：%s\n", pcTaskName);
    
    // 處理堆疊溢位
    // 可以重新啟動系統或任務
}
```

### **排程器控制**

**排程器控制函式：**
```c
// 排程器控制
void vSchedulerControl(void *pvParameters) {
    while (1) {
        // 暫停排程器
        vTaskSuspendAll();
        
        // 執行關鍵操作
        vCriticalOperation();
        
        // 恢復排程器
        xTaskResumeAll();
        
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}

// 關鍵操作
void vCriticalOperation(void) {
    // 不能被中斷的操作
    printf("正在執行關鍵操作...\n");
    
    // 模擬關鍵工作
    for (volatile uint32_t i = 0; i < 1000000; i++) {
        // 關鍵工作
    }
    
    printf("關鍵操作完成\n");
}
```

---

## 🚀 **實作**

### **完整排程系統**

**系統初始化：**
```c
// 含排程的系統初始化
void vSystemInit(void) {
    // 以不同優先權建立系統任務
    xTaskCreate(vSystemMonitorTask, "SysMon", 256, NULL, 5, NULL);
    xTaskCreate(vCommunicationTask, "Comm", 512, NULL, 4, NULL);
    xTaskCreate(vDataProcessingTask, "DataProc", 1024, NULL, 3, NULL);
    xTaskCreate(vBackgroundTask, "Background", 128, NULL, 2, NULL);
    xTaskCreate(vIdleTask, "Idle", 64, NULL, 1, NULL);
    
    // 啟動排程器
    vTaskStartScheduler();
}

// 主函式
int main(void) {
    // 硬體初始化
    SystemInit();
    HAL_Init();
    
    // 初始化周邊裝置
    MX_GPIO_Init();
    MX_USART1_UART_Init();
    
    // 初始化系統
    vSystemInit();
    
    // 不應該執行到這裡
    while (1) {
        // 錯誤處理
    }
}
```

---

## ⚠️ **常見陷阱**

### **優先權反轉**

**常見情境：**
- **資源競爭**：低優先權任務持有高優先權任務所需的資源
- **巢狀鎖定**：以錯誤順序取得多個互斥鎖
- **過長的臨界區段**：長時間停用中斷

**解決方案：**
- **優先權繼承**：在持有資源時自動提高任務優先權
- **優先權天花板**：為資源指定優先權上限
- **資源排序**：始終以一致的順序取得資源
- **逾時處理**：使用逾時防止無限期阻塞

### **排程額外開銷**

**額外開銷來源：**
- **上下文切換**：儲存和還原任務上下文的時間
- **排程決策**：做出排程決策的時間
- **中斷處理**：處理排程相關中斷的時間
- **記憶體管理**：記憶體配置和釋放的時間

**最佳化策略：**
- **最小化上下文切換**：減少不必要的任務切換
- **最佳化關鍵路徑**：將最佳化重點放在時間關鍵的區段
- **使用硬體功能**：在可用時利用硬體加速
- **剖析與量測**：使用剖析工具識別瓶頸

### **記憶體碎片化**

**碎片化原因：**
- **可變配置大小**：不同大小的記憶體區塊
- **頻繁配置/釋放**：記憶體流動
- **無記憶體壓縮**：碎片化的記憶體未被回收

**緩解措施：**
- **記憶體池**：使用固定大小的記憶體池
- **靜態配置**：在可能時預先配置記憶體
- **記憶體整理**：定期壓縮記憶體
- **垃圾回收**：自動記憶體管理

---

## ✅ **最佳實踐**

### **排程設計原則**

**優先權指派：**
- **明確的優先權層級**：建立明確的優先權等級
- **一致的指派**：使用一致的優先權指派策略
- **文件記錄**：記錄優先權指派的理由
- **檢視與更新**：定期檢視和更新優先權

**任務設計：**
- **單一職責**：每個任務應有一個主要功能
- **明確的介面**：定義良好的輸入/輸出介面
- **最小相依性**：減少任務之間的耦合
- **錯誤處理**：任務內的健全錯誤處理

### **效能最佳化**

**排程效率：**
- **最小化額外開銷**：減少排程決策的額外開銷
- **最佳化上下文切換**：最小化上下文切換時間
- **使用適當的演算法**：根據需求選擇演算法
- **監控效能**：持續監控排程效能

**資源管理：**
- **高效配置**：最小化資源配置的額外開銷
- **資源共享**：使用適當的同步機制
- **清理**：任務終止時適當清理資源
- **監控**：監控資源使用和可用性

---

## 🔬 **引導式實驗**

### **實驗 1：基於優先權的排程**
**目標**：理解任務優先權如何影響執行順序
**步驟**：
1. 建立三個不同優先權（1、2、3）的任務
2. 每個任務切換不同的 GPIO 腳位
3. 使用示波器觀察執行模式
4. 改變優先權並觀察差異

**預期結果**：較高優先權的任務獲得更多 CPU 時間並更頻繁地執行

### **實驗 2：速率單調排程**
**目標**：實作並觀察 RMS 行為
**步驟**：
1. 建立不同週期的任務（10ms、20ms、50ms）
2. 根據頻率指派優先權（較高頻率 = 較高優先權）
3. 監控任務執行與時序
4. 驗證所有截止期限都被滿足

**預期結果**：在適當的優先權指派下，所有任務都能滿足其截止期限

### **實驗 3：排程效能量測**
**目標**：量測排程額外開銷與效能
**步驟**：
1. 使用 GPIO 量測上下文切換時間
2. 在不同負載下監控 CPU 利用率
3. 量測最差情況回應時間
4. 剖析排程演算法效能

**預期結果**：理解排程額外開銷與最佳化機會

---

## ✅ **自我檢查**

### **理解檢查**
- [ ] 你能解釋為什麼搶占式排程對即時系統更好嗎？
- [ ] 你理解 RMS 和 EDF 排程之間的差異嗎？
- [ ] 你能識別何時發生優先權反轉嗎？
- [ ] 你知道如何判斷一個系統是否可排程嗎？

### **實務技能檢查**
- [ ] 你能在 FreeRTOS 中設定不同優先權的任務嗎？
- [ ] 你知道如何除錯排程問題嗎？
- [ ] 你能實作適當的優先權管理嗎？
- [ ] 你理解如何量測排程效能嗎？

### **進階概念檢查**
- [ ] 你能解釋回應時間分析嗎？
- [ ] 你理解如何最佳化排程演算法嗎？
- [ ] 你能實作自訂排程策略嗎？
- [ ] 你知道如何處理排程中的資源競爭嗎？

---

## 🔗 **交叉連結**

### **相關主題**
- **[FreeRTOS 基礎](./FreeRTOS_Basics.md)** - 理解 RTOS 上下文
- **[任務建立與管理](./Task_Creation_Management.md)** - 任務如何被建立與管理
- **[核心服務](./Kernel_Services.md)** - 支援排程的服務
- **[效能監控](./Performance_Monitoring.md)** - 量測排程效能

### **先備知識**
- **[C 語言基礎](../Embedded_C/C_Language_Fundamentals.md)** - 基本程式設計概念
- **[任務建立與管理](./Task_Creation_Management.md)** - 理解任務
- **[GPIO 設定](../Hardware_Fundamentals/GPIO_Configuration.md)** - 基本 I/O 設定

### **下一步**
- **[中斷處理](./Interrupt_Handling.md)** - 中斷如何影響排程
- **[即時除錯](./Real_Time_Debugging.md)** - 除錯排程問題
- **[回應時間分析](./Response_Time_Analysis.md)** - 分析任務時序

---

## 📋 **快速參考：關鍵事實**

### **排程基礎**
- **目的**：決定哪個任務何時執行以及執行多長時間
- **類型**：搶占式、非搶占式、靜態、動態
- **特性**：基於優先權、截止期限感知、資源高效
- **優點**：可預測的時序、高效的資源使用、即時保證

### **基於優先權的排程**
- **高優先權**：必須滿足嚴格截止期限的關鍵任務
- **中優先權**：正常的系統操作和資料處理
- **低優先權**：背景任務和狀態更新
- **優先權指派**：基於關鍵性、頻率和截止期限要求

### **常見排程演算法**
- **速率單調 (RMS)**：基於任務頻率的固定優先權
- **最早截止期限優先 (EDF)**：基於目前截止期限的動態優先權
- **循環排程**：相同優先權的任務平均分享 CPU 時間
- **優先權搶占**：較高優先權的任務可以中斷較低優先權的任務

### **排程分析**
- **利用率界限**：可排程性的最大 CPU 利用率
- **回應時間分析**：計算最差情況回應時間
- **截止期限錯失**：當任務無法滿足其時序要求時
- **可排程性測試**：判斷系統是否能滿足所有截止期限

---

## ❓ **面試問題**

### **基本概念**

1. **搶占式和非搶占式排程有什麼區別？**
   - 搶占式：較高優先權的任務可以中斷較低優先權的任務
   - 非搶占式：任務執行直到完成或主動讓出
   - 搶占式提供更好的回應能力但額外開銷更大

2. **如何判斷一個系統是否可排程？**
   - 使用利用率界限測試（RMS、EDF）
   - 執行回應時間分析
   - 考慮系統約束和需求
   - 以最差情況場景進行測試

3. **什麼是優先權反轉，如何預防？**
   - 低優先權任務阻塞高優先權任務
   - 使用優先權繼承或優先權天花板
   - 以一致的順序取得資源
   - 使用逾時機制

### **進階主題**

1. **解釋速率單調和 EDF 排程之間的差異。**
   - RMS：基於任務頻率的固定優先權
   - EDF：基於目前截止期限的動態優先權
   - RMS：較簡單但不是最佳的
   - EDF：較複雜但是最佳的

2. **如何分析一個任務的最差情況回應時間？**
   - 使用回應時間分析（RTA）
   - 計算來自較高優先權任務的干擾
   - 考慮來自共享資源的阻塞
   - 迭代直到收斂

3. **排程最佳化使用哪些策略？**
   - 最小化上下文切換額外開銷
   - 最佳化關鍵執行路徑
   - 使用適當的記憶體管理
   - 利用硬體功能

### **實務場景**

1. **為即時控制應用設計排程系統。**
   - 定義任務優先權和時序要求
   - 選擇適當的排程演算法
   - 實作優先權管理
   - 處理資源共享和同步

2. **如何在 RTOS 中除錯排程問題？**
   - 使用排程鉤子函式和監控
   - 分析任務狀態和優先權
   - 檢查優先權反轉
   - 監控系統效能

3. **解釋如何實作自訂排程演算法。**
   - 定義排程準則和策略
   - 實作優先權計算
   - 處理任務選擇和執行
   - 與 RTOS 框架整合

本增強版排程演算法文件現在提供了概念說明、實務見解和技術實作細節的全面平衡，嵌入式工程師可以用它來理解和實作穩健的 RTOS 排程系統。


