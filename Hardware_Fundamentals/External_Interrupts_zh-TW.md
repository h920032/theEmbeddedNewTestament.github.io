# 🔌 外部中斷

## 快速參考：重點摘要

- **外部中斷** 允許嵌入式系統在不需要輪詢的情況下立即回應外部事件
- **邊緣觸發中斷** 在訊號轉換（上升/下降沿）時觸發，屬於事件驅動型
- **準位觸發中斷** 在訊號保持在特定準位時觸發，需要手動清除
- **中斷優先級** 決定多個中斷同時發生時的優先順序
- **中斷延遲** 是從中斷發生到處理器開始執行的時間，影響即時效能
- **消彈** 對機械開關至關重要，可消除接觸彈跳產生的假觸發
- **中斷服務常式（ISR）** 必須快速、高效且避免阻塞操作
- **中斷遮罩** 在臨界區段或長時間處理期間防止中斷重入

> **掌握外部中斷處理以建構回應式嵌入式系統**  
> 學習實作邊緣/準位觸發中斷、消彈技術和中斷驅動設計

---

## 📋 **目錄**

- [概述](#概述)
- [中斷類型](#中斷類型)
- [邊緣觸發 vs 準位觸發](#邊緣觸發-vs-準位觸發)
- [中斷配置](#中斷配置)
- [消彈技術](#消彈技術)
- [中斷服務常式](#中斷服務常式)
- [中斷延遲](#中斷延遲)
- [最佳實踐](#最佳實踐)
- [常見陷阱](#常見陷阱)
- [範例](#範例)
- [面試問題](#面試問題)

---

## 🎯 **概述**

外部中斷允許嵌入式系統在不需要輪詢的情況下立即回應外部事件。它們對即時應用、使用者介面和高效的系統設計至關重要。

### 概念：邊緣 vs 準位，以及清除中斷源

選擇邊緣偵測來捕獲轉換；準位偵測用於持續條件。始終適當清除中斷源，並考慮在長時間處理期間進行遮罩。

### **重點概念**
- **中斷向量表** - 將中斷源映射到處理器函式
- **中斷優先級** - 決定中斷的優先順序
- **中斷延遲** - 從中斷發生到處理器開始執行的時間
- **消彈** - 消除機械開關產生的假觸發

### **面試官意圖（他們在探詢什麼）**
- 你能否正確選擇邊緣或準位觸發並解釋原因？
- 你是否知道如何保持 ISR 短小且安全？
- 你能否推理延遲、優先級和遮罩？

---

## 🔍 視覺化理解

### **邊緣觸發 vs 準位觸發中斷**
```
邊緣觸發中斷
輸入訊號
   ^
   │    ┌─────────────────┐
   │    │                 │
   │    │                 │
   │    │                 │
   +──────────────────────────-> 時間
   ▲         ▼
上升      下降
 沿        沿

中斷觸發
   ^
   │    │         │
   │    │         │
   │    │         │
   +──────────────────────────-> 時間
   │<->│ 中斷    │<->│ 中斷
       │  觸發   │   │  觸發

準位觸發中斷
輸入訊號
   ^
   │    ┌─────────────────┐
   │    │                 │
   │    │                 │
   │    │                 │
   +──────────────────────────-> 時間
   │<->│ 高準位  │<->│ 低準位
       │  觸發   │   │  觸發
```

### **中斷處理流程**
```
中斷處理管線
┌─────────────────────────────────────────────────────────────┐
│                    外部事件                                 │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐   │
│  │   硬體      │───▶│ 中斷        │───▶│   CPU 核心  │   │
│  │   偵測      │    │ 控制器      │    │   回應      │   │
│  └─────────────┘    └─────────────┘    └─────────────┘   │
│         │                   │                   │         │
│         ▼                   ▼                   ▼         │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐   │
│  │   訊號      │    │   優先級    │    │   上下文    │   │
│  │   條件      │    │   解析      │    │   切換      │   │
│  └─────────────┘    └─────────────┘    └─────────────┘   │
│         │                   │                   │         │
│         ▼                   ▼                   ▼         │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐   │
│  │   ISR       │    │   返回      │    │   恢復      │   │
│  │   執行      │    │   到 ISR    │    │   主程式    │   │
│  │             │    │             │    │             │   │
│  └─────────────┘    └─────────────┘    └─────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### **中斷優先級與巢狀**
```
中斷優先級層級
┌─────────────────────────────────────────────────────────────┐
│                    優先級階層                               │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐         │
│  │   高        │ │   中        │ │    低       │         │
│  │ 優先級     │ │  優先級     │ │  優先級     │         │
│  │ (Level 0)   │ │ (Level 1)   │ │ (Level 2)   │         │
│  └─────────────┘ └─────────────┘ └─────────────┘         │
│         │               │               │                 │
│         ▼               ▼               ▼                 │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐         │
│  │   可中斷    │ │   可中斷    │ │   無法中斷  │         │
│  │   所有      │ │   較低      │ │   較高      │         │
│  │   層級      │ │   層級      │ │   層級      │         │
│  └─────────────┘ └─────────────┘ └─────────────┘         │
└─────────────────────────────────────────────────────────────┘
```

### **🧠 概念基礎**

#### **中斷驅動範式**
外部中斷代表從輪詢式到事件驅動系統設計的根本轉變。系統不再持續檢查外部條件，而是等待事件發生並在事件到來時立即回應。

**關鍵特性：**
- **事件驅動**：系統回應外部事件而非持續監控
- **即時回應**：對外部刺激的立即反應，無軟體延遲
- **高效資源使用**：CPU 可在等待事件時執行其他任務
- **確定性延遲**：對時間關鍵應用的可預測回應時間

#### **為什麼外部中斷重要**
外部中斷對現代嵌入式系統至關重要：

- **即時效能**：對外部事件的立即回應對安全和控制系統至關重要
- **功耗效率**：系統可在等待事件時進入睡眠，大幅降低功耗
- **使用者體驗**：回應式介面需要對使用者輸入的立即反應
- **系統可靠性**：中斷使系統能夠回應關鍵事件，如電源故障或安全條件

#### **中斷設計挑戰**
設計有效的中斷系統涉及平衡多個相互競爭的需求：

- **回應時間**：快速回應需要高效的 ISR 和適當的優先級管理
- **可靠性**：穩健操作必須處理雜訊、突波和多個同時事件
- **功耗效率**：中斷應啟用省電模式同時保持回應能力
- **系統複雜度**：中斷驅動系統的除錯和維護可能更加複雜

## 🔄 **中斷類型**

### **1. 邊緣觸發中斷**
在訊號從一個狀態轉換到另一個狀態時觸發。

```c
// 邊緣觸發中斷配置
typedef enum {
    RISING_EDGE,    // 0 → 1 轉換
    FALLING_EDGE,   // 1 → 0 轉換
    BOTH_EDGES      // 兩種轉換
} edge_type_t;

typedef struct {
    uint8_t pin;
    edge_type_t edge;
    uint8_t priority;
    void (*handler)(void);
} external_interrupt_config_t;
```

### **2. 準位觸發中斷**
在訊號保持在特定準位時觸發。

```c
// 準位觸發中斷配置
typedef enum {
    HIGH_LEVEL,     // 高準位時觸發
    LOW_LEVEL       // 低準位時觸發
} level_type_t;

typedef struct {
    uint8_t pin;
    level_type_t level;
    uint8_t priority;
    void (*handler)(void);
} level_interrupt_config_t;
```

---

## ⚡ **邊緣觸發 vs 準位觸發**

### **邊緣觸發優點**
- **事件驅動** - 捕獲轉換而無需準位輪詢
- **較低功耗** - 中斷自動清除
- **更適合高頻訊號** - 無持續觸發

### **邊緣觸發缺點**
- **易受雜訊影響** - 突波可能產生假觸發
- **需要消彈** - 機械開關需要濾波

### **準位觸發優點**
- **實作簡單** - 直接準位偵測
- **適合慢速訊號** - 處理期間不會錯過事件

### **準位觸發缺點**
- **持續觸發** - 必須清除源頭和/或遮罩中斷
- **較高功耗** - 中斷在清除前保持活躍
- **可能需要遮罩** - 避免在長時間處理期間重入

---

## ⚙️ **中斷配置**

### **1. GPIO 中斷設定**

```c
// 為外部中斷配置 GPIO
void configure_external_interrupt(uint8_t pin, edge_type_t edge) {
    // 啟用 GPIO 時脈
    RCC->AHB1ENR |= (1 << GPIO_PORT(pin));
    
    // 將 GPIO 配置為帶上拉的輸入
    GPIO_TypeDef *port = GPIO_BASE(pin);
    uint8_t pin_num = GPIO_PIN_NUM(pin);
    
    // 設為輸入
    port->MODER &= ~(3 << (pin_num * 2));
    
    // 根據電路板設計配置上拉或下拉電阻
    port->PUPDR &= ~(3 << (pin_num * 2));
    if (edge == FALLING_EDGE) {
        port->PUPDR |= (1 << (pin_num * 2)); // 上拉
    } else if (edge == RISING_EDGE) {
        port->PUPDR |= (2 << (pin_num * 2)); // 下拉
    }
    
    // 配置中斷觸發
    configure_interrupt_trigger(pin, edge);
    
    // 以適當的優先級在 NVIC 中啟用中斷
    enable_nvic_interrupt(EXTI_IRQn);
}
```

### **2. 中斷觸發配置**

```c
// 配置中斷觸發類型
void configure_interrupt_trigger(uint8_t pin, edge_type_t edge) {
    uint8_t pin_num = GPIO_PIN_NUM(pin);
    uint8_t exti_line = pin_num;
    
    // 啟用 SYSCFG 時脈
    RCC->APB2ENR |= RCC_APB2ENR_SYSCFGEN;
    
    // 將 GPIO 連接到 EXTI 線路
    SYSCFG->EXTICR[exti_line / 4] &= ~(0xF << ((exti_line % 4) * 4));
    SYSCFG->EXTICR[exti_line / 4] |= (GPIO_PORT(pin) << ((exti_line % 4) * 4));
    
    // 配置觸發類型
    switch (edge) {
        case RISING_EDGE:
            EXTI->RTSR |= (1 << exti_line);
            EXTI->FTSR &= ~(1 << exti_line);
            break;
        case FALLING_EDGE:
            EXTI->FTSR |= (1 << exti_line);
            EXTI->RTSR &= ~(1 << exti_line);
            break;
        case BOTH_EDGES:
            EXTI->RTSR |= (1 << exti_line);
            EXTI->FTSR |= (1 << exti_line);
            break;
    }
    
    // 啟用中斷
    EXTI->IMR |= (1 << exti_line);
}
```

---

## 🔄 **消彈技術**

### **1. 軟體消彈**

```c
// 使用計時器的軟體消彈
typedef struct {
    uint8_t pin;
    uint32_t last_trigger_time;
    uint32_t debounce_delay;
    bool last_state;
} debounce_config_t;

debounce_config_t debounce_configs[MAX_INTERRUPTS];

// 消彈後的中斷處理器
void debounced_interrupt_handler(uint8_t pin) {
    uint32_t current_time = get_system_tick();
    debounce_config_t *config = &debounce_configs[pin];
    
    // 檢查自上次觸發是否經過足夠時間
    if (current_time - config->last_trigger_time < config->debounce_delay) {
        return; // 忽略此次觸發
    }
    
    // 讀取當前腳位狀態
    bool current_state = read_gpio_pin(pin);
    
    // 僅在狀態確實改變時才處理
    if (current_state != config->last_state) {
        config->last_state = current_state;
        config->last_trigger_time = current_time;
        
        // 處理實際中斷
        process_interrupt_event(pin, current_state);
    }
}
```

### **2. 硬體消彈**

```c
// 硬體消彈電路分析
/*
    硬體消彈選項：
    
    1. RC 濾波器：
       開關 --[R]--+--[C]-- GND
                   |
                 輸入腳位
    
    2. 施密特觸發器：
       開關 --[R]-- 施密特觸發器 -- 輸入腳位
    
    3. 彈跳抑制 IC：
       開關 -- 彈跳抑制 IC -- 輸入腳位
*/

// 計算消彈的 RC 值
void calculate_debounce_values(uint32_t bounce_time_ms, uint32_t *r_value, uint32_t *c_value) {
    // 典型彈跳時間：1-50ms
    // RC 時間常數應 > bounce_time/3
    
    uint32_t rc_time = bounce_time_ms * 3; // 3 倍彈跳時間以確保安全
    
    // 選擇標準值
    *c_value = 0.1; // 0.1μF
    *r_value = (rc_time * 1000) / (*c_value * 1000); // R = t/(C*1000) 用於 μF
}
```

### **3. 進階消彈**

```c
// 狀態機消彈
typedef enum {
    DEBOUNCE_IDLE,
    DEBOUNCE_WAIT,
    DEBOUNCE_CONFIRMED
} debounce_state_t;

typedef struct {
    debounce_state_t state;
    uint32_t start_time;
    uint32_t debounce_time;
    bool stable_state;
} debounce_state_machine_t;

void debounce_state_machine(debounce_state_machine_t *sm, bool current_input) {
    uint32_t current_time = get_system_tick();
    
    switch (sm->state) {
        case DEBOUNCE_IDLE:
            if (current_input != sm->stable_state) {
                sm->state = DEBOUNCE_WAIT;
                sm->start_time = current_time;
            }
            break;
            
        case DEBOUNCE_WAIT:
            if (current_time - sm->start_time >= sm->debounce_time) {
                if (current_input != sm->stable_state) {
                    sm->stable_state = current_input;
                    sm->state = DEBOUNCE_CONFIRMED;
                    // 處理狀態變化
                    process_state_change(sm->stable_state);
                } else {
                    sm->state = DEBOUNCE_IDLE;
                }
            }
            break;
            
        case DEBOUNCE_CONFIRMED:
            sm->state = DEBOUNCE_IDLE;
            break;
    }
}
```

---

## 🎯 **中斷服務常式**

### **1. 基本 ISR 結構**

```c
// 外部中斷服務常式
void EXTI0_IRQHandler(void) {
    // 檢查中斷是否待處理
    if (EXTI->PR & (1 << 0)) {
        // 清除待處理位元
        EXTI->PR = (1 << 0);
        
        // 處理中斷
        process_external_interrupt(0);
    }
}

// 通用外部中斷處理器
void process_external_interrupt(uint8_t pin) {
    // 讀取腳位狀態
    bool pin_state = read_gpio_pin(pin);
    
    // 根據應用處理
    switch (pin) {
        case BUTTON_PIN:
            handle_button_press(pin_state);
            break;
        case SENSOR_PIN:
            handle_sensor_interrupt(pin_state);
            break;
        default:
            // 未知腳位
            break;
    }
}
```

### **2. ISR 最佳實踐**

```c
// 最小化處理的 ISR
void EXTI15_10_IRQHandler(void) {
    // 檢查哪些腳位觸發
    uint16_t pending = EXTI->PR & 0xFC00; // 腳位 10-15
    
    if (pending) {
        // 清除待處理位元
        EXTI->PR = pending;
        
        // 設定旗標供主迴圈處理
        for (int i = 10; i <= 15; i++) {
            if (pending & (1 << i)) {
                set_interrupt_flag(i);
            }
        }
    }
}

// 主迴圈處理中斷旗標
void main_loop(void) {
    while (1) {
        // 檢查待處理中斷
        for (int i = 0; i < MAX_INTERRUPTS; i++) {
            if (check_interrupt_flag(i)) {
                clear_interrupt_flag(i);
                process_interrupt_event(i);
            }
        }
        
        // 其他主迴圈任務
        process_system_tasks();
    }
}
```

---

## ⏱️ **中斷延遲**

### **1. 延遲組成**

```c
// 中斷延遲分析
typedef struct {
    uint32_t hardware_latency;    // 進入 ISR 的時間
    uint32_t software_latency;    // ISR 中的時間
    uint32_t context_switch;      // 儲存/還原上下文的時間
    uint32_t total_latency;       // 總回應時間
} interrupt_latency_t;

// 量測中斷延遲
void measure_interrupt_latency(void) {
    uint32_t start_time, end_time;
    
    // 配置測試中斷
    configure_test_interrupt();
    
    // 開始量測
    start_time = get_high_resolution_timer();
    
    // 觸發中斷
    trigger_test_interrupt();
    
    // 在 ISR 中量測
    end_time = get_high_resolution_timer();
    
    // 計算延遲
    uint32_t latency = end_time - start_time;
    
    // 報告結果
    printf("中斷延遲：%lu 個週期\n", latency);
}
```

### **2. 延遲最佳化**

```c
// 最小化延遲的最佳化 ISR
__attribute__((interrupt)) void optimized_isr(void) {
    // 直接使用暫存器（無函式呼叫）
    // 最小化堆疊使用
    // 避免複雜操作
    
    // 快速狀態檢查
    if (GPIOA->IDR & (1 << 0)) {
        // 立即設定旗標
        interrupt_flags |= (1 << 0);
    }
    
    // 清除中斷
    EXTI->PR = (1 << 0);
}
```

---

## 🎯 **最佳實踐**

### **1. ISR 設計指南**

```c
// ISR 設計檢查清單
/*
    □ 保持 ISR 短小快速
    □ 盡可能避免函式呼叫
    □ 對共享變數使用 volatile
    □ 儘早清除中斷旗標
    □ 不要使用阻塞操作
    □ 避免浮點運算
    □ 使用適當的中斷優先級
    □ 測試中斷時序
    □ 正確處理中斷巢狀
    □ 記錄中斷依賴關係
*/

// 良好的 ISR 範例
volatile uint32_t interrupt_counter = 0;

void good_isr_example(void) {
    // 立即清除中斷
    EXTI->PR = (1 << 0);
    
    // 簡單操作
    interrupt_counter++;
    
    // 設定旗標供主迴圈處理
    interrupt_pending = true;
}
```

### **2. 中斷優先級管理**

```c
// 配置中斷優先級
void configure_interrupt_priorities(void) {
    // 設定優先級分組
    NVIC_SetPriorityGrouping(3); // 4 位元用於搶占，0 用於子優先級
    
    // 配置優先級（較低數字 = 較高優先級）
    NVIC_SetPriority(EXTI0_IRQn, 1);      // 高優先級
    NVIC_SetPriority(EXTI1_IRQn, 2);      // 中優先級
    NVIC_SetPriority(EXTI2_IRQn, 3);      // 低優先級
    
    // 啟用中斷
    NVIC_EnableIRQ(EXTI0_IRQn);
    NVIC_EnableIRQ(EXTI1_IRQn);
    NVIC_EnableIRQ(EXTI2_IRQn);
}
```

---

## ⚠️ **常見陷阱**

### **1. 缺少中斷清除**

```c
// 錯誤：缺少中斷清除
void bad_isr_example(void) {
    // 處理中斷
    process_interrupt();
    // 缺少：EXTI->PR = (1 << pin);
}

// 正確：清除中斷旗標
void good_isr_example(void) {
    // 先清除中斷旗標
    EXTI->PR = (1 << 0);
    
    // 處理中斷
    process_interrupt();
}
```

### **2. ISR 執行過長**

```c
// 錯誤：在 ISR 中進行長時間操作
void bad_long_isr(void) {
    EXTI->PR = (1 << 0);
    
    // 不要在 ISR 中這樣做！
    for (int i = 0; i < 1000; i++) {
        process_data();
    }
}

// 正確：設定旗標然後返回
void good_short_isr(void) {
    EXTI->PR = (1 << 0);
    
    // 設定旗標供主迴圈處理
    data_processing_pending = true;
}
```

### **3. 競爭條件**

```c
// 錯誤：共享變數的競爭條件
volatile bool button_pressed = false;

void isr_with_race(void) {
    EXTI->PR = (1 << 0);
    button_pressed = true; // 可能發生競爭條件
}

// 正確：原子操作
volatile uint32_t button_flags = 0;

void isr_without_race(void) {
    EXTI->PR = (1 << 0);
    __atomic_or_fetch(&button_flags, 1, __ATOMIC_RELEASE);
}
```

---

## 💡 **範例**

### **1. 帶消彈的按鈕中斷**

```c
// 按鈕中斷實作
#define BUTTON_PIN     0
#define DEBOUNCE_MS    50

volatile bool button_pressed = false;
volatile uint32_t last_button_time = 0;

void EXTI0_IRQHandler(void) {
    if (EXTI->PR & (1 << BUTTON_PIN)) {
        EXTI->PR = (1 << BUTTON_PIN);
        
        uint32_t current_time = get_system_tick();
        
        // 軟體消彈
        if (current_time - last_button_time > DEBOUNCE_MS) {
            button_pressed = true;
            last_button_time = current_time;
        }
    }
}

// 主迴圈處理按鈕按下
void main_loop(void) {
    while (1) {
        if (button_pressed) {
            button_pressed = false;
            handle_button_action();
        }
        
        // 其他任務
        process_system_tasks();
    }
}
```

### **2. 帶邊緣偵測的感測器中斷**

```c
// 帶邊緣偵測的感測器中斷
#define SENSOR_PIN     1
#define SENSOR_RISING  1
#define SENSOR_FALLING 0

volatile uint32_t sensor_rising_count = 0;
volatile uint32_t sensor_falling_count = 0;

void EXTI1_IRQHandler(void) {
    if (EXTI->PR & (1 << SENSOR_PIN)) {
        EXTI->PR = (1 << SENSOR_PIN);
        
        // 讀取當前腳位狀態
        bool current_state = (GPIOA->IDR & (1 << SENSOR_PIN)) ? 1 : 0;
        
        if (current_state == SENSOR_RISING) {
            sensor_rising_count++;
        } else {
            sensor_falling_count++;
        }
    }
}
```

### **3. 多重中斷源**

```c
// 帶優先級的多重中斷源
#define INT_PIN_1      0
#define INT_PIN_2      1
#define INT_PIN_3      2

volatile uint32_t interrupt_flags = 0;

void EXTI0_IRQHandler(void) {
    if (EXTI->PR & (1 << INT_PIN_1)) {
        EXTI->PR = (1 << INT_PIN_1);
        interrupt_flags |= (1 << INT_PIN_1);
    }
}

void EXTI1_IRQHandler(void) {
    if (EXTI->PR & (1 << INT_PIN_2)) {
        EXTI->PR = (1 << INT_PIN_2);
        interrupt_flags |= (1 << INT_PIN_2);
    }
}

void EXTI2_IRQHandler(void) {
    if (EXTI->PR & (1 << INT_PIN_3)) {
        EXTI->PR = (1 << INT_PIN_3);
        interrupt_flags |= (1 << INT_PIN_3);
    }
}

// 按優先級順序處理中斷
void process_interrupts(void) {
    if (interrupt_flags & (1 << INT_PIN_1)) {
        interrupt_flags &= ~(1 << INT_PIN_1);
        process_high_priority_interrupt();
    }
    
    if (interrupt_flags & (1 << INT_PIN_2)) {
        interrupt_flags &= ~(1 << INT_PIN_2);
        process_medium_priority_interrupt();
    }
    
    if (interrupt_flags & (1 << INT_PIN_3)) {
        interrupt_flags &= ~(1 << INT_PIN_3);
        process_low_priority_interrupt();
    }
}
```

---

## 🧪 引導式實驗

### 實驗 1：基本外部中斷實作
1. **設定**：配置 GPIO 腳位的外部中斷與邊緣偵測
2. **測試**：連接按鈕並驗證按下/釋放時的中斷觸發
3. **量測**：使用示波器量測中斷延遲和回應時間
4. **最佳化**：實作消彈並量測其對可靠性的影響

### 實驗 2：中斷優先級與巢狀
1. **配置**：設定具有不同優先級的多個中斷源
2. **測試**：同時觸發中斷並觀察優先級處理
3. **分析**：量測中斷巢狀行為和上下文切換開銷
4. **驗證**：驗證較高優先級中斷可以搶占較低優先級中斷

### 實驗 3：進階中斷技術
1. **實作**：帶有適當源頭清除的準位觸發中斷
2. **設計**：用於複雜事件處理的中斷驅動狀態機
3. **最佳化**：最小化 ISR 執行時間並量測效能影響
4. **除錯**：使用邏輯分析儀追蹤中斷時序並識別瓶頸

## ✅ 自我檢查

### 理解檢查
- [ ] 你能解釋邊緣觸發和準位觸發中斷之間的區別嗎？
- [ ] 你了解中斷優先級如何影響系統行為嗎？
- [ ] 你能描述中斷處理管線和延遲來源嗎？
- [ ] 你知道何時為不同應用使用邊緣觸發或準位觸發嗎？

### 應用檢查
- [ ] 你能配置帶有適當邊緣/準位偵測的外部中斷嗎？
- [ ] 你能為機械開關實作有效的消彈嗎？
- [ ] 你能設計最小化執行時間的中斷服務常式嗎？
- [ ] 你能用適當的優先級管理處理多個中斷源嗎？

### 分析檢查
- [ ] 你能量測和分析中斷延遲和回應時間嗎？
- [ ] 你能識別和解決中斷相關的競爭條件嗎？
- [ ] 你能為功耗效率和效能最佳化中斷系統嗎？
- [ ] 你能有效地除錯複雜的中斷驅動系統嗎？

## 🔗 交叉連結

- **[GPIO 配置](./GPIO_Configuration.md)** - 中斷腳位的 GPIO 設定
- **[數位 I/O 程式設計](./Digital_IO_Programming.md)** - 開關讀取和消彈技術
- **[中斷與例外](./Interrupts_Exceptions.md)** - 通用中斷處理概念
- **[即時系統](./../Real_Time_Systems/Real_Time_Systems_Overview.md)** - 即時中斷需求
- **[電源管理](./Power_Management.md)** - 中斷作為喚醒源

## 🎯 **面試問題**

### **基本問題**
1. **邊緣觸發和準位觸發中斷有什麼區別？**
   - 邊緣觸發：在訊號轉換時觸發（0→1 或 1→0）
   - 準位觸發：在訊號保持在特定準位時觸發
   - 邊緣觸發更常用於外部中斷

2. **如何為機械開關實作消彈？**
   - 軟體：基於計時器的延遲、狀態機
   - 硬體：RC 濾波器、施密特觸發器、彈跳抑制 IC
   - 最佳方法取決於需求和限制

3. **什麼是中斷延遲，如何最小化？**
   - 從中斷發生到處理器開始執行的時間
   - 最小化方法：短 ISR、適當優先級、避免函式呼叫

### **進階問題**
4. **如何處理具有不同優先級的多個中斷源？**
   - 配置 NVIC 優先級
   - 如果支援則使用中斷巢狀
   - 在主迴圈中按優先級順序處理

5. **實作外部中斷時常見的陷阱有哪些？**
   - 缺少中斷旗標清除
   - ISR 執行時間過長
   - 共享變數的競爭條件
   - 不正確的消彈

6. **如何量測和最佳化中斷延遲？**
   - 使用高解析度計時器
   - 分析 ISR 執行時間
   - 最佳化 ISR 中的最小化處理

### **實務問題**
7. **設計一個帶消彈的中斷驅動按鈕介面。**
   ```c
   // 配置按鈕中斷
   void configure_button_interrupt(void) {
       // GPIO 設為帶上拉的輸入
       GPIO_InitTypeDef gpio_init = {0};
       gpio_init.Pin = BUTTON_PIN;
       gpio_init.Mode = GPIO_MODE_IT_FALLING;
       gpio_init.Pull = GPIO_PULLUP;
       HAL_GPIO_Init(BUTTON_PORT, &gpio_init);
       
       // 啟用中斷
       HAL_NVIC_SetPriority(BUTTON_IRQn, 1, 0);
       HAL_NVIC_EnableIRQ(BUTTON_IRQn);
   }
   ```

8. **實作一個計算上升沿數量的感測器中斷。**
   ```c
   volatile uint32_t edge_count = 0;
   
   void sensor_isr(void) {
       if (EXTI->PR & (1 << SENSOR_PIN)) {
           EXTI->PR = (1 << SENSOR_PIN);
           edge_count++;
       }
   }
   ```

---

## 🧪 引導式實驗
1) 消彈比較
- 實作軟體消彈與硬體 RC 濾波器；量測回應時間和可靠性。

2) 邊緣觸發 vs 準位觸發
- 為同一腳位配置邊緣和準位中斷；觀察行為差異。

## ✅ 自我檢查
- 何時應該使用準位觸發中斷而非邊緣觸發？
- 如何處理同一腳位上的多個中斷源？

## 🔗 交叉連結
- `Hardware_Fundamentals/Interrupts_Exceptions.md` 中斷處理
- `Hardware_Fundamentals/Digital_IO_Programming.md` 腳位配置

---

## 🔗 **相關主題**

- **[計時器/計數器程式設計](./Timer_Counter_Programming.md)** - 輸入捕獲、輸出比較、頻率量測
- **[中斷與例外](./Interrupts_Exceptions.md)** - 中斷處理、ISR 設計、中斷延遲
- **[看門狗計時器](./Watchdog_Timers.md)** - 系統監控和恢復機制
- **[電源管理](./Power_Management.md)** - 睡眠模式、喚醒源、功耗最佳化

---

## 📚 **資源**

### **書籍**
- "Making Embedded Systems" by Elecia White
- "Programming Embedded Systems" by Michael Barr
- "Real-Time Systems" by Jane W. S. Liu

### **線上資源**
- [ARM Cortex-M 中斷處理](https://developer.arm.com/documentation/dui0552/a/the-cortex-m3-processor/interrupts-and-exceptions)
- [STM32 外部中斷](https://www.st.com/resource/en/user_manual/dm00031936-stm32f0x1stm32f0x2stm32f0x8-advanced-armbased-32bit-mcus-stmicroelectronics.pdf)

---

**下一個主題：** [看門狗計時器](./Watchdog_Timers.md) → [中斷與例外](./Interrupts_Exceptions.md)
