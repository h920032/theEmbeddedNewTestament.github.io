# RTOS 中的核心服務

> **了解即時作業系統中的核心服務、系統呼叫和核心 RTOS 功能，重點介紹 FreeRTOS 實作與即時核心原理**

## 🎯 **概念 → 重要性 → 最小範例 → 動手試試 → 重點摘要**

### **概念**
核心服務就像一座管理良好的圖書館，你不需要自己管理每個細節，只需「借閱」你需要的東西。RTOS 核心扮演著一位值得信賴的圖書館員角色，他確切知道每樣東西的位置，並能快速且可靠地為你取得。

### **重要性**
在嵌入式系統中，你不能為每個基本操作重新造輪子。核心服務為常見問題提供經過驗證的解決方案，例如記憶體管理、任務協調和計時。這讓你能專注於應用程式邏輯，而非底層系統細節。

### **最小範例**
```c
// 使用核心服務進行任務協調
SemaphoreHandle_t dataReady = xSemaphoreCreateBinary();
QueueHandle_t dataQueue = xQueueCreate(10, sizeof(sensor_data_t));

// 任務 1：生產者
void producerTask(void *pvParameters) {
    sensor_data_t data;
    while (1) {
        data = readSensor();
        xQueueSend(dataQueue, &data, portMAX_DELAY);
        xSemaphoreGive(dataReady);  // 通知消費者
        vTaskDelay(pdMS_TO_TICKS(100));
    }
}

// 任務 2：消費者
void consumerTask(void *pvParameters) {
    sensor_data_t data;
    while (1) {
        if (xSemaphoreTake(dataReady, pdMS_TO_TICKS(1000))) {
            if (xQueueReceive(dataQueue, &data, 0) == pdTRUE) {
                processData(data);
            }
        }
    }
}
```

### **動手試試**
- **實驗**：使用佇列建立一個簡單的生產者-消費者系統
- **挑戰**：使用核心服務實作一個可以暫停/恢復的任務
- **除錯**：使用核心掛鉤監控服務使用情況和效能

### **重點摘要**
核心服務提供了建構可靠嵌入式系統的基礎模組，讓你能專注於應用程式邏輯，同時 RTOS 在幕後處理複雜的協調工作。

---

## 📋 **目錄**
- [概述](#概述)
- [什麼是核心服務？](#什麼是核心服務)
- [為什麼核心服務如此重要？](#為什麼核心服務如此重要)
- [核心服務概念](#核心服務概念)
- [記憶體管理服務](#記憶體管理服務)
- [任務管理服務](#任務管理服務)
- [同步服務](#同步服務)
- [通訊服務](#通訊服務)
- [計時服務](#計時服務)
- [FreeRTOS 核心服務](#freertos-核心服務)
- [實作](#實作)
- [常見陷阱](#常見陷阱)
- [最佳實踐](#最佳實踐)
- [面試題目](#面試題目)

---

## 🎯 **概述**

核心服務構成即時作業系統的基礎，提供任務管理、記憶體分配、同步和通訊的基本功能。了解核心服務對於建構能夠有效管理資源、協調多個任務並提供可靠即時效能的嵌入式系統至關重要。

### **關鍵概念**
- **核心服務** - 核心作業系統函式與 API
- **系統呼叫** - 使用者任務與核心之間的介面
- **資源管理** - 系統資源的高效分配與管理
- **服務抽象** - 對應用程式隱藏硬體複雜性
- **即時保證** - 確保可預測的服務行為

---

## 🤔 **什麼是核心服務？**

核心服務是 RTOS 核心提供的基本函式，用於管理系統資源、協調任務執行，並為應用程式軟體提供一致的介面。它們抽象化硬體複雜性，並為即時應用程式提供可靠、可預測的服務。

### **核心概念**

**服務定義：**
- **系統函式**：作業系統提供的核心函式
- **資源管理**：CPU、記憶體和 I/O 資源的管理
- **任務協調**：多個任務的協調與同步
- **硬體抽象**：硬體資源的抽象介面

**服務特性：**
- **可靠性**：服務必須在所有條件下可靠運作
- **可預測性**：服務行為必須可預測且一致
- **效率**：服務必須高效以最小化開銷
- **可攜性**：服務應能跨不同硬體平台運作

**服務類別：**
- **記憶體服務**：記憶體分配、釋放和管理
- **任務服務**：任務建立、刪除和控制
- **同步服務**：號誌、互斥鎖和事件旗標
- **通訊服務**：佇列、信箱和訊息傳遞
- **計時服務**：延遲、逾時和週期性執行

### **核心架構**

**基本核心結構：**
```
┌─────────────────────────────────────────────────────────────┐
│                      應用程式層                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │   任務 1    │  │   任務 2    │  │   任務 3    │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      核心服務層                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │   記憶體    │  │    任務     │  │    同步     │        │
│  │   服務     │  │    服務     │  │    服務     │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    硬體抽象層                                │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │   CPU       │  │   記憶體    │  │   I/O      │        │
│  │   控制     │  │   控制     │  │   控制     │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
└─────────────────────────────────────────────────────────────┘
```

**服務呼叫流程：**
```
┌─────────────────────────────────────────────────────────────┐
│                    服務呼叫流程                              │
├─────────────────────────────────────────────────────────────┤
│  1. 任務呼叫核心服務                                        │
│  2. 核心驗證參數                                            │
│  3. 核心執行請求的操作                                       │
│  4. 核心將結果返回任務                                       │
│  5. 任務繼續執行                                            │
└─────────────────────────────────────────────────────────────┘
```

---

## 🎯 **為什麼核心服務如此重要？**

核心服務對於建構可靠、高效的嵌入式系統至關重要，因為它們為所有更高層級的功能提供基礎。如果沒有核心服務，應用程式將需要直接管理硬體資源，導致程式碼複雜、容易出錯且不可攜。

### **系統架構優勢**

**資源管理：**
- **集中控制**：系統資源的集中管理
- **高效分配**：資源的高效分配與釋放
- **資源保護**：保護資源免受未授權存取
- **資源共享**：任務之間的協調資源共享

**應用程式開發：**
- **簡化程式設計**：簡化應用程式設計介面
- **硬體獨立性**：應用程式獨立於特定硬體
- **程式碼重用性**：跨不同平台的可重用程式碼
- **標準化**：通用操作的標準介面

**即時效能：**
- **可預測行為**：可預測且一致的系統行為
- **計時保證**：關鍵操作的計時保證
- **資源效率**：有限系統資源的高效使用
- **效能最佳化**：針對即時需求的最佳化效能

### **設計考量**

**服務設計：**
- **API 設計**：設計良好的應用程式設計介面
- **錯誤處理**：全面的錯誤處理與回報
- **效能**：針對即時應用程式的最佳化效能
- **可靠性**：在所有條件下的可靠運作

**資源限制：**
- **記憶體限制**：在有限的記憶體資源內運作
- **CPU 限制**：最小化服務呼叫的 CPU 開銷
- **功耗考量**：在服務設計中考慮功耗
- **成本限制**：在功能與實作成本之間取得平衡

---

## 🔧 **核心服務概念**

### **服務架構**

**服務層次：**
- **應用程式介面**：面向應用程式的高階服務介面
- **服務實作**：核心服務實作邏輯
- **硬體介面**：底層硬體存取和控制
- **資源管理**：資源分配和管理邏輯

**服務類型：**
- **同步服務**：阻塞直到完成的服務
- **非同步服務**：立即返回的服務
- **阻塞服務**：可以阻塞呼叫任務的服務
- **非阻塞服務**：永不阻塞呼叫任務的服務

**服務特性：**
- **可重入性**：服務可從多個任務呼叫
- **執行緒安全**：服務對並行存取是安全的
- **錯誤處理**：全面的錯誤偵測與回報
- **效能**：針對即時效能需求的最佳化

### **服務呼叫機制**

**直接呼叫：**
- **函式呼叫**：直接函式呼叫核心服務
- **參數傳遞**：透過函式引數傳遞參數
- **返回值**：透過函式返回值返回結果
- **錯誤碼**：透過返回碼指示錯誤條件

**系統呼叫：**
- **軟體中斷**：透過軟體中斷存取服務
- **陷阱指令**：透過 CPU 陷阱指令存取服務
- **函式庫函式**：以函式庫函式包裝的服務
- **API 函式**：用於服務存取的高階 API 函式

**服務開銷：**
- **呼叫開銷**：進入和退出服務函式的時間
- **參數處理**：處理服務參數的時間
- **資源管理**：資源分配和管理的時間
- **上下文切換**：需要時的任務上下文切換時間

---

## 💾 **記憶體管理服務**

### **記憶體分配服務**

**動態記憶體分配：**
```c
// FreeRTOS 記憶體分配服務
void vMemoryAllocationExample(void) {
    // 分配記憶體
    void *ptr1 = pvPortMalloc(1024);
    if (ptr1 != NULL) {
        printf("在 %p 分配了 1KB\n", ptr1);
        
        // 使用已分配的記憶體
        memset(ptr1, 0xAA, 1024);
        
        // 釋放記憶體
        vPortFree(ptr1);
        printf("已釋放 1KB 記憶體\n");
    } else {
        printf("分配 1KB 失敗\n");
    }
    
    // 分配多個區塊
    void *blocks[10];
    for (int i = 0; i < 10; i++) {
        blocks[i] = pvPortMalloc(100);
        if (blocks[i] == NULL) {
            printf("分配區塊 %d 失敗\n", i);
            break;
        }
    }
    
    // 釋放所有區塊
    for (int i = 0; i < 10; i++) {
        if (blocks[i] != NULL) {
            vPortFree(blocks[i]);
        }
    }
}

// 帶錯誤處理的記憶體分配
void *vSafeMemoryAllocation(size_t size) {
    void *ptr = pvPortMalloc(size);
    
    if (ptr == NULL) {
        // 處理分配失敗
        printf("大小為 %zu 的記憶體分配失敗\n", size);
        
        // 嘗試釋放一些記憶體
        vTaskDelay(pdMS_TO_TICKS(100));
        
        // 重試分配
        ptr = pvPortMalloc(size);
        if (ptr == NULL) {
            printf("記憶體分配重試失敗\n");
            // 可在此觸發系統復原
        }
    }
    
    return ptr;
}
```

**靜態記憶體分配：**
```c
// 任務的靜態記憶體分配
void vStaticMemoryExample(void) {
    // 靜態任務堆疊和控制區塊
    static StackType_t xTaskStack[256];
    static StaticTask_t xTaskTCB;
    
    // 使用靜態分配建立任務
    TaskHandle_t xTaskHandle = xTaskCreateStatic(
        vExampleTask,           // 任務函式
        "Static_Task",          // 任務名稱
        256,                    // 堆疊大小
        NULL,                   // 參數
        2,                      // 優先權
        xTaskStack,             // 堆疊緩衝區
        &xTaskTCB               // 任務控制區塊
    );
    
    if (xTaskHandle != NULL) {
        printf("靜態任務建立成功\n");
    } else {
        printf("建立靜態任務失敗\n");
    }
}

// 靜態佇列分配
void vStaticQueueExample(void) {
    // 靜態佇列儲存區
    static uint8_t ucQueueStorageArea[100];
    static StaticQueue_t xStaticQueue;
    
    // 使用靜態分配建立佇列
    QueueHandle_t xQueue = xQueueCreateStatic(
        10,                     // 佇列長度
        sizeof(uint8_t),        // 項目大小
        ucQueueStorageArea,     // 儲存區域
        &xStaticQueue           // 佇列控制區塊
    );
    
    if (xQueue != NULL) {
        printf("靜態佇列建立成功\n");
    } else {
        printf("建立靜態佇列失敗\n");
    }
}
```

### **記憶體池服務**

**記憶體池實作：**
```c
// 記憶體池結構
typedef struct {
    uint8_t *pool_start;
    uint8_t *pool_end;
    uint32_t pool_size;
    uint32_t used_blocks;
    uint32_t total_blocks;
    uint32_t block_size;
    uint8_t *free_list;
} memory_pool_t;

// 建立記憶體池
memory_pool_t* vCreateMemoryPool(uint32_t block_size, uint32_t num_blocks) {
    memory_pool_t *pool = pvPortMalloc(sizeof(memory_pool_t));
    
    if (pool != NULL) {
        pool->block_size = block_size;
        pool->total_blocks = num_blocks;
        pool->pool_size = block_size * num_blocks;
        
        // 分配記憶體池記憶體
        pool->pool_start = pvPortMalloc(pool->pool_size);
        if (pool->pool_start != NULL) {
            pool->pool_end = pool->pool_start + pool->pool_size;
            
            // 初始化空閒鏈結串列
            pool->free_list = pool->pool_start;
            for (uint32_t i = 0; i < num_blocks - 1; i++) {
                *(uint32_t*)(pool->pool_start + i * block_size) = 
                    (uint32_t)(pool->pool_start + (i + 1) * block_size);
            }
            *(uint32_t*)(pool->pool_start + (num_blocks - 1) * block_size) = 0;
            
            pool->used_blocks = 0;
            printf("記憶體池已建立：%lu 個區塊，每個 %lu 位元組\n", 
                   num_blocks, block_size);
        } else {
            vPortFree(pool);
            pool = NULL;
        }
    }
    
    return pool;
}

// 從記憶體池分配區塊
void* vAllocateFromPool(memory_pool_t *pool) {
    if (pool->free_list && pool->used_blocks < pool->total_blocks) {
        void *block = pool->free_list;
        pool->free_list = (void*)*(uint32_t*)block;
        pool->used_blocks++;
        return block;
    }
    return NULL;
}

// 將區塊釋放回記憶體池
void vFreeToPool(memory_pool_t *pool, void *block) {
    if (block >= pool->pool_start && block < pool->pool_end) {
        *(uint32_t*)block = (uint32_t)pool->free_list;
        pool->free_list = block;
        pool->used_blocks--;
    }
}
```

---

## 🚀 **任務管理服務**

### **任務建立與控制**

**任務建立服務：**
```c
// 使用各種選項建立任務
void vTaskCreationExample(void) {
    TaskHandle_t xTaskHandle;
    BaseType_t xResult;
    
    // 建立基本任務
    xResult = xTaskCreate(
        vBasicTask,             // 任務函式
        "Basic_Task",           // 任務名稱
        128,                    // 堆疊大小
        NULL,                   // 參數
        2,                      // 優先權
        &xTaskHandle            // 任務控制代碼
    );
    
    if (xResult == pdPASS) {
        printf("基本任務建立成功\n");
    }
    
    // 建立帶參數的任務
    uint32_t task_param = 0x12345678;
    xResult = xTaskCreate(
        vParameterTask,         // 任務函式
        "Param_Task",           // 任務名稱
        256,                    // 堆疊大小
        (void*)task_param,      // 參數
        3,                      // 優先權
        NULL                    // 任務控制代碼（不需要）
    );
    
    if (xResult == pdPASS) {
        printf("帶參數任務建立成功\n");
    }
    
    // 使用靜態分配建立任務
    static StackType_t xStaticStack[512];
    static StaticTask_t xStaticTCB;
    
    TaskHandle_t xStaticTaskHandle = xTaskCreateStatic(
        vStaticTask,            // 任務函式
        "Static_Task",          // 任務名稱
        512,                    // 堆疊大小
        NULL,                   // 參數
        1,                      // 優先權
        xStaticStack,           // 堆疊緩衝區
        &xStaticTCB             // 任務控制區塊
    );
    
    if (xStaticTaskHandle != NULL) {
        printf("靜態任務建立成功\n");
    }
}

// 任務控制函式
void vTaskControlExample(TaskHandle_t xTaskHandle) {
    // 暫停任務
    vTaskSuspend(xTaskHandle);
    printf("任務已暫停\n");
    
    // 恢復任務
    vTaskResume(xTaskHandle);
    printf("任務已恢復\n");
    
    // 變更任務優先權
    UBaseType_t uxNewPriority = 4;
    vTaskPrioritySet(xTaskHandle, uxNewPriority);
    printf("任務優先權已變更為 %lu\n", uxNewPriority);
    
    // 取得任務優先權
    UBaseType_t uxCurrentPriority = uxTaskPriorityGet(xTaskHandle);
    printf("目前任務優先權：%lu\n", uxCurrentPriority);
    
    // 刪除任務
    vTaskDelete(xTaskHandle);
    printf("任務已刪除\n");
}
```

**任務資訊服務：**
```c
// 任務資訊與監控
void vTaskInformationExample(void) {
    // 取得目前任務控制代碼
    TaskHandle_t xCurrentTask = xTaskGetCurrentTaskHandle();
    printf("目前任務控制代碼：%p\n", xCurrentTask);
    
    // 取得目前任務名稱
    char *pcTaskName = pcTaskGetName(xCurrentTask);
    printf("目前任務名稱：%s\n", pcTaskName);
    
    // 取得任務狀態
    eTaskState eState = eTaskGetState(xCurrentTask);
    switch (eState) {
        case eRunning:
            printf("任務狀態：執行中\n");
            break;
        case eReady:
            printf("任務狀態：就緒\n");
            break;
        case eBlocked:
            printf("任務狀態：阻塞\n");
            break;
        case eSuspended:
            printf("任務狀態：暫停\n");
            break;
        case eDeleted:
            printf("任務狀態：已刪除\n");
            break;
        default:
            printf("任務狀態：未知\n");
            break;
    }
    
    // 取得任務數量
    UBaseType_t uxNumberOfTasks = uxTaskGetNumberOfTasks();
    printf("任務總數：%lu\n", uxNumberOfTasks);
    
    // 取得系統狀態
    UBaseType_t uxSchedulerRunning = xTaskGetSchedulerState();
    if (uxSchedulerRunning == taskSCHEDULER_RUNNING) {
        printf("排程器正在執行\n");
    } else if (uxSchedulerRunning == taskSCHEDULER_NOT_STARTED) {
        printf("排程器尚未啟動\n");
    } else if (uxSchedulerRunning == taskSCHEDULER_SUSPENDED) {
        printf("排程器已暫停\n");
    }
}
```

---

## 🔒 **同步服務**

### **號誌服務**

**二元號誌服務：**
```c
// 二元號誌範例
void vBinarySemaphoreExample(void) {
    // 建立二元號誌
    SemaphoreHandle_t xBinarySemaphore = xSemaphoreCreateBinary();
    
    if (xBinarySemaphore != NULL) {
        printf("二元號誌建立成功\n");
        
        // 釋放號誌
        xSemaphoreGive(xBinarySemaphore);
        printf("號誌已釋放\n");
        
        // 取得號誌
        if (xSemaphoreTake(xBinarySemaphore, pdMS_TO_TICKS(1000)) == pdTRUE) {
            printf("號誌取得成功\n");
        } else {
            printf("取得號誌失敗\n");
        }
        
        // 刪除號誌
        vSemaphoreDelete(xBinarySemaphore);
        printf("二元號誌已刪除\n");
    }
}

// 計數號誌範例
void vCountingSemaphoreExample(void) {
    // 建立計數號誌
    SemaphoreHandle_t xCountingSemaphore = xSemaphoreCreateCounting(
        5,                      // 最大計數
        0                       // 初始計數
    );
    
    if (xCountingSemaphore != NULL) {
        printf("計數號誌建立成功\n");
        
        // 多次釋放號誌
        for (int i = 0; i < 3; i++) {
            xSemaphoreGive(xCountingSemaphore);
            printf("號誌已釋放，計數：%d\n", i + 1);
        }
        
        // 多次取得號誌
        for (int i = 0; i < 3; i++) {
            if (xSemaphoreTake(xCountingSemaphore, pdMS_TO_TICKS(1000)) == pdTRUE) {
                printf("號誌已取得，剩餘：%d\n", 2 - i);
            }
        }
        
        vSemaphoreDelete(xCountingSemaphore);
    }
}
```

**互斥鎖服務：**
```c
// 互斥鎖範例
void vMutexExample(void) {
    // 建立互斥鎖
    SemaphoreHandle_t xMutex = xSemaphoreCreateMutex();
    
    if (xMutex != NULL) {
        printf("互斥鎖建立成功\n");
        
        // 取得互斥鎖
        if (xSemaphoreTake(xMutex, pdMS_TO_TICKS(1000)) == pdTRUE) {
            printf("互斥鎖已取得\n");
            
            // 臨界區段
            printf("正在執行臨界區段...\n");
            vTaskDelay(pdMS_TO_TICKS(100));
            
            // 釋放互斥鎖
            xSemaphoreGive(xMutex);
            printf("互斥鎖已釋放\n");
        }
        
        vSemaphoreDelete(xMutex);
    }
}

// 遞迴互斥鎖範例
void vRecursiveMutexExample(void) {
    // 建立遞迴互斥鎖
    SemaphoreHandle_t xRecursiveMutex = xSemaphoreCreateRecursiveMutex();
    
    if (xRecursiveMutex != NULL) {
        printf("遞迴互斥鎖建立成功\n");
        
        // 遞迴取得互斥鎖
        if (xSemaphoreTakeRecursive(xRecursiveMutex, pdMS_TO_TICKS(1000)) == pdTRUE) {
            printf("第一次取得互斥鎖\n");
            
            // 再次取得互斥鎖（遞迴）
            if (xSemaphoreTakeRecursive(xRecursiveMutex, pdMS_TO_TICKS(1000)) == pdTRUE) {
                printf("第二次取得互斥鎖（遞迴）\n");
                
                // 釋放互斥鎖（第二次取得）
                xSemaphoreGiveRecursive(xRecursiveMutex);
                printf("第二次釋放互斥鎖\n");
            }
            
            // 釋放互斥鎖（第一次取得）
            xSemaphoreGiveRecursive(xRecursiveMutex);
            printf("第一次釋放互斥鎖\n");
        }
        
        vSemaphoreDelete(xRecursiveMutex);
    }
}
```

---

## 📡 **通訊服務**

### **佇列服務**

**基本佇列操作：**
```c
// 佇列建立與使用
void vQueueExample(void) {
    // 建立佇列
    QueueHandle_t xQueue = xQueueCreate(
        10,                     // 佇列長度
        sizeof(uint32_t)        // 項目大小
    );
    
    if (xQueue != NULL) {
        printf("佇列建立成功\n");
        
        // 發送項目到佇列
        for (int i = 0; i < 5; i++) {
            uint32_t item = i * 10;
            if (xQueueSend(xQueue, &item, pdMS_TO_TICKS(1000)) == pdPASS) {
                printf("已發送項目：%lu\n", item);
            }
        }
        
        // 從佇列接收項目
        uint32_t received_item;
        for (int i = 0; i < 5; i++) {
            if (xQueueReceive(xQueue, &received_item, pdMS_TO_TICKS(1000)) == pdPASS) {
                printf("已接收項目：%lu\n", received_item);
            }
        }
        
        // 刪除佇列
        vQueueDelete(xQueue);
        printf("佇列已刪除\n");
    }
}

// 佇列窺視操作
void vQueuePeekExample(void) {
    QueueHandle_t xQueue = xQueueCreate(5, sizeof(char));
    
    if (xQueue != NULL) {
        // 發送資料
        char data[] = "Hello";
        for (int i = 0; i < strlen(data); i++) {
            xQueueSend(xQueue, &data[i], 0);
        }
        
        // 窺視第一個項目而不移除
        char peeked_item;
        if (xQueuePeek(xQueue, &peeked_item, pdMS_TO_TICKS(1000)) == pdPASS) {
            printf("窺視項目：%c\n", peeked_item);
        }
        
        // 接收所有項目
        char received_item;
        while (xQueueReceive(xQueue, &received_item, 0) == pdPASS) {
            printf("已接收：%c\n", received_item);
        }
        
        vQueueDelete(xQueue);
    }
}
```

**佇列集服務：**
```c
// 佇列集範例
void vQueueSetExample(void) {
    // 建立佇列
    QueueHandle_t xQueue1 = xQueueCreate(5, sizeof(uint32_t));
    QueueHandle_t xQueue2 = xQueueCreate(5, sizeof(uint32_t));
    
    if (xQueue1 != NULL && xQueue2 != NULL) {
        // 建立佇列集
        QueueSetHandle_t xQueueSet = xQueueCreateSet(10);
        
        if (xQueueSet != NULL) {
            // 將佇列加入集合
            xQueueAddToSet(xQueue1, xQueueSet);
            xQueueAddToSet(xQueue2, xQueueSet);
            
            // 發送資料到佇列
            uint32_t data1 = 100;
            uint32_t data2 = 200;
            xQueueSend(xQueue1, &data1, 0);
            xQueueSend(xQueue2, &data2, 0);
            
            // 等待任何佇列有資料
            QueueSetMemberHandle_t xActivatedQueue = xQueueSelectFromSet(xQueueSet, pdMS_TO_TICKS(1000));
            
            if (xActivatedQueue == xQueue1) {
                printf("佇列 1 有資料\n");
                uint32_t received_data;
                xQueueReceive(xQueue1, &received_data, 0);
                printf("從佇列 1 接收：%lu\n", received_data);
            } else if (xActivatedQueue == xQueue2) {
                printf("佇列 2 有資料\n");
                uint32_t received_data;
                xQueueReceive(xQueue2, &received_data, 0);
                printf("從佇列 2 接收：%lu\n", received_data);
            }
            
            vQueueDelete(xQueueSet);
        }
        
        vQueueDelete(xQueue1);
        vQueueDelete(xQueue2);
    }
}
```

---

## ⏰ **計時服務**

### **延遲與時間服務**

**基本計時服務：**
```c
// 延遲與時間服務
void vTimingServicesExample(void) {
    // 取得目前 tick 計數
    TickType_t xCurrentTicks = xTaskGetTickCount();
    printf("目前 tick 計數：%lu\n", xCurrentTicks);
    
    // 簡單延遲
    printf("開始延遲...\n");
    vTaskDelay(pdMS_TO_TICKS(1000));  // 延遲 1 秒
    printf("延遲完成\n");
    
    // 延遲到特定時間
    TickType_t xLastWakeTime = xTaskGetTickCount();
    printf("開始週期性延遲...\n");
    
    for (int i = 0; i < 5; i++) {
        // 延遲到下個週期
        vTaskDelayUntil(&xLastWakeTime, pdMS_TO_TICKS(500));
        printf("週期性延遲 %d，tick：%lu\n", i + 1, xTaskGetTickCount());
    }
    
    // 從 ISR 取得 tick 計數
    TickType_t xISRTicks = xTaskGetTickCountFromISR();
    printf("來自 ISR 的 tick 計數：%lu\n", xISRTicks);
}

// 時間轉換工具
void vTimeConversionExample(void) {
    // 將毫秒轉換為 tick
    uint32_t ms = 1000;
    TickType_t ticks = pdMS_TO_TICKS(ms);
    printf("%lu 毫秒 = %lu ticks\n", ms, ticks);
    
    // 將 tick 轉換為毫秒
    TickType_t tick_count = 100;
    uint32_t milliseconds = (tick_count * 1000) / configTICK_RATE_HZ;
    printf("%lu ticks = %lu 毫秒\n", tick_count, milliseconds);
    
    // 將秒轉換為 tick
    uint32_t seconds = 5;
    TickType_t seconds_ticks = pdSEC_TO_TICKS(seconds);
    printf("%lu 秒 = %lu ticks\n", seconds, seconds_ticks);
}
```

---

## ⚙️ **FreeRTOS 核心服務**

### **核心配置**

**基本核心配置：**
```c
// FreeRTOS 核心配置
#define configUSE_PREEMPTION                    1
#define configUSE_TIME_SLICING                  1
#define configUSE_TICKLESS_IDLE                 0
#define configUSE_IDLE_HOOK                     0
#define configUSE_TICK_HOOK                     0
#define configCPU_CLOCK_HZ                      16000000
#define configTICK_RATE_HZ                      1000
#define configMAX_PRIORITIES                    32
#define configMINIMAL_STACK_SIZE                128
#define configMAX_TASK_NAME_LEN                 16
#define configUSE_16_BIT_TICKS                  0
#define configIDLE_SHOULD_YIELD                 1
#define configUSE_MUTEXES                       1
#define configUSE_RECURSIVE_MUTEXES             0
#define configUSE_COUNTING_SEMAPHORES           1
#define configUSE_ALTERNATIVE_API               0
#define configCHECK_FOR_STACK_OVERFLOW          2
#define configUSE_MALLOC_FAILED_HOOK            1
#define configUSE_APPLICATION_TASK_TAG          0
#define configUSE_QUEUE_SETS                    1
#define configUSE_TASK_NOTIFICATIONS            1
#define configSUPPORT_STATIC_ALLOCATION         1
#define configSUPPORT_DYNAMIC_ALLOCATION        1
#define configUSE_MUTEXES                       1
#define configUSE_RECURSIVE_MUTEXES             0
#define configUSE_COUNTING_SEMAPHORES           1
#define configUSE_ALTERNATIVE_API               0
#define configCHECK_FOR_STACK_OVERFLOW          2
#define configUSE_MALLOC_FAILED_HOOK            1
#define configUSE_APPLICATION_TASK_TAG          0
#define configUSE_QUEUE_SETS                    1
#define configUSE_TASK_NOTIFICATIONS            1
#define configSUPPORT_STATIC_ALLOCATION         1
#define configSUPPORT_DYNAMIC_ALLOCATION        1
```

**核心掛鉤：**
```c
// FreeRTOS 核心掛鉤
void vApplicationIdleHook(void) {
    // 閒置任務執行時呼叫
    // 可用於電源管理
    __WFI();  // 等待中斷
}

void vApplicationTickHook(void) {
    // 每次 tick 時呼叫
    // 可用於週期性操作
    static uint32_t tick_count = 0;
    tick_count++;
    
    if (tick_count % 1000 == 0) {
        // 每 1000 次 tick
        printf("系統已執行 %lu 秒\n", tick_count / 1000);
    }
}

void vApplicationMallocFailedHook(void) {
    // 記憶體分配失敗時呼叫
    printf("記憶體分配失敗！\n");
    
    // 處理記憶體分配失敗
    // 可以重啟系統或釋放記憶體
}

void vApplicationStackOverflowHook(TaskHandle_t xTask, char *pcTaskName) {
    // 偵測到堆疊溢位時呼叫
    printf("任務堆疊溢位：%s\n", pcTaskName);
    
    // 處理堆疊溢位
    // 可以重啟系統或任務
}
```

---

## 🚀 **實作**

### **完整核心服務系統**

**系統初始化：**
```c
// 完整核心服務系統初始化
void vInitializeKernelServices(void) {
    // 初始化記憶體池
    xMemoryPool = vCreateMemoryPool(64, 20);
    
    // 建立系統佇列
    xSystemQueue = xQueueCreate(10, sizeof(system_message_t));
    xEventQueue = xQueueCreate(20, sizeof(event_t));
    
    // 建立系統號誌
    xSystemMutex = xSemaphoreCreateMutex();
    xResourceSemaphore = xSemaphoreCreateCounting(5, 5);
    
    // 建立系統任務
    xTaskCreate(vSystemMonitorTask, "SysMon", 256, NULL, 5, NULL);
    xTaskCreate(vResourceManagerTask, "ResMgr", 512, NULL, 4, NULL);
    xTaskCreate(vEventProcessorTask, "EventProc", 1024, NULL, 3, NULL);
    
    printf("核心服務初始化成功\n");
}

// 主函式
int main(void) {
    // 硬體初始化
    SystemInit();
    HAL_Init();
    
    // 初始化周邊
    MX_GPIO_Init();
    MX_USART1_UART_Init();
    
    // 初始化核心服務
    vInitializeKernelServices();
    
    // 啟動排程器
    vTaskStartScheduler();
    
    // 不應該執行到這裡
    while (1) {
        // 錯誤處理
    }
}
```

---

## ⚠️ **常見陷阱**

### **服務使用問題**

**常見問題：**
- **參數錯誤**：傳遞錯誤的參數給核心服務
- **資源洩漏**：未釋放已分配的資源
- **優先權反轉**：不正確的優先權指派
- **死結**：循環的資源相依性

**解決方案：**
- **參數驗證**：在呼叫服務前驗證參數
- **資源管理**：正確管理資源生命週期
- **優先權管理**：使用適當的優先權指派策略
- **死結預防**：設計系統以避免死結

### **效能問題**

**效能問題：**
- **服務開銷**：過多的服務呼叫開銷
- **資源爭用**：有限資源的衝突
- **記憶體碎片化**：分配造成的記憶體碎片化
- **上下文切換**：過多的任務上下文切換

**解決方案：**
- **最小化服務呼叫**：減少不必要的服務呼叫
- **資源最佳化**：最佳化資源使用模式
- **記憶體池**：對頻繁分配使用記憶體池
- **任務最佳化**：最佳化任務設計和排程

---

## ✅ **最佳實踐**

### **服務設計原則**

**API 設計：**
- **一致的介面**：使用一致的服務介面
- **錯誤處理**：實作全面的錯誤處理
- **文件**：記錄服務行為和使用方式
- **驗證**：驗證服務參數和狀態

**資源管理：**
- **高效分配**：最佳化資源分配策略
- **資源共享**：使用適當的資源共享機制
- **清理**：實作適當的資源清理程序
- **監控**：監控資源使用情況和可用性

### **效能最佳化**

**服務效率：**
- **最小化開銷**：減少服務呼叫開銷
- **最佳化演算法**：在服務中使用高效演算法
- **硬體特性**：在可用時利用硬體特性
- **分析與測量**：使用分析工具識別瓶頸

**資源最佳化：**
- **記憶體管理**：最佳化記憶體分配和使用
- **CPU 使用率**：高效使用 CPU 資源
- **電源管理**：在服務設計中考慮功耗
- **可擴展性**：為系統可擴展性設計服務

---

## 🔬 **引導式實驗**

### **實驗 1：基本佇列通訊**
**目標**：使用佇列建立兩個任務之間的通訊
**步驟**：
1. 建立一個佇列來保存感測器資料
2. 建立生產者任務讀取感測器並發送資料
3. 建立消費者任務接收並處理資料
4. 使用號誌在資料就緒時發出信號

**預期結果**：資料在任務之間順暢流動並具有適當的同步

### **實驗 2：記憶體池管理**
**目標**：了解記憶體池分配與釋放
**步驟**：
1. 為固定大小的緩衝區建立記憶體池
2. 為不同任務分配緩衝區
3. 監控記憶體池使用情況和碎片化
4. 實作適當的清理和錯誤處理

**預期結果**：高效的記憶體使用且無碎片化

### **實驗 3：服務效能測量**
**目標**：測量核心服務效能和開銷
**步驟**：
1. 使用 GPIO 測量服務呼叫計時
2. 比較不同的服務實作
3. 測量不同服務的記憶體使用
4. 在負載下分析服務效能

**預期結果**：了解服務開銷和最佳化機會

---

## ✅ **自我檢查**

### **概念理解檢查**
- [ ] 你能解釋為什麼核心服務比自行實作所有功能更好嗎？
- [ ] 你了解不同同步服務之間的差異嗎？
- [ ] 你能識別何時使用佇列、何時使用號誌嗎？
- [ ] 你知道如何測量核心服務效能嗎？

### **實務技能檢查**
- [ ] 你能使用核心服務建立生產者-消費者系統嗎？
- [ ] 你知道如何除錯核心服務問題嗎？
- [ ] 你能在核心服務中實作適當的錯誤處理嗎？
- [ ] 你了解記憶體管理策略嗎？

### **進階概念檢查**
- [ ] 你能解釋如何最佳化核心服務效能嗎？
- [ ] 你了解資源爭用以及如何處理它嗎？
- [ ] 你能實作自訂核心服務嗎？
- [ ] 你知道如何分析核心服務使用情況嗎？

---

## 🔗 **交叉連結**

### **相關主題**
- **[FreeRTOS 基礎](./FreeRTOS_Basics.md)** - 了解 RTOS 上下文
- **[任務建立與管理](./Task_Creation_Management.md)** - 任務如何使用核心服務
- **[中斷處理](./Interrupt_Handling.md)** - 在 ISR 中使用核心服務
- **[記憶體管理](../Embedded_C/Memory_Management.md)** - 記憶體分配策略

### **先備知識**
- **[C 語言基礎](../Embedded_C/C_Language_Fundamentals.md)** - 基本程式設計概念
- **[指標與記憶體位址](../Embedded_C/Pointers_Memory_Addresses.md)** - 記憶體概念
- **[GPIO 配置](../Hardware_Fundamentals/GPIO_Configuration.md)** - 基本 I/O 設定

### **下一步**
- **[排程演算法](./Scheduling_Algorithms.md)** - 核心服務如何影響排程
- **[效能監控](./Performance_Monitoring.md)** - 測量服務效能
- **[即時除錯](./Real_Time_Debugging.md)** - 除錯服務問題

---

## 📋 **快速參考：重點摘要**

### **核心服務基礎**
- **目的**：提供核心作業系統函式用於資源管理和任務協調
- **類型**：記憶體、任務、同步、通訊和計時服務
- **特性**：可靠、可預測、高效且可攜
- **優勢**：抽象化硬體複雜性，提供經過驗證的解決方案

### **記憶體管理服務**
- **靜態分配**：編譯時的固定記憶體分配
- **動態分配**：執行時的記憶體分配與釋放
- **記憶體池**：固定大小物件的高效分配
- **碎片化**：記憶體碎片化管理與預防

### **同步服務**
- **號誌**：資源計數和任務信號通知
- **互斥鎖**：共享資源的互斥存取
- **事件旗標**：任務同步和事件通知
- **佇列**：任務之間帶同步的資料傳輸

### **通訊服務**
- **佇列**：具有阻塞/非阻塞操作的 FIFO 資料傳輸
- **信箱**：任務之間的單一訊息傳輸
- **訊息傳遞**：任務之間的結構化通訊
- **共享記憶體**：高效能通訊的直接記憶體存取

---

## ❓ **面試題目**

### **基本概念**

1. **什麼是核心服務？為什麼它們很重要？**
   - 作業系統提供的核心函式
   - 為應用程式抽象化硬體複雜性
   - 提供可靠、可預測的服務
   - 實現高效的資源管理

2. **你如何在不同的記憶體分配策略之間做選擇？**
   - 靜態分配用於固定需求
   - 動態分配用於可變需求
   - 記憶體池用於頻繁分配
   - 考慮記憶體限制和效能

3. **不同類型的同步服務有哪些？**
   - 號誌用於資源計數
   - 互斥鎖用於互斥存取
   - 事件旗標用於任務同步
   - 根據同步需求選擇

### **進階主題**

1. **解釋如何在 RTOS 中實作高效的記憶體管理。**
   - 對頻繁分配使用記憶體池
   - 實作記憶體重組
   - 監控記憶體使用情況和碎片化
   - 最佳化分配模式

2. **你如何處理核心服務中的資源爭用？**
   - 使用適當的同步機制
   - 實作優先權繼承
   - 設計資源分配策略
   - 監控資源使用模式

3. **你使用哪些策略來最佳化核心服務效能？**
   - 最小化服務呼叫開銷
   - 最佳化資源分配演算法
   - 在可用時使用硬體特性
   - 分析和測量效能

### **實務場景**

1. **為即時應用程式設計核心服務系統。**
   - 定義服務需求和優先權
   - 設計服務介面和實作
   - 實作資源管理策略
   - 處理錯誤條件和復原

2. **你如何除錯核心服務問題？**
   - 使用除錯掛鉤和監控
   - 分析資源使用模式
   - 檢查資源洩漏和爭用
   - 使用分析工具

3. **解釋如何實作自訂核心服務。**
   - 定義服務介面和行為
   - 實作服務邏輯和錯誤處理
   - 與現有核心服務整合
   - 測試和驗證服務功能

此增強版核心服務文件現在提供了概念解釋、實務洞見和技術實作細節的全面平衡，嵌入式工程師可以使用它來了解並在 RTOS 環境中實作穩健的核心服務系統。
