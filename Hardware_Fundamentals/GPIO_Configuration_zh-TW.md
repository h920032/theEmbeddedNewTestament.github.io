# 🔌 GPIO 配置

## 快速參考：重點摘要

- **GPIO（通用輸入/輸出）** 提供嵌入式系統可配置的數位 I/O 腳位
- **輸入/輸出模式** 包括數位輸入、數位輸出、類比輸入和替代功能模式
- **配置暫存器** 控制模式、類型、速度、上拉/下拉及驅動強度
- **電氣特性** 包括電壓準位（3.3V/5V）、電流驅動能力和時序
- **保護功能** 包括 ESD 二極體，但過電壓/過電流需要外部保護
- **中斷功能** 支援邊緣觸發和準位觸發中斷
- **驅動強度** 決定電流源出/汲入能力，並影響訊號完整性
- **迴轉速率** 控制訊號上升/下降時間，影響 EMI 和訊號品質

> **掌握嵌入式系統的通用輸入/輸出**  
> 了解 GPIO 模式、配置和實際應用

## 📋 目錄

- [🎯 概述](#-概述)
- [🤔 什麼是 GPIO？](#-什麼是-gpio)
- [🎯 為什麼 GPIO 重要？](#-為什麼-gpio-重要)
- [🧠 GPIO 概念](#-gpio-概念)
- [🔧 GPIO 模式](#-gpio-模式)
- [⚙️ 配置暫存器](#️-配置暫存器)
- [🔌 輸入配置](#-輸入配置)
- [💡 輸出配置](#-輸出配置)
- [🔄 替代功能配置](#-替代功能配置)
- [⚡ 驅動強度與迴轉速率](#-驅動強度與迴轉速率)
- [🔒 上拉/下拉電阻](#-上拉下拉電阻)
- [🎯 常見應用](#-常見應用)
- [🔧 實作](#-實作)
- [⚠️ 常見陷阱](#️-常見陷阱)
- [✅ 最佳實踐](#-最佳實踐)
- [🎯 面試問題](#-面試問題)
- [📚 額外資源](#-額外資源)

---

## 🎯 概述

GPIO（通用輸入/輸出）是嵌入式系統 I/O 的基礎。了解 GPIO 配置對於與外部裝置、感測器和致動器的介面至關重要。

**重點概念：**
- **輸入/輸出模式**：數位輸入、數位輸出、類比輸入、替代功能
- **配置暫存器**：模式、類型、速度、上拉/下拉
- **電氣特性**：驅動強度、迴轉速率、電壓準位
- **中斷功能**：邊緣/準位觸發中斷

### **面試官意圖（他們在探詢什麼）**
- 你能否將資料手冊的腳位模式對應到正確的暫存器設定？
- 你是否了解電氣限制（驅動、上拉、電壓）？
- 你能否解釋安全的啟動預設值和故障模式？

### **🔍 視覺化理解**

#### **GPIO 腳位配置結構**
```
GPIO 腳位配置
┌─────────────────────────────────────────────────────────────┐
│                    GPIO 腳位                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              配置區塊                                │   │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐   │   │
│  │  │ 模式        │ │ 類型        │ │ 速度        │   │   │
│  │  │ (輸入/      │ │ (推挽/      │ │ (低/中/     │   │   │
│  │  │  輸出/      │ │  開漏)      │ │  高)        │   │   │
│  │  │  替代功能)  │ │             │ │             │   │   │
│  │  └─────────────┘ └─────────────┘ └─────────────┘   │   │
│  │                                                     │   │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐   │   │
│  │  │ 上拉/       │ │ 驅動        │ │ 中斷        │   │   │
│  │  │ 下拉        │ │ 強度        │ │ 啟用        │   │   │
│  │  │ (開/關)     │ │ (2/4/8/20mA)│ │ (邊緣/準位) │   │   │
│  │  └─────────────┘ └─────────────┘ └─────────────┘   │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

#### **GPIO 輸入與輸出操作**
```
輸入模式操作
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ 外部        │───▶│ 輸入        │───▶│ 輸入資料    │
│ 訊號        │    │ 緩衝器      │    │ 暫存器      │
│ (0V/3.3V)   │    │ (高阻抗)    │    │ (可讀取)    │
└─────────────┘    └─────────────┘    └─────────────┘

輸出模式操作
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ 輸出資料    │───▶│ 輸出        │───▶│ 外部        │
│ 暫存器      │    │ 驅動器      │    │ 負載        │
│ (可寫入)    │    │ (推挽)      │    │ (LED/繼電器)│
└─────────────┘    └─────────────┘    └─────────────┘
```

#### **GPIO 中斷觸發**
```
中斷觸發模式
┌─────────────────────────────────────────────────────────────┐
│                    上升沿                                    │
│  ┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐                │
│  │     │    │     │    │     │    │     │                │
│  │     │    │     │    │     │    │     │                │
│  └─────┘    └─────┘    └─────┘    └─────┘                │
│     ▲         ▲         ▲         ▲                       │
│     │         │         │         │                       │
│  中斷      中斷      中斷      中斷                      │
│  觸發      觸發      觸發      觸發                      │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    下降沿                                    │
│  ┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐                │
│  │     │    │     │    │     │    │     │                │
│  │     │    │     │    │     │    │     │                │
│  └─────┘    └─────┘    └─────┘    └─────┘                │
│     ▼         ▼         ▼         ▼                       │
│     │         │         │         │                       │
│  中斷      中斷      中斷      中斷                      │
│  觸發      觸發      觸發      觸發                      │
└─────────────────────────────────────────────────────────────┘
```

### **🧠 概念基礎**

#### **GPIO 在嵌入式系統中的角色**
GPIO 作為數位計算世界和物理世界之間的基本介面。它是使嵌入式系統能夠感知環境並控制外部裝置的最基本建構模組。

**關鍵特性：**
- **可配置性**：每個腳位可以動態配置為不同用途
- **即時回應**：對軟體命令和外部事件的立即回應
- **電氣介面**：提供適當的電壓準位和電流驅動能力
- **保護**：內建對常見電氣危害的保護

#### **為什麼 GPIO 配置很重要**
正確的 GPIO 配置對系統可靠性和效能至關重要：

- **訊號完整性**：不正確的驅動強度或迴轉速率會導致訊號劣化
- **功耗效率**：適當的上拉/下拉配置可防止浮接輸入
- **抗雜訊能力**：正確的配置可降低對電氣干擾的敏感度
- **系統可靠性**：適當的保護和配置可防止元件損壞

## 🤔 什麼是 GPIO？

GPIO（通用輸入/輸出）是微控制器或積體電路上的數位訊號腳位，可配置為輸入或輸出。它是嵌入式系統與外部世界互動的最基本和最根本的方式。

### **核心概念**

**數位訊號介面：**
- **二進位狀態**：GPIO 腳位以二進位狀態運作（HIGH/LOW、1/0、ON/OFF）
- **電壓準位**：HIGH 通常為 3.3V 或 5V，LOW 為 0V
- **數位邏輯**：乾淨、抗雜訊的數位訊號
- **可配置方向**：可配置為輸入或輸出

**硬體介面：**
- **實體腳位**：微控制器上的實際實體連接
- **電氣特性**：電壓準位、電流驅動能力、時序
- **保護**：通常僅限於 ESD 二極體/箝位。你必須遵守絕對最大額定值；對於過電壓/過電流條件，需要增加串聯電阻、電壓準位轉換或外部保護。
- **封裝**：腳位排列在封裝中（DIP、QFP、BGA 等）

**軟體控制：**
- **暫存器控制**：透過記憶體映射暫存器控制
- **位元級控制**：個別位元控制個別腳位
- **配置選項**：每個腳位有多個配置選項
- **即時控制**：對軟體命令的立即回應

### **GPIO 與其他 I/O 類型的比較**

**GPIO 與類比 I/O：**
- **GPIO**：數位訊號（離散準位）
- **類比 I/O**：連續電壓準位
- **GPIO**：簡單的二進位操作
- **類比 I/O**：複雜的訊號處理

**GPIO 與專用 I/O：**
- **GPIO**：通用、靈活
- **專用 I/O**：專門建構（UART、SPI、I2C 等）
- **GPIO**：需要手動控制
- **專用 I/O**：硬體輔助協議

**GPIO 與 PWM/ADC：**
- **GPIO**：數位開/關控制
- **PWM**：脈寬調變用於類比式控制
- **ADC**：類比至數位轉換
- **GPIO**：簡單、快速、可靠

## 🎯 為什麼 GPIO 重要？

### **嵌入式系統需求**

**硬體介面：**
- **感測器介面**：讀取數位感測器（按鈕、開關、編碼器）
- **致動器控制**：控制繼電器、馬達、LED、顯示器
- **狀態指示器**：LED 指示燈、狀態燈、警報器
- **使用者介面**：按鈕、開關、鍵盤、觸控感測器

**系統控制：**
- **配置**：設定系統配置選項
- **模式選擇**：選擇不同的操作模式
- **重設控制**：硬體重設和系統控制
- **除錯介面**：除錯訊號和測試點

**即時需求：**
- **快速回應**：對外部事件的立即回應
- **可預測時序**：即時系統的確定性時序
- **中斷功能**：對事件的快速中斷回應
- **低延遲**：輸入和輸出之間的最小延遲

### **真實世界的影響**

**硬體控制：**
```c
// LED 控制 - 簡單但必要
void control_led(bool state) {
    if (state) {
        GPIO_SetPin(GPIOA, 5);  // 開啟 LED
    } else {
        GPIO_ClearPin(GPIOA, 5); // 關閉 LED
    }
}

// 按鈕讀取 - 使用者介面
bool read_button(void) {
    return GPIO_ReadPin(GPIOB, 0);  // 讀取按鈕狀態
}
```

**系統狀態：**
```c
// 系統狀態監控
void check_system_status(void) {
    bool power_good = GPIO_ReadPin(GPIOA, 1);
    bool temperature_ok = GPIO_ReadPin(GPIOA, 2);
    bool communication_active = GPIO_ReadPin(GPIOA, 3);
    
    if (!power_good || !temperature_ok || !communication_active) {
        // 處理系統故障
        handle_system_fault();
    }
}
```

**即時控制：**
```c
// 即時控制範例
void emergency_stop(void) {
    // 對緊急停止按鈕的立即回應
    if (GPIO_ReadPin(GPIOA, 4)) {  // 緊急停止已按下
        GPIO_ClearPin(GPIOA, 5);   // 立即停止馬達
        GPIO_SetPin(GPIOA, 6);     // 啟動警報
    }
}
```

### **GPIO 何時重要**

**高影響場景：**
- 即時控制系統
- 使用者介面應用
- 感測器和致動器介面
- 系統監控和控制
- 除錯和測試介面

**低影響場景：**
- 純計算應用
- 僅網路系統
- 與外部互動最少的系統
- 資源充足的原型系統

## 🧠 核心概念

### **概念：GPIO 電氣操作與訊號完整性**
**重要性**：了解 GPIO 腳位的電氣操作方式對於可靠的系統設計至關重要。不正確的配置可能導致訊號劣化、雜訊問題，甚至元件損壞。

**GPIO 電氣模型**：
GPIO 腳位作為受控開關運作，可以感測外部訊號或驅動外部負載。可靠操作的關鍵在於了解電氣特性並將其與應用需求匹配。

**關鍵電氣考量**：
- **輸入阻抗**：高阻抗輸入對雜訊敏感但需要最小電流
- **輸出驅動能力**：必須匹配負載需求而不超過腳位限制
- **訊號時序**：上升/下降時間影響 EMI 和訊號完整性
- **抗雜訊能力**：適當的配置提供對電氣干擾的抵抗力

**最小範例**：
```c
// 基本 GPIO 配置結構
typedef struct {
    uint8_t mode;           // 輸入/輸出/替代/類比
    uint8_t type;           // 推挽/開漏
    uint8_t speed;          // 低/中/高速
    uint8_t pull_up_down;   // 無上拉/上拉/下拉
} gpio_config_t;

// 簡單 GPIO 配置
void configure_gpio_pin(uint8_t pin, gpio_config_t *config) {
    // 設定腳位模式
    set_pin_mode(pin, config->mode);
    
    // 如果是輸出模式則配置輸出類型和速度
    if (config->mode == GPIO_MODE_OUTPUT) {
        set_output_type(pin, config->type);
        set_output_speed(pin, config->speed);
    }
    
    // 設定上拉/下拉
    set_pull_up_down(pin, config->pull_up_down);
}
```

**試試看**：為不同負載條件配置 GPIO 腳位並量測訊號完整性。

**重點摘要**：
- 驅動強度須匹配負載需求
- 考慮輸入腳位的抗雜訊能力
- 適當的時序配置可減少 EMI
- 始終遵守絕對最大額定值

### **概念：GPIO 內部架構與暫存器組織**
**重要性**：了解 GPIO 腳位的內部結構有助於你做出明智的配置決策並有效地排除問題。

**GPIO 內部結構**：
每個 GPIO 腳位包含多個功能區塊，它們協同工作以提供靈活的 I/O 功能。內部架構決定了腳位的功能和限制。

**關鍵架構元件**：
- **輸入緩衝器**：提供具有施密特觸發器的高阻抗輸入以抗雜訊
- **輸出驅動器**：具有可調強度和類型的可配置驅動器
- **保護電路**：ESD 保護和過電壓箝位
- **配置邏輯**：控制所有腳位行為的暫存器

**最小範例**：
```c
// GPIO 暫存器存取結構
typedef struct {
    volatile uint32_t MODER;    // 模式暫存器
    volatile uint32_t OTYPER;   // 輸出類型暫存器
    volatile uint32_t OSPEEDR;  // 輸出速度暫存器
    volatile uint32_t PUPDR;    // 上拉/下拉暫存器
    volatile uint32_t IDR;      // 輸入資料暫存器
    volatile uint32_t ODR;      // 輸出資料暫存器
} GPIO_TypeDef;

// 使用暫存器配置腳位模式
void set_pin_mode_direct(GPIO_TypeDef *gpio, uint8_t pin, uint8_t mode) {
    // 清除並設定模式位元（每個腳位 2 位元）
    uint32_t mask = 3U << (pin * 2);
    uint32_t value = mode << (pin * 2);
    
    gpio->MODER = (gpio->MODER & ~mask) | value;
}
```

**試試看**：在除錯器中檢查 GPIO 暫存器以了解配置。

**重點摘要**：
- GPIO 腳位具有複雜的內部架構
- 基於暫存器的配置提供靈活性
- 了解暫存器佈局有助於除錯
- 每個配置位元影響特定的腳位行為

### **概念：GPIO 模式選擇與配置策略**
**重要性**：選擇正確的 GPIO 模式對系統可靠性和效能至關重要。不正確的模式選擇可能導致訊號完整性問題、過度功耗，甚至元件損壞。

**模式選擇過程**：
GPIO 模式選擇涉及了解你的應用需求並將其與可用的配置選項匹配。每種模式具有特定的電氣特性和使用案例。

**模式選擇考量**：
- **輸入需求**：浮接輸入對雜訊敏感但低功耗，上拉輸入提供抗雜訊能力
- **輸出需求**：推挽輸出可驅動高和低，開漏輸出需要外部上拉
- **負載特性**：考慮電流需求、電容性負載和切換速度需求
- **雜訊環境**：高雜訊環境受益於適當的上拉/下拉配置

**最小範例**：
```c
// 帶驗證的 GPIO 模式配置
typedef enum {
    GPIO_MODE_INPUT = 0,
    GPIO_MODE_OUTPUT = 1,
    GPIO_MODE_ALTERNATE = 2,
    GPIO_MODE_ANALOG = 3
} gpio_mode_t;

// 帶模式驗證的腳位配置
int configure_gpio_mode(uint8_t pin, gpio_mode_t mode, uint8_t pull_config) {
    // 驗證模式選擇
    if (mode > GPIO_MODE_ANALOG) {
        return -1;  // 無效模式
    }
    
    // 設定模式
    set_pin_mode(pin, mode);
    
    // 為輸入模式配置上拉/下拉
    if (mode == GPIO_MODE_INPUT) {
        set_pull_config(pin, pull_config);
    }
    
    return 0;  // 成功
}
```

**試試看**：為同一腳位配置不同模式並量測電氣特性。

**重點摘要**：
- 模式選擇影響電氣行為和效能
- 選擇輸入配置時考慮雜訊環境
- 輸出模式選擇取決於負載需求
- 始終驗證配置參數

## 🔧 GPIO 模式

### **什麼是 GPIO 模式？**

GPIO 模式定義了腳位的運作方式——無論是輸入、輸出，還是連接到特殊功能。每種模式具有特定的電氣特性和行為。

### **模式概念**

**模式選擇：**
- **輸入模式**：腳位感測外部訊號
- **輸出模式**：腳位驅動外部負載
- **替代功能**：腳位連接到硬體周邊設備
- **類比模式**：腳位連接到類比電路

**模式特性：**
- **電氣行為**：腳位的電氣表現
- **時序特性**：操作的速度和時序
- **負載能力**：腳位可驅動的負載
- **抗雜訊能力**：對電氣雜訊的抵抗力

### **數位輸入模式**
```c
// 將 GPIO 配置為數位輸入
void gpio_input_config(GPIO_TypeDef* GPIOx, uint16_t pin) {
    // 清除模式位元（00 = 輸入模式）
    GPIOx->MODER &= ~(3U << (pin * 2));
    
    // 配置為無上拉/下拉
    GPIOx->PUPDR &= ~(3U << (pin * 2));
}

// 讀取數位輸入
uint8_t gpio_read_input(GPIO_TypeDef* GPIOx, uint16_t pin) {
    return (GPIOx->IDR >> pin) & 0x01;
}
```

### **數位輸出模式**
```c
// 將 GPIO 配置為數位輸出
void gpio_output_config(GPIO_TypeDef* GPIOx, uint16_t pin) {
    // 設定模式位元（01 = 輸出模式）
    GPIOx->MODER &= ~(3U << (pin * 2));
    GPIOx->MODER |= (1U << (pin * 2));
    
    // 配置為推挽
    GPIOx->OTYPER &= ~(1U << pin);
    
    // 配置速度（11 = 非常高速）
    GPIOx->OSPEEDR &= ~(3U << (pin * 2));
    GPIOx->OSPEEDR |= (3U << (pin * 2));
}

// 寫入數位輸出
void gpio_write_output(GPIO_TypeDef* GPIOx, uint16_t pin, uint8_t state) {
    if (state) {
        GPIOx->BSRR = (1U << pin);  // 設定位元
    } else {
        GPIOx->BSRR = (1U << (pin + 16));  // 重設位元
    }
}
```

### **替代功能模式**
```c
// 將 GPIO 配置為替代功能
void gpio_alternate_config(GPIO_TypeDef* GPIOx, uint16_t pin, uint8_t alternate) {
    // 設定模式位元（10 = 替代功能模式）
    GPIOx->MODER &= ~(3U << (pin * 2));
    GPIOx->MODER |= (2U << (pin * 2));
    
    // 配置替代功能
    if (pin < 8) {
        GPIOx->AFR[0] &= ~(0xFU << (pin * 4));
        GPIOx->AFR[0] |= (alternate << (pin * 4));
    } else {
        GPIOx->AFR[1] &= ~(0xFU << ((pin - 8) * 4));
        GPIOx->AFR[1] |= (alternate << ((pin - 8) * 4));
    }
}
```

## ⚙️ 配置暫存器

### **什麼是配置暫存器？**

配置暫存器控制 GPIO 腳位的行為。它們決定每個腳位的模式、電氣特性和行為。

### **暫存器概念**

**暫存器組織：**
- **位元欄位**：每個暫存器包含多個位元欄位
- **腳位映射**：每個腳位在暫存器中有對應的位元
- **配置選項**：每個腳位有多個選項
- **原子操作**：安全的暫存器修改

**暫存器類型：**
- **模式暫存器**：控制腳位方向和模式
- **類型暫存器**：控制輸出類型
- **速度暫存器**：控制輸出速度和驅動強度
- **上拉/下拉暫存器**：控制內部電阻

### **模式暫存器（MODER）**
```c
// 模式暫存器位元定義
#define GPIO_MODE_INPUT     0x00  // 輸入模式
#define GPIO_MODE_OUTPUT    0x01  // 輸出模式
#define GPIO_MODE_ALTERNATE 0x02  // 替代功能模式
#define GPIO_MODE_ANALOG    0x03  // 類比模式

// 配置腳位模式
void gpio_set_mode(GPIO_TypeDef* GPIOx, uint16_t pin, uint8_t mode) {
    // 清除現有模式位元
    GPIOx->MODER &= ~(3U << (pin * 2));
    // 設定新模式位元
    GPIOx->MODER |= (mode << (pin * 2));
}
```

### **輸出類型暫存器（OTYPER）**
```c
// 輸出類型定義
#define GPIO_OTYPE_PUSH_PULL  0x00  // 推挽輸出
#define GPIO_OTYPE_OPEN_DRAIN 0x01  // 開漏輸出

// 配置輸出類型
void gpio_set_output_type(GPIO_TypeDef* GPIOx, uint16_t pin, uint8_t type) {
    if (type == GPIO_OTYPE_OPEN_DRAIN) {
        GPIOx->OTYPER |= (1U << pin);
    } else {
        GPIOx->OTYPER &= ~(1U << pin);
    }
}
```

### **輸出速度暫存器（OSPEEDR）**
```c
// 速度定義
#define GPIO_SPEED_LOW      0x00  // 低速
#define GPIO_SPEED_MEDIUM   0x01  // 中速
#define GPIO_SPEED_HIGH     0x02  // 高速
#define GPIO_SPEED_VERY_HIGH 0x03 // 非常高速

// 配置輸出速度
void gpio_set_speed(GPIO_TypeDef* GPIOx, uint16_t pin, uint8_t speed) {
    GPIOx->OSPEEDR &= ~(3U << (pin * 2));
    GPIOx->OSPEEDR |= (speed << (pin * 2));
}
```

## 🔌 輸入配置

### **什麼是輸入配置？**

輸入配置決定 GPIO 腳位配置為輸入時的行為方式。它包括上拉/下拉電阻、輸入濾波和中斷功能。

### **輸入配置概念**

**輸入特性：**
- **電壓閾值**：邏輯 HIGH 和 LOW 的電壓準位
- **遲滯**：施密特觸發器用於抗雜訊
- **輸入阻抗**：輸入電阻和電容
- **保護**：過電壓和過電流保護

**上拉/下拉電阻：**
- **上拉**：連接到 VCC 的電阻用於預設 HIGH 狀態
- **下拉**：連接到 GND 的電阻用於預設 LOW 狀態
- **浮接**：無內部電阻（需要外部電阻）
- **選擇**：根據外部電路需求選擇

### **輸入配置實作**

#### **基本輸入配置**
```c
// 配置基本輸入
void gpio_input_basic_config(GPIO_TypeDef* GPIOx, uint16_t pin) {
    // 設為輸入模式
    GPIOx->MODER &= ~(3U << (pin * 2));
    
    // 無上拉/下拉
    GPIOx->PUPDR &= ~(3U << (pin * 2));
}
```

#### **帶上拉的輸入**
```c
// 配置帶上拉的輸入
void gpio_input_pullup_config(GPIO_TypeDef* GPIOx, uint16_t pin) {
    // 設為輸入模式
    GPIOx->MODER &= ~(3U << (pin * 2));
    
    // 啟用上拉
    GPIOx->PUPDR &= ~(3U << (pin * 2));
    GPIOx->PUPDR |= (1U << (pin * 2));
}
```

#### **帶下拉的輸入**
```c
// 配置帶下拉的輸入
void gpio_input_pulldown_config(GPIO_TypeDef* GPIOx, uint16_t pin) {
    // 設為輸入模式
    GPIOx->MODER &= ~(3U << (pin * 2));
    
    // 啟用下拉
    GPIOx->PUPDR &= ~(3U << (pin * 2));
    GPIOx->PUPDR |= (2U << (pin * 2));
}
```

## 💡 輸出配置

### **什麼是輸出配置？**

輸出配置決定 GPIO 腳位配置為輸出時的行為方式。它包括輸出類型、驅動強度、速度和電氣特性。

### **輸出配置概念**

**輸出類型：**
- **推挽**：可驅動 HIGH 和 LOW（最常見）
- **開漏**：只能驅動 LOW（需要外部上拉）
- **開源**：只能驅動 HIGH（需要外部下拉）

**驅動特性：**
- **電流驅動**：腳位可源出/汲入的最大電流
- **電壓準位**：輸出 HIGH 和 LOW 的電壓準位
- **迴轉速率**：輸出狀態變化的速度
- **負載能力**：腳位可驅動的負載

### **輸出配置實作**

#### **推挽輸出**
```c
// 配置推挽輸出
void gpio_output_pushpull_config(GPIO_TypeDef* GPIOx, uint16_t pin) {
    // 設為輸出模式
    GPIOx->MODER &= ~(3U << (pin * 2));
    GPIOx->MODER |= (1U << (pin * 2));
    
    // 配置為推挽
    GPIOx->OTYPER &= ~(1U << pin);
}
```

#### **開漏輸出**
```c
// 配置開漏輸出
void gpio_output_opendrain_config(GPIO_TypeDef* GPIOx, uint16_t pin) {
    // 設為輸出模式
    GPIOx->MODER &= ~(3U << (pin * 2));
    GPIOx->MODER |= (1U << (pin * 2));
    
    // 配置為開漏
    GPIOx->OTYPER |= (1U << pin);
}
```

#### **高速輸出**
```c
// 配置高速輸出
void gpio_output_highspeed_config(GPIO_TypeDef* GPIOx, uint16_t pin) {
    // 設為輸出模式
    GPIOx->MODER &= ~(3U << (pin * 2));
    GPIOx->MODER |= (1U << (pin * 2));
    
    // 配置為高速
    GPIOx->OSPEEDR &= ~(3U << (pin * 2));
    GPIOx->OSPEEDR |= (3U << (pin * 2));
}
```

## 🔄 替代功能配置

### **什麼是替代功能配置？**

替代功能配置允許 GPIO 腳位連接到硬體周邊設備，如 UART、SPI、I2C、計時器和其他特殊功能。

### **替代功能概念**

**周邊連接：**
- **硬體路由**：到周邊設備的內部連接
- **功能選擇**：每個腳位有多個功能
- **配置**：特定於周邊設備的配置
- **時序**：硬體控制的時序

**常見替代功能：**
- **UART**：串列通訊
- **SPI**：串列周邊介面
- **I2C**：內部積體電路匯流排
- **計時器**：計時器輸入/輸出
- **ADC**：類比至數位轉換

### **替代功能實作**

#### **UART 配置**
```c
// 為 UART 配置 GPIO
void gpio_uart_config(GPIO_TypeDef* GPIOx, uint16_t tx_pin, uint16_t rx_pin) {
    // 配置 TX 腳位
    gpio_alternate_config(GPIOx, tx_pin, 7);  // AF7 用於 UART
    
    // 配置 RX 腳位
    gpio_alternate_config(GPIOx, rx_pin, 7);  // AF7 用於 UART
}
```

#### **SPI 配置**
```c
// 為 SPI 配置 GPIO
void gpio_spi_config(GPIO_TypeDef* GPIOx, uint16_t sck_pin, uint16_t miso_pin, uint16_t mosi_pin) {
    // 配置 SCK 腳位
    gpio_alternate_config(GPIOx, sck_pin, 5);   // AF5 用於 SPI
    
    // 配置 MISO 腳位
    gpio_alternate_config(GPIOx, miso_pin, 5);  // AF5 用於 SPI
    
    // 配置 MOSI 腳位
    gpio_alternate_config(GPIOx, mosi_pin, 5);  // AF5 用於 SPI
}
```

## ⚡ 驅動強度與迴轉速率

### **什麼是驅動強度與迴轉速率？**

驅動強度和迴轉速率決定 GPIO 腳位可以驅動多少電流以及狀態變化的速度。這些特性對於驅動不同類型的負載至關重要。

### **驅動特性概念**

**驅動強度：**
- **電流能力**：腳位可源出/汲入的最大電流
- **負載驅動**：驅動電容性和電阻性負載的能力
- **功耗**：較高的驅動強度使用更多功率
- **雜訊產生**：較高的驅動強度可能產生更多雜訊

**迴轉速率：**
- **轉換速度**：輸出狀態變化的速度
- **訊號完整性**：對訊號品質的影響
- **EMI 產生**：較快的轉換產生更多 EMI
- **功耗**：較快的轉換使用更多功率

### **驅動強度配置**

#### **低驅動強度**
```c
// 配置低驅動強度
void gpio_low_drive_config(GPIO_TypeDef* GPIOx, uint16_t pin) {
    GPIOx->OSPEEDR &= ~(3U << (pin * 2));
    GPIOx->OSPEEDR |= (0U << (pin * 2));  // 低速
}
```

#### **高驅動強度**
```c
// 配置高驅動強度
void gpio_high_drive_config(GPIO_TypeDef* GPIOx, uint16_t pin) {
    GPIOx->OSPEEDR &= ~(3U << (pin * 2));
    GPIOx->OSPEEDR |= (3U << (pin * 2));  // 非常高速
}
```

## 🔒 上拉/下拉電阻

### **什麼是上拉/下拉電阻？**

上拉和下拉電阻確保 GPIO 腳位在未被主動驅動時具有確定的狀態。它們防止浮接輸入並提供預設邏輯準位。

### **上拉/下拉概念**

**電阻類型：**
- **上拉**：連接到 VCC 的電阻（邏輯 HIGH）
- **下拉**：連接到 GND 的電阻（邏輯 LOW）
- **內部**：內建於微控制器中
- **外部**：為特定需求外部添加

**電阻值：**
- **典型值**：4.7kΩ 至 10kΩ
- **電流消耗**：較高的值消耗較少電流
- **抗雜訊能力**：較低的值提供更好的抗雜訊能力
- **速度**：較低的值允許更快的轉換

### **上拉/下拉配置**

#### **內部上拉**
```c
// 配置內部上拉
void gpio_pullup_config(GPIO_TypeDef* GPIOx, uint16_t pin) {
    GPIOx->PUPDR &= ~(3U << (pin * 2));
    GPIOx->PUPDR |= (1U << (pin * 2));
}
```

#### **內部下拉**
```c
// 配置內部下拉
void gpio_pulldown_config(GPIO_TypeDef* GPIOx, uint16_t pin) {
    GPIOx->PUPDR &= ~(3U << (pin * 2));
    GPIOx->PUPDR |= (2U << (pin * 2));
}
```

#### **無上拉/下拉**
```c
// 配置無上拉/下拉
void gpio_no_pull_config(GPIO_TypeDef* GPIOx, uint16_t pin) {
    GPIOx->PUPDR &= ~(3U << (pin * 2));
}
```

## 🎯 常見應用

### **什麼是常見的 GPIO 應用？**

GPIO 腳位在嵌入式系統中有無數應用。了解常見應用有助於設計有效的 GPIO 解決方案。

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

#### **LED 控制**
```c
// LED 控制應用
typedef struct {
    GPIO_TypeDef* port;
    uint16_t pin;
    bool state;
} led_t;

void led_init(led_t* led, GPIO_TypeDef* port, uint16_t pin) {
    led->port = port;
    led->pin = pin;
    led->state = false;
    
    // 配置為輸出
    gpio_output_config(port, pin);
    gpio_write_output(port, pin, false);
}

void led_toggle(led_t* led) {
    led->state = !led->state;
    gpio_write_output(led->port, led->pin, led->state);
}
```

#### **按鈕介面**
```c
// 按鈕介面應用
typedef struct {
    GPIO_TypeDef* port;
    uint16_t pin;
    bool last_state;
    bool current_state;
} button_t;

void button_init(button_t* button, GPIO_TypeDef* port, uint16_t pin) {
    button->port = port;
    button->pin = pin;
    button->last_state = false;
    button->current_state = false;
    
    // 配置為帶上拉的輸入
    gpio_input_pullup_config(port, pin);
}

bool button_read(button_t* button) {
    button->last_state = button->current_state;
    button->current_state = gpio_read_input(button->port, button->pin);
    return button->current_state;
}

bool button_pressed(button_t* button) {
    return !button->current_state && button->last_state;  // 低電位有效
}
```

## 🔧 實作

### **完整 GPIO 配置範例**

```c
#include <stdint.h>
#include <stdbool.h>

// GPIO 配置結構
typedef struct {
    GPIO_TypeDef* port;
    uint16_t pin;
    uint8_t mode;
    uint8_t type;
    uint8_t speed;
    uint8_t pull;
} gpio_config_t;

// GPIO 模式定義
#define GPIO_MODE_INPUT     0x00
#define GPIO_MODE_OUTPUT    0x01
#define GPIO_MODE_ALTERNATE 0x02
#define GPIO_MODE_ANALOG    0x03

// GPIO 類型定義
#define GPIO_OTYPE_PUSH_PULL  0x00
#define GPIO_OTYPE_OPEN_DRAIN 0x01

// GPIO 速度定義
#define GPIO_SPEED_LOW      0x00
#define GPIO_SPEED_MEDIUM   0x01
#define GPIO_SPEED_HIGH     0x02
#define GPIO_SPEED_VERY_HIGH 0x03

// GPIO 上拉定義
#define GPIO_PULL_NONE      0x00
#define GPIO_PULL_UP        0x01
#define GPIO_PULL_DOWN      0x02

// GPIO 配置函式
void gpio_configure(const gpio_config_t* config) {
    GPIO_TypeDef* GPIOx = config->port;
    uint16_t pin = config->pin;
    
    // 配置模式
    GPIOx->MODER &= ~(3U << (pin * 2));
    GPIOx->MODER |= (config->mode << (pin * 2));
    
    // 配置輸出類型（僅限輸出模式）
    if (config->mode == GPIO_MODE_OUTPUT) {
        if (config->type == GPIO_OTYPE_OPEN_DRAIN) {
            GPIOx->OTYPER |= (1U << pin);
        } else {
            GPIOx->OTYPER &= ~(1U << pin);
        }
    }
    
    // 配置速度（僅限輸出模式）
    if (config->mode == GPIO_MODE_OUTPUT) {
        GPIOx->OSPEEDR &= ~(3U << (pin * 2));
        GPIOx->OSPEEDR |= (config->speed << (pin * 2));
    }
    
    // 配置上拉/下拉
    GPIOx->PUPDR &= ~(3U << (pin * 2));
    GPIOx->PUPDR |= (config->pull << (pin * 2));
}

// GPIO 讀取函式
bool gpio_read(GPIO_TypeDef* GPIOx, uint16_t pin) {
    return (GPIOx->IDR >> pin) & 0x01;
}

// GPIO 寫入函式
void gpio_write(GPIO_TypeDef* GPIOx, uint16_t pin, bool state) {
    if (state) {
        GPIOx->BSRR = (1U << pin);
    } else {
        GPIOx->BSRR = (1U << (pin + 16));
    }
}

// GPIO 切換函式
void gpio_toggle(GPIO_TypeDef* GPIOx, uint16_t pin) {
    GPIOx->ODR ^= (1U << pin);
}

// LED 控制範例
typedef struct {
    GPIO_TypeDef* port;
    uint16_t pin;
    bool state;
} led_t;

void led_init(led_t* led, GPIO_TypeDef* port, uint16_t pin) {
    led->port = port;
    led->pin = pin;
    led->state = false;
    
    gpio_config_t config = {
        .port = port,
        .pin = pin,
        .mode = GPIO_MODE_OUTPUT,
        .type = GPIO_OTYPE_PUSH_PULL,
        .speed = GPIO_SPEED_MEDIUM,
        .pull = GPIO_PULL_NONE
    };
    
    gpio_configure(&config);
    gpio_write(port, pin, false);
}

void led_on(led_t* led) {
    led->state = true;
    gpio_write(led->port, led->pin, true);
}

void led_off(led_t* led) {
    led->state = false;
    gpio_write(led->port, led->pin, false);
}

void led_toggle(led_t* led) {
    led->state = !led->state;
    gpio_write(led->port, led->pin, led->state);
}

// 按鈕介面範例
typedef struct {
    GPIO_TypeDef* port;
    uint16_t pin;
    bool last_state;
    bool current_state;
} button_t;

void button_init(button_t* button, GPIO_TypeDef* port, uint16_t pin) {
    button->port = port;
    button->pin = pin;
    button->last_state = false;
    button->current_state = false;
    
    gpio_config_t config = {
        .port = port,
        .pin = pin,
        .mode = GPIO_MODE_INPUT,
        .type = GPIO_OTYPE_PUSH_PULL,
        .speed = GPIO_SPEED_LOW,
        .pull = GPIO_PULL_UP
    };
    
    gpio_configure(&config);
}

bool button_read(button_t* button) {
    button->last_state = button->current_state;
    button->current_state = gpio_read(button->port, button->pin);
    return button->current_state;
}

bool button_pressed(button_t* button) {
    return !button->current_state && button->last_state;  // 低電位有效
}

// 主函式
int main(void) {
    // 初始化 LED
    led_t led;
    led_init(&led, GPIOA, 5);
    
    // 初始化按鈕
    button_t button;
    button_init(&button, GPIOB, 0);
    
    // 主迴圈
    while (1) {
        if (button_pressed(&button)) {
            led_toggle(&led);
        }
    }
    
    return 0;
}
```

## ⚠️ 常見陷阱

### **1. 浮接輸入**

**問題**：輸入腳位沒有上拉/下拉電阻
**解決方案**：始終為輸入配置上拉/下拉

```c
// ❌ 錯誤：浮接輸入
void bad_input_config(GPIO_TypeDef* GPIOx, uint16_t pin) {
    GPIOx->MODER &= ~(3U << (pin * 2));  // 僅輸入模式
    // 無上拉/下拉 - 浮接！
}

// ✅ 正確：帶上拉的輸入
void good_input_config(GPIO_TypeDef* GPIOx, uint16_t pin) {
    GPIOx->MODER &= ~(3U << (pin * 2));  // 輸入模式
    GPIOx->PUPDR |= (1U << (pin * 2));   // 啟用上拉
}
```

### **2. 不正確的驅動強度**

**問題**：負載的驅動強度不足
**解決方案**：選擇適當的驅動強度

```c
// ❌ 錯誤：大電流負載使用低驅動強度
void bad_drive_config(GPIO_TypeDef* GPIOx, uint16_t pin) {
    GPIOx->OSPEEDR &= ~(3U << (pin * 2));
    GPIOx->OSPEEDR |= (0U << (pin * 2));  // 低速 - 可能無法驅動負載
}

// ✅ 正確：大電流負載使用高驅動強度
void good_drive_config(GPIO_TypeDef* GPIOx, uint16_t pin) {
    GPIOx->OSPEEDR &= ~(3U << (pin * 2));
    GPIOx->OSPEEDR |= (3U << (pin * 2));  // 非常高速
}
```

### **3. 缺少配置**

**問題**：未配置所有必要的暫存器
**解決方案**：配置所有相關暫存器

```c
// ❌ 錯誤：不完整的配置
void bad_config(GPIO_TypeDef* GPIOx, uint16_t pin) {
    GPIOx->MODER |= (1U << (pin * 2));  // 僅輸出模式
    // 缺少類型、速度和上拉配置
}

// ✅ 正確：完整的配置
void good_config(GPIO_TypeDef* GPIOx, uint16_t pin) {
    GPIOx->MODER &= ~(3U << (pin * 2));
    GPIOx->MODER |= (1U << (pin * 2));   // 輸出模式
    GPIOx->OTYPER &= ~(1U << pin);       // 推挽
    GPIOx->OSPEEDR |= (2U << (pin * 2)); // 高速
    GPIOx->PUPDR &= ~(3U << (pin * 2));  // 無上拉
}
```

### **4. 競爭條件**

**問題**：多執行緒應用中的競爭條件
**解決方案**：使用原子操作或適當的同步機制

```c
// ❌ 錯誤：競爭條件
void bad_write(GPIO_TypeDef* GPIOx, uint16_t pin, bool state) {
    if (state) {
        GPIOx->ODR |= (1U << pin);  // 非原子的讀取-修改-寫入
    } else {
        GPIOx->ODR &= ~(1U << pin); // 非原子的讀取-修改-寫入
    }
}

// ✅ 正確：原子操作
void good_write(GPIO_TypeDef* GPIOx, uint16_t pin, bool state) {
    if (state) {
        GPIOx->BSRR = (1U << pin);  // 原子設定
    } else {
        GPIOx->BSRR = (1U << (pin + 16)); // 原子重設
    }
}
```

## ✅ 最佳實踐

### **1. 始終配置上拉/下拉**

- **輸入腳位**：始終為輸入配置上拉/下拉
- **輸出腳位**：通常不需要上拉/下拉
- **浮接腳位**：在生產環境中避免浮接腳位
- **預設狀態**：選擇適當的預設狀態

### **2. 選擇適當的驅動強度**

- **輕負載**：使用低驅動強度以提高功耗效率
- **重負載**：使用高驅動強度以確保可靠操作
- **高速**：使用高驅動強度以實現快速轉換
- **EMI 考量**：較低的驅動強度可減少 EMI

### **3. 使用原子操作**

- **BSRR 暫存器**：使用 BSRR 進行原子位元操作
- **讀取-修改-寫入**：避免讀取-修改-寫入操作
- **中斷安全**：在中斷處理器中使用原子操作
- **執行緒安全**：在多執行緒程式碼中使用原子操作

### **4. 配置所有暫存器**

- **完整配置**：配置所有相關暫存器
- **預設值**：不要依賴預設暫存器值
- **初始化**：始終在使用前初始化 GPIO
- **文件記錄**：記錄 GPIO 配置

### **5. 考慮電氣特性**

- **電壓準位**：確保相容的電壓準位
- **電流限制**：不要超過電流限制
- **時序要求**：考慮時序要求
- **抗雜訊能力**：為抗雜訊設計

## 🎯 面試問題

### **基本問題**

1. **什麼是 GPIO，為什麼它很重要？**
   - 通用輸入/輸出腳位
   - 微控制器與外部世界之間的基本介面
   - 對感測器、致動器和使用者介面至關重要
   - 嵌入式系統 I/O 的基礎

2. **GPIO 有哪些不同的模式？**
   - 輸入模式：腳位感測外部訊號
   - 輸出模式：腳位驅動外部負載
   - 替代功能：腳位連接到硬體周邊設備
   - 類比模式：腳位連接到類比電路

3. **如何配置 GPIO 腳位？**
   - 設定模式暫存器的方向和模式
   - 配置輸出類型（推挽/開漏）
   - 設定速度暫存器的驅動強度
   - 配置上拉/下拉電阻

### **進階問題**

1. **如何為按鈕設計 GPIO 介面？**
   - 配置為帶上拉電阻的輸入
   - 實作消彈（硬體或軟體）
   - 處理按鈕按下的邊緣偵測
   - 考慮中斷功能以實現快速回應

2. **如何最佳化 GPIO 效能？**
   - 使用適當的驅動強度
   - 選擇正確的速度設定
   - 使用原子操作
   - 最小化暫存器存取

3. **如何在多執行緒應用中處理 GPIO？**
   - 使用原子操作確保執行緒安全
   - 實作適當的同步機制
   - 避免競爭條件
   - 考慮中斷安全

### **實作問題**

1. **編寫一個將 GPIO 配置為帶上拉輸入的函式**
2. **使用原子操作實作 GPIO 切換函式**
3. **創建 GPIO 配置結構和初始化函式**
4. **設計一個具有淡入淡出功能的 LED GPIO 介面**

## 🧪 引導式實驗

### 實驗 1：基本 GPIO 配置與控制
1. **設定**：為輸入和輸出模式配置 GPIO 腳位
2. **測試**：使用三用電表和示波器驗證腳位行為
3. **分析**：量測電壓準位、電流消耗和時序特性
4. **最佳化**：調整驅動強度和速度設定以獲得最佳效能

### 實驗 2：GPIO 中斷實作
1. **配置**：在 GPIO 輸入腳位上設定邊緣觸發中斷
2. **實作**：為按鈕按下編寫中斷服務常式
3. **測試**：量測中斷延遲和回應時間
4. **除錯**：使用邏輯分析儀驗證中斷時序

### 實驗 3：GPIO 保護與穩健性
1. **設計**：實作過電壓/過電流的外部保護電路
2. **測試**：施加壓力條件並量測保護效果
3. **驗證**：使用各種負載條件和雜訊源進行測試
4. **文件記錄**：建立穩健 GPIO 實作的設計指南

## ✅ 自我檢查

### 理解檢查
- [ ] 你能解釋推挽和開漏輸出之間的區別嗎？
- [ ] 你了解上拉/下拉電阻如何影響輸入行為嗎？
- [ ] 你能描述驅動強度和 EMI 之間的關係嗎？
- [ ] 你知道如何為給定負載計算適當的驅動強度嗎？

### 應用檢查
- [ ] 你能為不同的輸入和輸出場景配置 GPIO 嗎？
- [ ] 你能實作帶有正確邊緣偵測的 GPIO 中斷嗎？
- [ ] 你能為惡劣環境設計保護電路嗎？
- [ ] 你能為特定應用最佳化 GPIO 配置嗎？

### 分析檢查
- [ ] 你能分析 GPIO 訊號完整性問題嗎？
- [ ] 你能量測和最佳化 GPIO 時序特性嗎？
- [ ] 你能排除 GPIO 配置問題嗎？
- [ ] 你能為工業應用設計穩健的 GPIO 介面嗎？

## 🔗 交叉連結

- **[數位 I/O 程式設計](./Digital_IO_Programming.md)** - 實用的 GPIO 應用和程式設計技術
- **[外部中斷](./External_Interrupts.md)** - GPIO 中斷處理和邊緣偵測
- **[電源管理](./Power_Management.md)** - GPIO 功耗和最佳化
- **[硬體抽象層](./Hardware_Abstraction_Layer.md)** - GPIO 抽象和可攜性
- **[時脈管理](./Clock_Management.md)** - GPIO 時脈配置和時序

## 📚 額外資源

### **書籍**
- "The Definitive Guide to ARM Cortex-M3 and Cortex-M4 Processors" by Joseph Yiu
- "Embedded Systems: Introduction to ARM Cortex-M Microcontrollers" by Jonathan Valvano
- "Making Embedded Systems" by Elecia White

### **線上資源**
- [GPIO 教學](https://www.tutorialspoint.com/embedded_systems/es_gpio.htm)
- [ARM GPIO 文件](https://developer.arm.com/documentation/dui0552/a/the-cortex-m3-processor/peripherals/gpio)
- [STM32 GPIO 指南](https://www.st.com/resource/en/user_manual/dm00031936-stm32f0xxx-peripheral-controls-stmicroelectronics.pdf)

### **工具**
- **GPIO 模擬器**：GPIO 模擬工具
- **邏輯分析儀**：GPIO 訊號分析工具
- **示波器**：GPIO 時序分析工具
- **除錯器**：GPIO 除錯工具

### **標準**
- **GPIO 標準**：產業 GPIO 標準
- **電氣標準**：電壓和電流標準
- **時序標準**：GPIO 時序標準
- **安全標準**：GPIO 安全標準

---

**下一步**：探索 [數位 I/O 程式設計](./Digital_IO_Programming.md) 以了解數位輸入/輸出應用，或深入了解 [類比 I/O](./Analog_IO.md) 的類比訊號處理。
