# 記憶體映射 I/O

## 📋 目錄
- [概述](#-概述)
- [記憶體映射 I/O 基礎](#-記憶體映射-io-基礎)
- [硬體暫存器存取](#-硬體暫存器存取)
- [Volatile 關鍵字用法](#-volatile-關鍵字用法)
- [暫存器位元操作](#-暫存器位元操作)
- [周邊設定](#-周邊設定)
- [中斷處理](#-中斷處理)
- [DMA 與記憶體映射 I/O](#-dma-與記憶體映射-io)
- [即時考量](#-即時考量)
- [常見陷阱](#-常見陷阱)
- [最佳實踐](#-最佳實踐)
- [面試問題](#-面試問題)
- [額外資源](#-額外資源)

## 🎯 概述

### 概念：在固定位址上建立型別化的 volatile 視圖

將周邊暫存器視為已知位址上具有精確寬度的 `volatile` 物件。使用最少的型別化存取器，並避免意外的非 volatile 別名。

### 為何在嵌入式中重要
- 防止編譯器省略或重新排序關鍵的 I/O 操作。
- 釐清意圖（唯讀狀態暫存器 vs 純寫入命令暫存器）。
- 便於程式碼審查和靜態分析。

### 最小範例
```c
typedef struct {
  volatile uint32_t CTRL;
  volatile const uint32_t STAT;  // 唯讀
  volatile uint32_t DATA;
} periph_t;

#define PERIPH ((periph_t*)0x40010000u)

static inline void periph_enable(void) { PERIPH->CTRL |= 1u; }
static inline uint32_t periph_ready(void) { return (PERIPH->STAT & 1u) != 0u; }
```

### 動手試試
1. 刻意移除 `volatile` 並使用 `-O2` 編譯；展示被提升的載入或被移除的寫入。
2. 在架構要求的地方加入記憶體屏障（例如，在啟用時脈後），並量測行為。

### 重點結論
- 始終透過 `volatile` 限定型別/指標存取暫存器。
- 透過在 `volatile` 欄位上使用 `const` 明確表示唯讀/純寫入語義。
- 在需要排序的平台上考慮使用記憶體屏障。

### 面試官意圖（他們在探查什麼）
- 你能否在 C 中安全且清楚地建模暫存器？
- 你是否知道 `volatile` 為何重要以及它不能解決什麼？
- 你能否解釋讀取-修改-寫入風險和排序問題？

---

## 🧪 引導實驗
1) 最佳化驚喜
- 透過 `volatile` 和非 `volatile` 別名存取同一暫存器；比較產生的程式碼。

2) 排序
- 插入一個啟用周邊的寫入，緊接著一個依賴的讀取；在需要排序的平台上加入/移除屏障並觀察行為。

## ✅ 自我檢測
- `volatile` 不保證什麼（例如，原子性）？
- 你如何安全地建模純寫入暫存器欄位？

## 🔗 交叉連結
- `Embedded_C/Type_Qualifiers.md` 限定詞
- `Embedded_C/Compiler_Intrinsics.md` 屏障

記憶體映射 I/O 允許透過記憶體位址直接存取硬體暫存器，實現與周邊的高效通訊。此技術是嵌入式系統中控制硬體的基礎，無需專用的 I/O 指令。

## 🔧 記憶體映射 I/O 基礎

### 暫存器結構定義
```c
// 記憶體映射 I/O 暫存器結構
typedef struct {
    volatile uint32_t control;      // 控制暫存器
    volatile uint32_t status;       // 狀態暫存器
    volatile uint32_t data;         // 資料暫存器
    volatile uint32_t reserved;     // 對齊保留
} __attribute__((aligned(4))) peripheral_registers_t;

// 記憶體映射 I/O 基底位址
#define PERIPHERAL_BASE_ADDRESS    0x40000000
#define GPIO_BASE_ADDRESS          0x40020000
#define UART_BASE_ADDRESS          0x40021000
#define SPI_BASE_ADDRESS           0x40022000
#define I2C_BASE_ADDRESS           0x40023000

// 周邊暫存器映射
peripheral_registers_t* map_peripheral_registers(uint32_t base_address) {
    // 確保基底位址已對齊
    if (base_address & 0x3) {
        return NULL;  // 無效的對齊
    }
    
    return (peripheral_registers_t*)base_address;
}

// 使用範例
peripheral_registers_t* uart_regs = map_peripheral_registers(UART_BASE_ADDRESS);
if (uart_regs) {
    // 直接存取 UART 暫存器
    uart_regs->control = 0x01;  // 啟用 UART
}
```

### 暫存器存取巨集
```c
// 安全的暫存器存取巨集
#define REG_READ(addr)             (*(volatile uint32_t*)(addr))
#define REG_WRITE(addr, value)     (*(volatile uint32_t*)(addr) = (value))
#define REG_SET_BITS(addr, mask)   (*(volatile uint32_t*)(addr) |= (mask))
#define REG_CLEAR_BITS(addr, mask) (*(volatile uint32_t*)(addr) &= ~(mask))

// 位元操作巨集
#define BIT_SET(reg, bit)          ((reg) |= (1U << (bit)))
#define BIT_CLEAR(reg, bit)        ((reg) &= ~(1U << (bit)))
#define BIT_TOGGLE(reg, bit)       ((reg) ^= (1U << (bit)))
#define BIT_READ(reg, bit)         (((reg) >> (bit)) & 1U)

// 使用範例
void configure_gpio_pin(uint32_t gpio_base, uint8_t pin, uint8_t mode) {
    uint32_t* gpio_regs = (uint32_t*)gpio_base;
    
    // 設定腳位模式
    REG_SET_BITS(gpio_regs[0], (mode << (pin * 2)));
    
    // 啟用 GPIO 時脈
    REG_SET_BITS(gpio_regs[1], (1U << pin));
}
```

## 🔧 硬體暫存器存取

### GPIO 暫存器存取
```c
// GPIO 暫存器結構
typedef struct {
    volatile uint32_t moder;    // 模式暫存器
    volatile uint32_t otyper;   // 輸出類型暫存器
    volatile uint32_t ospeedr;  // 輸出速度暫存器
    volatile uint32_t pupdr;    // 上拉/下拉暫存器
    volatile uint32_t idr;      // 輸入資料暫存器
    volatile uint32_t odr;      // 輸出資料暫存器
    volatile uint32_t bsrr;     // 位元設定/重設暫存器
    volatile uint32_t lckr;     // 設定鎖定暫存器
    volatile uint32_t afrl;     // 替代功能低位暫存器
    volatile uint32_t afrh;     // 替代功能高位暫存器
} __attribute__((aligned(4))) gpio_registers_t;

// GPIO 設定函式
void gpio_set_pin_mode(gpio_registers_t* gpio, uint8_t pin, uint8_t mode) {
    uint32_t mask = 3U << (pin * 2);  // 每個腳位 2 位元
    uint32_t value = mode << (pin * 2);
    
    // 清除並設定模式位元
    gpio->moder &= ~mask;
    gpio->moder |= value;
}

void gpio_set_pin_output(gpio_registers_t* gpio, uint8_t pin, bool high) {
    if (high) {
        gpio->bsrr = (1U << pin);  // 設定位元
    } else {
        gpio->bsrr = (1U << (pin + 16));  // 重設位元
    }
}

bool gpio_read_pin_input(gpio_registers_t* gpio, uint8_t pin) {
    return (gpio->idr & (1U << pin)) != 0;
}
```

### UART 暫存器存取
```c
// UART 暫存器結構
typedef struct {
    volatile uint32_t sr;       // 狀態暫存器
    volatile uint32_t dr;       // 資料暫存器
    volatile uint32_t brr;      // 鮑率暫存器
    volatile uint32_t cr1;      // 控制暫存器 1
    volatile uint32_t cr2;      // 控制暫存器 2
    volatile uint32_t cr3;      // 控制暫存器 3
    volatile uint32_t gtpr;     // 保護時間和預分頻暫存器
} __attribute__((aligned(4))) uart_registers_t;

// UART 設定
void uart_configure(uart_registers_t* uart, uint32_t baud_rate, uint32_t system_clock) {
    // 計算鮑率除數
    uint32_t divisor = system_clock / baud_rate;
    uart->brr = divisor;
    
    // 啟用 UART
    uart->cr1 = UART_CR1_UE | UART_CR1_TE | UART_CR1_RE;
}

void uart_send_byte(uart_registers_t* uart, uint8_t data) {
    // 等待傳送資料暫存器為空
    while (!(uart->sr & UART_SR_TXE));
    
    // 傳送資料
    uart->dr = data;
}

uint8_t uart_receive_byte(uart_registers_t* uart) {
    // 等待接收資料暫存器非空
    while (!(uart->sr & UART_SR_RXNE));
    
    // 讀取資料
    return (uint8_t)(uart->dr & 0xFF);
}
```

## ⚡ Volatile 關鍵字用法

### Volatile 暫存器存取
```c
// Volatile 暫存器存取模式
typedef struct {
    volatile uint32_t control;
    volatile uint32_t status;
    volatile uint32_t data;
} volatile_peripheral_t;

// 正確的 volatile 用法
void safe_register_access(volatile_peripheral_t* peripheral) {
    // 讀取狀態暫存器（volatile 確保實際的記憶體讀取）
    uint32_t status = peripheral->status;
    
    // 檢查特定位元
    if (status & STATUS_BIT_READY) {
        // 寫入資料暫存器
        peripheral->data = 0x12345678;
        
        // 設定控制位元
        peripheral->control |= CONTROL_BIT_ENABLE;
    }
}

// 錯誤的非 volatile 用法
typedef struct {
    uint32_t control;  // 錯誤：缺少 volatile
    uint32_t status;   // 錯誤：缺少 volatile
    uint32_t data;     // 錯誤：缺少 volatile
} non_volatile_peripheral_t;

void unsafe_register_access(non_volatile_peripheral_t* peripheral) {
    // 編譯器可能最佳化掉這個讀取
    uint32_t status = peripheral->status;  // 可能被快取
    
    // 編譯器可能最佳化掉這個寫入
    peripheral->data = 0x12345678;  // 可能不會實際寫入硬體
}
```

### Volatile 指標用法
```c
// 指向非 volatile 資料的 volatile 指標
volatile uint32_t* const hardware_register = (volatile uint32_t*)0x40000000;

// 指向 volatile 資料的非 volatile 指標
uint32_t* volatile status_pointer = (uint32_t*)0x40000004;

// 指向 volatile 資料的 volatile 指標
volatile uint32_t* volatile config_register = (volatile uint32_t*)0x40000008;

// 安全的硬體存取函式
uint32_t read_hardware_register(volatile uint32_t* reg) {
    return *reg;  // volatile 確保實際的記憶體讀取
}

void write_hardware_register(volatile uint32_t* reg, uint32_t value) {
    *reg = value;  // volatile 確保實際的記憶體寫入
}

// 使用範例
void configure_hardware(void) {
    // 讀取目前狀態
    uint32_t status = read_hardware_register(hardware_register);
    
    // 修改設定
    write_hardware_register(config_register, 0x12345678);
}
```

## 🔧 暫存器位元操作

### 位元欄位定義
```c
// 暫存器位元欄位定義
#define GPIO_MODER_INPUT    0x00
#define GPIO_MODER_OUTPUT   0x01
#define GPIO_MODER_ALT      0x02
#define GPIO_MODER_ANALOG   0x03

#define GPIO_OTYPER_PUSH_PULL  0x00
#define GPIO_OTYPER_OPEN_DRAIN 0x01

#define GPIO_OSPEEDR_LOW     0x00
#define GPIO_OSPEEDR_MEDIUM  0x01
#define GPIO_OSPEEDR_HIGH    0x02
#define GPIO_OSPEEDR_VERY_HIGH 0x03

// 位元操作函式
void gpio_configure_pin(gpio_registers_t* gpio, uint8_t pin, 
                       uint8_t mode, uint8_t output_type, uint8_t speed) {
    uint32_t moder_mask = 3U << (pin * 2);
    uint32_t moder_value = mode << (pin * 2);
    
    uint32_t otyper_mask = 1U << pin;
    uint32_t otyper_value = output_type << pin;
    
    uint32_t ospeedr_mask = 3U << (pin * 2);
    uint32_t ospeedr_value = speed << (pin * 2);
    
    // 設定模式
    gpio->moder &= ~moder_mask;
    gpio->moder |= moder_value;
    
    // 設定輸出類型
    gpio->otyper &= ~otyper_mask;
    gpio->otyper |= otyper_value;
    
    // 設定速度
    gpio->ospeedr &= ~ospeedr_mask;
    gpio->ospeedr |= ospeedr_value;
}

// 原子位元操作
void gpio_atomic_set_pin(gpio_registers_t* gpio, uint8_t pin) {
    // 使用 BSRR 進行原子設定
    gpio->bsrr = (1U << pin);
}

void gpio_atomic_clear_pin(gpio_registers_t* gpio, uint8_t pin) {
    // 使用 BSRR 進行原子清除
    gpio->bsrr = (1U << (pin + 16));
}

void gpio_atomic_toggle_pin(gpio_registers_t* gpio, uint8_t pin) {
    // 讀取目前狀態並切換
    uint32_t current_state = gpio->odr;
    if (current_state & (1U << pin)) {
        gpio->bsrr = (1U << (pin + 16));  // 清除
    } else {
        gpio->bsrr = (1U << pin);  // 設定
    }
}
```

### 暫存器讀取-修改-寫入
```c
// 安全的讀取-修改-寫入操作
uint32_t register_read_modify_write(volatile uint32_t* reg, 
                                   uint32_t mask, uint32_t value) {
    uint32_t old_value = *reg;
    *reg = (old_value & ~mask) | (value & mask);
    return old_value;
}

// 範例：設定 UART 控制暫存器
void uart_configure_control(uart_registers_t* uart, uint32_t config_bits) {
    // 讀取目前控制暫存器
    uint32_t current_cr1 = uart->cr1;
    
    // 清除設定位元並設定新值
    current_cr1 &= ~(UART_CR1_CONFIG_MASK);
    current_cr1 |= (config_bits & UART_CR1_CONFIG_MASK);
    
    // 寫回暫存器
    uart->cr1 = current_cr1;
}

// 原子暫存器操作
void atomic_register_set_bits(volatile uint32_t* reg, uint32_t bits) {
    *reg |= bits;  // 原子 OR 操作
}

void atomic_register_clear_bits(volatile uint32_t* reg, uint32_t bits) {
    *reg &= ~bits;  // 原子 AND 操作
}

void atomic_register_toggle_bits(volatile uint32_t* reg, uint32_t bits) {
    *reg ^= bits;  // 原子 XOR 操作
}
```

## ⚙️ 周邊設定

### 周邊時脈控制
```c
// 時脈控制暫存器結構
typedef struct {
    volatile uint32_t ahb1enr;   // AHB1 周邊時脈啟用
    volatile uint32_t ahb2enr;   // AHB2 周邊時脈啟用
    volatile uint32_t apb1enr;   // APB1 周邊時脈啟用
    volatile uint32_t apb2enr;   // APB2 周邊時脈啟用
} __attribute__((aligned(4))) rcc_registers_t;

#define RCC_BASE_ADDRESS 0x40023800

// 時脈啟用/停用函式
void enable_peripheral_clock(uint32_t peripheral_bit, uint32_t clock_register) {
    volatile uint32_t* rcc = (volatile uint32_t*)RCC_BASE_ADDRESS;
    rcc[clock_register] |= (1U << peripheral_bit);
}

void disable_peripheral_clock(uint32_t peripheral_bit, uint32_t clock_register) {
    volatile uint32_t* rcc = (volatile uint32_t*)RCC_BASE_ADDRESS;
    rcc[clock_register] &= ~(1U << peripheral_bit);
}

// 範例：啟用 GPIOA 時脈
void enable_gpio_a_clock(void) {
    enable_peripheral_clock(0, 0);  // GPIOA 是 AHB1ENR 的位元 0
}
```

### 周邊重設控制
```c
// 重設控制函式
void reset_peripheral(uint32_t peripheral_bit, uint32_t reset_register) {
    volatile uint32_t* rcc = (volatile uint32_t*)RCC_BASE_ADDRESS;
    
    // 宣告重設
    rcc[reset_register] |= (1U << peripheral_bit);
    
    // 等待重設生效
    for (volatile int i = 0; i < 100; i++);
    
    // 解除重設
    rcc[reset_register] &= ~(1U << peripheral_bit);
    
    // 等待重設完成
    for (volatile int i = 0; i < 100; i++);
}

// 範例：重設 UART 周邊
void reset_uart_peripheral(void) {
    reset_peripheral(4, 1);  // UART1 是 APB1RSTR 的位元 4
}
```

## 🔄 中斷處理

### 中斷暫存器存取
```c
// NVIC（巢狀向量中斷控制器）暫存器
typedef struct {
    volatile uint32_t iser[8];    // 中斷設定啟用暫存器
    volatile uint32_t icer[8];    // 中斷清除啟用暫存器
    volatile uint32_t ispr[8];    // 中斷設定待處理暫存器
    volatile uint32_t icpr[8];    // 中斷清除待處理暫存器
    volatile uint32_t iabr[8];    // 中斷活動位元暫存器
    volatile uint32_t ipr[60];    // 中斷優先權暫存器
} __attribute__((aligned(4))) nvic_registers_t;

#define NVIC_BASE_ADDRESS 0xE000E100

// 中斷控制函式
void enable_interrupt(uint8_t irq_number) {
    volatile uint32_t* nvic = (volatile uint32_t*)NVIC_BASE_ADDRESS;
    uint8_t reg_index = irq_number / 32;
    uint8_t bit_position = irq_number % 32;
    
    nvic->iser[reg_index] = (1U << bit_position);
}

void disable_interrupt(uint8_t irq_number) {
    volatile uint32_t* nvic = (volatile uint32_t*)NVIC_BASE_ADDRESS;
    uint8_t reg_index = irq_number / 32;
    uint8_t bit_position = irq_number % 32;
    
    nvic->icer[reg_index] = (1U << bit_position);
}

void set_interrupt_priority(uint8_t irq_number, uint8_t priority) {
    volatile uint32_t* nvic = (volatile uint32_t*)NVIC_BASE_ADDRESS;
    uint8_t reg_index = irq_number / 4;
    uint8_t byte_position = (irq_number % 4) * 8;
    
    uint32_t mask = 0xFF << byte_position;
    uint32_t value = priority << byte_position;
    
    nvic->ipr[reg_index] = (nvic->ipr[reg_index] & ~mask) | value;
}
```

### 周邊中斷設定
```c
// UART 中斷設定
void uart_enable_interrupts(uart_registers_t* uart, uint32_t interrupt_bits) {
    // 在 UART 中啟用特定中斷
    uart->cr1 |= interrupt_bits;
    
    // 在 NVIC 中啟用 UART 中斷
    enable_interrupt(UART_IRQ_NUMBER);
}

void uart_disable_interrupts(uart_registers_t* uart, uint32_t interrupt_bits) {
    // 在 UART 中停用特定中斷
    uart->cr1 &= ~interrupt_bits;
    
    // 檢查是否還有中斷啟用
    if (!(uart->cr1 & UART_INTERRUPT_MASK)) {
        disable_interrupt(UART_IRQ_NUMBER);
    }
}

// UART 中斷處理常式
void uart_interrupt_handler(uart_registers_t* uart) {
    uint32_t status = uart->sr;
    
    // 檢查接收中斷
    if (status & UART_SR_RXNE) {
        uint8_t data = (uint8_t)(uart->dr & 0xFF);
        process_received_data(data);
    }
    
    // 檢查傳送中斷
    if (status & UART_SR_TXE) {
        if (has_data_to_transmit()) {
            uart->dr = get_next_byte_to_transmit();
        } else {
            uart->cr1 &= ~UART_CR1_TXEIE;  // 停用 TXE 中斷
        }
    }
}
```

## 🔄 DMA 與記憶體映射 I/O

### DMA 周邊設定
```c
// DMA 周邊設定
typedef struct {
    volatile uint32_t cr;         // 控制暫存器
    volatile uint32_t ndtr;       // 資料數量暫存器
    volatile uint32_t par;        // 周邊位址暫存器
    volatile uint32_t mar;        // 記憶體位址暫存器
    volatile uint32_t reserved;
    volatile uint32_t fcr;        // FIFO 控制暫存器
} __attribute__((aligned(4))) dma_stream_registers_t;

void configure_dma_for_peripheral(dma_stream_registers_t* dma_stream,
                                 uint32_t peripheral_address,
                                 uint32_t memory_address,
                                 uint32_t transfer_count) {
    // 停用 DMA 串流
    dma_stream->cr &= ~DMA_CR_EN;
    
    // 等待 DMA 停用
    while (dma_stream->cr & DMA_CR_EN);
    
    // 設定周邊位址
    dma_stream->par = peripheral_address;
    
    // 設定記憶體位址
    dma_stream->mar = memory_address;
    
    // 設定傳輸數量
    dma_stream->ndtr = transfer_count;
    
    // 設定控制暫存器
    dma_stream->cr = DMA_CR_DIR_PERIPH_TO_MEM |
                     DMA_CR_MINC |
                     DMA_CR_PSIZE_WORD |
                     DMA_CR_MSIZE_WORD |
                     DMA_CR_PL_HIGH;
    
    // 啟用 DMA 串流
    dma_stream->cr |= DMA_CR_EN;
}
```

### DMA 與 UART 範例
```c
// UART DMA 設定
void uart_configure_dma_receive(uart_registers_t* uart, 
                               dma_stream_registers_t* dma_stream,
                               uint8_t* buffer, uint32_t buffer_size) {
    // 設定 DMA 用於 UART 接收
    configure_dma_for_peripheral(dma_stream,
                                (uint32_t)&uart->dr,  // UART 資料暫存器
                                (uint32_t)buffer,     // 記憶體緩衝區
                                buffer_size);
    
    // 啟用 UART DMA 接收
    uart->cr3 |= UART_CR3_DMAR;
    
    // 啟用 UART 接收
    uart->cr1 |= UART_CR1_RE;
}

void uart_configure_dma_transmit(uart_registers_t* uart,
                                dma_stream_registers_t* dma_stream,
                                const uint8_t* buffer, uint32_t buffer_size) {
    // 設定 DMA 用於 UART 傳送
    configure_dma_for_peripheral(dma_stream,
                                (uint32_t)&uart->dr,  // UART 資料暫存器
                                (uint32_t)buffer,     // 記憶體緩衝區
                                buffer_size);
    
    // 啟用 UART DMA 傳送
    uart->cr3 |= UART_CR3_DMAT;
    
    // 啟用 UART 傳送
    uart->cr1 |= UART_CR1_TE;
}
```

## ⏱️ 即時考量

### 暫存器存取時序
```c
// 時序關鍵的暫存器存取
typedef struct {
    volatile uint32_t* register_address;
    uint32_t access_time_ns;
    bool is_timing_critical;
} timing_critical_register_t;

// 針對即時系統最佳化的暫存器存取
uint32_t fast_register_read(volatile uint32_t* reg) {
    // 確保單指令存取
    return *reg;
}

void fast_register_write(volatile uint32_t* reg, uint32_t value) {
    // 確保單指令寫入
    *reg = value;
}

// 關鍵時序暫存器存取
void critical_timing_register_access(volatile uint32_t* reg, uint32_t value) {
    // 在關鍵存取期間停用中斷
    uint32_t primask = __get_PRIMASK();
    __disable_irq();
    
    // 執行暫存器存取
    *reg = value;
    
    // 恢復中斷狀態
    if (!primask) {
        __enable_irq();
    }
}
```

### 暫存器存取延遲
```c
// 量測暫存器存取延遲
typedef struct {
    uint32_t read_latency_ns;
    uint32_t write_latency_ns;
    uint32_t access_count;
} register_latency_monitor_t;

register_latency_monitor_t* create_register_latency_monitor(void) {
    register_latency_monitor_t* monitor = malloc(sizeof(register_latency_monitor_t));
    if (monitor) {
        monitor->read_latency_ns = 0;
        monitor->write_latency_ns = 0;
        monitor->access_count = 0;
    }
    return monitor;
}

uint32_t measure_register_read_latency(volatile uint32_t* reg) {
    uint32_t start_time = get_system_tick_count();
    uint32_t value = *reg;
    uint32_t end_time = get_system_tick_count();
    
    return (end_time - start_time) * SYSTEM_TICK_PERIOD_NS;
}
```

## ⚠️ 常見陷阱

### 1. 缺少 Volatile 關鍵字
```c
// 錯誤：缺少 volatile 關鍵字
typedef struct {
    uint32_t control;  // 缺少 volatile
    uint32_t status;   // 缺少 volatile
} incorrect_peripheral_t;

void incorrect_access(incorrect_peripheral_t* peripheral) {
    // 編譯器可能最佳化掉這個存取
    peripheral->control = 0x01;
    // 可能不會實際寫入硬體
}

// 正確：使用 volatile 關鍵字
typedef struct {
    volatile uint32_t control;  // 正確
    volatile uint32_t status;   // 正確
} correct_peripheral_t;

void correct_access(correct_peripheral_t* peripheral) {
    // volatile 確保實際的硬體存取
    peripheral->control = 0x01;
}
```

### 2. 競爭條件
```c
// 錯誤：暫存器存取中的競爭條件
void unsafe_register_modification(volatile uint32_t* reg, uint32_t mask, uint32_t value) {
    uint32_t current = *reg;  // 讀取
    current &= ~mask;         // 修改
    current |= value;
    *reg = current;           // 寫入
    // 讀取和寫入之間存在競爭條件
}

// 正確：原子暫存器修改
void safe_register_modification(volatile uint32_t* reg, uint32_t mask, uint32_t value) {
    // 使用原子操作或停用中斷
    uint32_t primask = __get_PRIMASK();
    __disable_irq();
    
    uint32_t current = *reg;
    current = (current & ~mask) | (value & mask);
    *reg = current;
    
    if (!primask) {
        __enable_irq();
    }
}
```

### 3. 不正確的暫存器對齊
```c
// 錯誤：未對齊的暫存器存取
void incorrect_register_access(void) {
    uint8_t* unaligned_ptr = (uint8_t*)0x40000001;  // 未對齊的位址
    uint32_t* reg = (uint32_t*)unaligned_ptr;        // 可能導致對齊錯誤
    *reg = 0x12345678;  // 潛在的對齊錯誤
}

// 正確：對齊的暫存器存取
void correct_register_access(void) {
    uint32_t* reg = (uint32_t*)0x40000000;  // 對齊的位址
    *reg = 0x12345678;  // 安全存取
}
```

## ✅ 最佳實踐

### 1. 暫存器存取安全
```c
// 安全的暫存器存取模式
typedef struct {
    volatile uint32_t* register_ptr;
    uint32_t register_mask;
    uint32_t register_shift;
    const char* register_name;
} safe_register_access_t;

safe_register_access_t* create_safe_register_access(volatile uint32_t* reg,
                                                   uint32_t mask,
                                                   uint32_t shift,
                                                   const char* name) {
    safe_register_access_t* access = malloc(sizeof(safe_register_access_t));
    if (access) {
        access->register_ptr = reg;
        access->register_mask = mask;
        access->register_shift = shift;
        access->register_name = name;
    }
    return access;
}

uint32_t safe_register_read(safe_register_access_t* access) {
    if (!access || !access->register_ptr) {
        return 0;
    }
    
    uint32_t value = *access->register_ptr;
    return (value >> access->register_shift) & access->register_mask;
}

void safe_register_write(safe_register_access_t* access, uint32_t value) {
    if (!access || !access->register_ptr) {
        return;
    }
    
    uint32_t current = *access->register_ptr;
    uint32_t masked_value = (value & access->register_mask) << access->register_shift;
    
    *access->register_ptr = (current & ~(access->register_mask << access->register_shift)) | masked_value;
}
```

### 2. 暫存器驗證
```c
// 暫存器存取驗證
bool is_valid_register_address(uint32_t address) {
    // 檢查位址是否在有效的周邊範圍內
    return (address >= PERIPHERAL_BASE_ADDRESS && 
            address < (PERIPHERAL_BASE_ADDRESS + PERIPHERAL_SIZE));
}

bool is_aligned_register_address(uint32_t address, uint32_t alignment) {
    return (address % alignment) == 0;
}

volatile uint32_t* validate_and_map_register(uint32_t address, uint32_t alignment) {
    if (!is_valid_register_address(address)) {
        return NULL;
    }
    
    if (!is_aligned_register_address(address, alignment)) {
        return NULL;
    }
    
    return (volatile uint32_t*)address;
}
```

### 3. 暫存器存取記錄
```c
// 用於除錯的暫存器存取記錄
typedef struct {
    uint32_t address;
    uint32_t value;
    bool is_write;
    uint32_t timestamp;
} register_access_log_t;

typedef struct {
    register_access_log_t* log_entries;
    size_t log_size;
    size_t current_index;
    bool logging_enabled;
} register_logger_t;

register_logger_t* create_register_logger(size_t log_size) {
    register_logger_t* logger = malloc(sizeof(register_logger_t));
    if (!logger) return NULL;
    
    logger->log_entries = malloc(log_size * sizeof(register_access_log_t));
    logger->log_size = log_size;
    logger->current_index = 0;
    logger->logging_enabled = true;
    
    return logger;
}

void log_register_access(register_logger_t* logger, uint32_t address, 
                        uint32_t value, bool is_write) {
    if (!logger || !logger->logging_enabled) {
        return;
    }
    
    logger->log_entries[logger->current_index].address = address;
    logger->log_entries[logger->current_index].value = value;
    logger->log_entries[logger->current_index].is_write = is_write;
    logger->log_entries[logger->current_index].timestamp = get_system_tick_count();
    
    logger->current_index = (logger->current_index + 1) % logger->log_size;
}
```

## 🎯 面試問題

### 基礎問題
1. **什麼是記憶體映射 I/O，為什麼要使用它？**
   - 透過記憶體位址直接存取硬體暫存器
   - 用於高效的周邊控制，無需專用的 I/O 指令

2. **為什麼 volatile 關鍵字在記憶體映射 I/O 中很重要？**
   - 防止編譯器最佳化暫存器存取
   - 確保對硬體的實際記憶體讀寫

3. **暫存器存取的關鍵考量有哪些？**
   - 正確的對齊
   - volatile 關鍵字的使用
   - 競爭條件防護
   - 原子操作

### 進階問題
1. **如何實作原子暫存器操作？**
   - 使用硬體原子指令
   - 在關鍵區段期間停用中斷
   - 使用帶有適當同步的讀取-修改-寫入

2. **記憶體映射 I/O 在多核心系統中的挑戰有哪些？**
   - 快取一致性問題
   - 核心之間的競爭條件
   - 記憶體排序要求

3. **如何針對即時系統最佳化暫存器存取？**
   - 最小化存取延遲
   - 使用適當的記憶體屏障
   - 實作關鍵區段保護

## 📚 額外資源

### 標準和文件
- **ARM 架構參考**：記憶體映射 I/O 規格
- **C 標準**：volatile 關鍵字行為
- **硬體參考手冊**：暫存器規格

### 相關主題
- **[DMA 緩衝區管理](./DMA_Buffer_Management.md)** - DMA 與記憶體映射 I/O
- **[快取感知程式設計](./Cache_Aware_Programming.md)** - 快取考量
- **[中斷處理](./Interrupt_Handling.md)** - 中斷管理
- **[效能最佳化](./performance_optimization.md)** - 暫存器存取最佳化

### 工具和函式庫
- **暫存器存取函式庫**：硬體抽象層
- **除錯工具**：暫存器監控和分析
- **效能分析器**：暫存器存取時序分析

---

**下一個主題：** [共享記憶體程式設計](./Shared_Memory_Programming.md) → [即時系統](./Real_Time_Systems.md)
