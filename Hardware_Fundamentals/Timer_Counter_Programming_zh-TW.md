# ⏱️ 計時器/計數器程式設計

> **掌握嵌入式系統的計時器與計數器操作**  
> 輸入捕獲、輸出比較、頻率量測和計時應用

---

## 📋 **目錄**

- [概述](#概述)
- [快速參考：重點摘要](#快速參考重點摘要)
- [視覺化理解](#視覺化理解)
- [概念基礎](#概念基礎)
- [核心概念](#核心概念)
- [實務考量](#實務考量)
- [額外資源](#額外資源)

---

## 🎯 **概述**

計時器和計數器是嵌入式系統中不可或缺的周邊設備，用於精確計時、頻率量測、PWM 產生和事件計數。了解計時器程式設計對即時應用至關重要。

---

## 🚀 **快速參考：重點摘要**

- **計時器模式**：向上計數、向下計數、中央對齊
- **輸入捕獲**：邊緣偵測、頻率量測、脈衝寬度量測
- **輸出比較**：PWM 產生、計時控制、波形產生
- **預分頻器**：分頻時脈頻率以達到所需的計時解析度
- **自動重裝載**：自動重新載入計數器值以實現連續操作
- **中斷**：計時器中斷用於精確的計時事件
- **DMA 整合**：無需 CPU 介入的高速資料傳輸

### **面試官意圖（他們在探詢什麼）**
- 你能否從時脈和預分頻器計算計時器頻率/週期？
- 你是否了解捕獲與比較的使用場景？
- 你能否推理抖動、ISR 開銷和 DMA 權衡？

---

## 🔍 **視覺化理解**

### **計時器方塊圖**
```
時脈源 → 預分頻器 → 計數器 → 比較/輸出 → 中斷/DMA
     ↓            ↓         ↓           ↓              ↓
系統時脈      除以       向上/向下/   產生         觸發
(84 MHz)    PSC+1     中央計數    PWM/事件    CPU 事件
```

### **計時器模式視覺化**
```
向上計數：    0 → 1 → 2 → ... → ARR → 0 → 1 → ...
向下計數：    ARR → ARR-1 → ... → 1 → 0 → ARR → ...
中央對齊：    0 → 1 → ... → ARR → ARR-1 → ... → 1 → 0 → ...
```

### **輸入捕獲 vs 輸出比較**
```
輸入捕獲：  外部事件 → 捕獲計時器值 → 計算時序
                     ↓                ↓                ↓
                GPIO 邊緣         儲存計數值      頻率/脈衝
                                 至暫存器        寬度

輸出比較：  計時器計數 → 與 CCR 比較 → 產生輸出事件
                     ↓            ↓              ↓
                遞增          匹配值        PWM/中斷
                計數器      (CCR 暫存器)    產生
```

---

## 🧠 **概念基礎**

### **計時器作為時間基礎**
計時器作為嵌入式系統中所有時間相關操作的基本建構模組。它們提供：
- **精確計時**：基於硬體的計時，抖動最小
- **事件排程**：可預測的週期性任務執行
- **量測功能**：精確的頻率和時間間隔量測
- **波形產生**：建立精確的時序模式和 PWM 訊號

### **為什麼計時器程式設計重要**
計時器程式設計至關重要，因為：
- **即時需求**：許多嵌入式應用需要精確的計時
- **系統同步**：協調多個系統事件和周邊設備
- **功耗效率**：計時器啟用睡眠模式和喚醒計時
- **效能最佳化**：硬體計時器將計時任務從 CPU 卸載

### **計時器設計挑戰**
設計計時器系統涉及平衡多個相互競爭的關注點：
- **解析度 vs 範圍**：較高的解析度（較小的預分頻器）減少最大週期
- **精確度 vs 複雜度**：更精確的計時需要細心的配置
- **硬體 vs 軟體**：硬體計時器 vs 軟體計時迴圈
- **中斷頻率**：平衡計時精度與系統開銷

---

## 🎯 **核心概念**

### **概念：計時器配置與頻率計算**

**重要性**：正確的計時器配置確保精確的計時並防止溢位錯誤。了解時脈頻率、預分頻器和週期之間的關係對於實現所需的計時解析度和範圍至關重要。

**最小範例**
```c
// 基本計時器配置結構
typedef struct {
    uint32_t prescaler;      // 計時器預分頻器值
    uint32_t period;         // 計時器週期（ARR 值）
    uint32_t clock_freq;     // 計時器時脈頻率
    uint32_t mode;           // 計時器模式（UP、DOWN、CENTER）
} timer_config_t;

// 計算計時器頻率
uint32_t calculate_timer_frequency(uint32_t clock_freq, uint32_t prescaler, uint32_t period) {
    return clock_freq / ((prescaler + 1) * (period + 1));
}

// 計算目標頻率的計時器週期
uint32_t calculate_timer_period(uint32_t clock_freq, uint32_t prescaler, uint32_t target_freq) {
    return (clock_freq / ((prescaler + 1) * target_freq)) - 1;
}
```

**試試看**：使用 84MHz 時脈計算 1kHz 計時器所需的預分頻器和週期值。

**重點摘要**
- 預分頻器分頻輸入時脈頻率
- 週期決定計時器溢位頻率
- 較高的預分頻器值提供較長的計時週期
- 始終驗證計算以防止溢位

### **概念：用於精確時序量測的輸入捕獲**

**重要性**：輸入捕獲能夠精確量測外部事件，使其對嵌入式系統中的頻率量測、脈衝寬度分析和事件計時至關重要。

**最小範例**
```c
// 輸入捕獲配置
typedef struct {
    uint32_t channel;        // 計時器通道（1-4）
    uint32_t edge;           // 上升/下降沿
    uint32_t filter;         // 輸入濾波器值
    bool interrupt_enable;   // 啟用捕獲中斷
} input_capture_config_t;

// 配置輸入捕獲
void configure_input_capture(TIM_HandleTypeDef* htim, input_capture_config_t* config) {
    // 將通道配置為輸入
    TIM_IC_InitTypeDef sConfigIC = {0};
    sConfigIC.ICPolarity = config->edge;
    sConfigIC.ICSelection = TIM_ICSELECTION_DIRECTTI;
    sConfigIC.ICPrescaler = TIM_ICPSC_DIV1;
    sConfigIC.ICFilter = config->filter;
    
    HAL_TIM_IC_ConfigChannel(htim, &sConfigIC, config->channel);
    
    // 如果需要則啟用中斷
    if (config->interrupt_enable) {
        __HAL_TIM_ENABLE_IT(htim, TIM_IT_CC1 << (config->channel - 1));
    }
}
```

**試試看**：實作輸入捕獲來量測外部訊號的頻率。

**重點摘要**
- 輸入捕獲在外部事件發生時儲存計時器值
- 邊緣選擇會影響量測精確度
- 輸入濾波降低雜訊敏感度
- 中斷啟用即時事件處理

### **概念：用於精確事件產生的輸出比較**

**重要性**：輸出比較能夠精確產生計時事件、PWM 訊號和週期性輸出。它是馬達控制、音訊產生和計時同步的基礎。

**最小範例**
```c
// 輸出比較配置
typedef struct {
    uint32_t channel;        // 計時器通道（1-4）
    uint32_t compare_value;  // 比較值（CCR）
    uint32_t mode;           // 輸出模式（切換、PWM、強制）
    bool interrupt_enable;   // 啟用比較中斷
} output_compare_config_t;

// 配置輸出比較
void configure_output_compare(TIM_HandleTypeDef* htim, output_compare_config_t* config) {
    // 將通道配置為輸出
    TIM_OC_InitTypeDef sConfigOC = {0};
    sConfigOC.OCMode = config->mode;
    sConfigOC.Pulse = config->compare_value;
    sConfigOC.OCPolarity = TIM_OCPOLARITY_HIGH;
    sConfigOC.OCFastMode = TIM_OCFAST_DISABLE;
    
    HAL_TIM_OC_ConfigChannel(htim, &sConfigOC, config->channel);
    
    // 如果需要則啟用中斷
    if (config->interrupt_enable) {
        __HAL_TIM_ENABLE_IT(htim, TIM_IT_CC1 << (config->channel - 1));
    }
}
```

**試試看**：使用輸出比較產生 50% 佔空比的 1kHz 方波。

**重點摘要**
- 輸出比較在計時器匹配比較值時產生事件
- 比較值決定輸出事件的時序
- 多通道可啟用複雜的計時模式
- 中斷提供精確的事件通知

### **概念：計時器中斷與 DMA 整合**

**重要性**：計時器中斷與 DMA 整合能夠高效處理計時事件和高速資料傳輸，降低 CPU 開銷並提升系統效能。

**最小範例**
```c
// 計時器中斷配置
void configure_timer_interrupt(TIM_HandleTypeDef* htim, uint32_t priority) {
    // 啟用計時器更新中斷
    __HAL_TIM_ENABLE_IT(htim, TIM_IT_UPDATE);
    
    // 配置 NVIC 優先級
    HAL_NVIC_SetPriority(TIM2_IRQn, priority, 0);
    HAL_NVIC_EnableIRQ(TIM2_IRQn);
}

// 計時器中斷處理器
void TIM2_IRQHandler(void) {
    if (__HAL_TIM_GET_FLAG(&htim2, TIM_FLAG_UPDATE) != RESET) {
        if (__HAL_TIM_GET_IT_SOURCE(&htim2, TIM_IT_UPDATE) != RESET) {
            __HAL_TIM_CLEAR_IT(&htim2, TIM_IT_UPDATE);
            
            // 處理計時器事件
            handle_timer_event();
        }
    }
}
```

**試試看**：實作一個每 500ms 切換 LED 的計時器中斷。

**重點摘要**
- 計時器中斷為事件處理提供精確的計時
- 保持中斷處理器短小高效
- 對高速計時器應用使用 DMA
- 適當的優先級配置可防止計時衝突

---

## 🧪 **引導式實驗**

### **實驗 1：基本計時器配置與中斷**
**目標**：配置計時器以產生週期性中斷並量測計時精確度。

**步驟**：
1. 配置計時器為 1kHz 操作
2. 啟用計時器中斷
3. 在中斷處理器中切換 GPIO 腳位
4. 使用示波器量測計時精確度

**預期結果**：理解計時器配置和中斷計時。

### **實驗 2：用於頻率量測的輸入捕獲**
**目標**：使用輸入捕獲量測外部訊號的頻率。

**步驟**：
1. 配置計時器通道用於輸入捕獲
2. 使用函數產生器產生測試訊號
3. 實作頻率計算演算法
4. 比較量測值與實際頻率

**預期結果**：獲得輸入捕獲和頻率量測的實務經驗。

### **實驗 3：用於 PWM 產生的輸出比較**
**目標**：使用輸出比較產生可變佔空比的 PWM 訊號。

**步驟**：
1. 配置計時器為 PWM 模式
2. 實作佔空比控制
3. 產生不同的 PWM 頻率
4. 使用示波器量測 PWM 特性

**預期結果**：理解 PWM 產生和輸出比較操作。

---

## ✅ **自我檢查**

### **基本理解**
- 計時器模式和計數器模式之間的區別是什麼？
- 如何從時脈、預分頻器和週期計算計時器頻率？
- 輸入捕獲和輸出比較的主要應用是什麼？

### **實務應用**
- 如何配置具有 1ms 解析度的 100Hz 計時器？
- 選擇計時器預分頻器值時有哪些重要考量？
- 如何使用計時器實作精確的時序量測？

### **進階概念**
- 如何在長時間量測中處理計時器溢位？
- 硬體和軟體計時之間的權衡是什麼？
- 如何為複雜應用同步多個計時器？

---

## 🔗 **交叉連結**

- **[GPIO 配置](./GPIO_Configuration.md)** - GPIO 模式、配置、電氣特性
- **[脈寬調變](./Pulse_Width_Modulation.md)** - PWM 產生、頻率控制、佔空比
- **[外部中斷](./External_Interrupts.md)** - 邊緣/準位觸發中斷、消彈
- **[中斷與例外](./Interrupts_Exceptions.md)** - 中斷處理和 ISR 設計
- **[硬體抽象層](./Hardware_Abstraction_Layer.md)** - 計時器抽象和可攜性

---

## 🎯 **實務考量**

### **系統級設計決策**
- **計時器選擇**：根據解析度和範圍需求選擇適當的計時器
- **中斷頻率**：平衡計時精度與系統開銷
- **資源分配**：考慮多個應用之間的計時器共享

### **效能與最佳化**
- **預分頻器選擇**：最佳化所需的計時解析度和範圍
- **中斷效率**：最小化 ISR 執行時間
- **DMA 使用**：對高速計時器應用使用 DMA

### **除錯與測試**
- **計時驗證**：使用示波器或邏輯分析儀驗證計時
- **中斷除錯**：監控中斷計時和頻率
- **效能分析**：量測計時器精確度和抖動

---

## 📚 **額外資源**

### **文件**
- [STM32 計時器參考手冊](https://www.st.com/resource/en/reference_manual/dm00031020-stm32f405-415-stm32f407-417-stm32f427-437-and-stm32f429-439-advanced-arm-based-32-bit-mcus-stmicroelectronics.pdf)
- [ARM Cortex-M 計時器程式設計](https://developer.arm.com/documentation/dui0552/a/the-cortex-m3-processor/peripherals/general-purpose-timers)

### **工具**
- [STM32CubeMX](https://www.st.com/en/development-tools/stm32cubemx.html) - 計時器配置
- [計時器計算器](https://www.st.com/resource/en/user_manual/dm00104712-stm32cubemx-user-manual-stmicroelectronics.pdf)

### **書籍**
- "Embedded Systems: Introduction to ARM Cortex-M Microcontrollers" by Jonathan Valvano
- "Making Embedded Systems" by Elecia White

---

**下一個主題：** [看門狗計時器](./Watchdog_Timers.md) → [中斷與例外](./Interrupts_Exceptions.md)
