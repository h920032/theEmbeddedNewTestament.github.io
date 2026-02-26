# ⚡ 電源管理

## 快速參考：關鍵要點

- **電源管理**對於電池供電的嵌入式系統和節能應用至關重要
- **電源模式**包括主動、閒置、睡眠和深度睡眠狀態，各有不同的電流消耗特性
- **睡眠模式**透過停用未使用的周邊設備和降低時脈頻率來減少功耗
- **喚醒來源**包括外部中斷、計時器、看門狗計時器和周邊設備事件
- **電源最佳化**技術包括時脈閘控、周邊設備停用和動態頻率調整
- **電池管理**涉及監控電壓、電流和電量狀態以達到最佳運作
- **功率預算**需要測量並分配不同系統狀態下的功耗
- **能源效率**以每次操作的微焦耳來衡量，而非僅僅是電流消耗

> **為電池供電和節能嵌入式系統最佳化功耗**  
> 學習實作睡眠模式、喚醒來源和電源最佳化技術

---

## 📋 **目錄**

- [概述](#overview)
- [電源模式](#power-modes)
- [睡眠模式](#sleep-modes)
- [喚醒來源](#wake-up-sources)
- [電源最佳化](#power-optimization)
- [時脈管理](#clock-management)
- [周邊設備電源管理](#peripheral-power-management)
- [電池管理](#battery-management)
- [電源監控](#power-monitoring)
- [最佳實踐](#best-practices)
- [常見陷阱](#common-pitfalls)
- [範例](#examples)
- [面試問題](#interview-questions)

---

## 🎯 **概述**

電源管理對於電池供電的嵌入式系統和節能應用至關重要。有效的電源管理可延長電池壽命、減少發熱，並實現可攜式和 IoT 裝置。

### 概念：按狀態和按喚醒分配功率預算

功率由您的狀態機管理：定義主動/閒置/睡眠狀態，確定已知的電流和喚醒來源。要測量，不要猜測。

### 最小範例
```c
typedef enum { RUN, IDLE, SLEEP } pm_state_t;
void enter_idle(void){ /* 降低時脈，閘控周邊設備 */ }
void enter_sleep(void){ /* 無滴答閒置，停止時脈，啟用喚醒 */ }
```

### 重點總結
- 當延遲預算允許時使用無滴答閒置；驗證喚醒來源。
- 閘控未使用的時脈/周邊設備；停用會漏電的上拉電阻。
- 量化每事件能量（每次感測器讀取的微安培）以比較設計方案。

---

## 🔍 視覺化理解

### **電源狀態轉換**
```
電源狀態機
┌─────────────────────────────────────────────────────────────┐
│                      電源狀態轉換                           │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐   │
│  │    主動     │───▶│    閒置     │───▶│    睡眠     │   │
│  │ (全功率)    │    │(降低        │    │(最低        │   │
│  │             │    │ 功率)       │    │ 功率)       │   │
│  └─────────────┘    └─────────────┘    └─────────────┘   │
│         ▲                   ▲                   ▲         │
│         │                   │                   │         │
│         └───────────────────┴───────────────────┘         │
│                      喚醒事件                              │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐         │
│  │   計時器    │ │    外部     │ │   周邊設備  │         │
│  │    中斷    │ │    中斷     │ │    事件     │         │
│  └─────────────┘ └─────────────┘ └─────────────┘         │
└─────────────────────────────────────────────────────────────┘
```

### **功耗特性圖**
```
功耗 vs. 時間
功率 (mW)
   ^
   │    ┌─────────────────────────────────────────┐
   │    │              主動模式                   │
   │    │         (全功率運作)                    │
   │    └─────────────────────────────────────────┘
   │
   │    ┌─────────────────────────────────────────┐
   │    │              閒置模式                   │
   │    │         (降低功率狀態)                  │
   │    └─────────────────────────────────────────┘
   │
   │    ┌─────────────────────────────────────────┐
   │    │              睡眠模式                   │
   │    │         (最低功率狀態)                  │
   │    └─────────────────────────────────────────┘
   │
   +───────────────────────────────────────────────> 時間
   │<->│  喚醒    │<->│  主動     │<->│  睡眠    │
```

### **時脈閘控與功率降低**
```
用於電源最佳化的時脈閘控
┌─────────────────────────────────────────────────────────────┐
│                    時脈閘控控制                              │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐         │
│  │   模組 1    │ │   模組 2    │ │   模組 3    │         │
│  │  時脈閘控   │ │  時脈閘控   │ │  時脈閘控   │         │
│  │    [開]     │ │    [關]     │ │    [開]     │         │
│  └─────────────┘ └─────────────┘ └─────────────┘         │
│         │               │               │                 │
│         ▼               ▼               ▼                 │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐         │
│  │    啟用     │ │    停用     │ │    啟用     │         │
│  │ (消耗       │ │ (無功率     │ │ (消耗       │         │
│  │   電力)     │ │  消耗)      │ │   電力)     │         │
│  └─────────────┘ └─────────────┘ └─────────────┘         │
└─────────────────────────────────────────────────────────────┘
```

### **🧠 概念基礎**

#### **電源管理的挑戰**
嵌入式系統中的電源管理涉及在效能需求和能量限制之間取得平衡。與市電供電的系統不同，電池供電的裝置必須仔細管理每一微焦耳的能量，以最大化運作壽命。

**關鍵特性：**
- **能量預算**：有限的能量儲存需要在各系統狀態間謹慎分配
- **動態調整**：系統必須根據當前需求調整功耗
- **喚醒延遲**：省電與回應時間之間的取捨
- **狀態管理**：複雜的狀態機管理電源模式之間的轉換

#### **為什麼電源管理很重要**
有效的電源管理對現代嵌入式系統至關重要：

- **電池壽命**：適當的電源管理可將電池壽命延長 10 倍或更多
- **散熱管理**：降低功耗可減少發熱
- **成本降低**：較低的功率需求可使用更小、更便宜的電源供應器
- **環境影響**：節能系統可減少環境足跡

#### **功率與效能的取捨**
電源管理涉及必須仔細考慮的根本取捨：

- **主動 vs. 睡眠**：較高的效能需要更多功率，睡眠模式可省電但會增加延遲
- **頻率 vs. 效率**：較高的時脈頻率可提升效能但會增加功耗
- **周邊設備管理**：啟用更多周邊設備可提升功能但會增加功率消耗
- **喚醒策略**：快速喚醒來源消耗更多功率但提供更好的回應能力

## 🧪 實作練習
1) 電源狀態測量
- 使用萬用表或功率分析儀測量不同電源狀態下的電流消耗。

2) 喚醒來源測試
- 測試不同的喚醒來源並測量喚醒時間和功耗。

## ✅ 自我檢查
- 您如何計算系統的總功率預算？
- 何時應該使用深度睡眠而非淺度睡眠模式？

## 🔗 交叉連結
- `Hardware_Fundamentals/Clock_Management.md` 了解時脈閘控
- `Hardware_Fundamentals/Watchdog_Timers.md` 了解喚醒來源

### **關鍵概念**
- **睡眠模式** - 用於節能的不同電源狀態
- **喚醒來源** - 將系統從睡眠中喚醒的事件
- **電源最佳化** - 最小化功耗的技術
- **電池管理** - 監控和最佳化電池使用

---

## 🔄 **電源模式**

### **1. 主動模式**
系統全功率運作，所有周邊設備已啟用。

```c
// 主動模式配置
typedef struct {
    uint32_t cpu_frequency;      // CPU 頻率（Hz）
    bool peripherals_enabled;    // 所有周邊設備已啟用
    uint32_t power_consumption;  // 功耗（mW）
} active_mode_config_t;

// 主動模式功耗
void configure_active_mode(active_mode_config_t *config) {
    // 設定 CPU 頻率
    set_cpu_frequency(config->cpu_frequency);
    
    // 啟用所有需要的周邊設備
    if (config->peripherals_enabled) {
        enable_all_peripherals();
    }
    
    // 監控功耗
    monitor_power_consumption();
}
```

### **2. 睡眠模式**
降低功耗，部分周邊設備已停用。

```c
// 睡眠模式配置
typedef struct {
    sleep_mode_t mode;           // 睡眠模式類型
    uint32_t wake_up_time;       // 喚醒時間（ms）
    wake_up_source_t sources;    // 喚醒來源
    bool peripherals_disabled;   // 停用未使用的周邊設備
} sleep_mode_config_t;

// 睡眠模式類型
typedef enum {
    SLEEP_MODE_LIGHT,    // 淺度睡眠 - CPU 停止，周邊設備仍啟用
    SLEEP_MODE_DEEP,     // 深度睡眠 - CPU 和大部分周邊設備停止
    SLEEP_MODE_STANDBY,  // 待機 - 僅備份域啟用
    SLEEP_MODE_HIBERNATE // 休眠 - 僅 RTC 啟用
} sleep_mode_t;
```

### **3. 電源狀態轉換**

```c
// 電源狀態轉換
typedef enum {
    POWER_STATE_ACTIVE,
    POWER_STATE_SLEEP,
    POWER_STATE_DEEP_SLEEP,
    POWER_STATE_STANDBY
} power_state_t;

// 電源狀態管理
typedef struct {
    power_state_t current_state;
    power_state_t target_state;
    uint32_t transition_time;
    bool transition_in_progress;
} power_state_manager_t;

// 轉換到電源狀態
void transition_to_power_state(power_state_t target_state) {
    power_state_manager_t *pm = get_power_state_manager();
    
    if (pm->current_state != target_state) {
        // 準備轉換
        prepare_power_transition(target_state);
        
        // 執行轉換
        execute_power_transition(target_state);
        
        // 更新狀態
        pm->current_state = target_state;
    }
}
```

---

## 😴 **睡眠模式**

### **1. 淺度睡眠模式**
CPU 停止但周邊設備和記憶體保持啟用。

```c
// 淺度睡眠模式實作
void enter_light_sleep(void) {
    // 儲存當前狀態
    save_system_state();
    
    // 停用 CPU
    __WFI(); // 等待中斷
    
    // 喚醒後恢復狀態
    restore_system_state();
}

// 淺度睡眠配置
void configure_light_sleep(void) {
    // 配置喚醒來源
    configure_wake_up_sources();
    
    // 設定睡眠模式
    SCB->SCR |= SCB_SCR_SLEEPDEEP_Msk;
    
    // 啟用睡眠模式
    __enable_irq();
}
```

### **2. 深度睡眠模式**
CPU 和大部分周邊設備停止，僅保留必要功能。

```c
// 深度睡眠模式實作
void enter_deep_sleep(void) {
    // 儲存關鍵資料
    save_critical_data();
    
    // 停用未使用的周邊設備
    disable_unused_peripherals();
    
    // 配置深度睡眠
    SCB->SCR |= SCB_SCR_SLEEPDEEP_Msk;
    
    // 進入深度睡眠
    __WFI();
    
    // 喚醒後恢復
    restore_critical_data();
    enable_peripherals();
}

// 深度睡眠配置
void configure_deep_sleep(void) {
    // 配置喚醒來源
    configure_deep_sleep_wake_up();
    
    // 設定深度睡眠模式
    PWR->CR |= PWR_CR_LPDS;
    
    // 配置電壓調整
    PWR->CR |= PWR_CR_VOS;
}
```

### **3. 待機模式**
僅備份域和 RTC 啟用，其他所有功能停止。

```c
// 待機模式實作
void enter_standby_mode(void) {
    // 將關鍵資料儲存到備份暫存器
    save_to_backup_registers();
    
    // 配置待機模式
    PWR->CR |= PWR_CR_CWUF;
    PWR->CR |= PWR_CR_PDDS;
    
    // 進入待機
    SCB->SCR |= SCB_SCR_SLEEPDEEP_Msk;
    __WFI();
    
    // 系統將在喚醒後重置
}

// 待機模式配置
void configure_standby_mode(void) {
    // 配置 RTC 作為喚醒來源
    configure_rtc_wake_up();
    
    // 啟用備份域
    RCC->APB1ENR |= RCC_APB1ENR_PWREN;
    PWR->CR |= PWR_CR_DBP;
}
```

---

## 🔔 **喚醒來源**

### **1. 外部中斷**

```c
// 外部中斷喚醒配置
typedef struct {
    uint8_t pin;
    edge_type_t edge;
    bool enabled;
} external_wake_up_config_t;

// 配置外部喚醒
void configure_external_wake_up(external_wake_up_config_t *config) {
    // 配置 GPIO 為輸入
    configure_gpio_input(config->pin);
    
    // 配置中斷
    configure_external_interrupt(config->pin, config->edge);
    
    // 啟用喚醒功能
    EXTI->IMR |= (1 << config->pin);
    
    // 在 NVIC 中啟用
    NVIC_EnableIRQ(EXTI0_IRQn + config->pin);
}

// 外部喚醒處理常式
void external_wake_up_handler(void) {
    // 清除喚醒旗標
    PWR->CR |= PWR_CR_CWUF;
    
    // 處理喚醒事件
    process_wake_up_event();
}
```

### **2. 計時器喚醒**

```c
// 計時器喚醒配置
typedef struct {
    uint32_t wake_up_time_ms;
    timer_type_t timer_type;
    bool enabled;
} timer_wake_up_config_t;

// 配置計時器喚醒
void configure_timer_wake_up(timer_wake_up_config_t *config) {
    // 配置計時器
    configure_timer(config->timer_type, config->wake_up_time_ms);
    
    // 啟用計時器中斷
    enable_timer_interrupt(config->timer_type);
    
    // 配置為喚醒來源
    configure_timer_wake_up_source(config->timer_type);
}

// 計時器喚醒處理常式
void timer_wake_up_handler(void) {
    // 清除計時器中斷
    clear_timer_interrupt();
    
    // 處理計時器喚醒
    process_timer_wake_up();
}
```

### **3. RTC 喚醒**

```c
// RTC 喚醒配置
typedef struct {
    uint32_t wake_up_time;
    rtc_wake_up_source_t source;
    bool enabled;
} rtc_wake_up_config_t;

// 配置 RTC 喚醒
void configure_rtc_wake_up(rtc_wake_up_config_t *config) {
    // 配置 RTC
    configure_rtc();
    
    // 設定喚醒時間
    set_rtc_wake_up_time(config->wake_up_time);
    
    // 啟用 RTC 喚醒
    RTC->CR |= RTC_CR_WUTE;
    
    // 啟用 RTC 中斷
    NVIC_EnableIRQ(RTC_IRQn);
}

// RTC 喚醒處理常式
void rtc_wake_up_handler(void) {
    // 清除 RTC 喚醒旗標
    RTC->ISR &= ~RTC_ISR_WUTF;
    
    // 處理 RTC 喚醒
    process_rtc_wake_up();
}
```

---

## ⚡ **電源最佳化**

### **1. CPU 電源最佳化**

```c
// CPU 電源最佳化
typedef struct {
    uint32_t frequency;
    voltage_scale_t voltage;
    bool dynamic_scaling;
} cpu_power_config_t;

// 配置 CPU 電源
void configure_cpu_power(cpu_power_config_t *config) {
    // 設定電壓調整
    set_voltage_scaling(config->voltage);
    
    // 設定 CPU 頻率
    set_cpu_frequency(config->frequency);
    
    // 啟用動態頻率調整
    if (config->dynamic_scaling) {
        enable_dynamic_frequency_scaling();
    }
}

// 動態頻率調整
void enable_dynamic_frequency_scaling(void) {
    // 監控 CPU 負載
    uint32_t cpu_load = get_cpu_load();
    
    if (cpu_load < 30) {
        // 低負載時降低頻率
        set_cpu_frequency(CPU_FREQ_LOW);
    } else if (cpu_load > 80) {
        // 高負載時提高頻率
        set_cpu_frequency(CPU_FREQ_HIGH);
    }
}
```

### **2. 周邊設備電源最佳化**

```c
// 周邊設備電源管理
typedef struct {
    peripheral_type_t peripheral;
    bool enabled;
    uint32_t power_consumption;
} peripheral_power_config_t;

// 停用未使用的周邊設備
void disable_unused_peripherals(void) {
    // 停用未使用的 UART
    if (!uart1_used) {
        disable_peripheral(UART1);
    }
    
    // 停用未使用的計時器
    if (!timer1_used) {
        disable_peripheral(TIM1);
    }
    
    // 停用未使用的 ADC
    if (!adc1_used) {
        disable_peripheral(ADC1);
    }
}

// 僅在需要時啟用周邊設備
void enable_peripheral_on_demand(peripheral_type_t peripheral) {
    // 啟用周邊設備
    enable_peripheral(peripheral);
    
    // 使用周邊設備
    use_peripheral(peripheral);
    
    // 使用後停用周邊設備
    disable_peripheral(peripheral);
}
```

### **3. 記憶體電源最佳化**

```c
// 記憶體電源最佳化
typedef struct {
    bool flash_power_down;
    bool sram_retention;
    bool cache_enabled;
} memory_power_config_t;

// 配置記憶體電源
void configure_memory_power(memory_power_config_t *config) {
    if (config->flash_power_down) {
        // 不使用時關閉快閃記憶體電源
        power_down_flash();
    }
    
    if (config->sram_retention) {
        // 在睡眠模式下啟用 SRAM 保持
        enable_sram_retention();
    }
    
    if (config->cache_enabled) {
        // 啟用快取以獲得更好的效能
        enable_cache();
    }
}
```

---

## ⏰ **時脈管理**

### **1. 時脈配置**

```c
// 時脈配置
typedef struct {
    uint32_t system_clock;
    uint32_t peripheral_clock;
    bool pll_enabled;
    clock_source_t source;
} clock_config_t;

// 配置系統時脈
void configure_system_clock(clock_config_t *config) {
    // 如果啟用則配置 PLL
    if (config->pll_enabled) {
        configure_pll(config->system_clock);
    }
    
    // 設定系統時脈
    set_system_clock(config->system_clock);
    
    // 配置周邊設備時脈
    configure_peripheral_clocks(config->peripheral_clock);
}

// 動態時脈調整
void dynamic_clock_scaling(void) {
    uint32_t current_load = get_system_load();
    
    if (current_load < 20) {
        // 低負載 - 降低時脈頻率
        set_system_clock(SYSTEM_CLOCK_LOW);
    } else if (current_load > 80) {
        // 高負載 - 提高時脈頻率
        set_system_clock(SYSTEM_CLOCK_HIGH);
    }
}
```

### **2. 時脈閘控**

```c
// 用於省電的時脈閘控
void enable_clock_gating(void) {
    // 閘控未使用的周邊設備時脈
    RCC->AHB1ENR &= ~(RCC_AHB1ENR_GPIOAEN | RCC_AHB1ENR_GPIOBEN);
    RCC->APB1ENR &= ~(RCC_APB1ENR_USART2EN | RCC_APB1ENR_TIM3EN);
    RCC->APB2ENR &= ~(RCC_APB2ENR_USART1EN | RCC_APB2ENR_TIM1EN);
}

// 僅在需要時啟用時脈
void enable_peripheral_clock(peripheral_type_t peripheral) {
    switch (peripheral) {
        case PERIPHERAL_UART1:
            RCC->APB2ENR |= RCC_APB2ENR_USART1EN;
            break;
        case PERIPHERAL_TIM1:
            RCC->APB2ENR |= RCC_APB2ENR_TIM1EN;
            break;
        case PERIPHERAL_ADC1:
            RCC->APB2ENR |= RCC_APB2ENR_ADC1EN;
            break;
    }
}
```

---

## 🔋 **電池管理**

### **1. 電池監控**

```c
// 電池監控
typedef struct {
    uint32_t voltage;
    uint32_t capacity;
    uint32_t remaining;
    battery_status_t status;
} battery_info_t;

// 電池狀態
typedef enum {
    BATTERY_STATUS_UNKNOWN,
    BATTERY_STATUS_CHARGING,
    BATTERY_STATUS_DISCHARGING,
    BATTERY_STATUS_FULL,
    BATTERY_STATUS_LOW,
    BATTERY_STATUS_CRITICAL
} battery_status_t;

// 監控電池
void monitor_battery(void) {
    battery_info_t battery;
    
    // 讀取電池電壓
    battery.voltage = read_battery_voltage();
    
    // 計算剩餘容量
    battery.remaining = calculate_battery_capacity(battery.voltage);
    
    // 更新電池狀態
    battery.status = get_battery_status(battery.voltage);
    
    // 處理低電量
    if (battery.status == BATTERY_STATUS_CRITICAL) {
        handle_critical_battery();
    }
}
```

### **2. 電池最佳化**

```c
// 電池最佳化策略
void optimize_for_battery_life(void) {
    // 降低 CPU 頻率
    set_cpu_frequency(CPU_FREQ_LOW);
    
    // 停用未使用的周邊設備
    disable_unused_peripherals();
    
    // 啟用睡眠模式
    enable_sleep_modes();
    
    // 最佳化通訊
    optimize_communication_power();
    
    // 降低感測器取樣率
    reduce_sensor_sampling_rate();
}

// 嚴重低電量處理
void handle_critical_battery(void) {
    // 儲存關鍵資料
    save_critical_data();
    
    // 進入深度睡眠模式
    enter_deep_sleep();
    
    // 僅配置關鍵事件的喚醒
    configure_critical_wake_up_sources();
}
```

---

## 📊 **電源監控**

### **1. 功耗監控**

```c
// 功耗監控
typedef struct {
    uint32_t current_consumption;
    uint32_t average_consumption;
    uint32_t peak_consumption;
    uint32_t total_energy;
} power_consumption_t;

// 監控功耗
void monitor_power_consumption(void) {
    power_consumption_t power;
    
    // 讀取當前消耗
    power.current_consumption = read_current_consumption();
    
    // 更新平均消耗
    update_average_consumption(power.current_consumption);
    
    // 檢查峰值消耗
    if (power.current_consumption > power.peak_consumption) {
        power.peak_consumption = power.current_consumption;
    }
    
    // 計算總能量
    power.total_energy += power.current_consumption;
    
    // 記錄功耗
    log_power_consumption(&power);
}
```

### **2. 功率分析**

```c
// 功率分析
typedef struct {
    uint32_t timestamp;
    power_state_t state;
    uint32_t consumption;
    uint32_t duration;
} power_profile_entry_t;

// 功率分析
void profile_power_consumption(void) {
    static power_profile_entry_t profile[MAX_PROFILE_ENTRIES];
    static uint8_t profile_index = 0;
    
    // 記錄功率分析條目
    profile[profile_index].timestamp = get_system_tick();
    profile[profile_index].state = get_current_power_state();
    profile[profile_index].consumption = read_current_consumption();
    profile[profile_index].duration = calculate_duration();
    
    // 遞增索引
    profile_index = (profile_index + 1) % MAX_PROFILE_ENTRIES;
    
    // 分析功率特性
    analyze_power_profile(profile, profile_index);
}
```

---

## 🎯 **最佳實踐**

### **1. 電源管理指南**

```c
// 電源管理檢查清單
/*
    □ 使用適當的睡眠模式
    □ 停用未使用的周邊設備
    □ 最佳化時脈頻率
    □ 實作動態電源調整
    □ 監控功耗
    □ 處理電池管理
    □ 使用高效的喚醒來源
    □ 最佳化通訊協定
    □ 實作功率感知排程
    □ 測試功耗
*/

// 良好的電源管理範例
void good_power_management(void) {
    // 配置電源管理
    configure_power_management();
    
    // 帶有電源最佳化的主迴圈
    while (1) {
        // 處理任務
        process_tasks();
        
        // 檢查系統是否可以進入睡眠
        if (can_enter_sleep_mode()) {
            // 進入睡眠模式
            enter_sleep_mode();
        }
        
        // 監控功耗
        monitor_power_consumption();
    }
}
```

### **2. 睡眠模式指南**

```c
// 睡眠模式檢查清單
/*
    □ 選擇適當的睡眠模式
    □ 配置喚醒來源
    □ 儲存關鍵資料
    □ 停用未使用的周邊設備
    □ 處理喚醒事件
    □ 恢復系統狀態
    □ 監控睡眠持續時間
    □ 測試睡眠模式功能
    □ 記錄睡眠行為
    □ 考慮安全需求
*/

// 良好的睡眠模式實作
void good_sleep_mode(void) {
    // 儲存系統狀態
    save_system_state();
    
    // 配置喚醒來源
    configure_wake_up_sources();
    
    // 進入睡眠模式
    enter_sleep_mode();
    
    // 喚醒後恢復系統狀態
    restore_system_state();
    
    // 處理喚醒事件
    process_wake_up_events();
}
```

---

## ⚠️ **常見陷阱**

### **1. 低效的睡眠模式**

```c
// 錯誤：未使用睡眠模式
void bad_no_sleep(void) {
    while (1) {
        // 處理任務
        process_tasks();
        
        // 始終處於啟用狀態 - 浪費電力
        delay_ms(100);
    }
}

// 正確：使用睡眠模式
void good_sleep_usage(void) {
    while (1) {
        // 處理任務
        process_tasks();
        
        // 閒置時進入睡眠模式
        if (is_system_idle()) {
            enter_sleep_mode();
        }
    }
}
```

### **2. 未使用的周邊設備**

```c
// 錯誤：未停用未使用的周邊設備
void bad_unused_peripherals(void) {
    // 啟用所有周邊設備
    enable_all_peripherals();
    
    // 僅使用部分周邊設備
    use_some_peripherals();
    
    // 讓未使用的周邊設備保持啟用
}

// 正確：停用未使用的周邊設備
void good_peripheral_management(void) {
    // 僅啟用需要的周邊設備
    enable_needed_peripherals();
    
    // 使用周邊設備
    use_peripherals();
    
    // 不需要時停用
    disable_unused_peripherals();
}
```

### **3. 低效的時脈管理**

```c
// 錯誤：固定高頻率
void bad_fixed_clock(void) {
    // 始終使用高頻率
    set_cpu_frequency(CPU_FREQ_HIGH);
    
    // 處理任務
    process_tasks();
}

// 正確：動態時脈調整
void good_clock_management(void) {
    // 根據負載調整時脈
    uint32_t load = get_cpu_load();
    
    if (load < 30) {
        set_cpu_frequency(CPU_FREQ_LOW);
    } else if (load > 80) {
        set_cpu_frequency(CPU_FREQ_HIGH);
    }
    
    // 處理任務
    process_tasks();
}
```

---

## 💡 **範例**

### **1. 簡單的電源管理**

```c
// 簡單的電源管理實作
void simple_power_management(void) {
    // 配置電源管理
    configure_power_management();
    
    while (1) {
        // 處理應用程式任務
        process_application_tasks();
        
        // 檢查系統是否可以進入睡眠
        if (is_system_idle()) {
            // 進入淺度睡眠
            enter_light_sleep();
        }
        
        // 監控電池
        monitor_battery();
    }
}

// 系統閒置檢查
bool is_system_idle(void) {
    // 檢查是否沒有待處理的任務
    if (task_queue_empty() && !communication_pending()) {
        return true;
    }
    
    return false;
}
```

### **2. 進階電源管理**

```c
// 具有多種模式的進階電源管理
typedef enum {
    POWER_MODE_ACTIVE,
    POWER_MODE_LIGHT_SLEEP,
    POWER_MODE_DEEP_SLEEP,
    POWER_MODE_STANDBY
} power_mode_t;

void advanced_power_management(void) {
    power_mode_t current_mode = POWER_MODE_ACTIVE;
    
    while (1) {
        // 決定最佳電源模式
        power_mode_t optimal_mode = determine_optimal_power_mode();
        
        // 轉換到最佳模式
        if (optimal_mode != current_mode) {
            transition_to_power_mode(optimal_mode);
            current_mode = optimal_mode;
        }
        
        // 根據模式處理任務
        switch (current_mode) {
            case POWER_MODE_ACTIVE:
                process_active_tasks();
                break;
            case POWER_MODE_LIGHT_SLEEP:
                process_light_sleep_tasks();
                break;
            case POWER_MODE_DEEP_SLEEP:
                process_deep_sleep_tasks();
                break;
            case POWER_MODE_STANDBY:
                process_standby_tasks();
                break;
        }
    }
}

// 決定最佳電源模式
power_mode_t determine_optimal_power_mode(void) {
    uint32_t battery_level = get_battery_level();
    uint32_t system_load = get_system_load();
    
    if (battery_level < 20) {
        return POWER_MODE_DEEP_SLEEP;
    } else if (system_load < 10) {
        return POWER_MODE_LIGHT_SLEEP;
    } else {
        return POWER_MODE_ACTIVE;
    }
}
```

### **3. 電池最佳化系統**

```c
// 電池最佳化系統
void battery_optimized_system(void) {
    // 配置電池最佳化
    configure_battery_optimization();
    
    while (1) {
        // 監控電池電量
        uint32_t battery_level = get_battery_level();
        
        if (battery_level < 10) {
            // 嚴重低電量 - 進入深度睡眠
            enter_critical_battery_mode();
        } else if (battery_level < 30) {
            // 低電量 - 降低功耗
            enter_low_battery_mode();
        } else {
            // 正常電量 - 標準運作
            enter_normal_mode();
        }
        
        // 根據電池電量處理任務
        process_tasks_based_on_battery(battery_level);
    }
}

// 嚴重低電量模式
void enter_critical_battery_mode(void) {
    // 停用非必要周邊設備
    disable_non_essential_peripherals();
    
    // 降低 CPU 頻率
    set_cpu_frequency(CPU_FREQ_MIN);
    
    // 以最少喚醒來源進入深度睡眠
    configure_minimal_wake_up_sources();
    enter_deep_sleep();
}
```

---

## 🎯 **面試問題**

### **基礎問題**
1. **嵌入式系統中有哪些不同的睡眠模式？**
   - 淺度睡眠：CPU 停止，周邊設備仍啟用
   - 深度睡眠：CPU 和大部分周邊設備停止
   - 待機：僅備份域啟用
   - 休眠：僅 RTC 啟用

2. **您如何在嵌入式系統中實作電源管理？**
   - 使用適當的睡眠模式
   - 停用未使用的周邊設備
   - 最佳化時脈頻率
   - 監控功耗

3. **常見的喚醒來源有哪些？**
   - 外部中斷
   - 計時器中斷
   - RTC 鬧鐘
   - 通訊事件

### **進階問題**
4. **您如何為電池供電的裝置最佳化功耗？**
   - 實作動態電源調整
   - 使用高效的睡眠模式
   - 最佳化通訊協定
   - 監控和管理電池使用

5. **電源管理中有哪些取捨？**
   - 效能 vs 功耗
   - 回應時間 vs 睡眠持續時間
   - 功能性 vs 電池壽命
   - 成本 vs 電源效率

6. **您如何在即時系統中處理電源管理？**
   - 確保滿足時序需求
   - 使用適當的喚醒來源
   - 平衡省電與回應能力
   - 徹底測試電源管理

### **實作問題**
7. **為 IoT 裝置設計電源管理系統。**
   ```c
   void iot_power_management(void) {
       // 配置 IoT 運作
       configure_iot_power_management();
       
       while (1) {
           // 處理 IoT 任務
           process_iot_tasks();
           
           // 檢查是否需要通訊
           if (communication_needed()) {
               enable_communication();
               send_data();
               disable_communication();
           }
           
           // 進入睡眠模式
           enter_sleep_mode();
       }
   }
   ```

8. **實作電池監控系統。**
   ```c
   void battery_monitoring_system(void) {
       // 配置電池監控
       configure_battery_monitoring();
       
       while (1) {
           // 監控電池
           uint32_t battery_level = get_battery_level();
           
           // 處理不同的電池電量
           if (battery_level < 10) {
               handle_critical_battery();
           } else if (battery_level < 30) {
               handle_low_battery();
           }
           
           // 記錄電池狀態
           log_battery_status(battery_level);
           
           // 按監控間隔睡眠
           delay_ms(BATTERY_MONITOR_INTERVAL);
       }
   }
   ```

---

## 🔗 **相關主題**

- **[外部中斷](./External_Interrupts.md)** - 邊緣/位準觸發中斷、防彈跳
- **[看門狗計時器](./Watchdog_Timers.md)** - 系統監控與恢復機制
- **[中斷與例外](./Interrupts_Exceptions.md)** - 中斷處理、ISR 設計、中斷延遲
- **[時脈管理](./Clock_Management.md)** - 系統時脈配置、PLL 設定

---

## 📚 **資源**

### **書籍**
- "Making Embedded Systems" by Elecia White
- "Programming Embedded Systems" by Michael Barr
- "Real-Time Systems" by Jane W. S. Liu

### **線上資源**
- [ARM Cortex-M Power Management](https://developer.arm.com/documentation/dui0552/a/the-cortex-m3-processor/power-management)
- [STM32 Power Management](https://www.st.com/resource/en/reference_manual/dm00031020-stm32f405-415-stm32f407-417-stm32f427-437-and-stm32f429-439-advanced-arm-based-32-bit-mcus-stmicroelectronics.pdf)

---

**下一個主題：** [時脈管理](./Clock_Management.md) → [重置管理](./Reset_Management.md)
