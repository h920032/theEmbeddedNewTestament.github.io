# ⏱️ 脈寬調變

## 快速參考：重點摘要

- **脈寬調變（PWM）** 透過快速在開/關狀態之間切換來控制功率傳遞
- **佔空比** 是訊號為高電位的時間百分比，控制平均功率輸出
- **頻率** 決定切換速率，影響效率、雜訊和解析度
- **解析度** 是離散佔空比等級的數量，與頻率成反比
- **計時器硬體** 使用比較暫存器和輸出比較模式產生 PWM
- **應用** 包括馬達控制、LED 調光、電源供應器和音訊產生
- **濾波** 可將 PWM 轉換為類比訊號，有效地建立 DAC
- **權衡** 存在於頻率（雜訊）、解析度（精度）和效率之間

> **掌握嵌入式系統的 PWM**  
> PWM 產生、頻率控制、佔空比和實際應用

## 📋 目錄

- [🎯 概述](#-概述)
- [🔧 PWM 基礎](#-pwm-基礎)
- [⚙️ PWM 配置](#️-pwm-配置)
- [🎛️ 佔空比控制](#️-佔空比控制)
- [🔄 頻率控制](#-頻率控制)
- [📊 PWM 應用](#-pwm-應用)
- [⚡ 進階 PWM 技術](#-進階-pwm-技術)
- [🎯 常見應用](#-常見應用)
- [⚠️ 常見陷阱](#️-常見陷阱)
- [✅ 最佳實踐](#-最佳實踐)
- [🎯 面試問題](#-面試問題)
- [📚 額外資源](#-額外資源)

---

## 🎯 概述

### 概念：佔空比、頻率和解析度是耦合的

PWM 是計時器比較硬體。你選擇的週期（ARR）和時脈預分頻器設定了頻率和佔空比解析度。將這些與致動器需求和 EMC 約束匹配。

### 最小範例
```c
// 如果時脈允許，將 PWM 設為 20 kHz、10 位元解析度
void pwm_init(void){ /* 設定 PSC/ARR；配置 CCx 模式；啟用輸出 */ }
void pwm_set_duty(uint16_t duty){ /* 寫入 CCRx，箝位到 ARR */ }
```

### 試試看
1. 掃描頻率，將切換雜訊移出敏感頻段。
2. 評估馬達/LED 上的線性度 vs 死區時間/驅動延遲。

### 重點摘要
- 頻率 vs 解析度的權衡由計時器時脈決定。
- 對半橋使用互補輸出和死區時間。
- 濾波後的 PWM（RC）表現如同 DAC；設計濾波器。

### **面試官意圖（他們在探詢什麼）**
- 你能否從計時器設定計算 PWM 頻率/解析度？
- 你是否了解權衡：雜訊 vs 解析度 vs 效率？
- 你能否解釋實際致動器約束（死區時間、驅動器限制）？

---

## 🔍 視覺化理解

### **PWM 訊號特性**
```
PWM 訊號參數
┌─────────────────────────────────────────────────────────────┐
│                    PWM 訊號分析                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              高頻 PWM                                │   │
│  │  ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐       │   │
│  │  │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │       │   │
│  │  │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │       │   │
│  │  └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘       │   │
│  │  │<->│ 週期  │<->│ 週期  │<->│ 週期  │<->│        │   │
│  │  │<->│ 佔空比│<->│ 佔空比│<->│ 佔空比│<->│        │   │
│  └─────────────────────────────────────────────────────┘   │
│                            │                               │
│                            ▼                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              低頻 PWM                                │   │
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────┐ │   │
│  │  │             │    │             │    │         │ │   │
│  │  │             │    │             │    │         │ │   │
│  │  └─────────────┘    └─────────────┘    └─────────┘ │   │
│  │  │<---------->│ 週期  │<---------->│ 週期  │<->│  │   │
│  │  │<---------->│ 佔空比│<---------->│ 佔空比│<->│  │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### **佔空比與功率控制**
```
佔空比 vs 功率輸出
┌─────────────────────────────────────────────────────────────┐
│                    佔空比控制                               │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐         │
│  │  25% 佔空比 │ │  50% 佔空比 │ │  75% 佔空比 │         │
│  │   ┌─┐ ┌─┐  │ │  ┌───┐ ┌───┐│ │ ┌─────┐ ┌─┐│         │
│  │   │ │ │ │  │ │  │   │ │   │ │ │ │     │ │ ││         │
│  │   │ │ │ │  │ │  │   │ │   │ │ │ │     │ │ ││         │
│  │   └─┘ └─┘  │ │  └───┘ └───┘│ │ └─────┘ └─┘│         │
│  │   │<->│     │ │  │<--->│<--->│ │ │<----->│<->│         │
│  │   │   │     │ │  │     │     │ │ │       │   │         │
│  └─────────────┘ └─────────────┘ └─────────────┘         │
│         │               │               │                 │
│         ▼               ▼               ▼                 │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐         │
│  │   低        │ │  中等       │ │   高        │         │
│  │   功率      │ │   功率      │ │   功率      │         │
│  │  (25% 平均) │ │  (50% 平均) │ │  (75% 平均) │         │
│  └─────────────┘ └─────────────┘ └─────────────┘         │
└─────────────────────────────────────────────────────────────┘
```

### **PWM 濾波與 DAC 效果**
```
PWM 至類比轉換
┌─────────────────────────────────────────────────────────────┐
│                    PWM 濾波過程                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              原始 PWM 訊號                          │   │
│  │  ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐       │   │
│  │  │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │       │   │
│  │  │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │       │   │
│  │  └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘       │   │
│  └─────────────────────────────────────────────────────┘   │
│                            │                               │
│                            ▼                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              RC 低通濾波器                           │   │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐   │   │
│  │  │     R       │ │     C       │ │             │   │   │
│  │  │             │ │             │ │             │   │   │
│  │  └─────────────┘ └─────────────┘ └─────────────┘   │   │
│  └─────────────────────────────────────────────────────┘   │
│                            │                               │
│                            ▼                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              濾波後的類比輸出                        │   │
│  │  ┌─────────────────────────────────────────────┐   │   │
│  │  │                                             │   │   │
│  │  │                                             │   │   │
│  │  │                                             │   │   │
│  │  │                                             │   │   │
│  │  └─────────────────────────────────────────────┘   │   │
│  │  │<---------->│ 平均值    │<---------->│            │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### **🧠 概念基礎**

#### **PWM 原理**
脈寬調變代表數位控制類比系統的基本技術。透過快速在高和低狀態之間切換數位訊號，PWM 建立一個有效的類比輸出，其平均值與佔空比成正比。

**關鍵特性：**
- **數位控制**：PWM 使用數位訊號控制類比功率等級
- **效率**：切換操作最小化控制元件中的功率損耗
- **靈活性**：佔空比和頻率可獨立控制
- **可擴展性**：相同技術適用於毫瓦到千瓦

#### **為什麼 PWM 重要**
PWM 對現代嵌入式系統至關重要：

- **功率控制**：啟用馬達速度、LED 亮度和功率轉換的精確控制
- **效率**：切換操作比線性控制方法更高效
- **數位整合**：PWM 可由微控制器計時器直接產生
- **雜訊控制**：頻率選擇可將切換雜訊移離敏感頻段
- **成本效益**：PWM 控制通常比類比替代方案更便宜

#### **PWM 設計挑戰**
設計有效的 PWM 系統涉及平衡多個相互競爭的需求：

- **頻率選擇**：較高頻率提供更平滑的輸出但增加切換損耗
- **解析度 vs 頻率**：計時器約束在精度和切換速率之間產生權衡
- **雜訊管理**：需要選擇切換頻率以最小化干擾
- **濾波器設計**：RC 濾波器可將 PWM 轉換為類比但引入延遲和漣波

## 🔧 PWM 基礎

### **PWM 參數**
```c
// PWM 參數結構
typedef struct {
    uint32_t frequency;      // PWM 頻率（Hz）
    uint32_t period;         // 計時器週期值
    uint32_t prescaler;      // 計時器預分頻器值
    uint16_t duty_cycle;     // 佔空比（0-100）
    uint16_t resolution;     // PWM 解析度（位元）
} PWM_Config_t;

// 計算 PWM 參數
void pwm_calculate_parameters(PWM_Config_t* config, uint32_t clock_freq, uint32_t target_freq) {
    // 計算目標頻率的預分頻器和週期
    uint32_t psc = 1;
    uint32_t arr = clock_freq / target_freq;
    
    // 如果週期太大則調整預分頻器
    while (arr > 65535 && psc < 65535) {
        psc++;
        arr = (clock_freq / psc) / target_freq;
    }
    
    config->frequency = target_freq;
    config->period = arr - 1;
    config->prescaler = psc - 1;
    config->resolution = 16; // 假設 16 位元計時器
}
```

### **佔空比計算**
```c
// 將佔空比百分比轉換為計時器比較值
uint16_t duty_cycle_to_compare(uint16_t duty_cycle, uint16_t period) {
    return (uint16_t)((duty_cycle * period) / 100);
}

// 將計時器比較值轉換為佔空比百分比
uint16_t compare_to_duty_cycle(uint16_t compare_value, uint16_t period) {
    return (uint16_t)((compare_value * 100) / period);
}

// 設定 PWM 佔空比
void pwm_set_duty_cycle(TIM_HandleTypeDef* htim, uint32_t channel, uint16_t duty_cycle) {
    uint16_t compare_value = duty_cycle_to_compare(duty_cycle, htim->Init.Period);
    __HAL_TIM_SET_COMPARE(htim, channel, compare_value);
}
```

---

## ⚙️ PWM 配置

### **基本 PWM 配置**
```c
// 配置計時器用於 PWM 輸出
void pwm_timer_config(TIM_HandleTypeDef* htim, PWM_Config_t* config) {
    TIM_OC_InitTypeDef sConfigOC = {0};
    
    // 配置計時器基礎
    htim->Instance = TIM2;
    htim->Init.Prescaler = config->prescaler;
    htim->Init.CounterMode = TIM_COUNTERMODE_UP;
    htim->Init.Period = config->period;
    htim->Init.ClockDivision = TIM_CLOCKDIVISION_DIV1;
    htim->Init.AutoReloadPreload = TIM_AUTORELOAD_PRELOAD_DISABLE;
    
    HAL_TIM_Base_Init(htim);
    
    // 配置 PWM 通道
    sConfigOC.OCMode = TIM_OCMODE_PWM1;
    sConfigOC.Pulse = 0;
    sConfigOC.OCPolarity = TIM_OCPOLARITY_HIGH;
    sConfigOC.OCFastMode = TIM_OCFAST_DISABLE;
    
    HAL_TIM_PWM_ConfigChannel(htim, &sConfigOC, TIM_CHANNEL_1);
    
    // 啟動 PWM
    HAL_TIM_PWM_Start(htim, TIM_CHANNEL_1);
}
```

### **多通道 PWM 配置**
```c
// 配置多個 PWM 通道
typedef struct {
    TIM_HandleTypeDef* htim;
    uint32_t channels[4];
    uint8_t channel_count;
    PWM_Config_t config;
} MultiChannelPWM_t;

void multi_channel_pwm_init(MultiChannelPWM_t* mpwm, TIM_HandleTypeDef* htim,
                           uint32_t* channels, uint8_t count, uint32_t frequency) {
    mpwm->htim = htim;
    mpwm->channel_count = count;
    
    for (int i = 0; i < count; i++) {
        mpwm->channels[i] = channels[i];
    }
    
    // 計算 PWM 參數
    pwm_calculate_parameters(&mpwm->config, SystemCoreClock, frequency);
    
    // 配置計時器基礎
    htim->Instance = TIM2;
    htim->Init.Prescaler = mpwm->config.prescaler;
    htim->Init.CounterMode = TIM_COUNTERMODE_UP;
    htim->Init.Period = mpwm->config.period;
    htim->Init.ClockDivision = TIM_CLOCKDIVISION_DIV1;
    htim->Init.AutoReloadPreload = TIM_AUTORELOAD_PRELOAD_DISABLE;
    
    HAL_TIM_Base_Init(htim);
    
    // 配置每個通道
    TIM_OC_InitTypeDef sConfigOC = {0};
    sConfigOC.OCMode = TIM_OCMODE_PWM1;
    sConfigOC.Pulse = 0;
    sConfigOC.OCPolarity = TIM_OCPOLARITY_HIGH;
    sConfigOC.OCFastMode = TIM_OCFAST_DISABLE;
    
    for (int i = 0; i < count; i++) {
        HAL_TIM_PWM_ConfigChannel(htim, &sConfigOC, channels[i]);
        HAL_TIM_PWM_Start(htim, channels[i]);
    }
}
```

### **帶中斷的 PWM**
```c
// 配置帶更新中斷的 PWM
void pwm_with_interrupt_config(TIM_HandleTypeDef* htim, PWM_Config_t* config) {
    // 配置計時器基礎
    htim->Instance = TIM2;
    htim->Init.Prescaler = config->prescaler;
    htim->Init.CounterMode = TIM_COUNTERMODE_UP;
    htim->Init.Period = config->period;
    htim->Init.ClockDivision = TIM_CLOCKDIVISION_DIV1;
    htim->Init.AutoReloadPreload = TIM_AUTORELOAD_PRELOAD_DISABLE;
    
    HAL_TIM_Base_Init(htim);
    
    // 配置 PWM 通道
    TIM_OC_InitTypeDef sConfigOC = {0};
    sConfigOC.OCMode = TIM_OCMODE_PWM1;
    sConfigOC.Pulse = 0;
    sConfigOC.OCPolarity = TIM_OCPOLARITY_HIGH;
    sConfigOC.OCFastMode = TIM_OCFAST_DISABLE;
    
    HAL_TIM_PWM_ConfigChannel(htim, &sConfigOC, TIM_CHANNEL_1);
    
    // 啟用更新中斷
    __HAL_TIM_ENABLE_IT(htim, TIM_IT_UPDATE);
    HAL_NVIC_SetPriority(TIM2_IRQn, 0, 0);
    HAL_NVIC_EnableIRQ(TIM2_IRQn);
    
    // 啟動 PWM
    HAL_TIM_PWM_Start(htim, TIM_CHANNEL_1);
}

// PWM 中斷處理器
void TIM2_IRQHandler(void) {
    if (__HAL_TIM_GET_FLAG(&htim2, TIM_FLAG_UPDATE) != RESET) {
        if (__HAL_TIM_GET_IT_SOURCE(&htim2, TIM_IT_UPDATE) != RESET) {
            __HAL_TIM_CLEAR_IT(&htim2, TIM_IT_UPDATE);
            
            // 處理 PWM 更新
            pwm_update_callback();
        }
    }
}
```

---

## 🎛️ 佔空比控制

### **平滑佔空比轉換**
```c
// 平滑佔空比轉換
typedef struct {
    TIM_HandleTypeDef* htim;
    uint32_t channel;
    uint16_t current_duty;
    uint16_t target_duty;
    uint16_t step_size;
    uint32_t transition_time;
    uint32_t last_update_time;
} PWM_Transition_t;

void pwm_transition_init(PWM_Transition_t* transition, TIM_HandleTypeDef* htim,
                        uint32_t channel, uint16_t step_size, uint32_t transition_time_ms) {
    transition->htim = htim;
    transition->channel = channel;
    transition->current_duty = 0;
    transition->target_duty = 0;
    transition->step_size = step_size;
    transition->transition_time = transition_time_ms;
    transition->last_update_time = 0;
}

void pwm_transition_set_target(PWM_Transition_t* transition, uint16_t target_duty) {
    transition->target_duty = target_duty;
}

void pwm_transition_update(PWM_Transition_t* transition) {
    uint32_t current_time = HAL_GetTick();
    
    if (current_time - transition->last_update_time >= transition->transition_time) {
        if (transition->current_duty < transition->target_duty) {
            transition->current_duty += transition->step_size;
            if (transition->current_duty > transition->target_duty) {
                transition->current_duty = transition->target_duty;
            }
        } else if (transition->current_duty > transition->target_duty) {
            transition->current_duty -= transition->step_size;
            if (transition->current_duty < transition->target_duty) {
                transition->current_duty = transition->target_duty;
            }
        }
        
        pwm_set_duty_cycle(transition->htim, transition->channel, transition->current_duty);
        transition->last_update_time = current_time;
    }
}
```

### **佔空比漸變**
```c
// 佔空比漸變
typedef struct {
    TIM_HandleTypeDef* htim;
    uint32_t channel;
    uint16_t start_duty;
    uint16_t end_duty;
    uint16_t current_duty;
    uint16_t step_size;
    uint32_t ramp_time;
    uint32_t step_time;
    uint32_t last_step_time;
    uint8_t ramping;
} PWM_Ramp_t;

void pwm_ramp_init(PWM_Ramp_t* ramp, TIM_HandleTypeDef* htim, uint32_t channel,
                   uint16_t step_size, uint32_t ramp_time_ms) {
    ramp->htim = htim;
    ramp->channel = channel;
    ramp->step_size = step_size;
    ramp->ramp_time = ramp_time_ms;
    ramp->ramping = 0;
    ramp->current_duty = 0;
}

void pwm_ramp_start(PWM_Ramp_t* ramp, uint16_t start_duty, uint16_t end_duty) {
    ramp->start_duty = start_duty;
    ramp->end_duty = end_duty;
    ramp->current_duty = start_duty;
    ramp->ramping = 1;
    ramp->last_step_time = HAL_GetTick();
    
    // 計算步進時間
    uint16_t total_steps = abs(end_duty - start_duty) / ramp->step_size;
    ramp->step_time = ramp->ramp_time / total_steps;
    
    // 設定初始佔空比
    pwm_set_duty_cycle(ramp->htim, ramp->channel, ramp->current_duty);
}

void pwm_ramp_update(PWM_Ramp_t* ramp) {
    if (!ramp->ramping) return;
    
    uint32_t current_time = HAL_GetTick();
    
    if (current_time - ramp->last_step_time >= ramp->step_time) {
        if (ramp->current_duty < ramp->end_duty) {
            ramp->current_duty += ramp->step_size;
            if (ramp->current_duty > ramp->end_duty) {
                ramp->current_duty = ramp->end_duty;
                ramp->ramping = 0;
            }
        } else if (ramp->current_duty > ramp->end_duty) {
            ramp->current_duty -= ramp->step_size;
            if (ramp->current_duty < ramp->end_duty) {
                ramp->current_duty = ramp->end_duty;
                ramp->ramping = 0;
            }
        }
        
        pwm_set_duty_cycle(ramp->htim, ramp->channel, ramp->current_duty);
        ramp->last_step_time = current_time;
    }
}
```

---

## 🔄 頻率控制

### **動態頻率控制**
```c
// 動態頻率控制
typedef struct {
    TIM_HandleTypeDef* htim;
    uint32_t current_frequency;
    uint32_t target_frequency;
    uint32_t min_frequency;
    uint32_t max_frequency;
} PWM_Frequency_Control_t;

void pwm_frequency_control_init(PWM_Frequency_Control_t* freq_ctrl, TIM_HandleTypeDef* htim,
                               uint32_t min_freq, uint32_t max_freq) {
    freq_ctrl->htim = htim;
    freq_ctrl->min_frequency = min_freq;
    freq_ctrl->max_frequency = max_freq;
    freq_ctrl->current_frequency = min_freq;
    freq_ctrl->target_frequency = min_freq;
}

void pwm_set_frequency(PWM_Frequency_Control_t* freq_ctrl, uint32_t frequency) {
    if (frequency < freq_ctrl->min_frequency) {
        frequency = freq_ctrl->min_frequency;
    } else if (frequency > freq_ctrl->max_frequency) {
        frequency = freq_ctrl->max_frequency;
    }
    
    freq_ctrl->target_frequency = frequency;
    
    // 計算新的計時器參數
    PWM_Config_t config;
    pwm_calculate_parameters(&config, SystemCoreClock, frequency);
    
    // 更新計時器配置
    freq_ctrl->htim->Init.Prescaler = config.prescaler;
    freq_ctrl->htim->Init.Period = config.period;
    
    HAL_TIM_Base_Init(freq_ctrl->htim);
    freq_ctrl->current_frequency = frequency;
}
```

### **頻率掃描**
```c
// 頻率掃描
typedef struct {
    PWM_Frequency_Control_t* freq_ctrl;
    uint32_t start_frequency;
    uint32_t end_frequency;
    uint32_t sweep_time;
    uint32_t current_time;
    uint8_t sweeping;
} PWM_Frequency_Sweep_t;

void pwm_frequency_sweep_init(PWM_Frequency_Sweep_t* sweep, PWM_Frequency_Control_t* freq_ctrl,
                             uint32_t sweep_time_ms) {
    sweep->freq_ctrl = freq_ctrl;
    sweep->sweep_time = sweep_time_ms;
    sweep->sweeping = 0;
}

void pwm_frequency_sweep_start(PWM_Frequency_Sweep_t* sweep, uint32_t start_freq, uint32_t end_freq) {
    sweep->start_frequency = start_freq;
    sweep->end_frequency = end_freq;
    sweep->current_time = 0;
    sweep->sweeping = 1;
    
    // 設定初始頻率
    pwm_set_frequency(sweep->freq_ctrl, start_freq);
}

void pwm_frequency_sweep_update(PWM_Frequency_Sweep_t* sweep) {
    if (!sweep->sweeping) return;
    
    sweep->current_time += 10; // 每 10ms 更新
    
    // 計算當前頻率
    float progress = (float)sweep->current_time / sweep->sweep_time;
    if (progress > 1.0f) progress = 1.0f;
    
    uint32_t current_freq = sweep->start_frequency + 
                           (uint32_t)(progress * (sweep->end_frequency - sweep->start_frequency));
    
    pwm_set_frequency(sweep->freq_ctrl, current_freq);
    
    if (progress >= 1.0f) {
        sweep->sweeping = 0;
    }
}
```

---

## 📊 PWM 應用

### **LED 調光**
```c
// LED 調光控制
typedef struct {
    TIM_HandleTypeDef* htim;
    uint32_t channel;
    uint16_t brightness;
    uint8_t fade_direction;
    uint16_t fade_speed;
} LED_Dimmer_t;

void led_dimmer_init(LED_Dimmer_t* dimmer, TIM_HandleTypeDef* htim, uint32_t channel) {
    dimmer->htim = htim;
    dimmer->channel = channel;
    dimmer->brightness = 0;
    dimmer->fade_direction = 0; // 0 = 淡入，1 = 淡出
    dimmer->fade_speed = 1;
}

void led_dimmer_set_brightness(LED_Dimmer_t* dimmer, uint16_t brightness) {
    dimmer->brightness = brightness;
    pwm_set_duty_cycle(dimmer->htim, dimmer->channel, brightness);
}

void led_dimmer_fade(LED_Dimmer_t* dimmer) {
    if (dimmer->fade_direction == 0) {
        // 淡入
        dimmer->brightness += dimmer->fade_speed;
        if (dimmer->brightness >= 100) {
            dimmer->brightness = 100;
            dimmer->fade_direction = 1;
        }
    } else {
        // 淡出
        dimmer->brightness -= dimmer->fade_speed;
        if (dimmer->brightness <= 0) {
            dimmer->brightness = 0;
            dimmer->fade_direction = 0;
        }
    }
    
    pwm_set_duty_cycle(dimmer->htim, dimmer->channel, dimmer->brightness);
}
```

### **馬達速度控制**
```c
// 馬達速度控制
typedef struct {
    TIM_HandleTypeDef* htim;
    uint32_t channel;
    uint16_t speed;
    uint16_t max_speed;
    uint16_t min_speed;
    uint8_t direction;
} Motor_Controller_t;

void motor_controller_init(Motor_Controller_t* motor, TIM_HandleTypeDef* htim, uint32_t channel,
                          uint16_t min_speed, uint16_t max_speed) {
    motor->htim = htim;
    motor->channel = channel;
    motor->min_speed = min_speed;
    motor->max_speed = max_speed;
    motor->speed = 0;
    motor->direction = 0;
}

void motor_set_speed(Motor_Controller_t* motor, uint16_t speed) {
    if (speed > motor->max_speed) {
        speed = motor->max_speed;
    } else if (speed < motor->min_speed) {
        speed = motor->min_speed;
    }
    
    motor->speed = speed;
    pwm_set_duty_cycle(motor->htim, motor->channel, speed);
}

void motor_set_direction(Motor_Controller_t* motor, uint8_t direction) {
    motor->direction = direction;
    // 如果需要，可使用額外的 GPIO 控制方向
}
```

### **音訊產生**
```c
// 音訊音調產生
typedef struct {
    TIM_HandleTypeDef* htim;
    uint32_t channel;
    uint16_t frequency;
    uint16_t volume;
    uint8_t playing;
} Audio_Generator_t;

void audio_generator_init(Audio_Generator_t* audio, TIM_HandleTypeDef* htim, uint32_t channel) {
    audio->htim = htim;
    audio->channel = channel;
    audio->frequency = 440; // A4 音符
    audio->volume = 50;
    audio->playing = 0;
}

void audio_generator_play_tone(Audio_Generator_t* audio, uint16_t frequency, uint16_t volume) {
    audio->frequency = frequency;
    audio->volume = volume;
    audio->playing = 1;
    
    // 設定頻率
    PWM_Config_t config;
    pwm_calculate_parameters(&config, SystemCoreClock, frequency);
    
    audio->htim->Init.Prescaler = config.prescaler;
    audio->htim->Init.Period = config.period;
    HAL_TIM_Base_Init(audio->htim);
    
    // 設定音量（佔空比）
    pwm_set_duty_cycle(audio->htim, audio->channel, volume);
}

void audio_generator_stop(Audio_Generator_t* audio) {
    audio->playing = 0;
    pwm_set_duty_cycle(audio->htim, audio->channel, 0);
}
```

---

## ⚡ 進階 PWM 技術

### **移相 PWM**
```c
// 多相系統的移相 PWM
typedef struct {
    TIM_HandleTypeDef* htim;
    uint32_t channels[3];
    uint16_t duty_cycle;
    uint16_t phase_shift;
} Phase_Shifted_PWM_t;

void phase_shifted_pwm_init(Phase_Shifted_PWM_t* pspwm, TIM_HandleTypeDef* htim,
                           uint32_t* channels, uint16_t phase_shift) {
    pspwm->htim = htim;
    pspwm->phase_shift = phase_shift;
    
    for (int i = 0; i < 3; i++) {
        pspwm->channels[i] = channels[i];
    }
}

void phase_shifted_pwm_set_duty(Phase_Shifted_PWM_t* pspwm, uint16_t duty_cycle) {
    pspwm->duty_cycle = duty_cycle;
    
    // 為每個通道設定帶移相的佔空比
    for (int i = 0; i < 3; i++) {
        uint16_t phase_adjusted_duty = (duty_cycle + (i * pspwm->phase_shift)) % 100;
        pwm_set_duty_cycle(pspwm->htim, pspwm->channels[i], phase_adjusted_duty);
    }
}
```

### **死區時間插入**
```c
// H 橋控制的死區時間插入
typedef struct {
    TIM_HandleTypeDef* htim;
    uint32_t channel1;
    uint32_t channel2;
    uint16_t dead_time;
} Dead_Time_PWM_t;

void dead_time_pwm_init(Dead_Time_PWM_t* dtpwm, TIM_HandleTypeDef* htim,
                       uint32_t channel1, uint32_t channel2, uint16_t dead_time) {
    dtpwm->htim = htim;
    dtpwm->channel1 = channel1;
    dtpwm->channel2 = channel2;
    dtpwm->dead_time = dead_time;
    
    // 配置死區時間
    __HAL_TIM_SET_AUTORELOAD(htim, htim->Init.Period);
    __HAL_TIM_SET_COMPARE(htim, channel1, 0);
    __HAL_TIM_SET_COMPARE(htim, channel2, dead_time);
}
```

---

## 🎯 常見應用

### **電源供應器控制**
```c
// 降壓轉換器控制
typedef struct {
    TIM_HandleTypeDef* htim;
    uint32_t channel;
    uint16_t duty_cycle;
    float output_voltage;
    float target_voltage;
    float input_voltage;
} Buck_Converter_t;

void buck_converter_init(Buck_Converter_t* buck, TIM_HandleTypeDef* htim, uint32_t channel,
                        float input_voltage) {
    buck->htim = htim;
    buck->channel = channel;
    buck->input_voltage = input_voltage;
    buck->output_voltage = 0;
    buck->target_voltage = 0;
    buck->duty_cycle = 0;
}

void buck_converter_set_voltage(Buck_Converter_t* buck, float target_voltage) {
    buck->target_voltage = target_voltage;
    
    // 計算佔空比：Vout = Vin * 佔空比
    float duty_cycle = (target_voltage / buck->input_voltage) * 100.0f;
    
    if (duty_cycle > 100.0f) duty_cycle = 100.0f;
    if (duty_cycle < 0.0f) duty_cycle = 0.0f;
    
    buck->duty_cycle = (uint16_t)duty_cycle;
    pwm_set_duty_cycle(buck->htim, buck->channel, buck->duty_cycle);
}
```

### **伺服馬達控制**
```c
// 伺服馬達控制
typedef struct {
    TIM_HandleTypeDef* htim;
    uint32_t channel;
    uint16_t position;
    uint16_t min_position;
    uint16_t max_position;
} Servo_Controller_t;

void servo_controller_init(Servo_Controller_t* servo, TIM_HandleTypeDef* htim, uint32_t channel) {
    servo->htim = htim;
    servo->channel = channel;
    servo->min_position = 5;   // 0.5ms 脈衝
    servo->max_position = 25;  // 2.5ms 脈衝
    servo->position = 15;      // 中間位置
}

void servo_set_position(Servo_Controller_t* servo, uint16_t position) {
    if (position < servo->min_position) {
        position = servo->min_position;
    } else if (position > servo->max_position) {
        position = servo->max_position;
    }
    
    servo->position = position;
    pwm_set_duty_cycle(servo->htim, servo->channel, position);
}
```

---

## ⚠️ 常見陷阱

### **1. 不正確的頻率計算**
```c
// ❌ 錯誤：不正確的頻率計算
void pwm_frequency_wrong(TIM_HandleTypeDef* htim, uint32_t frequency) {
    htim->Init.Period = 1000 / frequency; // 錯誤計算
}

// ✅ 正確：正確的頻率計算
void pwm_frequency_correct(TIM_HandleTypeDef* htim, uint32_t frequency) {
    uint32_t clock_freq = SystemCoreClock;
    uint32_t period = (clock_freq / frequency) - 1;
    htim->Init.Period = period;
}
```

### **2. 缺少死區時間**
```c
// ❌ 錯誤：H 橋無死區時間
void h_bridge_control_wrong(TIM_HandleTypeDef* htim) {
    // 無死區時間的直接 PWM 控制
}

// ✅ 正確：帶死區時間
void h_bridge_control_correct(TIM_HandleTypeDef* htim) {
    // 配置安全切換的死區時間
    __HAL_TIM_SET_COMPARE(htim, TIM_CHANNEL_1, 0);
    __HAL_TIM_SET_COMPARE(htim, TIM_CHANNEL_2, DEAD_TIME_VALUE);
}
```

### **3. 解析度不足**
```c
// ❌ 錯誤：低解析度 PWM
void pwm_low_resolution_wrong(TIM_HandleTypeDef* htim) {
    htim->Init.Period = 100; // 僅 100 步
}

// ✅ 正確：高解析度 PWM
void pwm_high_resolution_correct(TIM_HandleTypeDef* htim) {
    htim->Init.Period = 4095; // 4096 步（12 位元）
}
```

---

## ✅ 最佳實踐

### **1. 使用適當的頻率**
```c
// 為不同應用使用適當的頻率
void configure_pwm_frequency(PWM_Config_t* config, uint8_t application_type) {
    switch (application_type) {
        case PWM_LED_DIMMING:
            config->frequency = 1000; // LED 調光用 1kHz
            break;
        case PWM_MOTOR_CONTROL:
            config->frequency = 20000; // 馬達控制用 20kHz
            break;
        case PWM_AUDIO:
            config->frequency = 44100; // 音訊用 44.1kHz
            break;
        case PWM_POWER_SUPPLY:
            config->frequency = 100000; // 電源供應器用 100kHz
            break;
    }
}
```

### **2. 實作平滑轉換**
```c
// 始終使用平滑轉換以獲得更好的效能
void smooth_pwm_transition(TIM_HandleTypeDef* htim, uint32_t channel,
                          uint16_t start_duty, uint16_t end_duty, uint16_t steps) {
    uint16_t step_size = (end_duty - start_duty) / steps;
    
    for (int i = 0; i < steps; i++) {
        uint16_t current_duty = start_duty + (i * step_size);
        pwm_set_duty_cycle(htim, channel, current_duty);
        HAL_Delay(10); // 平滑轉換的小延遲
    }
    
    pwm_set_duty_cycle(htim, channel, end_duty);
}
```

### **3. 監控 PWM 效能**
```c
// 監控 PWM 效能和效率
typedef struct {
    uint32_t frequency;
    uint16_t duty_cycle;
    float efficiency;
    uint32_t switching_losses;
} PWM_Performance_t;

void pwm_performance_monitor(PWM_Performance_t* perf, TIM_HandleTypeDef* htim) {
    // 計算切換損耗
    perf->switching_losses = perf->frequency * perf->duty_cycle / 100;
    
    // 計算效率（簡化版）
    perf->efficiency = 100.0f - (perf->switching_losses / 1000.0f);
}
```

---

## 🎯 面試問題

### **基本問題**
1. **什麼是 PWM，它如何運作？**
   - 脈寬調變，在開/關狀態之間快速切換

2. **什麼是佔空比，如何計算？**
   - 訊號為高電位的時間百分比，(導通時間 / 總時間) * 100

3. **頻率和週期之間的關係是什麼？**
   - 頻率 = 1 / 週期

### **進階問題**
1. **如何實作移相 PWM？**
   - 使用具有不同相位延遲的多個通道

2. **什麼是死區時間，為什麼它重要？**
   - 切換之間的延遲，防止 H 橋中的上下管直通

3. **如何為不同應用最佳化 PWM？**
   - 選擇適當的頻率、解析度和切換策略

### **實務問題**
1. **使用 PWM 設計馬達速度控制器**
   - 計時器配置、佔空比控制、回授迴路

2. **實作帶平滑轉換的 LED 調光**
   - PWM 產生、佔空比漸變、時序控制

3. **建立音訊音調產生器**
   - 頻率控制、音量控制、波形產生

---

## 📚 額外資源

### **文件**
- [STM32 計時器參考手冊](https://www.st.com/resource/en/reference_manual/dm00031020-stm32f405-415-stm32f407-417-stm32f427-437-and-stm32f429-439-advanced-arm-based-32-bit-mcus-stmicroelectronics.pdf)
- [ARM Cortex-M 計時器程式設計](https://developer.arm.com/documentation/dui0552/a/the-cortex-m3-processor/peripherals/general-purpose-timers)

### **工具**
- [STM32CubeMX](https://www.st.com/en/development-tools/stm32cubemx.html) - PWM 配置
- [PWM 計算器](https://www.st.com/resource/en/user_manual/dm00104712-stm32cubemx-user-manual-stmicroelectronics.pdf)

### **相關主題**
- **[GPIO 配置](./GPIO_Configuration.md)** - GPIO 模式、配置、電氣特性
- **[計時器/計數器程式設計](./Timer_Counter_Programming.md)** - 輸入捕獲、輸出比較、頻率量測
- **[類比 I/O](./Analog_IO.md)** - ADC 取樣技術、DAC 輸出產生

---

**下一個主題：** [計時器/計數器程式設計](./Timer_Counter_Programming.md) → [外部中斷](./External_Interrupts.md)
