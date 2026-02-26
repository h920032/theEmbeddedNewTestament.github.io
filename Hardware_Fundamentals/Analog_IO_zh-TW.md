# 📊 類比 I/O

## 快速參考：重點摘要

- **類比 I/O** 處理代表真實世界現象的連續電壓/電流訊號
- **ADC（類比轉數位轉換器）** 將類比訊號轉換為數位值，具有解析度和取樣率
- **DAC（數位轉類比轉換器）** 將數位值轉換為類比訊號，具有穩定時間和線性度
- **參考電壓 (Vref)** 決定 ADC/DAC 範圍並影響量測精確度
- **取樣率** 必須至少為訊號頻率的 2 倍（奈奎斯特定理）以避免混疊
- **輸入阻抗** 影響訊號完整性；高阻抗來源需要緩衝
- **ENOB（有效位元數）** 代表考慮雜訊和失真後的實際解析度
- **訊號調理** 包括濾波、放大和保護，以實現可靠的量測

> **掌握嵌入式系統的類比輸入/輸出**  
> ADC 取樣技術、DAC 輸出產生，以及類比訊號處理

## 🎯 概述

類比 I/O 對於與真實世界訊號（如溫度感測器、壓力感測器、音訊訊號和控制系統）的介面至關重要。瞭解 ADC（類比轉數位轉換器）和 DAC（數位轉類比轉換器）對嵌入式系統至關重要。

### **面試官意圖（他們在探測什麼）**
- 你能否清楚解釋取樣、量化和混疊？
- 你是否瞭解 Vref、輸入阻抗和縮放？
- 你能否根據訊號需求選擇取樣率和解析度？

### **🔍 視覺化理解**

#### **類比訊號與數位訊號表示**
```
類比訊號（連續）
電壓
   ^
   |    /\
   |   /  \    /\
   |  /    \  /  \
   | /      \/    \
   |/              \
   +-------------------> 時間
   
數位訊號（取樣）
電壓
   ^
   |    |    |    |
   |    |    |    |
   |    |    |    |
   |    |    |    |
   +-------------------> 時間
   |<->| 取樣週期
```

#### **ADC 取樣過程**
```
輸入訊號
   ^
   |    /\
   |   /  \    /\
   |  /    \  /  \
   | /      \/    \
   |/              \
   +-------------------> 時間
   
取樣點
   ^
   |    |    |    |
   |    |    |    |
   |    |    |    |
   |    |    |    |
   +-------------------> 時間
   
量化輸出
   ^
   |    |    |    |
   |    |    |    |
   |    |    |    |
   |    |    |    |
   +-------------------> 時間
   |<->| 量化階級
```

#### **DAC 重建過程**
```
數位輸入
   ^
   |    |    |    |
   |    |    |    |
   |    |    |    |
   |    |    |    |
   +-------------------> 時間
   
重建輸出
   ^
   |    /\
   |   /  \    /\
   |  /    \  /  \
   | /      \/    \
   |/              \
   +-------------------> 時間
```

### **🧠 概念基礎**

#### **類比訊號的本質**
類比訊號代表隨時間平滑變化的連續物理現象。與具有離散位準的數位訊號不同，類比訊號可以在其範圍內取任何值。這個根本差異在嵌入式系統中創造了獨特的挑戰和機會。

**主要特性：**
- **連續性**：類比訊號在其範圍內具有無限解析度
- **即時性**：它們代表物理量的瞬時值
- **雜訊敏感性**：類比訊號容易受到電氣干擾
- **頻寬限制**：物理系統具有自然的頻率響應限制

#### **為何類比 I/O 在嵌入式系統中很重要**
嵌入式系統必須在數位計算世界和類比物理世界之間架起橋樑。此介面對以下方面至關重要：
- **感測器整合**：將物理量測（溫度、壓力、光線）轉換為數位資料
- **致動器控制**：為馬達、顯示器和音訊產生精確的類比輸出
- **訊號調理**：濾波、放大和處理真實世界的訊號
- **系統監控**：閉迴路控制系統的即時回饋

#### **取樣定理及其含義**
奈奎斯特-夏農取樣定理指出，要準確重建訊號，取樣率必須至少為最高頻率分量的兩倍。這個基本原理具有深遠的影響：

**實務考量：**
- **抗混疊濾波器**：必須在取樣前應用，以移除高於取樣率一半的頻率
- **過取樣**：使用高於嚴格必要的取樣率可以改善訊號品質
- **欠取樣**：可以策略性地用於降頻轉換高頻訊號
- **抖動效應**：取樣中的時序變異可能引入額外的雜訊

## 🧠 核心概念

### **概念：ADC 取樣與訊號完整性**
**為何重要**：ADC 效能取決於適當的取樣配置、參考穩定性和訊號調理。不正確的取樣可能導致不準確的量測和混疊。

**取樣過程說明**：
ADC 取樣過程涉及幾個必須仔細管理的關鍵階段：

1. **擷取階段**：必須捕獲輸入訊號並在轉換期間保持穩定
2. **轉換階段**：保持的電壓被量化為離散的數位值
3. **穩定階段**：系統必須在下一次取樣前穩定

**影響訊號完整性的關鍵因素**：
- **來源阻抗**：高阻抗來源需要更長的取樣時間來對內部取樣電容充電
- **訊號頻寬**：快速變化的訊號需要更高的取樣率以避免混疊
- **雜訊環境**：電氣雜訊可能損壞量測結果，需要濾波和平均
- **參考穩定性**：參考電壓的任何漂移都會直接影響量測精確度

**最小範例**：
```c
// 基本 ADC 配置結構
typedef struct {
    uint32_t sample_time;      // 以 ADC 時鐘週期為單位的取樣時間
    uint32_t resolution;       // ADC 解析度（位元數）
    float reference_voltage;   // 參考電壓
} adc_config_t;

// 具有基本錯誤檢查的簡單 ADC 讀取
uint16_t read_adc_safe(uint8_t channel) {
    // 開始轉換
    start_adc_conversion(channel);
    
    // 等待完成並設置逾時
    uint32_t timeout = 1000;
    while (!is_adc_complete() && timeout--) {
        delay_us(1);
    }
    
    if (timeout == 0) {
        return ADC_ERROR_VALUE;  // 逾時錯誤
    }
    
    return read_adc_result();
}
```

**動手試試**：考慮改變取樣時間如何影響不同來源阻抗下的量測精確度。

**重點摘要**： 
- 取樣時間必須適應來源阻抗特性
- 參考電壓穩定性對精確度至關重要
- 始終實作逾時保護以確保穩健運作
- 考慮取樣速度和精確度之間的取捨

### **概念：DAC 輸出產生與穩定時間**
**為何重要**：DAC 效能取決於穩定時間、線性度和輸出範圍。瞭解這些參數可確保準確的類比輸出產生。

**DAC 輸出過程說明**：
數位轉類比轉換涉及幾個影響輸出品質的關鍵考量：

1. **數位輸入處理**：數位值被載入 DAC 暫存器
2. **轉換階段**：數位值被轉換為類比電壓
3. **穩定階段**：輸出必須穩定到指定精確度範圍內
4. **輸出緩衝**：可選的輸出緩衝器影響穩定時間和驅動能力

**主要效能參數**：
- **穩定時間**：輸出在指定精確度內達到最終值所需的時間
- **線性度**：輸出電壓遵循理想直線關係的程度
- **突波能量**：代碼轉換期間不需要的電壓尖峰
- **輸出阻抗**：影響驅動外部負載而不失真的能力

**最小範例**：
```c
// 基本 DAC 配置
typedef struct {
    uint32_t resolution;       // DAC 解析度（位元數）
    float reference_voltage;   // 參考電壓
} dac_config_t;

// 具有穩定時間考量的簡單 DAC 輸出
void set_dac_output_safe(uint16_t value, uint32_t settling_time_us) {
    // 設定 DAC 值
    set_dac_value(value);
    
    // 等待穩定時間
    delay_us(settling_time_us);
    
    // 現在可以安全使用輸出
}
```

// 具有穩定時間考量的類比輸出產生
void generate_analog_output(uint16_t digital_value, dac_config_t *config) {
    // 將值寫入 DAC 資料暫存器
    DAC->DHR12R1 = digital_value;
    
    // 等待穩定時間
    uint32_t settling_cycles = (config->settling_time_us * SystemCoreClock) / 1000000;
    for (volatile uint32_t i = 0; i < settling_cycles; i++) {
        __NOP();
    }
}

// 產生正弦波輸出
void generate_sine_wave(float frequency_hz, uint32_t samples_per_cycle) {
    static uint32_t sample_index = 0;
    
    // 計算正弦值
    float angle = (2.0f * M_PI * sample_index) / samples_per_cycle;
    float sine_value = sinf(angle);
    
    // 轉換為 DAC 範圍（12 位元 DAC 為 0 到 4095）
    uint16_t dac_value = (uint16_t)((sine_value + 1.0f) * 2047.5f);
    
    // 輸出到 DAC
    DAC->DHR12R1 = dac_value;
    
    // 更新取樣索引
    sample_index = (sample_index + 1) % samples_per_cycle;
}
```

**動手試試**：使用 DAC 產生不同的波形並量測穩定時間和線性度。

**重點摘要**： 
- 必須尊重穩定時間以確保準確的輸出產生
- 輸出緩衝器的選擇影響穩定時間和驅動能力
- 在使用類比訊號之前，始終驗證輸出穩定性

### **概念：訊號調理與濾波**
**為何重要**：原始類比訊號通常包含雜訊、直流偏移和不需要的頻率分量。適當的訊號調理可確保可靠的量測和乾淨的輸出。

**訊號調理鏈**：
訊號調理涉及多個協同工作以改善訊號品質的階段：

1. **保護**：防止過電壓、反極性和 ESD
2. **放大**：將微弱訊號提升到適合 ADC 輸入的位準
3. **濾波**：移除不需要的頻率分量和雜訊
4. **隔離**：將敏感電路與雜訊環境分離

**濾波器設計考量**：
- **低通濾波器**：移除訊號頻寬以上的高頻雜訊
- **高通濾波器**：移除直流偏移和低頻漂移
- **帶通濾波器**：隔離特定頻率範圍內的訊號
- **陷波濾波器**：移除特定的干擾頻率（例如 50/60 Hz 電源線）

**最小範例**：
```c
// 基本訊號調理結構
typedef struct {
    float gain;              // 放大因子
    float cutoff_freq;       // 濾波器截止頻率
    bool enable_filter;      // 濾波器啟用旗標
} signal_conditioning_t;

// 簡單訊號調理
float condition_signal(float input, signal_conditioning_t *config) {
    float output = input;
    
    // 應用增益
    output *= config->gain;
    
    // 如果啟用則應用簡單低通濾波器
    if (config->enable_filter) {
        output = apply_low_pass_filter(output, config->cutoff_freq);
    }
    
    return output;
}
```

**動手試試**：嘗試不同的濾波器類型和截止頻率，觀察它們對訊號品質的影響。

**重點摘要**： 
- 訊號調理對可靠的類比 I/O 至關重要
- 濾波器設計必須考慮訊號需求和雜訊特性
- 保護電路防止敏感元件損壞
- 始終在調理的每個階段驗證訊號品質

### **概念：訊號調理與雜訊降低**
**為何重要**：真實世界的類比訊號通常包含雜訊，需要調理以進行可靠的量測和處理。

**最小範例**：
```c
// 訊號調理配置
typedef struct {
    float filter_cutoff_freq;  // 低通濾波器截止頻率
    uint8_t averaging_samples; // 平均取樣數
    float calibration_offset;  // 校準偏移
    float calibration_gain;    // 校準增益
} signal_conditioning_t;

// 低通濾波器實作
float low_pass_filter(float new_value, float old_value, float alpha) {
    // alpha = dt / (dt + RC)，其中 RC 是濾波器時間常數
    return alpha * new_value + (1.0f - alpha) * old_value;
}

// 應用訊號調理
float condition_analog_signal(float raw_value, signal_conditioning_t *config) {
    static float filtered_value = 0.0f;
    
    // 應用低通濾波器
    float alpha = 0.1f;  // 根據所需響應調整
    filtered_value = low_pass_filter(raw_value, filtered_value, alpha);
    
    // 應用校準
    float calibrated_value = (filtered_value + config->calibration_offset) * config->calibration_gain;
    
    return calibrated_value;
}

// 溫度感測器訊號調理範例
float read_temperature_sensor(void) {
    // 讀取 ADC 值
    uint16_t adc_value = read_adc_averaged(TEMP_SENSOR_CHANNEL, 16);
    
    // 轉換為電壓
    float voltage = (adc_value * 3.3f) / 4095.0f;
    
    // 轉換為溫度（LM35 範例：10mV/°C）
    float temperature = voltage * 100.0f;  // 100°C/V
    
    // 應用訊號調理
    signal_conditioning_t temp_config = {
        .filter_cutoff_freq = 1.0f,    // 1 Hz 截止
        .averaging_samples = 16,        // 16 個取樣
        .calibration_offset = -0.5f,   // -0.5°C 偏移
        .calibration_gain = 1.02f      // 2% 增益修正
    };
    
    return condition_analog_signal(temperature, &temp_config);
}
```

**動手試試**：為感測器實作訊號調理並量測訊號品質的改善。

**重點摘要**：使用濾波降低雜訊，使用平均提高穩定性，使用校準提高精確度。

---

## 📡 ADC 基礎

### **什麼是 ADC？**

ADC（類比轉數位轉換器）將連續的類比訊號轉換為離散的數位值，可由微控制器和數位系統處理。

### **ADC 概念**

**轉換過程：**
- **取樣**：在特定時間間隔進行量測
- **量化**：將連續值轉換為離散位準
- **編碼**：將量化值轉換為數位碼
- **輸出**：類比訊號的數位表示

**ADC 類型：**
- **逐次逼近**：最常見的類型
- **快閃式 ADC**：最快但最複雜
- **Delta-Sigma**：高解析度，較慢
- **管線式 ADC**：高速，中等解析度

### **ADC 解析度與範圍**
```c
// ADC 解析度定義
#define ADC_RESOLUTION_8BIT  256
#define ADC_RESOLUTION_10BIT 1024
#define ADC_RESOLUTION_12BIT 4096
#define ADC_RESOLUTION_16BIT 65536

// ADC 電壓計算
float adc_to_voltage(uint16_t adc_value, float vref, uint16_t resolution) {
    return (float)adc_value * vref / resolution;
}

uint16_t voltage_to_adc(float voltage, float vref, uint16_t resolution) {
    return (uint16_t)(voltage * resolution / vref);
}
```

### **ADC 配置結構**
```c
typedef struct {
    ADC_HandleTypeDef* hadc;
    uint32_t channel;
    uint32_t resolution;
    float vref;
    uint32_t sampling_time;
    uint8_t continuous_mode;
} ADC_Config_t;

void adc_config_init(ADC_Config_t* config, ADC_HandleTypeDef* hadc, 
                     uint32_t channel, uint32_t resolution, float vref) {
    config->hadc = hadc;
    config->channel = channel;
    config->resolution = resolution;
    config->vref = vref;
    config->sampling_time = ADC_SAMPLETIME_480CYCLES;
    config->continuous_mode = 0;
}
```

## 📊 ADC 配置

### **什麼是 ADC 配置？**

ADC 配置涉及為特定應用設定 ADC 硬體，包括解析度、取樣率、參考電壓和轉換模式。

### **配置概念**

**硬體配置：**
- **解析度**：數位表示的位元數
- **參考電壓**：轉換的電壓參考
- **取樣時間**：訊號取樣的時間
- **轉換模式**：單次或連續轉換

**通道配置：**
- **輸入通道**：使用哪個類比輸入
- **輸入範圍**：輸入訊號的電壓範圍
- **輸入阻抗**：輸入阻抗要求
- **輸入保護**：過電壓保護

### **基本 ADC 配置**
```c
// 配置 ADC 進行單次轉換
void adc_single_config(ADC_Config_t* config) {
    ADC_ChannelConfTypeDef sConfig = {0};
    
    // 配置 ADC
    config->hadc->Instance = ADC1;
    config->hadc->Init.ClockPrescaler = ADC_CLOCK_SYNC_PCLK_DIV4;
    config->hadc->Init.Resolution = ADC_RESOLUTION_12B;
    config->hadc->Init.ScanConvMode = DISABLE;
    config->hadc->Init.ContinuousConvMode = DISABLE;
    config->hadc->Init.DiscontinuousConvMode = DISABLE;
    config->hadc->Init.ExternalTrigConvEdge = ADC_EXTERNALTRIGCONVEDGE_NONE;
    config->hadc->Init.ExternalTrigConv = ADC_SOFTWARE_START;
    config->hadc->Init.DataAlign = ADC_DATAALIGN_RIGHT;
    config->hadc->Init.NbrOfConversion = 1;
    config->hadc->Init.DMAContinuousRequests = DISABLE;
    config->hadc->Init.EOCSelection = ADC_EOC_SINGLE_CONV;
    
    HAL_ADC_Init(config->hadc);
    
    // 配置通道
    sConfig.Channel = config->channel;
    sConfig.Rank = 1;
    sConfig.SamplingTime = config->sampling_time;
    HAL_ADC_ConfigChannel(config->hadc, &sConfig);
}
```

### **連續 ADC 配置**
```c
// 配置 ADC 進行連續轉換
void adc_continuous_config(ADC_Config_t* config) {
    ADC_ChannelConfTypeDef sConfig = {0};
    
    // 配置 ADC 為連續模式
    config->hadc->Init.ContinuousConvMode = ENABLE;
    config->hadc->Init.DMAContinuousRequests = ENABLE;
    config->hadc->Init.EOCSelection = ADC_EOC_SEQ_CONV;
    
    HAL_ADC_Init(config->hadc);
    
    // 配置通道
    sConfig.Channel = config->channel;
    sConfig.Rank = 1;
    sConfig.SamplingTime = config->sampling_time;
    HAL_ADC_ConfigChannel(config->hadc, &sConfig);
    
    // 開始連續轉換
    HAL_ADC_Start_IT(config->hadc);
}
```

## 🔍 ADC 取樣技術

### **什麼是 ADC 取樣技術？**

ADC 取樣技術涉及高效且準確地進行類比量測的方法，包括取樣率選擇、濾波和平均。

### **取樣概念**

**取樣率：**
- **奈奎斯特率**：最小取樣率（訊號頻率的 2 倍）
- **過取樣**：以更高的速率取樣以獲得更好的精確度
- **欠取樣**：以較低的速率取樣用於特定應用
- **自適應取樣**：根據訊號調整取樣率

**取樣方法：**
- **單次取樣**：進行單次量測
- **平均**：進行多次量測並取平均
- **過取樣**：進行多次量測以獲得更好的精確度
- **觸發取樣**：基於外部觸發進行取樣

### **單次取樣**
```c
// 單次 ADC 讀取
uint16_t adc_single_read(ADC_HandleTypeDef* hadc) {
    HAL_ADC_Start(hadc);
    HAL_ADC_PollForConversion(hadc, 100);
    uint16_t value = HAL_ADC_GetValue(hadc);
    HAL_ADC_Stop(hadc);
    return value;
}
```

### **平均取樣**
```c
// 多次 ADC 讀取取平均
uint16_t adc_average_read(ADC_HandleTypeDef* hadc, uint8_t samples) {
    uint32_t sum = 0;
    
    for (int i = 0; i < samples; i++) {
        HAL_ADC_Start(hadc);
        HAL_ADC_PollForConversion(hadc, 100);
        sum += HAL_ADC_GetValue(hadc);
        HAL_ADC_Stop(hadc);
    }
    
    return (uint16_t)(sum / samples);
}
```

### **過取樣提高解析度**
```c
// 過取樣以獲得更高解析度
uint16_t adc_oversample_read(ADC_HandleTypeDef* hadc, uint8_t oversample_factor) {
    uint32_t sum = 0;
    uint16_t samples = 1 << oversample_factor;  // 2^oversample_factor
    
    for (int i = 0; i < samples; i++) {
        HAL_ADC_Start(hadc);
        HAL_ADC_PollForConversion(hadc, 100);
        sum += HAL_ADC_GetValue(hadc);
        HAL_ADC_Stop(hadc);
    }
    
    // 右移 oversample_factor 位以獲得更高解析度
    return (uint16_t)(sum >> oversample_factor);
}
```

## 📈 DAC 基礎

### **什麼是 DAC？**

DAC（數位轉類比轉換器）將數位值轉換為連續的類比訊號，可用於控制類比裝置和系統。

### **DAC 概念**

**轉換過程：**
- **數位輸入**：要轉換的數位值
- **解碼**：將數位碼轉換為類比值
- **重建**：產生連續的類比訊號
- **輸出**：給外部裝置的類比訊號

**DAC 類型：**
- **R-2R 梯形**：最常見的類型
- **加權電阻**：簡單但解析度有限
- **Delta-Sigma**：高解析度，較慢
- **電流引導**：高速，中等解析度

### **DAC 解析度與範圍**
```c
// DAC 解析度定義
#define DAC_RESOLUTION_8BIT  256
#define DAC_RESOLUTION_10BIT 1024
#define DAC_RESOLUTION_12BIT 4096
#define DAC_RESOLUTION_16BIT 65536

// DAC 電壓計算
float dac_to_voltage(uint16_t dac_value, float vref, uint16_t resolution) {
    return (float)dac_value * vref / resolution;
}

uint16_t voltage_to_dac(float voltage, float vref, uint16_t resolution) {
    return (uint16_t)(voltage * resolution / vref);
}
```

### **DAC 配置結構**
```c
typedef struct {
    DAC_HandleTypeDef* hdac;
    uint32_t channel;
    uint32_t resolution;
    float vref;
    uint32_t output_buffer;
} DAC_Config_t;

void dac_config_init(DAC_Config_t* config, DAC_HandleTypeDef* hdac, 
                     uint32_t channel, uint32_t resolution, float vref) {
    config->hdac = hdac;
    config->channel = channel;
    config->resolution = resolution;
    config->vref = vref;
    config->output_buffer = DAC_OUTPUTBUFFER_ENABLE;
}
```

## 🎛️ DAC 配置

### **什麼是 DAC 配置？**

DAC 配置涉及為特定應用設定 DAC 硬體，包括解析度、輸出範圍、穩定時間和輸出緩衝器配置。

### **配置概念**

**硬體配置：**
- **解析度**：數位輸入的位元數
- **參考電壓**：轉換的電壓參考
- **輸出範圍**：輸出訊號的電壓範圍
- **輸出緩衝器**：內部輸出緩衝器配置

**輸出配置：**
- **輸出通道**：使用哪個 DAC 輸出
- **輸出阻抗**：輸出阻抗特性
- **穩定時間**：輸出穩定的時間
- **輸出保護**：過電流保護

### **基本 DAC 配置**
```c
// 配置 DAC 進行基本輸出
void dac_basic_config(DAC_Config_t* config) {
    DAC_ChannelConfTypeDef sConfig = {0};
    
    // 配置 DAC
    config->hdac->Instance = DAC1;
    HAL_DAC_Init(config->hdac);
    
    // 配置通道
    sConfig.DAC_Trigger = DAC_TRIGGER_SOFTWARE;
    sConfig.DAC_OutputBuffer = config->output_buffer;
    HAL_DAC_ConfigChannel(config->hdac, &sConfig, config->channel);
}
```

### **DAC 波形產生**
```c
// 使用 DAC 產生正弦波
void dac_sine_wave(DAC_HandleTypeDef* hdac, uint32_t channel, float frequency, float amplitude) {
    static uint32_t phase = 0;
    static const uint16_t sine_table[256] = {
        // 正弦波查找表（0-255）
        128, 131, 134, 137, 140, 143, 146, 149, 152, 155, 158, 161, 164, 167, 170, 173,
        // ...（完整正弦表）
    };
    
    // 計算正弦波值
    uint8_t index = (phase >> 8) & 0xFF;
    uint16_t sine_value = sine_table[index];
    
    // 按振幅縮放
    uint16_t dac_value = (uint16_t)(sine_value * amplitude / 128.0f);
    
    // 寫入 DAC
    HAL_DAC_SetValue(hdac, channel, DAC_ALIGN_12B_R, dac_value);
    
    // 更新相位
    phase += (uint32_t)(frequency * 256.0f / 1000.0f);  // 假設 1kHz 更新率
}
```

## 🔧 類比訊號處理

### **什麼是類比訊號處理？**

類比訊號處理涉及操作類比訊號以改善品質、提取資訊或為進一步處理準備訊號。

### **訊號處理概念**

**濾波：**
- **低通濾波器**：移除高頻雜訊
- **高通濾波器**：移除低頻雜訊
- **帶通濾波器**：通過特定頻率範圍
- **陷波濾波器**：移除特定頻率

**放大：**
- **增益控制**：調整訊號振幅
- **偏移調整**：調整訊號偏移
- **線性化**：修正非線性響應
- **校準**：根據感測器特性調整

### **數位濾波**
```c
// 簡單移動平均濾波器
typedef struct {
    uint16_t buffer[16];
    uint8_t index;
    uint8_t count;
} moving_average_filter_t;

void filter_init(moving_average_filter_t* filter) {
    filter->index = 0;
    filter->count = 0;
    for (int i = 0; i < 16; i++) {
        filter->buffer[i] = 0;
    }
}

uint16_t filter_update(moving_average_filter_t* filter, uint16_t new_value) {
    filter->buffer[filter->index] = new_value;
    filter->index = (filter->index + 1) % 16;
    
    if (filter->count < 16) {
        filter->count++;
    }
    
    uint32_t sum = 0;
    for (int i = 0; i < filter->count; i++) {
        sum += filter->buffer[i];
    }
    
    return (uint16_t)(sum / filter->count);
}
```

### **訊號校準**
```c
// 訊號校準結構
typedef struct {
    float slope;
    float offset;
    float min_input;
    float max_input;
    float min_output;
    float max_output;
} calibration_t;

void calibration_init(calibration_t* cal, float min_in, float max_in, float min_out, float max_out) {
    cal->min_input = min_in;
    cal->max_input = max_in;
    cal->min_output = min_out;
    cal->max_output = max_out;
    cal->slope = (max_out - min_out) / (max_in - min_in);
    cal->offset = min_out - (min_in * cal->slope);
}

float calibrate_signal(calibration_t* cal, float input) {
    return input * cal->slope + cal->offset;
}
```

## ⚡ 效能最佳化

### **什麼影響類比 I/O 效能？**

類比 I/O 效能取決於多個因素，包括解析度、取樣率、雜訊和訊號調理。

### **效能因素**

**解析度與精確度：**
- **位元深度**：更高的解析度帶來更好的精確度
- **量化誤差**：離散表示造成的誤差
- **線性度**：輸出追隨輸入的程度
- **穩定性**：隨時間和溫度的一致性

**時序與速度：**
- **轉換時間**：ADC/DAC 轉換所需的時間
- **取樣率**：取樣的速率
- **穩定時間**：輸出穩定的時間
- **響應時間**：從輸入變化到輸出響應的時間

### **效能最佳化技術**

#### **過取樣提高解析度**
```c
// 過取樣以獲得更高解析度
uint16_t adc_oversample_high_res(ADC_HandleTypeDef* hadc, uint8_t oversample_bits) {
    uint32_t sum = 0;
    uint16_t samples = 1 << (oversample_bits * 2);  // 4^oversample_bits
    
    for (int i = 0; i < samples; i++) {
        HAL_ADC_Start(hadc);
        HAL_ADC_PollForConversion(hadc, 100);
        sum += HAL_ADC_GetValue(hadc);
        HAL_ADC_Stop(hadc);
    }
    
    // 右移 oversample_bits 位以獲得更高解析度
    return (uint16_t)(sum >> oversample_bits);
}
```

#### **雜訊降低**
```c
// 使用多次取樣降低雜訊
uint16_t adc_noise_reduction(ADC_HandleTypeDef* hadc, uint8_t samples) {
    uint32_t sum = 0;
    uint16_t min_val = 65535;
    uint16_t max_val = 0;
    
    // 進行多次取樣
    for (int i = 0; i < samples; i++) {
        HAL_ADC_Start(hadc);
        HAL_ADC_PollForConversion(hadc, 100);
        uint16_t value = HAL_ADC_GetValue(hadc);
        sum += value;
        
        if (value < min_val) min_val = value;
        if (value > max_val) max_val = value;
        
        HAL_ADC_Stop(hadc);
    }
    
    // 移除離群值（最小值和最大值）並取平均
    sum = sum - min_val - max_val;
    return (uint16_t)(sum / (samples - 2));
}
```

## 🎯 常見應用

### **常見的類比 I/O 應用有哪些？**

類比 I/O 在嵌入式系統中有無數的應用。瞭解常見應用有助於設計有效的類比 I/O 解決方案。

### **應用類別**

**感測器介面：**
- **溫度感測器**：熱敏電阻、RTD、熱電偶
- **壓力感測器**：應變規、壓力變送器
- **光感測器**：光電二極體、光電晶體
- **位置感測器**：電位計、編碼器

**控制系統：**
- **馬達控制**：可變速馬達控制
- **閥門控制**：比例閥控制
- **加熱控制**：溫度控制系統
- **照明控制**：調光和亮度控制

**音訊處理：**
- **音訊輸入**：麥克風和音訊輸入
- **音訊輸出**：揚聲器和音訊放大器
- **音訊效果**：濾波器、等化器、效果
- **音訊錄製**：數位音訊錄製

### **應用範例**

#### **溫度監測系統**
```c
// 溫度監測系統
typedef struct {
    ADC_HandleTypeDef* hadc;
    uint32_t channel;
    float temperature;
    moving_average_filter_t filter;
} temperature_monitor_t;

void temperature_monitor_init(temperature_monitor_t* monitor, ADC_HandleTypeDef* hadc, uint32_t channel) {
    monitor->hadc = hadc;
    monitor->channel = channel;
    monitor->temperature = 0.0f;
    filter_init(&monitor->filter);
}

float temperature_monitor_read(temperature_monitor_t* monitor) {
    // 讀取 ADC 值
    uint16_t adc_value = adc_single_read(monitor->hadc);
    
    // 應用濾波
    uint16_t filtered_value = filter_update(&monitor->filter, adc_value);
    
    // 轉換為電壓
    float voltage = adc_to_voltage(filtered_value, 3.3f, 4096);
    
    // 轉換為溫度（假設使用熱敏電阻）
    monitor->temperature = voltage_to_temperature(voltage);
    return monitor->temperature;
}
```

#### **馬達速度控制系統**
```c
// 馬達速度控制系統
typedef struct {
    DAC_HandleTypeDef* hdac;
    uint32_t channel;
    float speed;
    float max_speed;
    calibration_t calibration;
} motor_speed_control_t;

void motor_speed_control_init(motor_speed_control_t* control, DAC_HandleTypeDef* hdac, uint32_t channel) {
    control->hdac = hdac;
    control->channel = channel;
    control->speed = 0.0f;
    control->max_speed = 100.0f;
    calibration_init(&control->calibration, 0.0f, 100.0f, 0.0f, 3.3f);
}

void motor_speed_control_set_speed(motor_speed_control_t* control, float speed) {
    if (speed >= 0.0f && speed <= control->max_speed) {
        control->speed = speed;
        
        // 應用校準
        float voltage = calibrate_signal(&control->calibration, speed);
        
        // 轉換為 DAC 值
        uint16_t dac_value = voltage_to_dac(voltage, 3.3f, 4096);
        
        // 寫入 DAC
        HAL_DAC_SetValue(control->hdac, control->channel, DAC_ALIGN_12B_R, dac_value);
    }
}
```

## 🔧 實作

### **完整類比 I/O 範例**

```c
#include <stdint.h>
#include <stdbool.h>
#include <math.h>

// 類比 I/O 配置結構
typedef struct {
    ADC_HandleTypeDef* hadc;
    DAC_HandleTypeDef* hdac;
    uint32_t adc_channel;
    uint32_t dac_channel;
    uint32_t resolution;
    float vref;
} analog_io_config_t;

// 類比 I/O 初始化
void analog_io_init(const analog_io_config_t* config) {
    // 初始化 ADC
    ADC_ChannelConfTypeDef sConfig = {0};
    
    config->hadc->Instance = ADC1;
    config->hadc->Init.ClockPrescaler = ADC_CLOCK_SYNC_PCLK_DIV4;
    config->hadc->Init.Resolution = ADC_RESOLUTION_12B;
    config->hadc->Init.ScanConvMode = DISABLE;
    config->hadc->Init.ContinuousConvMode = DISABLE;
    config->hadc->Init.DiscontinuousConvMode = DISABLE;
    config->hadc->Init.ExternalTrigConvEdge = ADC_EXTERNALTRIGCONVEDGE_NONE;
    config->hadc->Init.ExternalTrigConv = ADC_SOFTWARE_START;
    config->hadc->Init.DataAlign = ADC_DATAALIGN_RIGHT;
    config->hadc->Init.NbrOfConversion = 1;
    config->hadc->Init.DMAContinuousRequests = DISABLE;
    config->hadc->Init.EOCSelection = ADC_EOC_SINGLE_CONV;
    
    HAL_ADC_Init(config->hadc);
    
    sConfig.Channel = config->adc_channel;
    sConfig.Rank = 1;
    sConfig.SamplingTime = ADC_SAMPLETIME_480CYCLES;
    HAL_ADC_ConfigChannel(config->hadc, &sConfig);
    
    // 初始化 DAC
    DAC_ChannelConfTypeDef dacConfig = {0};
    
    config->hdac->Instance = DAC1;
    HAL_DAC_Init(config->hdac);
    
    dacConfig.DAC_Trigger = DAC_TRIGGER_SOFTWARE;
    dacConfig.DAC_OutputBuffer = DAC_OUTPUTBUFFER_ENABLE;
    HAL_DAC_ConfigChannel(config->hdac, &dacConfig, config->dac_channel);
}

// ADC 讀取函式
uint16_t analog_io_read(ADC_HandleTypeDef* hadc) {
    HAL_ADC_Start(hadc);
    HAL_ADC_PollForConversion(hadc, 100);
    uint16_t value = HAL_ADC_GetValue(hadc);
    HAL_ADC_Stop(hadc);
    return value;
}

// DAC 寫入函式
void analog_io_write(DAC_HandleTypeDef* hdac, uint32_t channel, uint16_t value) {
    HAL_DAC_SetValue(hdac, channel, DAC_ALIGN_12B_R, value);
}

// 電壓轉換函式
float adc_to_voltage(uint16_t adc_value, float vref, uint16_t resolution) {
    return (float)adc_value * vref / resolution;
}

uint16_t voltage_to_dac(float voltage, float vref, uint16_t resolution) {
    return (uint16_t)(voltage * resolution / vref);
}

// 溫度感測器介面
typedef struct {
    ADC_HandleTypeDef* hadc;
    uint32_t channel;
    float temperature;
    moving_average_filter_t filter;
} temperature_sensor_t;

void temperature_sensor_init(temperature_sensor_t* sensor, ADC_HandleTypeDef* hadc, uint32_t channel) {
    sensor->hadc = hadc;
    sensor->channel = channel;
    sensor->temperature = 0.0f;
    filter_init(&sensor->filter);
}

float temperature_sensor_read(temperature_sensor_t* sensor) {
    uint16_t adc_value = analog_io_read(sensor->hadc);
    uint16_t filtered_value = filter_update(&sensor->filter, adc_value);
    float voltage = adc_to_voltage(filtered_value, 3.3f, 4096);
    
    // 將電壓轉換為溫度（假設使用熱敏電阻）
    sensor->temperature = voltage_to_temperature(voltage);
    return sensor->temperature;
}

// 馬達控制介面
typedef struct {
    DAC_HandleTypeDef* hdac;
    uint32_t channel;
    float speed;
    float max_speed;
} motor_control_t;

void motor_control_init(motor_control_t* motor, DAC_HandleTypeDef* hdac, uint32_t channel) {
    motor->hdac = hdac;
    motor->channel = channel;
    motor->speed = 0.0f;
    motor->max_speed = 100.0f;
}

void motor_control_set_speed(motor_control_t* motor, float speed) {
    if (speed >= 0.0f && speed <= motor->max_speed) {
        motor->speed = speed;
        float voltage = speed_to_voltage(speed);
        uint16_t dac_value = voltage_to_dac(voltage, 3.3f, 4096);
        analog_io_write(motor->hdac, motor->channel, dac_value);
    }
}

// 主函式
int main(void) {
    // 初始化系統
    system_init();
    
    // 初始化類比 I/O
    analog_io_config_t analog_config = {
        .hadc = &hadc1,
        .hdac = &hdac1,
        .adc_channel = ADC_CHANNEL_0,
        .dac_channel = DAC_CHANNEL_1,
        .resolution = 4096,
        .vref = 3.3f
    };
    
    analog_io_init(&analog_config);
    
    // 初始化溫度感測器
    temperature_sensor_t temp_sensor;
    temperature_sensor_init(&temp_sensor, &hadc1, ADC_CHANNEL_0);
    
    // 初始化馬達控制
    motor_control_t motor;
    motor_control_init(&motor, &hdac1, DAC_CHANNEL_1);
    
    // 主迴圈
    while (1) {
        // 讀取溫度
        float temperature = temperature_sensor_read(&temp_sensor);
        
        // 根據溫度控制馬達
        if (temperature > 25.0f) {
            motor_control_set_speed(&motor, 50.0f);
        } else {
            motor_control_set_speed(&motor, 0.0f);
        }
        
        // 更新系統
        system_update();
    }
    
    return 0;
}
```

## ⚠️ 常見陷阱

### **1. 解析度不足**

**問題**：在高精度應用中使用低解析度 ADC/DAC
**解決方案**：根據應用需求選擇適當的解析度

```c
// ❌ 錯誤：高精度應用使用低解析度
void bad_precision_config(ADC_HandleTypeDef* hadc) {
    hadc->Init.Resolution = ADC_RESOLUTION_8B;  // 僅 8 位元解析度
}

// ✅ 正確：高精度應用使用高解析度
void good_precision_config(ADC_HandleTypeDef* hadc) {
    hadc->Init.Resolution = ADC_RESOLUTION_12B;  // 12 位元解析度
}
```

### **2. 雜訊處理不當**

**問題**：未處理類比訊號中的電氣雜訊
**解決方案**：實作適當的濾波和雜訊降低

```c
// ❌ 錯誤：無雜訊降低
uint16_t bad_adc_read(ADC_HandleTypeDef* hadc) {
    return analog_io_read(hadc);  // 單次讀取 - 可能有雜訊
}

// ✅ 正確：使用平均降低雜訊
uint16_t good_adc_read(ADC_HandleTypeDef* hadc) {
    return adc_average_read(hadc, 16);  // 16 次讀取的平均值
}
```

### **3. 參考電壓不正確**

**問題**：計算時使用錯誤的參考電壓
**解決方案**：為 ADC/DAC 使用正確的參考電壓

```c
// ❌ 錯誤：錯誤的參考電壓
float bad_voltage_calc(uint16_t adc_value) {
    return (float)adc_value * 5.0f / 4096;  // 錯誤的參考電壓
}

// ✅ 正確：正確的參考電壓
float good_voltage_calc(uint16_t adc_value) {
    return (float)adc_value * 3.3f / 4096;  // 正確的參考電壓
}
```

### **4. 校準不良**

**問題**：未校準類比感測器
**解決方案**：實作適當的校準程序

```c
// ❌ 錯誤：無校準
float bad_sensor_read(ADC_HandleTypeDef* hadc) {
    uint16_t adc_value = analog_io_read(hadc);
    return adc_to_voltage(adc_value, 3.3f, 4096);  // 無校準
}

// ✅ 正確：包含校準
float good_sensor_read(temperature_sensor_t* sensor) {
    return temperature_sensor_read(sensor);  // 包含校準
}
```

## ✅ 最佳實踐

### **1. 選擇適當的解析度**

- **應用需求**：根據應用需求匹配解析度
- **成本考量**：平衡解析度與成本
- **效能影響**：考慮對效能的影響
- **未來需求**：規劃未來的需求

### **2. 實作適當的濾波**

- **雜訊降低**：使用濾波降低雜訊
- **訊號調理**：在處理前調理訊號
- **平均**：使用平均以獲得更好的精確度
- **過取樣**：使用過取樣以獲得更高解析度

### **3. 校準感測器**

- **初始校準**：在設定期間校準感測器
- **定期校準**：定期重新校準
- **溫度補償**：補償溫度影響
- **線性化**：修正非線性感測器響應

### **4. 處理雜訊與干擾**

- **遮蔽**：對敏感訊號使用適當的遮蔽
- **接地**：實作適當的接地
- **濾波**：使用硬體和軟體濾波
- **佈局**：考慮類比訊號的 PCB 佈局

### **5. 效能最佳化**

- **取樣率**：選擇適當的取樣率
- **轉換時間**：考慮轉換時間需求
- **處理開銷**：最小化處理開銷
- **記憶體使用**：最佳化緩衝區的記憶體使用

## 🎯 面試問題

### **基礎問題**

1. **什麼是類比 I/O，為什麼它很重要？**
   - 處理連續電壓或電流訊號
   - 與真實世界的類比現象介面
   - 對感測器、致動器和控制系統至關重要
   - 實現高精度量測和控制

2. **ADC 和 DAC 之間的主要差異是什麼？**
   - ADC：將類比訊號轉換為數位訊號
   - DAC：將數位訊號轉換為類比訊號
   - ADC：用於感測器介面和量測
   - DAC：用於致動器控制和訊號產生

3. **如何處理類比訊號中的雜訊？**
   - 使用硬體濾波（電容、電感）
   - 實作軟體濾波（平均、數位濾波器）
   - 使用適當的遮蔽和接地
   - 實作過取樣技術

### **進階問題**

1. **如何設計高精度溫度量測系統？**
   - 使用高解析度 ADC（16 位元或更高）
   - 實作適當的訊號調理
   - 使用校準和線性化
   - 實作雜訊降低技術

2. **如何最佳化類比 I/O 效能？**
   - 選擇適當的取樣率
   - 使用高效的濾波演算法
   - 實作適當的緩衝
   - 針對即時需求最佳化

3. **如何處理類比訊號校準？**
   - 實作兩點校準
   - 使用溫度補償
   - 對非線性感測器實作線性化
   - 將校準資料存儲在非揮發性記憶體中

### **實作問題**

1. **編寫一個實作 ADC 過取樣的函式**
2. **實作用於雜訊降低的移動平均濾波器**
3. **建立帶有校準的溫度感測器介面**
4. **設計使用 DAC 的馬達速度控制系統**

## 📚 額外資源

### **書籍**
- "The Art of Electronics" by Paul Horowitz and Winfield Hill
- "Analog Circuit Design" by Jim Williams
- "Embedded Systems: Introduction to ARM Cortex-M Microcontrollers" by Jonathan Valvano

### **線上資源**
- [Analog I/O Tutorial](https://www.tutorialspoint.com/embedded_systems/es_analog_io.htm)
- [ADC Fundamentals](https://www.allaboutcircuits.com/technical-articles/analog-to-digital-conversion/)
- [DAC Fundamentals](https://www.allaboutcircuits.com/technical-articles/digital-to-analog-conversion/)

### **工具**
- **示波器**：類比訊號分析工具
- **訊號產生器**：類比訊號產生工具
- **萬用表**：電壓和電流量測工具
- **頻譜分析儀**：頻域分析工具

### **標準**
- **ADC 標準**：業界 ADC 標準
- **DAC 標準**：業界 DAC 標準
- **訊號標準**：類比訊號標準
- **校準標準**：校準和量測標準

---

**下一步**：探索 [脈寬調變](./Pulse_Width_Modulation.md) 以瞭解 PWM 控制技術，或深入 [計時器/計數器程式設計](./Timer_Counter_Programming.md) 以瞭解計時應用。

## 🎯 **實務考量與取捨**

### **系統層級設計決策**
在設計類比 I/O 系統時，工程師必須平衡多個競爭需求：

**精確度與速度的取捨**：
- 更高解析度的 ADC 提供更好的精確度，但需要更長的轉換時間
- 更快的取樣率改善時間解析度，但可能增加雜訊
- 過取樣可以改善有效解析度，但需要更多的處理能力

**成本與效能的取捨**：
- 高精度元件（低漂移參考、精密電阻）改善精確度但增加成本
- 整合式解決方案減少電路板空間，但可能限制客製化
- 離散式設計提供靈活性，但需要仔細的元件選擇和佈局

### **環境因素**
真實世界的部署引入了必須解決的挑戰：

**溫度效應**：
- 參考電壓隨溫度的漂移影響 ADC 和 DAC 的精確度
- 元件值隨溫度變化，影響濾波器特性
- 熱雜訊隨溫度增加，降低訊號品質

**電源供應考量**：
- 電源雜訊直接耦合到類比訊號
- 電壓穩壓器必須提供乾淨、穩定的輸出
- 接地平面設計對最小化干擾至關重要

**EMI/EMC 挑戰**：
- 高頻切換可能耦合到敏感的類比電路
- 適當的遮蔽和濾波對工業環境至關重要
- 符合 EMC 標準需要仔細的設計和測試

### **整合挑戰**
現代嵌入式系統通常需要多個類比 I/O 通道：

**通道隔離**：
- 通道間的串擾可能損壞量測
- 多工 ADC 需要仔細的時序以避免干擾
- 對於高電壓或浮動量測，可能需要接地隔離

**同步**：
- 對於相位敏感量測，多個 ADC 必須同步
- DAC 輸出可能需要精確的時序以產生波形
- 時鐘抖動影響取樣精確度和輸出品質

**資料管理**：
- 高速取樣產生大量資料
- 即時處理需求限制了緩衝選項
- 資料完整性必須在系統邊界間維護

### **設計方法論與最佳實踐**
成功的類比 I/O 設計需要系統化的方法：

**由上而下的設計流程**：
1. **需求分析**：定義精確度、頻寬和環境需求
2. **架構選擇**：在整合式和離散式解決方案之間選擇
3. **元件選擇**：選擇適當的 ADC、DAC 和支援元件
4. **佈局與實作**：以適當的類比考量設計 PCB
5. **測試與驗證**：在所有操作條件下驗證效能

**應避免的常見陷阱**：
- **接地迴路**：不當的接地可能造成量測誤差
- **訊號完整性**：過長的走線或不良的阻抗匹配會降低訊號品質
- **電源雜訊**：不足的濾波導致效能不佳
- **熱管理**：溫度變化影響元件效能
- **EMI 敏感性**：不良的遮蔽導致干擾

**驗證策略**：
- **蒙地卡羅分析**：考慮元件公差和變異
- **溫度測試**：驗證整個操作溫度範圍的效能
- **雜訊分析**：量測並分析雜訊來源及其影響
- **長期穩定性**：在較長時間內監控效能

## 🧪 引導式實驗

### 實驗 1：ADC 配置與量測
1. **設定**：使用不同的取樣時間和解析度配置 ADC
2. **量測**：使用已知電壓源測試並量測精確度
3. **分析**：計算 ENOB 並與資料手冊規格比較
4. **最佳化**：針對不同來源阻抗調整取樣時間

### 實驗 2：DAC 波形產生
1. **配置**：使用適當的參考電壓和解析度設定 DAC
2. **產生**：建立正弦波、方波和三角波
3. **量測**：使用示波器驗證輸出精確度和穩定時間
4. **最佳化**：調整輸出緩衝器設定以取捨速度與精確度

### 實驗 3：訊號調理實作
1. **設計**：實作低通濾波器和平均演算法
2. **測試**：應用於有雜訊的感測器訊號並量測改善
3. **校準**：實作偏移和增益校準
4. **驗證**：使用已知輸入訊號測試並驗證精確度

## ✅ 自我檢查

### 理解檢查
- [ ] 你能解釋 ADC 解析度和量測精確度之間的關係嗎？
- [ ] 你瞭解取樣率如何影響訊號重建嗎？
- [ ] 你能描述取樣時間和轉換精確度之間的取捨嗎？
- [ ] 你知道如何從量測資料計算 ENOB 嗎？

### 應用檢查
- [ ] 你能為不同的訊號類型和來源阻抗配置 ADC 嗎？
- [ ] 你能以適當的時序實作 DAC 輸出產生嗎？
- [ ] 你能設計用於雜訊降低的訊號調理濾波器嗎？
- [ ] 你能實作用於感測器精確度的校準程序嗎？

### 分析檢查
- [ ] 你能分析 ADC 效能並識別瓶頸嗎？
- [ ] 你能量測並最佳化 DAC 穩定時間嗎？
- [ ] 你能設計多階段訊號調理管線嗎？
- [ ] 你能排除類比訊號完整性問題嗎？

## 🔗 交叉連結

- **[型別限定詞](../Embedded_C/Type_Qualifiers.md)** - ADC/DAC 的 volatile 暫存器存取
- **[計時器計數器程式設計](./Timer_Counter_Programming.md)** - 取樣時序和 PWM 產生
- **[GPIO 配置](./GPIO_Configuration.md)** - 類比腳位配置
- **[時鐘管理](./Clock_Management.md)** - ADC/DAC 時鐘配置
- **[電源管理](./Power_Management.md)** - 類比電源供應考量

## 結論
