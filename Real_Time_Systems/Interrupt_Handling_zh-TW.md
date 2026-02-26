# RTOS 中的中斷處理

> **了解中斷處理、中斷服務常式，以及即時作業系統中的中斷管理，重點介紹 FreeRTOS 實作與即時中斷原理**

## 🎯 **概念 → 為何重要 → 最小範例 → 動手試試 → 重點回顧**

### **概念**
中斷就像緊急電話，可以打斷 CPU 正在做的任何事情，以便立即處理緊急事件。系統不需要持續檢查是否有事情需要處理（輪詢），而是等待重要事件來「呼叫」它。

### **為何重要**
在嵌入式系統中，時序就是一切。感測器讀數晚到 1ms 可能就是安全降落與墜機之間的差別。中斷確保關鍵事件能獲得即時關注，使系統既具備回應性又高效率。

### **最小範例**
```c
// 簡單的中斷處理程式
void UART_IRQHandler(void) {
    if (UART->SR & UART_SR_RXNE) {  // 接收到資料
        uint8_t data = UART->DR;     // 讀取資料
        // 通知任務處理資料
        xSemaphoreGiveFromISR(uart_semaphore, NULL);
    }
}
```

### **動手試試**
- **實驗**：在 ISR 中加入 GPIO 切換來量測時序
- **挑戰**：設計一個中斷系統，使溫度感測器能在 100μs 內回應
- **除錯**：使用示波器量測中斷延遲

### **重點回顧**
中斷將被動反應的系統轉變為主動回應的系統，確保關鍵事件能獲得即時關注，同時維持系統效率。

---

## 📋 **目錄**
- [概述](#overview)
- [什麼是中斷？](#what-are-interrupts)
- [為什麼中斷處理很重要？](#why-is-interrupt-handling-important)
- [中斷概念](#interrupt-concepts)
- [中斷服務常式](#interrupt-service-routines)
- [中斷優先權管理](#interrupt-priority-management)
- [中斷延遲分析](#interrupt-latency-analysis)
- [FreeRTOS 中斷處理](#freertos-interrupt-handling)
- [實作](#implementation)
- [常見陷阱](#common-pitfalls)
- [最佳實踐](#best-practices)
- [面試問題](#interview-questions)

---

## 🎯 **概述**

中斷處理是即時作業系統的關鍵組成部分，使系統能夠快速回應外部事件和硬體訊號。了解中斷處理對於建構能夠滿足即時需求、處理多個並行事件，以及提供可預測回應時間的嵌入式系統至關重要。

### **關鍵概念**
- **中斷處理** - 管理硬體和軟體中斷
- **中斷服務常式** - 處理中斷事件的函式
- **中斷優先權** - 管理中斷優先權和巢狀中斷
- **中斷延遲** - 從中斷發生到 ISR 開始執行的時間
- **即時回應** - 滿足關鍵事件的時序需求

---

## 🤔 **什麼是中斷？**

中斷是暫時中止正常程式執行以處理緊急事件的訊號。它們提供了硬體和軟體與 CPU 通訊的機制，使系統能夠快速回應外部事件，而無需持續輪詢。

### **核心概念**

**中斷定義：**
- **硬體中斷**：由硬體裝置產生（計時器、I/O、通訊）
- **軟體中斷**：由軟體產生，用於系統呼叫或例外
- **外部中斷**：由外部裝置或訊號產生
- **內部中斷**：由 CPU 產生，用於例外或錯誤

**中斷特性：**
- **非同步性**：可在程式執行的任何時間點發生
- **基於優先權**：不同的中斷具有不同的優先權等級
- **巢狀**：較高優先權的中斷可以中斷較低優先權的中斷
- **上下文保存**：CPU 狀態會被保存和恢復

**中斷 vs 輪詢：**
- **中斷驅動**：系統在事件發生時回應
- **輪詢**：系統持續檢查事件
- **效率**：中斷對偶發性事件更有效率
- **即時性**：中斷提供更好的即時回應

### **中斷系統架構**

**基本中斷系統：**
```
┌─────────────────────────────────────────────────────────────┐
│                      中斷來源                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │   計時器    │  │    UART     │  │   GPIO      │        │
│  │   中斷      │  │   中斷      │  │   中斷      │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    中斷控制器                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              優先權編碼器                            │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┘
│                    CPU                                     │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              中斷處理程式                            │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

**中斷處理流程：**
```
┌─────────────────────────────────────────────────────────────┐
│                      中斷處理流程                           │
├─────────────────────────────────────────────────────────────┤
│  1. 中斷發生                                              │
│  2. CPU 保存當前上下文                                     │
│  3. CPU 跳轉至中斷向量                                     │
│  4. 執行中斷服務常式                                       │
│  5. CPU 恢復上下文                                         │
│  6. 恢復正常執行                                           │
└─────────────────────────────────────────────────────────────┘
```

**即時 vs 非即時中斷處理：**
```
┌─────────────────────────────────────────────────────────────┐
│                   非即時系統                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │   任務 A    │  │   任務 B    │  │   任務 C    │        │
│  │  (10ms)     │  │  (20ms)     │  │  (15ms)     │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
│                              │                            │
│                              ▼                            │
│                    中斷在佇列中等待                        │
│                    (回應時間：45ms 之後)                   │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    即時系統                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │   任務 A    │  │   任務 B    │  │   任務 C    │        │
│  │  (10ms)     │  │  (20ms)     │  │  (15ms)     │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
│                              │                            │
│                              ▼                            │
│                    中斷立即搶佔                            │
│                    (回應時間：<1ms)                        │
└─────────────────────────────────────────────────────────────┘
```

---

## 🎯 **為什麼中斷處理很重要？**

有效的中斷處理對即時系統至關重要，因為它直接影響系統的回應性、可靠性，以及滿足時序需求的能力。適當的中斷設計確保系統能夠快速回應關鍵事件，同時維持可預測的行為。

### **即時系統需求**

**時序約束：**
- **回應時間**：系統必須在要求的時間範圍內回應事件
- **中斷延遲**：從中斷到 ISR 執行的時間必須有界限
- **抖動控制**：最小化中斷回應時序的變異
- **可預測性**：中斷行為在所有條件下必須可預測

**系統可靠性：**
- **事件處理**：可靠地處理硬體和軟體事件
- **錯誤恢復**：正確處理中斷錯誤和例外
- **系統穩定性**：在變化的中斷負載下維持穩定性
- **容錯能力**：即使中斷失敗也能繼續運作

**效能需求：**
- **低額外負擔**：最小化中斷處理的額外負擔
- **高效上下文切換**：快速保存和恢復 CPU 上下文
- **資源管理**：中斷期間有效使用系統資源
- **可擴展性**：有效處理多個中斷來源

### **中斷設計考量**

**系統架構：**
- **硬體支援**：可用的中斷硬體和功能
- **軟體框架**：RTOS 中斷處理機制
- **資源限制**：可用的記憶體和處理資源
- **效能需求**：所需的回應時間和吞吐量

**應用需求：**
- **事件類型**：需要中斷處理的事件類型
- **頻率**：中斷事件的預期頻率
- **關鍵程度**：不同中斷類型的關鍵程度
- **處理需求**：ISR 中所需的處理量

---

## 🔧 **中斷概念**

### **中斷類型與來源**

**硬體中斷：**
- **計時器中斷**：由硬體計時器和計數器產生
- **I/O 中斷**：由輸入/輸出裝置產生
- **通訊中斷**：由通訊介面產生
- **電源管理**：由電源管理事件產生

**軟體中斷：**
- **系統呼叫**：由軟體產生的中斷，用於系統服務
- **例外**：由 CPU 產生的中斷，用於錯誤條件
- **陷阱**：由軟體產生的中斷，用於除錯
- **排程器中斷**：由 RTOS 產生的中斷，用於任務切換

**依優先權排列的中斷來源：**
- **重置**：最高優先權，系統重置
- **NMI（不可遮蔽中斷）**：無法被停用
- **硬體錯誤**：系統錯誤處理
- **外部中斷**：硬體裝置中斷
- **Systick**：系統計時器中斷
- **軟體中斷**：最低優先權

### **中斷優先權與巢狀**

**優先權等級：**
- **等級 0**：最高優先權（重置、NMI）
- **等級 1-3**：高優先權（硬體錯誤、記憶體管理）
- **等級 4-7**：中優先權（外部中斷）
- **等級 8-15**：低優先權（周邊裝置中斷）

**中斷巢狀：**
- **巢狀規則**：較高優先權的中斷可以中斷較低優先權的中斷
- **巢狀深度**：巢狀中斷的最大數量
- **上下文堆疊**：每個巢狀層級都會保存 CPU 上下文
- **返回順序**：中斷以巢狀的反序返回

**優先權管理：**
- **靜態優先權**：固定的中斷優先權
- **動態優先權**：執行期間可以變更的優先權
- **優先權分配**：根據系統需求分配優先權
- **優先權反轉**：防止中斷處理中的優先權反轉

### **中斷延遲與時序**

**延遲組成部分：**
- **硬體延遲**：硬體產生中斷所需的時間
- **CPU 延遲**：CPU 回應中斷所需的時間
- **上下文保存**：保存 CPU 上下文所需的時間
- **ISR 執行**：執行中斷服務常式所需的時間
- **上下文恢復**：恢復 CPU 上下文所需的時間

**時序分析：**
- **最壞情況延遲**：最大可能的中斷延遲
- **平均延遲**：一段時間內的平均中斷延遲
- **抖動分析**：中斷延遲的變異
- **延遲預算**：為不同延遲組成部分分配時間

**延遲最佳化：**
- **最小化上下文保存**：最佳化上下文保存和恢復操作
- **高效 ISR**：設計高效的中斷服務常式
- **硬體最佳化**：使用硬體功能來減少延遲
- **中斷合併**：在可能時合併多個中斷

---

## 🚀 **中斷服務常式**

### **ISR 設計原則**

**ISR 特性：**
- **最短執行時間**：盡可能縮短 ISR 的執行時間
- **無阻塞操作**：避免可能阻塞執行的操作
- **有限的函式呼叫**：減少函式呼叫以降低額外負擔
- **高效資料處理**：使用高效的資料結構和演算法

**ISR 職責：**
- **事件處理**：處理特定的中斷事件
- **資料處理**：處理與中斷相關的資料
- **狀態清除**：清除中斷狀態旗標
- **任務通知**：將中斷事件通知任務

**ISR 設計模式：**
- **上半部/下半部**：將中斷處理分為快速部分和慢速部分
- **事件通知**：使用事件與任務通訊
- **資料緩衝**：緩衝資料供任務稍後處理
- **狀態回報**：回報中斷狀態以進行系統監控

### **ISR 實作**

**基本 ISR 結構：**
```c
// 基本中斷服務常式
void TIM2_IRQHandler(void) {
    // 清除中斷旗標
    if (TIM_GetITStatus(TIM2, TIM_IT_Update) != RESET) {
        TIM_ClearITPendingBit(TIM2, TIM_IT_Update);
        
        // 處理計時器中斷
        timer_interrupt_count++;
        
        // 如有需要，通知任務
        BaseType_t xHigherPriorityTaskWoken = pdFALSE;
        xTaskNotifyFromISR(xTimerTask, timer_interrupt_count, 
                          eSetValueWithOverwrite, &xHigherPriorityTaskWoken);
        
        // 如果喚醒了更高優先權的任務則讓出
        portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
    }
}

// UART 中斷處理程式
void USART1_IRQHandler(void) {
    if (USART_GetITStatus(USART1, USART_IT_RXNE) != RESET) {
        // 讀取接收到的資料
        uint8_t received_data = USART_ReceiveData(USART1);
        
        // 將資料緩衝以供任務處理
        if (uart_rx_buffer_index < UART_RX_BUFFER_SIZE) {
            uart_rx_buffer[uart_rx_buffer_index++] = received_data;
        }
        
        // 通知 UART 任務
        BaseType_t xHigherPriorityTaskWoken = pdFALSE;
        xTaskNotifyFromISR(xUARTTask, 1, eIncrement, &xHigherPriorityTaskWoken);
        
        portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
    }
}
```

**進階 ISR 與任務通訊：**
```c
// 具有多重事件處理的進階 ISR
void EXTI15_10_IRQHandler(void) {
    BaseType_t xHigherPriorityTaskWoken = pdFALSE;
    
    // 檢查哪條 GPIO 線路產生了中斷
    if (EXTI_GetITStatus(EXTI_Line15) != RESET) {
        EXTI_ClearITPendingBit(EXTI_Line15);
        
        // 處理 GPIO 中斷
        uint32_t gpio_state = GPIO_ReadInputDataBit(GPIOA, GPIO_Pin_15);
        
        // 建立事件資料
        gpio_event_t event = {
            .line = 15,
            .state = gpio_state,
            .timestamp = xTaskGetTickCountFromISR()
        };
        
        // 發送事件給任務
        xQueueSendFromISR(xGPIOEventQueue, &event, &xHigherPriorityTaskWoken);
    }
    
    if (EXTI_GetITStatus(EXTI_Line14) != RESET) {
        EXTI_ClearITPendingBit(EXTI_Line14);
        
        // 處理另一條 GPIO 線路
        uint32_t gpio_state = GPIO_ReadInputDataBit(GPIOA, GPIO_Pin_14);
        
        gpio_event_t event = {
            .line = 14,
            .state = gpio_state,
            .timestamp = xTaskGetTickCountFromISR()
        };
        
        xQueueSendFromISR(xGPIOEventQueue, &event, &xHigherPriorityTaskWoken);
    }
    
    // 如果喚醒了更高優先權的任務則讓出
    portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
}
```

### **ISR 與任務的通訊**

**任務通知：**
```c
// 使用任務通知的 ISR
void ADC1_IRQHandler(void) {
    if (ADC_GetITStatus(ADC1, ADC_IT_EOC) != RESET) {
        // 讀取 ADC 值
        uint16_t adc_value = ADC_GetConversionValue(ADC1);
        
        // 將值存入緩衝區
        if (adc_buffer_index < ADC_BUFFER_SIZE) {
            adc_buffer[adc_buffer_index++] = adc_value;
        }
        
        // 通知 ADC 任務
        BaseType_t xHigherPriorityTaskWoken = pdFALSE;
        xTaskNotifyFromISR(xADCTask, adc_value, eSetValueWithOverwrite, 
                          &xHigherPriorityTaskWoken);
        
        portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
    }
}

// 接收通知的任務
void vADCTask(void *pvParameters) {
    uint32_t notification_value;
    
    while (1) {
        // 等待來自 ISR 的通知
        if (xTaskNotifyWait(0, ULONG_MAX, &notification_value, portMAX_DELAY) == pdTRUE) {
            // 處理 ADC 值
            printf("ADC 值: %lu\n", notification_value);
            
            // 處理緩衝的資料
            while (adc_buffer_index > 0) {
                uint16_t value = adc_buffer[--adc_buffer_index];
                process_adc_value(value);
            }
        }
    }
}
```

**佇列通訊：**
```c
// 使用佇列進行通訊的 ISR
void DMA1_Channel1_IRQHandler(void) {
    if (DMA_GetITStatus(DMA1_Channel1, DMA_IT_TC) != RESET) {
        DMA_ClearITPendingBit(DMA1_Channel1, DMA_IT_TC);
        
        // DMA 傳輸完成
        dma_transfer_complete = true;
        
        // 發送完成事件給任務
        BaseType_t xHigherPriorityTaskWoken = pdFALSE;
        dma_event_t event = {
            .channel = 1,
            .status = DMA_COMPLETE,
            .timestamp = xTaskGetTickCountFromISR()
        };
        
        xQueueSendFromISR(xDMAEventQueue, &event, &xHigherPriorityTaskWoken);
        portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
    }
}
```

---

## 🎯 **中斷優先權管理**

### **優先權設定**

**硬體優先權設定：**
```c
// 設定中斷優先權
void vConfigureInterruptPriorities(void) {
    // 設定優先權分組
    NVIC_SetPriorityGrouping(NVIC_PriorityGroup_4);
    
    // 設定特定中斷的優先權
    NVIC_SetPriority(TIM2_IRQn, NVIC_EncodePriority(NVIC_PriorityGroup_4, 0, 0));
    NVIC_SetPriority(USART1_IRQn, NVIC_EncodePriority(NVIC_PriorityGroup_4, 1, 0));
    NVIC_SetPriority(EXTI15_10_IRQn, NVIC_EncodePriority(NVIC_PriorityGroup_4, 2, 0));
    NVIC_SetPriority(ADC1_IRQn, NVIC_EncodePriority(NVIC_PriorityGroup_4, 3, 0));
    
    // 啟用中斷
    NVIC_EnableIRQ(TIM2_IRQn);
    NVIC_EnableIRQ(USART1_IRQn);
    NVIC_EnableIRQ(EXTI15_10_IRQn);
    NVIC_EnableIRQ(ADC1_IRQn);
}

// 優先權分組說明
void vExplainPriorityGrouping(void) {
    printf("優先權群組 4：4 位元用於搶佔，0 位元用於子優先權\n");
    printf("優先權群組 3：3 位元用於搶佔，1 位元用於子優先權\n");
    printf("優先權群組 2：2 位元用於搶佔，2 位元用於子優先權\n");
    printf("優先權群組 1：1 位元用於搶佔，3 位元用於子優先權\n");
    printf("優先權群組 0：0 位元用於搶佔，4 位元用於子優先權\n");
}
```

**動態優先權管理：**
```c
// 動態中斷優先權管理
void vDynamicPriorityManagement(void) {
    // 儲存原始優先權
    uint32_t original_timer_priority = NVIC_GetPriority(TIM2_IRQn);
    uint32_t original_uart_priority = NVIC_GetPriority(USART1_IRQn);
    
    // 根據系統狀態調整優先權
    if (system_under_high_load()) {
        // 提高計時器優先權以獲得更好的時序
        NVIC_SetPriority(TIM2_IRQn, 
                        NVIC_EncodePriority(NVIC_PriorityGroup_4, 0, 0));
        
        // 降低 UART 優先權以減少額外負擔
        NVIC_SetPriority(USART1_IRQn, 
                        NVIC_EncodePriority(NVIC_PriorityGroup_4, 3, 0));
    } else {
        // 恢復原始優先權
        NVIC_SetPriority(TIM2_IRQn, original_timer_priority);
        NVIC_SetPriority(USART1_IRQn, original_uart_priority);
    }
}
```

### **優先權反轉預防**

**中斷優先權天花板：**
```c
// 中斷優先權天花板實作
typedef struct {
    uint32_t base_priority;
    uint32_t ceiling_priority;
    bool is_active;
} interrupt_priority_ceiling_t;

interrupt_priority_ceiling_t timer_ceiling = {
    .base_priority = 1,
    .ceiling_priority = 0,
    .is_active = false
};

// 將中斷優先權提升至天花板
void vRaiseInterruptPriority(interrupt_priority_ceiling_t *ceiling) {
    if (!ceiling->is_active) {
        ceiling->is_active = true;
        
        // 儲存當前優先權
        uint32_t current_priority = NVIC_GetPriority(TIM2_IRQn);
        
        // 提升至天花板優先權
        NVIC_SetPriority(TIM2_IRQn, ceiling->ceiling_priority);
    }
}

// 恢復中斷優先權
void vRestoreInterruptPriority(interrupt_priority_ceiling_t *ceiling) {
    if (ceiling->is_active) {
        ceiling->is_active = false;
        
        // 恢復基本優先權
        NVIC_SetPriority(TIM2_IRQn, ceiling->base_priority);
    }
}
```

---

## ⏱️ **中斷延遲分析**

### **延遲量測**

**中斷延遲量測：**
```c
// 使用 GPIO 量測中斷延遲
volatile uint32_t interrupt_entry_time = 0;
volatile uint32_t interrupt_latency = 0;

void EXTI0_IRQHandler(void) {
    // 記錄進入時間
    interrupt_entry_time = DWT->CYCCNT;
    
    // 清除中斷旗標
    EXTI_ClearITPendingBit(EXTI_Line0);
    
    // 切換 GPIO 用於量測
    GPIO_SetBits(GPIOA, GPIO_Pin_0);
    
    // 模擬 ISR 工作
    volatile uint32_t i;
    for (i = 0; i < 1000; i++);
    
    // 切換 GPIO 回原狀態
    GPIO_ResetBits(GPIOA, GPIO_Pin_0);
    
    // 計算延遲
    interrupt_latency = DWT->CYCCNT - interrupt_entry_time;
}

// 分析中斷延遲的任務
void vInterruptLatencyAnalyzer(void *pvParameters) {
    uint32_t max_latency = 0;
    uint32_t min_latency = UINT32_MAX;
    uint32_t total_latency = 0;
    uint32_t sample_count = 0;
    
    while (1) {
        if (interrupt_latency > 0) {
            // 更新統計資料
            if (interrupt_latency > max_latency) {
                max_latency = interrupt_latency;
            }
            if (interrupt_latency < min_latency) {
                min_latency = interrupt_latency;
            }
            
            total_latency += interrupt_latency;
            sample_count++;
            
            // 列印統計資料
            printf("延遲 - 最大: %lu, 最小: %lu, 平均: %lu, 樣本數: %lu\n",
                   max_latency, min_latency, total_latency / sample_count, sample_count);
            
            // 重置以進行下次量測
            interrupt_latency = 0;
        }
        
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}
```

**延遲預算分析：**
```c
// 中斷延遲預算分析
typedef struct {
    uint32_t max_allowed_latency;
    uint32_t measured_latency;
    uint32_t margin;
    bool within_budget;
} latency_budget_t;

latency_budget_t timer_latency_budget = {
    .max_allowed_latency = 1000,  // 1000 個週期
    .measured_latency = 0,
    .margin = 0,
    .within_budget = true
};

void vAnalyzeLatencyBudget(latency_budget_t *budget) {
    // 計算餘裕
    budget->margin = budget->max_allowed_latency - budget->measured_latency;
    
    // 檢查是否在預算內
    budget->within_budget = (budget->measured_latency <= budget->max_allowed_latency);
    
    // 列印分析結果
    printf("延遲預算分析：\n");
    printf("  最大允許量：%lu 個週期\n", budget->max_allowed_latency);
    printf("  量測值：%lu 個週期\n", budget->measured_latency);
    printf("  餘裕：%lu 個週期\n", budget->margin);
    printf("  在預算內：%s\n", budget->within_budget ? "是" : "否");
    
    if (!budget->within_budget) {
        printf("  警告：延遲超出預算！\n");
    }
}
```

### **抖動分析**

**中斷抖動量測：**
```c
// 中斷抖動量測
typedef struct {
    uint32_t last_interrupt_time;
    uint32_t jitter_samples[100];
    uint8_t sample_index;
    uint32_t total_jitter;
    uint32_t max_jitter;
} jitter_measurement_t;

jitter_measurement_t timer_jitter = {0};

void TIM2_IRQHandler(void) {
    uint32_t current_time = DWT->CYCCNT;
    
    if (timer_jitter.last_interrupt_time > 0) {
        // 計算抖動
        uint32_t expected_interval = 16000;  // 在 16MHz 時為 1ms
        uint32_t actual_interval = current_time - timer_jitter.last_interrupt_time;
        uint32_t jitter = abs((int32_t)actual_interval - (int32_t)expected_interval);
        
        // 儲存抖動樣本
        timer_jitter.jitter_samples[timer_jitter.sample_index] = jitter;
        timer_jitter.sample_index = (timer_jitter.sample_index + 1) % 100;
        
        // 更新統計資料
        if (jitter > timer_jitter.max_jitter) {
            timer_jitter.max_jitter = jitter;
        }
        
        timer_jitter.total_jitter += jitter;
    }
    
    timer_jitter.last_interrupt_time = current_time;
    
    // 清除中斷旗標
    TIM_ClearITPendingBit(TIM2, TIM_IT_Update);
}

// 分析抖動的任務
void vJitterAnalyzer(void *pvParameters) {
    while (1) {
        // 計算平均抖動
        uint32_t total = 0;
        for (int i = 0; i < 100; i++) {
            total += timer_jitter.jitter_samples[i];
        }
        uint32_t avg_jitter = total / 100;
        
        printf("抖動分析 - 最大: %lu, 平均: %lu 個週期\n",
               timer_jitter.max_jitter, avg_jitter);
        
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}
```

---

## ⚙️ **FreeRTOS 中斷處理**

### **FreeRTOS 中斷設定**

**基本設定：**
```c
// FreeRTOS 中斷設定
#define configMAX_SYSCALL_INTERRUPT_PRIORITY    191
#define configKERNEL_INTERRUPT_PRIORITY         255
#define configMAX_API_CALL_INTERRUPT_PRIORITY   191

// 中斷安全的 FreeRTOS 函式
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
```

**中斷安全函式：**
```c
// 中斷安全的 FreeRTOS 函式列表
void vListInterruptSafeFunctions(void) {
    printf("中斷安全的 FreeRTOS 函式：\n");
    printf("  - xTaskNotifyFromISR()\n");
    printf("  - xTaskNotifyGiveFromISR()\n");
    printf("  - xQueueSendFromISR()\n");
    printf("  - xQueueReceiveFromISR()\n");
    printf("  - xSemaphoreGiveFromISR()\n");
    printf("  - xSemaphoreTakeFromISR()\n");
    printf("  - xEventGroupSetBitsFromISR()\n");
    printf("  - xTimerPendFunctionCallFromISR()\n");
    printf("  - portYIELD_FROM_ISR()\n");
    printf("  - xTaskGetTickCountFromISR()\n");
}
```

### **FreeRTOS 中斷掛鉤**

**中斷掛鉤函式：**
```c
// FreeRTOS 中斷掛鉤
void vApplicationTickHook(void) {
    // 每個 tick 從中斷上下文呼叫
    static uint32_t tick_count = 0;
    tick_count++;
    
    // 執行週期性操作
    if (tick_count % 1000 == 0) {
        // 每 1000 個 tick
        system_heartbeat();
    }
}

void vApplicationIdleHook(void) {
    // 閒置任務執行時呼叫
    // 可用於電源管理
    if (system_can_sleep()) {
        // 進入低功耗模式
        __WFI();  // 等待中斷
    }
}

void vApplicationMallocFailedHook(void) {
    // 記憶體配置失敗時呼叫
    printf("中斷上下文中記憶體配置失敗！\n");
    
    // 處理記憶體配置失敗
    // 可以重新啟動系統或釋放記憶體
}

void vApplicationStackOverflowHook(TaskHandle_t xTask, char *pcTaskName) {
    // 偵測到堆疊溢位時呼叫
    printf("任務堆疊溢位: %s\n", pcTaskName);
    
    // 處理堆疊溢位
    // 可以重新啟動系統或任務
}
```

---

## 🚀 **實作**

### **完整中斷系統**

**系統初始化：**
```c
// 完整中斷系統初始化
void vInitializeInterruptSystem(void) {
    // 啟用 DWT 以進行時序量測
    CoreDebug->DEMCR |= CoreDebug_DEMCR_TRCENA_Msk;
    DWT->CTRL |= DWT_CTRL_CYCCNTENA_Msk;
    
    // 設定 GPIO 用於中斷產生
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_2MHz;
    GPIO_Init(GPIOA, &GPIO_InitStructure);
    
    // 設定外部中斷
    EXTI_InitTypeDef EXTI_InitStructure;
    EXTI_InitStructure.EXTI_Line = EXTI_Line0;
    EXTI_InitStructure.EXTI_Mode = EXTI_Mode_Interrupt;
    EXTI_InitStructure.EXTI_Trigger = EXTI_Trigger_Rising;
    EXTI_InitStructure.EXTI_LineCmd = ENABLE;
    EXTI_Init(&EXTI_InitStructure);
    
    // 設定計時器中斷
    TIM_TimeBaseInitTypeDef TIM_TimeBaseStructure;
    TIM_TimeBaseStructure.TIM_Period = 15999;  // 在 16MHz 時為 1ms
    TIM_TimeBaseStructure.TIM_Prescaler = 0;
    TIM_TimeBaseStructure.TIM_ClockDivision = 0;
    TIM_TimeBaseStructure.TIM_CounterMode = TIM_CounterMode_Up;
    TIM_TimeBaseInit(TIM2, &TIM_TimeBaseStructure);
    
    // 啟用計時器中斷
    TIM_ITConfig(TIM2, TIM_IT_Update, ENABLE);
    
    // 設定中斷優先權
    vConfigureInterruptPriorities();
    
    // 啟用中斷
    __enable_irq();
    
    // 啟動計時器
    TIM_Cmd(TIM2, ENABLE);
}

// 主函式
int main(void) {
    // 硬體初始化
    SystemInit();
    HAL_Init();
    
    // 初始化周邊裝置
    MX_GPIO_Init();
    MX_TIM2_Init();
    
    // 初始化中斷系統
    vInitializeInterruptSystem();
    
    // 建立 FreeRTOS 任務
    xTaskCreate(vInterruptLatencyAnalyzer, "Latency", 256, NULL, 2, NULL);
    xTaskCreate(vJitterAnalyzer, "Jitter", 256, NULL, 1, NULL);
    
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

### **中斷設計問題**

**常見問題：**
- **過長的 ISR**：ISR 執行時間過長
- **阻塞操作**：在 ISR 中執行可能阻塞的操作
- **優先權衝突**：不正確的中斷優先權分配
- **資源競爭**：ISR 與任務之間的衝突

**解決方案：**
- **保持 ISR 簡短**：最小化 ISR 執行時間
- **使用任務通訊**：與任務通訊而不是在 ISR 中執行工作
- **正確的優先權分配**：根據系統需求分配優先權
- **資源保護**：保護共享資源免受衝突

### **時序問題**

**時序問題：**
- **中斷延遲**：過長的中斷回應時間
- **抖動**：不可預測的中斷時序
- **遺失中斷**：未被處理的中斷
- **優先權反轉**：低優先權中斷阻塞高優先權中斷

**解決方案：**
- **最佳化 ISR**：設計高效的中斷服務常式
- **使用硬體功能**：善用硬體時序功能
- **正確的優先權管理**：正確管理中斷優先權
- **延遲分析**：分析並最佳化中斷延遲

### **記憶體問題**

**記憶體問題：**
- **堆疊溢位**：ISR 的堆疊空間不足
- **記憶體碎片化**：ISR 配置造成的記憶體碎片化
- **緩衝區溢位**：中斷資料的緩衝區空間不足
- **記憶體洩漏**：中斷處理後未釋放記憶體

**解決方案：**
- **充足的堆疊大小**：為 ISR 配置足夠的堆疊空間
- **靜態配置**：盡可能使用靜態配置
- **緩衝區管理**：正確調整和管理中斷緩衝區大小
- **記憶體清理**：確保中斷後正確清理記憶體

---

## ✅ **最佳實踐**

### **中斷設計原則**

**ISR 設計：**
- **最小執行量**：保持 ISR 執行時間最短
- **無阻塞**：避免在 ISR 中執行阻塞操作
- **高效通訊**：使用高效的方式與任務通訊
- **錯誤處理**：在 ISR 中實作適當的錯誤處理

**優先權管理：**
- **明確的優先權階層**：建立明確的中斷優先權等級
- **一致的分配**：使用一致的優先權分配策略
- **文件記錄**：記錄優先權分配的理由
- **審查與更新**：定期審查和更新優先權

### **效能最佳化**

**延遲最佳化：**
- **最小化上下文保存**：最佳化上下文保存和恢復操作
- **高效 ISR**：設計高效的中斷服務常式
- **硬體最佳化**：使用硬體功能來減少延遲
- **中斷合併**：在可能時合併多個中斷

**資源管理：**
- **高效配置**：最小化資源配置的額外負擔
- **資源共享**：使用適當的同步機制
- **清理**：中斷處理後正確清理資源
- **監控**：監控資源使用量和可用性

---

## 🔬 **引導式實驗**

### **實驗 1：基本中斷設定**
**目標**：設定一個簡單的 GPIO 中斷系統
**步驟**：
1. 將 GPIO 腳位設定為帶上拉電阻的輸入
2. 啟用下降沿外部中斷
3. 編寫一個切換 LED 的最小 ISR
4. 使用示波器量測中斷延遲

**預期結果**：LED 在按鈕按下後數微秒內切換

### **實驗 2：中斷優先權實驗**
**目標**：了解中斷優先權和巢狀
**步驟**：
1. 設定兩個具有不同優先權的計時器中斷
2. 設定高優先權計時器中斷低優先權計時器
3. 使用 GPIO 視覺化中斷巢狀
4. 量測最壞情況的中斷延遲

**預期結果**：高優先權中斷可以搶佔低優先權中斷

### **實驗 3：ISR 到任務的通訊**
**目標**：學習 ISR 與任務之間的正確通訊方式
**步驟**：
1. 建立一個等待號誌的任務
2. 設定 UART 中斷來釋放號誌
3. 任務處理接收到的資料
4. 量測端到端延遲

**預期結果**：資料在可預測的時間範圍內被處理

---

## ✅ **自我檢核**

### **理解檢核**
- [ ] 你能解釋為什麼中斷對偶發性事件比輪詢更好嗎？
- [ ] 你了解中斷優先權和任務優先權之間的差異嗎？
- [ ] 你能識別哪些操作在 ISR 中是安全的嗎？
- [ ] 你知道如何量測中斷延遲嗎？

### **實務技能檢核**
- [ ] 你能在微控制器上設定基本的中斷系統嗎？
- [ ] 你知道如何除錯中斷時序問題嗎？
- [ ] 你能實作適當的 ISR 到任務通訊嗎？
- [ ] 你了解中斷優先權管理嗎？

### **進階概念檢核**
- [ ] 你能解釋中斷合併以及何時使用它嗎？
- [ ] 你了解中斷優先權反轉嗎？
- [ ] 你能最佳化中斷效能嗎？
- [ ] 你知道如何處理巢狀中斷嗎？

---

## 🔗 **交叉連結**

### **相關主題**
- **[FreeRTOS 基礎](./FreeRTOS_Basics.md)** - 了解 RTOS 上下文
- **[任務建立與管理](./Task_Creation_Management.md)** - 任務如何與中斷互動
- **[排程演算法](./Scheduling_Algorithms.md)** - 中斷如何影響排程
- **[即時除錯](./Real_Time_Debugging.md)** - 除錯中斷問題

### **先備知識**
- **[GPIO 設定](../Hardware_Fundamentals/GPIO_Configuration.md)** - 基本 I/O 設定
- **[計時器/計數器程式設計](../Hardware_Fundamentals/Timer_Counter_Programming.md)** - 計時器中斷
- **[外部中斷](../Hardware_Fundamentals/External_Interrupts.md)** - 硬體中斷設定

### **下一步**
- **[核心服務](./Kernel_Services.md)** - 在 ISR 中使用 RTOS 服務
- **[效能監控](./Performance_Monitoring.md)** - 量測中斷效能
- **[記憶體保護](./Memory_Protection.md)** - 在 ISR 中保護記憶體

---

## 📋 **快速參考：重要事實**

### **中斷基礎**
- **定義**：暫時中止正常執行以處理緊急事件的訊號
- **類型**：硬體（外部）、軟體（系統呼叫）、內部（例外）
- **特性**：非同步、基於優先權、可巢狀、保存上下文
- **優點**：對偶發性事件比輪詢更有效率

### **中斷服務常式 (ISR)**
- **目的**：快速且高效地處理中斷事件
- **約束條件**：必須快速、無阻塞、操作最少
- **通訊**：使用 FromISR API 與任務通訊
- **返回**：必須快速返回以最小化延遲

### **優先權管理**
- **階層**：較高優先權的中斷可以搶佔較低優先權的中斷
- **分配**：根據系統關鍵程度和時序需求決定
- **巢狀**：考慮中斷巢狀深度和堆疊使用量
- **反轉**：使用優先權繼承或天花板協定

### **效能考量**
- **延遲**：從中斷到 ISR 開始執行的時間
- **抖動**：中斷回應時間的變異
- **最佳化**：最小化上下文保存/恢復，使用高效 ISR
- **量測**：使用 GPIO、示波器或效能計數器

---

## ❓ **面試問題**

### **基本概念**

1. **中斷和輪詢有什麼差別？**
   - 中斷：系統在事件發生時回應
   - 輪詢：系統持續檢查事件
   - 中斷對偶發性事件更有效率
   - 中斷提供更好的即時回應

2. **如何決定中斷優先權？**
   - 根據系統關鍵程度和時序需求
   - 頻率較高的中斷通常獲得較高優先權
   - 關鍵系統功能獲得最高優先權
   - 考慮中斷巢狀和系統穩定性

3. **中斷延遲的組成部分有哪些？**
   - 硬體延遲：硬體產生中斷所需的時間
   - CPU 延遲：CPU 回應中斷所需的時間
   - 上下文保存：保存 CPU 上下文所需的時間
   - ISR 執行：執行中斷服務常式所需的時間

### **進階主題**

1. **說明如何處理中斷優先權反轉。**
   - 使用優先權繼承或優先權天花板
   - 實作中斷優先權管理
   - 以一致的順序處理中斷
   - 使用逾時機制

2. **如何最佳化中斷效能？**
   - 最小化 ISR 執行時間
   - 使用高效的方式與任務通訊
   - 最佳化上下文保存和恢復
   - 在可用時使用硬體功能

3. **你使用哪些策略來除錯中斷？**
   - 使用 GPIO 進行時序量測
   - 實作中斷掛鉤和監控
   - 分析中斷延遲和抖動
   - 使用除錯工具和示波器

### **實務情境**

1. **為即時控制應用設計一個中斷系統。**
   - 定義中斷來源和優先權
   - 為不同中斷類型設計 ISR
   - 實作任務通訊機制
   - 處理時序和效能需求

2. **你如何除錯中斷時序問題？**
   - 量測中斷延遲和抖動
   - 分析中斷優先權衝突
   - 檢查 ISR 中的阻塞操作
   - 使用硬體除錯工具

3. **說明如何實作中斷合併。**
   - 將多個中斷合併為單一事件
   - 使用計時器進行中斷批次處理
   - 實作中斷過濾機制
   - 平衡延遲和效率

本增強版中斷處理文件現在提供了概念說明、實務見解和技術實作細節之間的全面平衡，讓嵌入式工程師可以用來了解和實作 RTOS 環境中穩健的中斷處理系統。
