# 🔌 數位 I/O 程式設計

## 快速參考：重點摘要

- **數位 I/O 程式設計** 涉及透過 GPIO 腳位控制二進位訊號（HIGH/LOW）以實現嵌入式系統互動
- **輸入操作** 包括讀取開關、感測器和數位訊號，含消彈和邊緣偵測
- **輸出操作** 包括驅動 LED、繼電器、顯示器和致動器，具有精確的時序控制
- **消彈** 對可靠的開關讀取至關重要，使用硬體濾波器或軟體演算法
- **邊緣偵測** 識別狀態轉換（上升/下降）用於事件驅動應用
- **狀態機** 管理複雜的 I/O 序列和使用者介面互動
- **效能最佳化** 包括原子操作、中斷處理和時序一致性
- **介面設計** 涵蓋鍵盤、顯示器和多工技術以實現高效 I/O

> **掌握嵌入式系統的數位輸入/輸出操作**  
> 讀取開關、驅動 LED、鍵盤掃描和數位訊號處理

## 📋 目錄

- [🎯 概述](#-概述)
- [🤔 什麼是數位 I/O 程式設計？](#-什麼是數位-io-程式設計)
- [🎯 為什麼數位 I/O 重要？](#-為什麼數位-io-重要)
- [🧠 數位 I/O 概念](#-數位-io-概念)
- [🔌 基本數位 I/O 操作](#-基本數位-io-操作)
- [🔘 開關讀取技術](#-開關讀取技術)
- [💡 LED 控制模式](#-led-控制模式)
- [⌨️ 鍵盤掃描](#️-鍵盤掃描)
- [🔢 七段顯示器控制](#-七段顯示器控制)
- [🔄 狀態機實作](#-狀態機實作)
- [⚡ 效能最佳化](#-效能最佳化)
- [🎯 常見應用](#-常見應用)
- [🔧 實作](#-實作)
- [⚠️ 常見陷阱](#️-常見陷阱)
- [✅ 最佳實踐](#-最佳實踐)
- [🎯 面試問題](#-面試問題)
- [📚 額外資源](#-額外資源)

---

## 🎯 概述

### 概念：具有明確腳位擁有權和時序的確定性 I/O

數位 I/O 是關於確定性地配置腳位方向、電位和時序。將每個腳位視為具有明確擁有權和轉換的資源。

### 為什麼在嵌入式中重要
- 防止競爭（兩個驅動器在同一網路上）和未定義的電位。
- 確保邊緣滿足外部裝置時序（建立/保持時間、消彈）。
- 使行為在中斷和 RTOS 排程下可預測。

### 最小範例
```c
// 帶有明確初始化的簡單 LED 切換
static inline void led_init(void){ /* 配置 GPIO 埠/腳位模式 */ }
static inline void led_on(void){ /* 設定 ODR 位元 */ }
static inline void led_off(void){ /* 清除 ODR 位元 */ }
static inline void led_toggle(void){ /* XOR ODR 位元 */ }
```

### 試試看
1. 以已知週期切換腳位；使用邏輯分析儀量測以驗證抖動。
2. 加入 ISR 並觀察抖動變化；調整優先級或將工作移出 ISR。

### 重點摘要
- 使用前先初始化；記錄上拉/下拉和預設狀態。
- 可用時使用原子設定/重設暫存器避免讀取-修改-寫入競爭。
- 將腳位控制封裝在函式/巨集後面以確保可攜性。

---

## 🔍 視覺化理解

### **數位 I/O 訊號特性**
```
數位訊號狀態
┌─────────────────────────────────────────────────────────────┐
│                    HIGH 狀態（邏輯 1）                      │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ 電壓：3.3V/5V（取決於邏輯準位）                    │   │
│  │ 電流：可向外部負載源出電流                          │   │
│  │ 狀態：活躍/開/真                                    │   │
│  └─────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│                    LOW 狀態（邏輯 0）                       │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ 電壓：0V（接地參考）                                │   │
│  │ 電流：可從外部源汲入電流                            │   │
│  │ 狀態：非活躍/關/假                                  │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### **開關消彈過程**
```
開關彈跳與消彈
原始開關訊號
   ^
   |    ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐
   |    │ │ │ │ │ │ │ │ │ │
   |    │ │ │ │ │ │ │ │ │ │
   |    │ │ │ │ │ │ │ │ │ │
   +──────────────────────────-> 時間
   |<->| 彈跳週期

消彈後的訊號
   ^
   |    ┌─────────────────┐
   |    │                 │
   |    │                 │
   |    │                 │
   +──────────────────────────-> 時間
   |<->| 穩定週期
```

### **邊緣偵測時序**
```
邊緣偵測與時序
輸入訊號
   ^
   |    ┌─────────────────┐
   |    │                 │
   |    │                 │
   |    │                 │
   +──────────────────────────-> 時間
   ▲         ▼
上升      下降
 沿        沿

中斷回應
   ^
   |    │         │
   |    │         │
   |    │         │
   +──────────────────────────-> 時間
   │<->│ 回應時間
```

### **🧠 概念基礎**

#### **數位 I/O 範式**
數位 I/O 代表嵌入式系統互動的最基本層級。與處理連續值的類比 I/O 不同，數位 I/O 在離散的二進位狀態上操作，本質上具有抗雜訊和快速的特性。

**關鍵特性：**
- **二進位本質**：僅兩個狀態簡化了邏輯並減少錯誤
- **抗雜訊能力**：高雜訊裕度使訊號可靠
- **快速回應**：立即的狀態變化啟用即時控制
- **確定性**：可預測的時序和行為

#### **為什麼數位 I/O 程式設計重要**
數位 I/O 程式設計對系統可靠性和效能至關重要：

- **訊號完整性**：適當的時序和消彈確保可靠操作
- **即時回應**：對外部事件的快速、可預測回應
- **資源管理**：有限 GPIO 腳位的高效使用
- **系統可靠性**：在雜訊環境中的穩健操作

#### **時序挑戰**
數位 I/O 引入了必須處理的獨特時序挑戰：

- **消彈**：機械開關產生必須過濾的多次轉換
- **邊緣偵測**：事件驅動應用需要精確的時序
- **中斷延遲**：回應時間必須可預測且有界
- **抖動控制**：時序變化可能影響系統效能

## 🧪 引導式實驗
1) 抖動量測
- 在緊密迴圈中切換腳位；使用示波器或邏輯分析儀量測邊緣-邊緣時序。

2) RMW 競爭避免
- 實作一個設定/清除個別位元而不影響其他位元的函式；驗證原子性。

## ✅ 自我檢查
- 何時需要在 I/O 操作期間禁用中斷？
- 如何確保不同最佳化等級下的一致時序？

## 🔗 交叉連結
- `Embedded_C/Type_Qualifiers.md` volatile 使用
- `Embedded_C/Bit_Manipulation.md` 位元操作

數位 I/O 程式設計是嵌入式系統與物理世界互動的基礎。它涉及讀取數位輸入（開關、感測器）和控制數位輸出（LED、繼電器、顯示器）。

**重點概念：**
- **輸入讀取**：消彈、邊緣偵測、狀態機
- **輸出控制**：PWM、模式、時序控制
- **介面設計**：鍵盤、顯示器、多工
- **效能**：最佳化、即時約束

## 🤔 什麼是數位 I/O 程式設計？

數位 I/O 程式設計涉及透過 GPIO 腳位控制和讀取二進位訊號（HIGH/LOW、1/0、ON/OFF）。它是嵌入式系統與外部世界互動的最基本方式，能夠與開關、感測器、致動器和顯示器進行通訊。

### **核心概念**

**二進位訊號處理：**
- **數位狀態**：僅兩個狀態 - HIGH（1）或 LOW（0）
- **電壓準位**：HIGH 通常為 3.3V 或 5V，LOW 為 0V
- **乾淨訊號**：抗雜訊的數位訊號
- **快速回應**：對狀態變化的立即回應

**輸入/輸出操作：**
- **輸入讀取**：感測外部數位訊號
- **輸出控制**：驅動外部數位負載
- **雙向**：腳位可配置為輸入或輸出
- **即時**：對外部事件的立即回應

**訊號特性：**
- **時序**：上升/下降時間和傳播延遲
- **抗雜訊能力**：對電氣雜訊的抵抗力
- **負載驅動**：驅動外部負載的能力
- **保護**：內建的電氣損壞保護

### **數位 vs 類比 I/O**

**數位 I/O：**
- **離散狀態**：僅兩個狀態（HIGH/LOW）
- **簡單處理**：直接二進位操作
- **抗雜訊**：對小雜訊變化免疫
- **快速回應**：立即狀態變化

**類比 I/O：**
- **連續值**：電壓準位範圍
- **複雜處理**：需要 ADC/DAC 轉換
- **雜訊敏感**：受雜訊和干擾影響
- **較慢回應**：需要轉換時間

### **數位 I/O 應用**

**輸入應用：**
- **開關和按鈕**：使用者介面裝置
- **感測器**：數位感測器（溫度、壓力、動作）
- **編碼器**：位置和速度回饋
- **偵測器**：接近、液位和存在偵測器

**輸出應用：**
- **LED**：狀態指示燈和顯示器
- **繼電器**：大功率切換
- **顯示器**：LCD、OLED 和七段顯示器
- **致動器**：馬達、電磁閥和閥門

## 🎯 為什麼數位 I/O 重要？

### **嵌入式系統需求**

**使用者介面：**
- **人機互動**：按鈕、開關、鍵盤用於使用者輸入
- **狀態回饋**：LED、顯示器用於系統狀態
- **控制介面**：使用者控制系統功能
- **除錯介面**：除錯訊號和測試點

**感測器介面：**
- **環境感測**：溫度、壓力、動作感測器
- **位置感測**：編碼器、限位開關、位置感測器
- **安全感測**：安全開關、緊急停止
- **狀態感測**：電源狀態、通訊狀態

**致動器控制：**
- **馬達控制**：直流馬達、步進馬達、伺服馬達
- **繼電器控制**：大功率切換和控制
- **閥門控制**：流體和氣體控制系統
- **顯示器控制**：LED 顯示器、LCD 顯示器

**系統控制：**
- **配置**：系統配置和模式選擇
- **重設控制**：硬體重設和系統控制
- **通訊**：數位通訊介面
- **時序**：時序和同步訊號

### **真實世界的影響**

**使用者介面應用：**
```c
// 使用者控制的按鈕介面
typedef struct {
    GPIO_TypeDef* port;
    uint16_t pin;
    bool pressed;
    uint32_t press_time;
} user_button_t;

void handle_user_input(user_button_t* button) {
    if (button->pressed) {
        // 處理按鈕按下
        system_mode_toggle();
        button->pressed = false;
    }
}
```

**感測器介面應用：**
```c
// 數位感測器介面
typedef struct {
    GPIO_TypeDef* port;
    uint16_t pin;
    bool triggered;
    uint32_t trigger_count;
} digital_sensor_t;

void handle_sensor_event(digital_sensor_t* sensor) {
    if (sensor->triggered) {
        // 處理感測器事件
        sensor->trigger_count++;
        process_sensor_data();
        sensor->triggered = false;
    }
}
```

**致動器控制應用：**
```c
// 馬達控制介面
typedef struct {
    GPIO_TypeDef* direction_port;
    uint16_t direction_pin;
    GPIO_TypeDef* enable_port;
    uint16_t enable_pin;
    bool running;
    bool direction;
} motor_control_t;

void control_motor(motor_control_t* motor, bool enable, bool direction) {
    if (enable) {
        gpio_write(motor->enable_port, motor->enable_pin, true);
        gpio_write(motor->direction_port, motor->direction_pin, direction);
        motor->running = true;
        motor->direction = direction;
    } else {
        gpio_write(motor->enable_port, motor->enable_pin, false);
        motor->running = false;
    }
}
```

### **數位 I/O 何時重要**

**高影響場景：**
- 即時控制系統
- 使用者介面應用
- 感測器和致動器介面
- 系統監控和控制
- 安全關鍵系統

**低影響場景：**
- 純計算應用
- 僅網路系統
- 與外部互動最少的系統
- 資源充足的原型系統

## 🧠 數位 I/O 概念

### **數位 I/O 如何運作**

**訊號處理：**
1. **輸入感測**：GPIO 腳位感測外部電壓準位
2. **訊號調節**：雜訊濾波和訊號調節
3. **狀態偵測**：將電壓轉換為數位狀態
4. **輸出驅動**：以電壓驅動外部負載

**時序考量：**
- **回應時間**：從輸入變化到輸出回應的時間
- **消彈**：濾除機械開關彈跳
- **邊緣偵測**：偵測上升和下降沿
- **狀態機**：管理複雜的輸入/輸出模式

**電氣特性：**
- **電壓準位**：邏輯 HIGH 和 LOW 的電壓準位
- **電流驅動**：腳位可源出/汲入的最大電流
- **負載能力**：腳位可驅動的負載
- **抗雜訊能力**：對電氣雜訊的抵抗力

### **數位 I/O 模式**

**輸入模式：**
- **準位偵測**：偵測 HIGH/LOW 準位
- **邊緣偵測**：偵測上升/下降沿
- **脈衝偵測**：偵測脈衝和時序
- **模式識別**：識別輸入模式

**輸出模式：**
- **準位控制**：設定 HIGH/LOW 準位
- **脈衝產生**：產生脈衝和時序
- **模式產生**：產生輸出模式
- **PWM 控制**：脈寬調變控制

**介面模式：**
- **輪詢**：定期檢查輸入狀態
- **中斷驅動**：回應輸入變化
- **狀態機**：管理複雜的輸入/輸出狀態
- **事件驅動**：回應特定事件

### **數位 I/O 時序**

**輸入時序：**
- **建立時間**：讀取前輸入必須穩定的時間
- **保持時間**：讀取後輸入必須保持穩定的時間
- **消彈時間**：濾除開關彈跳的時間
- **回應時間**：從輸入變化到偵測的時間

**輸出時序：**
- **上升時間**：輸出從 LOW 到 HIGH 的時間
- **下降時間**：輸出從 HIGH 到 LOW 的時間
- **傳播延遲**：從命令到輸出變化的時間
- **穩定時間**：輸出穩定的時間

## 🔌 基本數位 I/O 操作

### **什麼是基本數位 I/O 操作？**

基本數位 I/O 操作是讀取數位輸入和寫入數位輸出的基礎操作。它們構成所有數位 I/O 程式設計的基礎。

### **操作概念**

**輸入操作：**
- **讀取**：讀取輸入腳位的當前狀態
- **取樣**：隨時間進行多次讀取
- **濾波**：去除雜訊和不需要的訊號
- **調節**：準備訊號以供處理

**輸出操作：**
- **寫入**：設定輸出腳位的狀態
- **切換**：改變輸出腳位的狀態
- **模式產生**：產生特定的輸出模式
- **時序控制**：控制輸出時序

### **讀取數位輸入**
```c
// 基本數位輸入讀取
uint8_t read_digital_input(GPIO_TypeDef* GPIOx, uint16_t pin) {
    return (GPIOx->IDR >> pin) & 0x01;
}

// 一次讀取多個輸入
uint16_t read_multiple_inputs(GPIO_TypeDef* GPIOx, uint16_t mask) {
    return GPIOx->IDR & mask;
}
```

### **寫入數位輸出**
```c
// 基本數位輸出寫入
void write_digital_output(GPIO_TypeDef* GPIOx, uint16_t pin, uint8_t state) {
    if (state) {
        GPIOx->BSRR = (1U << pin);  // 設定位元
    } else {
        GPIOx->BSRR = (1U << (pin + 16));  // 重設位元
    }
}

// 一次寫入多個輸出
void write_multiple_outputs(GPIO_TypeDef* GPIOx, uint16_t mask, uint16_t state) {
    GPIOx->BSRR = (state & mask) | ((~state & mask) << 16);
}
```

### **切換輸出**
```c
// 切換數位輸出
void toggle_output(GPIO_TypeDef* GPIOx, uint16_t pin) {
    GPIOx->ODR ^= (1U << pin);
}

// 切換多個輸出
void toggle_multiple_outputs(GPIO_TypeDef* GPIOx, uint16_t mask) {
    GPIOx->ODR ^= mask;
}
```

## 🔘 開關讀取技術

### **什麼是開關讀取技術？**

開關讀取技術涉及讀取機械開關和按鈕，同時處理消彈、邊緣偵測和狀態管理等問題。

### **開關讀取概念**

**機械開關特性：**
- **接觸彈跳**：機械開關在按下/釋放時會彈跳
- **接觸電阻**：開關閉合時的電阻
- **接觸磨損**：開關隨時間磨損
- **環境因素**：溫度、濕度、振動

**消彈技術：**
- **硬體消彈**：使用電容和電阻
- **軟體消彈**：使用計時器和狀態機
- **混合消彈**：結合硬體和軟體
- **進階消彈**：使用濾波器和演算法

### **簡單開關讀取**
```c
// 基本開關讀取（無消彈）
uint8_t read_switch_simple(GPIO_TypeDef* GPIOx, uint16_t pin) {
    return !read_digital_input(GPIOx, pin);  // 上拉反轉
}
```

### **消彈開關讀取**
```c
typedef struct {
    GPIO_TypeDef* GPIOx;
    uint16_t pin;
    uint8_t last_state;
    uint8_t current_state;
    uint32_t debounce_time;
    uint32_t last_change_time;
} DebouncedSwitch_t;

void switch_init(DebouncedSwitch_t* sw, GPIO_TypeDef* GPIOx, uint16_t pin, uint32_t debounce_ms) {
    sw->GPIOx = GPIOx;
    sw->pin = pin;
    sw->debounce_time = debounce_ms;
    sw->last_state = 0;
    sw->current_state = 0;
    sw->last_change_time = 0;
    
    // 配置為帶上拉的輸入
    gpio_input_pullup_config(GPIOx, pin);
}

uint8_t read_switch_debounced(DebouncedSwitch_t* sw) {
    uint8_t raw_state = !read_digital_input(sw->GPIOx, sw->pin);
    uint32_t current_time = HAL_GetTick();
    
    if (raw_state != sw->last_state) {
        if (current_time - sw->last_change_time > sw->debounce_time) {
            sw->current_state = raw_state;
            sw->last_state = raw_state;
            sw->last_change_time = current_time;
        }
    }
    
    return sw->current_state;
}
```

### **邊緣偵測**
```c
typedef enum {
    EDGE_NONE = 0,
    EDGE_RISING = 1,
    EDGE_FALLING = 2,
    EDGE_BOTH = 3
} EdgeType_t;

typedef struct {
    DebouncedSwitch_t switch_data;
    EdgeType_t edge_type;
    uint8_t last_stable_state;
} EdgeDetector_t;

void edge_detector_init(EdgeDetector_t* detector, GPIO_TypeDef* GPIOx, uint16_t pin, 
                       EdgeType_t edge_type, uint32_t debounce_ms) {
    switch_init(&detector->switch_data, GPIOx, pin, debounce_ms);
    detector->edge_type = edge_type;
    detector->last_stable_state = 0;
}

uint8_t detect_edge(EdgeDetector_t* detector) {
    uint8_t current_state = read_switch_debounced(&detector->switch_data);
    uint8_t edge_detected = 0;
    
    if (current_state != detector->last_stable_state) {
        if (detector->edge_type == EDGE_RISING && current_state == 1) {
            edge_detected = 1;
        } else if (detector->edge_type == EDGE_FALLING && current_state == 0) {
            edge_detected = 1;
        } else if (detector->edge_type == EDGE_BOTH) {
            edge_detected = 1;
        }
        
        detector->last_stable_state = current_state;
    }
    
    return edge_detected;
}
```

## 💡 LED 控制模式

### **什麼是 LED 控制模式？**

LED 控制模式涉及控制 LED 用於狀態指示、顯示和視覺回饋。它們包括簡單的開/關控制、閃爍模式和複雜的顯示模式。

### **LED 控制概念**

**LED 特性：**
- **正向電壓**：開啟 LED 所需的電壓
- **正向電流**：正確亮度所需的電流
- **亮度控制**：控制 LED 亮度
- **顏色控制**：控制 LED 顏色（RGB LED）

**控制模式：**
- **簡單開/關**：基本 LED 控制
- **閃爍模式**：定時閃爍序列
- **淡入淡出模式**：亮度漸變
- **顯示模式**：複雜的顯示序列

### **基本 LED 控制**
```c
typedef struct {
    GPIO_TypeDef* GPIOx;
    uint16_t pin;
    uint8_t state;
} LED_t;

void led_init(LED_t* led, GPIO_TypeDef* GPIOx, uint16_t pin) {
    led->GPIOx = GPIOx;
    led->pin = pin;
    led->state = 0;
    
    gpio_pushpull_output_config(GPIOx, pin);
    write_digital_output(GPIOx, pin, 0);
}

void led_on(LED_t* led) {
    write_digital_output(led->GPIOx, led->pin, 1);
    led->state = 1;
}

void led_off(LED_t* led) {
    write_digital_output(led->GPIOx, led->pin, 0);
    led->state = 0;
}

void led_toggle(LED_t* led) {
    led->state = !led->state;
    write_digital_output(led->GPIOx, led->pin, led->state);
}
```

### **LED 閃爍模式**
```c
typedef struct {
    LED_t led;
    uint32_t blink_period;
    uint32_t last_toggle_time;
    bool blinking;
} BlinkingLED_t;

void blinking_led_init(BlinkingLED_t* bled, GPIO_TypeDef* GPIOx, uint16_t pin, uint32_t period_ms) {
    led_init(&bled->led, GPIOx, pin);
    bled->blink_period = period_ms;
    bled->last_toggle_time = 0;
    bled->blinking = false;
}

void blinking_led_start(BlinkingLED_t* bled) {
    bled->blinking = true;
    bled->last_toggle_time = HAL_GetTick();
}

void blinking_led_stop(BlinkingLED_t* bled) {
    bled->blinking = false;
    led_off(&bled->led);
}

void blinking_led_update(BlinkingLED_t* bled) {
    if (bled->blinking) {
        uint32_t current_time = HAL_GetTick();
        if (current_time - bled->last_toggle_time >= bled->blink_period) {
            led_toggle(&bled->led);
            bled->last_toggle_time = current_time;
        }
    }
}
```

## ⌨️ 鍵盤掃描

### **什麼是鍵盤掃描？**

鍵盤掃描涉及讀取矩陣鍵盤和按鈕陣列以偵測使用者輸入。它需要掃描行和列以確定按下了哪個鍵。

### **鍵盤掃描概念**

**矩陣鍵盤結構：**
- **行和列**：鍵盤以矩陣格式組織
- **掃描技術**：掃描行/列以偵測按下
- **鬼鍵**：多鍵按下導致的假偵測
- **按鍵翻轉**：處理多個同時按下

**掃描方法：**
- **行掃描**：一次掃描一行
- **列掃描**：一次掃描一列
- **中斷掃描**：使用中斷進行按鍵偵測
- **輪詢掃描**：定期輪詢按鍵按下

### **矩陣鍵盤實作**
```c
#define KEYPAD_ROWS 4
#define KEYPAD_COLS 4

typedef struct {
    GPIO_TypeDef* row_ports[KEYPAD_ROWS];
    uint16_t row_pins[KEYPAD_ROWS];
    GPIO_TypeDef* col_ports[KEYPAD_COLS];
    uint16_t col_pins[KEYPAD_COLS];
    char keymap[KEYPAD_ROWS][KEYPAD_COLS];
    uint8_t last_key;
} MatrixKeypad_t;

void keypad_init(MatrixKeypad_t* keypad) {
    // 初始化行腳位為輸出
    for (int i = 0; i < KEYPAD_ROWS; i++) {
        gpio_pushpull_output_config(keypad->row_ports[i], keypad->row_pins[i]);
        write_digital_output(keypad->row_ports[i], keypad->row_pins[i], 1);
    }
    
    // 初始化列腳位為帶上拉的輸入
    for (int i = 0; i < KEYPAD_COLS; i++) {
        gpio_input_pullup_config(keypad->col_ports[i], keypad->col_pins[i]);
    }
    
    keypad->last_key = 0;
}

char keypad_scan(MatrixKeypad_t* keypad) {
    char pressed_key = 0;
    
    // 掃描每一行
    for (int row = 0; row < KEYPAD_ROWS; row++) {
        // 將當前行設為 LOW
        write_digital_output(keypad->row_ports[row], keypad->row_pins[row], 0);
        
        // 檢查每一列
        for (int col = 0; col < KEYPAD_COLS; col++) {
            if (!read_digital_input(keypad->col_ports[col], keypad->col_pins[col])) {
                pressed_key = keypad->keymap[row][col];
                break;
            }
        }
        
        // 將行設回 HIGH
        write_digital_output(keypad->row_ports[row], keypad->row_pins[row], 1);
        
        if (pressed_key) break;
    }
    
    return pressed_key;
}
```

## 🔢 七段顯示器控制

### **什麼是七段顯示器控制？**

七段顯示器控制涉及驅動七段 LED 顯示器以顯示數字、字母和符號。它需要控制個別段落並實作多位數的多工。

### **七段顯示器概念**

**顯示器結構：**
- **七個段落**：個別 LED 段落（a-g）
- **共陽/共陰**：共同連接類型
- **位數多工**：驅動多個位數
- **字元編碼**：將字元轉換為段落圖案

**控制方法：**
- **直接控制**：直接控制每個段落
- **多工控制**：多位數的時分多工控制
- **移位暫存器控制**：使用移位暫存器進行控制
- **I2C/SPI 控制**：使用通訊協議

### **七段顯示器實作**
```c
// 七段圖案（共陰）
const uint8_t seven_seg_patterns[16] = {
    0x3F, // 0
    0x06, // 1
    0x5B, // 2
    0x4F, // 3
    0x66, // 4
    0x6D, // 5
    0x7D, // 6
    0x07, // 7
    0x7F, // 8
    0x6F, // 9
    0x77, // A
    0x7C, // b
    0x39, // C
    0x5E, // d
    0x79, // E
    0x71  // F
};

typedef struct {
    GPIO_TypeDef* segment_ports[7];
    uint16_t segment_pins[7];
    GPIO_TypeDef* digit_ports[4];
    uint16_t digit_pins[4];
    uint8_t current_digit;
    uint8_t display_value[4];
} SevenSegmentDisplay_t;

void seven_seg_init(SevenSegmentDisplay_t* display) {
    // 初始化段落腳位為輸出
    for (int i = 0; i < 7; i++) {
        gpio_pushpull_output_config(display->segment_ports[i], display->segment_pins[i]);
    }
    
    // 初始化位數腳位為輸出
    for (int i = 0; i < 4; i++) {
        gpio_pushpull_output_config(display->digit_ports[i], display->digit_pins[i]);
        write_digital_output(display->digit_ports[i], display->digit_pins[i], 0);
    }
    
    display->current_digit = 0;
}

void seven_seg_display_digit(SevenSegmentDisplay_t* display, uint8_t digit, uint8_t value) {
    if (digit < 4 && value < 16) {
        display->display_value[digit] = value;
    }
}

void seven_seg_update(SevenSegmentDisplay_t* display) {
    // 關閉所有位數
    for (int i = 0; i < 4; i++) {
        write_digital_output(display->digit_ports[i], display->digit_pins[i], 0);
    }
    
    // 設定當前位數的段落
    uint8_t pattern = seven_seg_patterns[display->display_value[display->current_digit]];
    for (int i = 0; i < 7; i++) {
        write_digital_output(display->segment_ports[i], display->segment_pins[i], 
                           (pattern >> i) & 0x01);
    }
    
    // 開啟當前位數
    write_digital_output(display->digit_ports[display->current_digit], 
                        display->digit_pins[display->current_digit], 1);
    
    // 移到下一位數
    display->current_digit = (display->current_digit + 1) % 4;
}
```

## 🔄 狀態機實作

### **什麼是狀態機實作？**

狀態機實作涉及使用有限狀態機管理複雜的輸入/輸出模式。它對處理複雜的使用者介面和系統行為至關重要。

### **狀態機概念**

**狀態機結構：**
- **狀態**：不同的系統狀態
- **轉換**：基於輸入的狀態變化
- **動作**：在每個狀態中執行的動作
- **事件**：觸發狀態變化的輸入

**狀態機類型：**
- **Moore 機**：輸出僅取決於當前狀態
- **Mealy 機**：輸出取決於當前狀態和輸入
- **階層式狀態機**：狀態中的狀態
- **並行狀態機**：多個平行狀態機

### **狀態機實作**
```c
typedef enum {
    STATE_IDLE,
    STATE_BUTTON_PRESSED,
    STATE_BUTTON_HELD,
    STATE_BUTTON_RELEASED
} ButtonState_t;

typedef struct {
    ButtonState_t current_state;
    uint32_t state_entry_time;
    uint32_t button_press_time;
    bool button_pressed;
} ButtonStateMachine_t;

void button_state_machine_init(ButtonStateMachine_t* sm) {
    sm->current_state = STATE_IDLE;
    sm->state_entry_time = 0;
    sm->button_press_time = 0;
    sm->button_pressed = false;
}

void button_state_machine_update(ButtonStateMachine_t* sm, bool button_input) {
    uint32_t current_time = HAL_GetTick();
    
    switch (sm->current_state) {
        case STATE_IDLE:
            if (button_input) {
                sm->current_state = STATE_BUTTON_PRESSED;
                sm->state_entry_time = current_time;
                sm->button_press_time = current_time;
                sm->button_pressed = true;
            }
            break;
            
        case STATE_BUTTON_PRESSED:
            if (!button_input) {
                sm->current_state = STATE_BUTTON_RELEASED;
                sm->state_entry_time = current_time;
            } else if (current_time - sm->button_press_time > 1000) {
                sm->current_state = STATE_BUTTON_HELD;
                sm->state_entry_time = current_time;
            }
            break;
            
        case STATE_BUTTON_HELD:
            if (!button_input) {
                sm->current_state = STATE_BUTTON_RELEASED;
                sm->state_entry_time = current_time;
            }
            break;
            
        case STATE_BUTTON_RELEASED:
            sm->current_state = STATE_IDLE;
            sm->button_pressed = false;
            break;
    }
}
```

## ⚡ 效能最佳化

### **什麼是效能最佳化？**

效能最佳化涉及改善數位 I/O 操作的效率和回應性。它對即時系統和具有嚴格時序要求的應用至關重要。

### **最佳化概念**

**時序最佳化：**
- **回應時間**：最小化從輸入到輸出的時間
- **輪詢頻率**：最佳化輪詢頻率
- **中斷延遲**：最小化中斷回應時間
- **處理開銷**：降低處理開銷

**記憶體最佳化：**
- **暫存器使用**：高效使用硬體暫存器
- **資料結構**：最佳化的資料結構
- **程式碼大小**：最小化程式碼大小
- **記憶體存取**：高效的記憶體存取模式

### **效能最佳化技術**

#### **中斷驅動 I/O**
```c
// 中斷驅動的按鈕介面
typedef struct {
    GPIO_TypeDef* port;
    uint16_t pin;
    bool pressed;
    void (*callback)(void);
} InterruptButton_t;

void interrupt_button_init(InterruptButton_t* button, GPIO_TypeDef* port, uint16_t pin, 
                          void (*callback)(void)) {
    button->port = port;
    button->pin = pin;
    button->pressed = false;
    button->callback = callback;
    
    // 配置為帶上拉的輸入
    gpio_input_pullup_config(port, pin);
    
    // 配置中斷
    gpio_interrupt_config(port, pin, GPIO_IRQ_FALLING_EDGE);
    gpio_interrupt_enable(port, pin);
}

void interrupt_button_handler(InterruptButton_t* button) {
    if (button->callback) {
        button->callback();
    }
}
```

#### **高效輪詢**
```c
// 多輸入的高效輪詢
typedef struct {
    GPIO_TypeDef* port;
    uint16_t mask;
    uint16_t last_state;
    uint16_t current_state;
} EfficientPoller_t;

void efficient_poller_init(EfficientPoller_t* poller, GPIO_TypeDef* port, uint16_t mask) {
    poller->port = port;
    poller->mask = mask;
    poller->last_state = 0;
    poller->current_state = 0;
}

uint16_t efficient_poller_update(EfficientPoller_t* poller) {
    poller->last_state = poller->current_state;
    poller->current_state = read_multiple_inputs(poller->port, poller->mask);
    return poller->current_state ^ poller->last_state;  // 返回改變的位元
}
```

## 🎯 常見應用

### **什麼是常見的數位 I/O 應用？**

數位 I/O 在嵌入式系統中有無數應用。了解常見應用有助於設計有效的數位 I/O 解決方案。

### **應用類別**

**使用者介面：**
- **按鈕和開關**：使用者輸入裝置
- **LED 指示燈**：狀態和回饋
- **顯示器**：LCD、OLED 和七段顯示器
- **鍵盤**：數字和英數字輸入

**感測器介面：**
- **數位感測器**：溫度、壓力、動作感測器
- **編碼器**：位置和速度回饋
- **開關**：限位開關、安全開關
- **偵測器**：接近、液位和存在偵測器

**致動器控制：**
- **繼電器**：大功率切換
- **馬達**：直流馬達、步進馬達
- **電磁閥**：線性和旋轉致動器
- **閥門**：流體和氣體控制

### **應用範例**

#### **使用者介面系統**
```c
// 完整使用者介面系統
typedef struct {
    DebouncedSwitch_t buttons[4];
    LED_t status_leds[4];
    MatrixKeypad_t keypad;
    SevenSegmentDisplay_t display;
    ButtonStateMachine_t state_machine;
} UserInterface_t;

void user_interface_init(UserInterface_t* ui) {
    // 初始化按鈕
    for (int i = 0; i < 4; i++) {
        switch_init(&ui->buttons[i], GPIOA, i, 50);
        led_init(&ui->status_leds[i], GPIOB, i);
    }
    
    // 初始化鍵盤和顯示器
    keypad_init(&ui->keypad);
    seven_seg_init(&ui->display);
    button_state_machine_init(&ui->state_machine);
}

void user_interface_update(UserInterface_t* ui) {
    // 更新按鈕
    for (int i = 0; i < 4; i++) {
        bool pressed = read_switch_debounced(&ui->buttons[i]);
        if (pressed) {
            led_toggle(&ui->status_leds[i]);
        }
    }
    
    // 更新鍵盤
    char key = keypad_scan(&ui->keypad);
    if (key) {
        // 處理按鍵
        handle_key_press(key);
    }
    
    // 更新顯示器
    seven_seg_update(&ui->display);
}
```

## 🔧 實作

### **完整數位 I/O 程式設計範例**

```c
#include <stdint.h>
#include <stdbool.h>

// 數位 I/O 配置結構
typedef struct {
    GPIO_TypeDef* port;
    uint16_t pin;
    uint8_t mode;  // 0 = 輸入，1 = 輸出
    uint8_t pull;  // 0 = 無，1 = 上拉，2 = 下拉
} dio_config_t;

// 數位 I/O 初始化
void dio_init(const dio_config_t* config) {
    if (config->mode == 0) {
        // 輸入模式
        gpio_input_config(config->port, config->pin);
        if (config->pull == 1) {
            gpio_pullup_config(config->port, config->pin);
        } else if (config->pull == 2) {
            gpio_pulldown_config(config->port, config->pin);
        }
    } else {
        // 輸出模式
        gpio_output_config(config->port, config->pin);
    }
}

// 數位 I/O 讀取
bool dio_read(GPIO_TypeDef* port, uint16_t pin) {
    return (port->IDR >> pin) & 0x01;
}

// 數位 I/O 寫入
void dio_write(GPIO_TypeDef* port, uint16_t pin, bool state) {
    if (state) {
        port->BSRR = (1U << pin);
    } else {
        port->BSRR = (1U << (pin + 16));
    }
}

// 數位 I/O 切換
void dio_toggle(GPIO_TypeDef* port, uint16_t pin) {
    port->ODR ^= (1U << pin);
}

// 消彈開關結構
typedef struct {
    GPIO_TypeDef* port;
    uint16_t pin;
    bool last_state;
    bool current_state;
    uint32_t debounce_time;
    uint32_t last_change_time;
} debounced_switch_t;

// 消彈開關初始化
void debounced_switch_init(debounced_switch_t* sw, GPIO_TypeDef* port, uint16_t pin, uint32_t debounce_ms) {
    sw->port = port;
    sw->pin = pin;
    sw->debounce_time = debounce_ms;
    sw->last_state = false;
    sw->current_state = false;
    sw->last_change_time = 0;
    
    // 配置為帶上拉的輸入
    dio_config_t config = {port, pin, 0, 1};
    dio_init(&config);
}

// 消彈開關讀取
bool debounced_switch_read(debounced_switch_t* sw) {
    bool raw_state = dio_read(sw->port, sw->pin);
    uint32_t current_time = HAL_GetTick();
    
    if (raw_state != sw->last_state) {
        if (current_time - sw->last_change_time > sw->debounce_time) {
            sw->current_state = raw_state;
            sw->last_state = raw_state;
            sw->last_change_time = current_time;
        }
    }
    
    return sw->current_state;
}

// LED 結構
typedef struct {
    GPIO_TypeDef* port;
    uint16_t pin;
    bool state;
} led_t;

// LED 初始化
void led_init(led_t* led, GPIO_TypeDef* port, uint16_t pin) {
    led->port = port;
    led->pin = pin;
    led->state = false;
    
    dio_config_t config = {port, pin, 1, 0};
    dio_init(&config);
    dio_write(port, pin, false);
}

// LED 控制函式
void led_on(led_t* led) {
    dio_write(led->port, led->pin, true);
    led->state = true;
}

void led_off(led_t* led) {
    dio_write(led->port, led->pin, false);
    led->state = false;
}

void led_toggle(led_t* led) {
    led->state = !led->state;
    dio_write(led->port, led->pin, led->state);
}

// 主函式
int main(void) {
    // 初始化系統
    system_init();
    
    // 初始化數位 I/O
    debounced_switch_t button;
    debounced_switch_init(&button, GPIOA, 0, 50);
    
    led_t led;
    led_init(&led, GPIOB, 0);
    
    // 主迴圈
    while (1) {
        // 讀取按鈕
        if (debounced_switch_read(&button)) {
            led_toggle(&led);
        }
        
        // 更新系統
        system_update();
    }
    
    return 0;
}
```

## ⚠️ 常見陷阱

### **1. 缺少消彈**

**問題**：未對機械開關進行消彈
**解決方案**：始終為機械開關實作消彈

```c
// ❌ 錯誤：無消彈
bool read_switch_bad(GPIO_TypeDef* port, uint16_t pin) {
    return dio_read(port, pin);  // 可能因彈跳讀取多次
}

// ✅ 正確：帶消彈
bool read_switch_good(debounced_switch_t* sw) {
    return debounced_switch_read(sw);  // 正確消彈
}
```

### **2. 競爭條件**

**問題**：多執行緒應用中的競爭條件
**解決方案**：使用原子操作或適當的同步機制

```c
// ❌ 錯誤：競爭條件
void toggle_led_bad(led_t* led) {
    led->state = !led->state;  // 非原子操作
    dio_write(led->port, led->pin, led->state);
}

// ✅ 正確：原子操作
void toggle_led_good(led_t* led) {
    dio_toggle(led->port, led->pin);  // 原子操作
    led->state = !led->state;
}
```

### **3. 不正確的上拉/下拉**

**問題**：未配置上拉/下拉電阻
**解決方案**：始終配置適當的上拉/下拉

```c
// ❌ 錯誤：浮接輸入
void bad_input_config(GPIO_TypeDef* port, uint16_t pin) {
    dio_config_t config = {port, pin, 0, 0};  // 無上拉/下拉
    dio_init(&config);
}

// ✅ 正確：帶上拉的輸入
void good_input_config(GPIO_TypeDef* port, uint16_t pin) {
    dio_config_t config = {port, pin, 0, 1};  // 啟用上拉
    dio_init(&config);
}
```

### **4. 效能不佳**

**問題**：低效的輪詢或處理
**解決方案**：使用中斷或高效輪詢

```c
// ❌ 錯誤：低效輪詢
void bad_polling(void) {
    while (1) {
        if (dio_read(GPIOA, 0)) {
            // 處理輸入
        }
        // 無延遲 - 浪費 CPU 週期
    }
}

// ✅ 正確：高效輪詢
void good_polling(void) {
    while (1) {
        if (dio_read(GPIOA, 0)) {
            // 處理輸入
        }
        HAL_Delay(10);  // 合理的輪詢間隔
    }
}
```

## ✅ 最佳實踐

### **1. 始終實作消彈**

- **機械開關**：始終對機械開關進行消彈
- **軟體消彈**：使用計時器和狀態機
- **硬體消彈**：盡可能使用電容和電阻
- **混合方法**：結合硬體和軟體消彈

### **2. 使用原子操作**

- **BSRR 暫存器**：使用 BSRR 進行原子位元操作
- **讀取-修改-寫入**：避免讀取-修改-寫入操作
- **中斷安全**：在中斷處理器中使用原子操作
- **執行緒安全**：在多執行緒程式碼中使用原子操作

### **3. 為效能最佳化**

- **中斷驅動**：使用中斷實現快速回應
- **高效輪詢**：使用合理的輪詢間隔
- **批次操作**：一起處理多個 I/O 操作
- **記憶體存取**：最小化記憶體存取開銷

### **4. 處理錯誤條件**

- **輸入驗證**：驗證所有輸入
- **錯誤恢復**：實作錯誤恢復機制
- **逾時處理**：處理逾時條件
- **故障偵測**：偵測和處理故障

### **5. 為可靠性設計**

- **冗餘**：盡可能使用冗餘輸入
- **容錯**：為容錯設計
- **錯誤報告**：適當地報告錯誤
- **測試**：使用各種條件進行徹底測試

## 🎯 面試問題

### **基本問題**

1. **什麼是數位 I/O 程式設計，為什麼它很重要？**
   - 透過 GPIO 腳位控制和讀取二進位訊號
   - 嵌入式系統與外部世界互動的基礎
   - 對感測器、致動器和使用者介面至關重要
   - 啟用即時控制和監控

2. **數位 I/O 程式設計中的主要挑戰是什麼？**
   - 機械開關的消彈
   - 多執行緒應用中的競爭條件
   - 即時系統的效能最佳化
   - 錯誤處理和容錯

3. **如何實作開關消彈？**
   - 使用計時器延遲狀態變化
   - 實作消彈狀態機
   - 使用電容進行硬體消彈
   - 結合硬體和軟體方法

### **進階問題**

1. **如何設計鍵盤掃描系統？**
   - 使用矩陣掃描技術
   - 實作行/列掃描
   - 處理鬼鍵和按鍵翻轉
   - 使用中斷實現高效掃描

2. **如何最佳化數位 I/O 效能？**
   - 使用中斷驅動 I/O
   - 實作高效輪詢
   - 使用原子操作
   - 最小化處理開銷

3. **如何高效處理多個數位輸入？**
   - 使用位元遮罩處理多個輸入
   - 實作高效輪詢
   - 對關鍵輸入使用中斷
   - 批次處理多個輸入

### **實作問題**

1. **編寫一個實作開關消彈的函式**
2. **實作矩陣鍵盤掃描函式**
3. **創建七段顯示器控制系統**
4. **設計按鈕處理的狀態機**

## 📚 額外資源

### **書籍**
- "The Definitive Guide to ARM Cortex-M3 and Cortex-M4 Processors" by Joseph Yiu
- "Embedded Systems: Introduction to ARM Cortex-M Microcontrollers" by Jonathan Valvano
- "Making Embedded Systems" by Elecia White

### **線上資源**
- [數位 I/O 教學](https://www.tutorialspoint.com/embedded_systems/es_digital_io.htm)
- [GPIO 程式設計](https://developer.arm.com/documentation/dui0552/a/the-cortex-m3-processor/peripherals/gpio)
- [開關消彈](https://www.allaboutcircuits.com/technical-articles/switch-bounce-how-to-deal-with-it/)

### **工具**
- **邏輯分析儀**：數位訊號分析工具
- **示波器**：時序分析工具
- **GPIO 模擬器**：GPIO 模擬工具
- **除錯器**：數位 I/O 除錯工具

### **標準**
- **GPIO 標準**：產業 GPIO 標準
- **電氣標準**：電壓和電流標準
- **時序標準**：數位 I/O 時序標準
- **安全標準**：數位 I/O 安全標準

---

**下一步**：探索 [類比 I/O](./Analog_IO.md) 以了解類比訊號處理，或深入了解 [脈寬調變](./Pulse_Width_Modulation.md) 的 PWM 控制技術。
