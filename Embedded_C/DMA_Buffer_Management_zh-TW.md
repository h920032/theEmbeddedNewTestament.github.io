# DMA 緩衝區管理

## 📋 目錄
- [概述](#-概述)
- [DMA 緩衝區需求](#-dma-緩衝區需求)
- [快取一致性 DMA](#-快取一致性-dma)
- [DMA 緩衝區配置](#-dma-緩衝區配置)
- [DMA 描述符管理](#-dma-描述符管理)
- [DMA 傳輸類型](#-dma-傳輸類型)
- [DMA 緩衝區池化](#-dma-緩衝區池化)
- [即時 DMA 考量](#-即時-dma-考量)
- [DMA 錯誤處理](#-dma-錯誤處理)
- [常見陷阱](#-常見陷阱)
- [最佳實踐](#-最佳實踐)
- [面試問題](#-面試問題)
- [其他資源](#-其他資源)

## 🎯 概述

DMA（直接記憶體存取）緩衝區管理對於在周邊裝置和記憶體之間進行高效資料傳輸而無需 CPU 介入至關重要。適當的 DMA 緩衝區管理可確保快取一致性、最佳效能以及嵌入式系統中可靠的資料傳輸。

## 🔧 DMA 緩衝區需求

### 記憶體對齊需求
```c
// DMA 緩衝區對齊需求
#define DMA_BUFFER_ALIGNMENT 32  // 大多數 DMA 控制器需要 32 位元組對齊
#define DMA_BUFFER_SIZE_ALIGNMENT 4  // 4 位元組大小對齊

typedef struct {
    void* buffer;
    size_t size;
    uint32_t physical_address;
    bool is_cacheable;
} dma_buffer_t;

// 檢查 DMA 緩衝區對齊
bool is_dma_buffer_aligned(const void* buffer, size_t size) {
    uintptr_t addr = (uintptr_t)buffer;
    
    // 檢查地址對齊
    if (addr % DMA_BUFFER_ALIGNMENT != 0) {
        return false;
    }
    
    // 檢查大小對齊
    if (size % DMA_BUFFER_SIZE_ALIGNMENT != 0) {
        return false;
    }
    
    return true;
}

// 取得 DMA 緩衝區需求
typedef struct {
    uint32_t min_alignment;
    uint32_t max_transfer_size;
    bool requires_cache_flush;
    bool requires_cache_invalidate;
} dma_requirements_t;

dma_requirements_t get_dma_requirements(void) {
    dma_requirements_t req = {
        .min_alignment = 32,
        .max_transfer_size = 65536,
        .requires_cache_flush = true,
        .requires_cache_invalidate = true
    };
    return req;
}
```

### 實體記憶體映射
```c
// 為 DMA 映射虛擬地址到實體地址
typedef struct {
    void* virtual_address;
    uint32_t physical_address;
    size_t size;
    bool is_mapped;
} dma_memory_mapping_t;

dma_memory_mapping_t* create_dma_memory_mapping(size_t size) {
    dma_memory_mapping_t* mapping = malloc(sizeof(dma_memory_mapping_t));
    if (!mapping) return NULL;
    
    // 配置對齊的記憶體
    mapping->virtual_address = aligned_alloc(DMA_BUFFER_ALIGNMENT, size);
    if (!mapping->virtual_address) {
        free(mapping);
        return NULL;
    }
    
    // 取得實體地址（平台特定）
    mapping->physical_address = get_physical_address(mapping->virtual_address);
    mapping->size = size;
    mapping->is_mapped = true;
    
    return mapping;
}

uint32_t get_physical_address(void* virtual_addr) {
    // 平台特定實作
    // 這會使用 MMU 或記憶體映射函式
    return (uint32_t)virtual_addr;  // 簡化示範
}

void destroy_dma_memory_mapping(dma_memory_mapping_t* mapping) {
    if (mapping && mapping->virtual_address) {
        free(mapping->virtual_address);
        free(mapping);
    }
}
```

## 🔄 快取一致性 DMA

### 快取刷新與無效化
```c
// DMA 的快取操作
void flush_cache_for_dma(void* buffer, size_t size) {
    // 刷新快取以確保 DMA 看到最新資料
    __builtin___clear_cache((char*)buffer, (char*)buffer + size);
}

void invalidate_cache_after_dma(void* buffer, size_t size) {
    // 無效化快取以確保 CPU 看到 DMA 資料
    __builtin___clear_cache((char*)buffer, (char*)buffer + size);
}

// DMA 安全緩衝區操作
typedef struct {
    void* buffer;
    size_t size;
    bool needs_flush;
    bool needs_invalidate;
} dma_safe_buffer_t;

dma_safe_buffer_t* create_dma_safe_buffer(size_t size) {
    dma_safe_buffer_t* safe_buffer = malloc(sizeof(dma_safe_buffer_t));
    if (!safe_buffer) return NULL;
    
    safe_buffer->buffer = aligned_alloc(DMA_BUFFER_ALIGNMENT, size);
    safe_buffer->size = size;
    safe_buffer->needs_flush = true;
    safe_buffer->needs_invalidate = true;
    
    return safe_buffer;
}

void prepare_dma_buffer_for_write(dma_safe_buffer_t* safe_buffer) {
    if (safe_buffer->needs_flush) {
        flush_cache_for_dma(safe_buffer->buffer, safe_buffer->size);
    }
}

void prepare_dma_buffer_for_read(dma_safe_buffer_t* safe_buffer) {
    if (safe_buffer->needs_invalidate) {
        invalidate_cache_after_dma(safe_buffer->buffer, safe_buffer->size);
    }
}
```

### 不可快取的 DMA 緩衝區
```c
// 為 DMA 配置不可快取的記憶體
void* allocate_non_cacheable_dma_buffer(size_t size) {
    // 使用非快取記憶體區域
    void* buffer = allocate_uncached_memory(size);
    if (buffer) {
        // 確保適當對齊
        uintptr_t addr = (uintptr_t)buffer;
        if (addr % DMA_BUFFER_ALIGNMENT != 0) {
            // 如有必要重新對齊
            free(buffer);
            buffer = aligned_alloc(DMA_BUFFER_ALIGNMENT, size);
        }
    }
    return buffer;
}

// 平台特定的非快取記憶體配置
void* allocate_uncached_memory(size_t size) {
    // 這會使用平台特定的函式
    // 例如，在具有 MMU 的 ARM 上，將記憶體映射為裝置記憶體
    return malloc(size);  // 簡化示範
}
```

## 🗄️ DMA 緩衝區配置

### DMA 緩衝區池
```c
// 用於高效配置的 DMA 緩衝區池
typedef struct {
    void* pool_start;
    size_t pool_size;
    size_t buffer_size;
    size_t num_buffers;
    bool* buffer_used;
    void** buffer_addresses;
} dma_buffer_pool_t;

dma_buffer_pool_t* create_dma_buffer_pool(size_t buffer_size, size_t num_buffers) {
    dma_buffer_pool_t* pool = malloc(sizeof(dma_buffer_pool_t));
    if (!pool) return NULL;
    
    pool->buffer_size = buffer_size;
    pool->num_buffers = num_buffers;
    pool->pool_size = buffer_size * num_buffers;
    
    // 配置對齊的池記憶體
    pool->pool_start = aligned_alloc(DMA_BUFFER_ALIGNMENT, pool->pool_size);
    if (!pool->pool_start) {
        free(pool);
        return NULL;
    }
    
    // 初始化緩衝區追蹤
    pool->buffer_used = calloc(num_buffers, sizeof(bool));
    pool->buffer_addresses = malloc(num_buffers * sizeof(void*));
    
    if (!pool->buffer_used || !pool->buffer_addresses) {
        free(pool->pool_start);
        free(pool->buffer_used);
        free(pool->buffer_addresses);
        free(pool);
        return NULL;
    }
    
    // 初始化緩衝區地址
    for (size_t i = 0; i < num_buffers; i++) {
        pool->buffer_addresses[i] = (char*)pool->pool_start + (i * buffer_size);
    }
    
    return pool;
}

void* allocate_dma_buffer_from_pool(dma_buffer_pool_t* pool) {
    for (size_t i = 0; i < pool->num_buffers; i++) {
        if (!pool->buffer_used[i]) {
            pool->buffer_used[i] = true;
            return pool->buffer_addresses[i];
        }
    }
    return NULL;  // 無可用緩衝區
}

void free_dma_buffer_to_pool(dma_buffer_pool_t* pool, void* buffer) {
    for (size_t i = 0; i < pool->num_buffers; i++) {
        if (pool->buffer_addresses[i] == buffer) {
            pool->buffer_used[i] = false;
            break;
        }
    }
}
```

### 分散-聚集 DMA
```c
// 分散-聚集 DMA 描述符
typedef struct {
    uint32_t source_address;
    uint32_t destination_address;
    uint32_t transfer_size;
    uint32_t next_descriptor;
} dma_sg_descriptor_t;

typedef struct {
    dma_sg_descriptor_t* descriptors;
    size_t num_descriptors;
    size_t max_descriptors;
} dma_scatter_gather_t;

dma_scatter_gather_t* create_scatter_gather_dma(size_t max_descriptors) {
    dma_scatter_gather_t* sg = malloc(sizeof(dma_scatter_gather_t));
    if (!sg) return NULL;
    
    sg->descriptors = aligned_alloc(DMA_BUFFER_ALIGNMENT, 
                                   max_descriptors * sizeof(dma_sg_descriptor_t));
    sg->num_descriptors = 0;
    sg->max_descriptors = max_descriptors;
    
    return sg;
}

void add_sg_descriptor(dma_scatter_gather_t* sg, 
                      uint32_t src_addr, uint32_t dst_addr, uint32_t size) {
    if (sg->num_descriptors < sg->max_descriptors) {
        dma_sg_descriptor_t* desc = &sg->descriptors[sg->num_descriptors];
        desc->source_address = src_addr;
        desc->destination_address = dst_addr;
        desc->transfer_size = size;
        desc->next_descriptor = (sg->num_descriptors + 1 < sg->max_descriptors) ?
                               (uint32_t)&sg->descriptors[sg->num_descriptors + 1] : 0;
        sg->num_descriptors++;
    }
}
```

## 📋 DMA 描述符管理

### DMA 描述符結構
```c
// 不同傳輸類型的 DMA 描述符
typedef struct {
    uint32_t source_address;
    uint32_t destination_address;
    uint32_t transfer_size;
    uint32_t control;
    uint32_t next_descriptor;
} dma_descriptor_t;

#define DMA_CONTROL_ENABLE         (1 << 0)
#define DMA_CONTROL_INC_SRC        (1 << 1)
#define DMA_CONTROL_INC_DST        (1 << 2)
#define DMA_CONTROL_MEM_TO_MEM     (1 << 3)
#define DMA_CONTROL_MEM_TO_PERIPH  (1 << 4)
#define DMA_CONTROL_PERIPH_TO_MEM  (1 << 5)
#define DMA_CONTROL_INTERRUPT      (1 << 6)

// 建立 DMA 描述符
dma_descriptor_t* create_dma_descriptor(uint32_t src, uint32_t dst, 
                                       uint32_t size, uint32_t control) {
    dma_descriptor_t* desc = aligned_alloc(DMA_BUFFER_ALIGNMENT, 
                                          sizeof(dma_descriptor_t));
    if (!desc) return NULL;
    
    desc->source_address = src;
    desc->destination_address = dst;
    desc->transfer_size = size;
    desc->control = control;
    desc->next_descriptor = 0;
    
    return desc;
}

// 連結 DMA 描述符
void link_dma_descriptors(dma_descriptor_t* first, dma_descriptor_t* second) {
    if (first && second) {
        first->next_descriptor = (uint32_t)second;
    }
}
```

### DMA 通道管理
```c
// DMA 通道配置
typedef struct {
    uint8_t channel_id;
    dma_descriptor_t* descriptor_chain;
    bool is_active;
    uint32_t transfer_count;
    void (*completion_callback)(void*);
    void* callback_data;
} dma_channel_t;

typedef struct {
    dma_channel_t channels[MAX_DMA_CHANNELS];
    uint8_t active_channels;
} dma_controller_t;

dma_controller_t* initialize_dma_controller(void) {
    dma_controller_t* controller = malloc(sizeof(dma_controller_t));
    if (!controller) return NULL;
    
    // 初始化所有通道
    for (int i = 0; i < MAX_DMA_CHANNELS; i++) {
        controller->channels[i].channel_id = i;
        controller->channels[i].descriptor_chain = NULL;
        controller->channels[i].is_active = false;
        controller->channels[i].transfer_count = 0;
        controller->channels[i].completion_callback = NULL;
        controller->channels[i].callback_data = NULL;
    }
    
    controller->active_channels = 0;
    return controller;
}

bool start_dma_transfer(dma_controller_t* controller, uint8_t channel_id) {
    if (channel_id >= MAX_DMA_CHANNELS) return false;
    
    dma_channel_t* channel = &controller->channels[channel_id];
    if (!channel->descriptor_chain) return false;
    
    // 配置 DMA 硬體
    configure_dma_channel(channel_id, channel->descriptor_chain);
    
    // 開始傳輸
    start_dma_channel(channel_id);
    channel->is_active = true;
    controller->active_channels++;
    
    return true;
}
```

## 🔄 DMA 傳輸類型

### 記憶體到記憶體傳輸
```c
// 記憶體到記憶體 DMA 傳輸
typedef struct {
    void* source_buffer;
    void* destination_buffer;
    size_t transfer_size;
    dma_safe_buffer_t* safe_source;
    dma_safe_buffer_t* safe_destination;
} dma_mem_to_mem_transfer_t;

dma_mem_to_mem_transfer_t* create_mem_to_mem_transfer(void* src, void* dst, 
                                                       size_t size) {
    dma_mem_to_mem_transfer_t* transfer = malloc(sizeof(dma_mem_to_mem_transfer_t));
    if (!transfer) return NULL;
    
    transfer->source_buffer = src;
    transfer->destination_buffer = dst;
    transfer->transfer_size = size;
    
    // 如有需要建立安全緩衝區
    transfer->safe_source = create_dma_safe_buffer(size);
    transfer->safe_destination = create_dma_safe_buffer(size);
    
    return transfer;
}

bool execute_mem_to_mem_transfer(dma_mem_to_mem_transfer_t* transfer) {
    // 準備來源緩衝區
    prepare_dma_buffer_for_read(transfer->safe_source);
    
    // 準備目標緩衝區
    prepare_dma_buffer_for_write(transfer->safe_destination);
    
    // 建立 DMA 描述符
    dma_descriptor_t* desc = create_dma_descriptor(
        (uint32_t)transfer->source_buffer,
        (uint32_t)transfer->destination_buffer,
        transfer->transfer_size,
        DMA_CONTROL_ENABLE | DMA_CONTROL_INC_SRC | DMA_CONTROL_INC_DST | 
        DMA_CONTROL_MEM_TO_MEM | DMA_CONTROL_INTERRUPT
    );
    
    // 開始 DMA 傳輸
    return start_dma_transfer_with_descriptor(desc);
}
```

### 周邊到記憶體傳輸
```c
// 周邊到記憶體 DMA 傳輸（例如 ADC 取樣）
typedef struct {
    uint32_t peripheral_address;
    void* memory_buffer;
    size_t buffer_size;
    uint32_t transfer_count;
    dma_safe_buffer_t* safe_buffer;
} dma_periph_to_mem_transfer_t;

dma_periph_to_mem_transfer_t* create_adc_dma_transfer(uint32_t adc_data_reg, 
                                                      void* buffer, size_t size) {
    dma_periph_to_mem_transfer_t* transfer = malloc(sizeof(dma_periph_to_mem_transfer_t));
    if (!transfer) return NULL;
    
    transfer->peripheral_address = adc_data_reg;
    transfer->memory_buffer = buffer;
    transfer->buffer_size = size;
    transfer->transfer_count = 0;
    transfer->safe_buffer = create_dma_safe_buffer(size);
    
    return transfer;
}

bool start_adc_dma_transfer(dma_periph_to_mem_transfer_t* transfer) {
    // 準備記憶體緩衝區
    prepare_dma_buffer_for_write(transfer->safe_buffer);
    
    // 建立 DMA 描述符
    dma_descriptor_t* desc = create_dma_descriptor(
        transfer->peripheral_address,
        (uint32_t)transfer->memory_buffer,
        transfer->buffer_size,
        DMA_CONTROL_ENABLE | DMA_CONTROL_INC_DST | 
        DMA_CONTROL_PERIPH_TO_MEM | DMA_CONTROL_INTERRUPT
    );
    
    // 配置 ADC 用於 DMA
    configure_adc_for_dma();
    
    // 開始 DMA 傳輸
    return start_dma_transfer_with_descriptor(desc);
}
```

## 🗂️ DMA 緩衝區池化

### 多大小緩衝區池
```c
// 具有多種大小的緩衝區池
typedef struct {
    size_t buffer_size;
    size_t num_buffers;
    void** buffers;
    bool* buffer_used;
} dma_buffer_pool_size_t;

typedef struct {
    dma_buffer_pool_size_t* pools;
    size_t num_pool_sizes;
} dma_multi_size_pool_t;

dma_multi_size_pool_t* create_multi_size_dma_pool(size_t* sizes, 
                                                  size_t* counts, 
                                                  size_t num_sizes) {
    dma_multi_size_pool_t* multi_pool = malloc(sizeof(dma_multi_size_pool_t));
    if (!multi_pool) return NULL;
    
    multi_pool->pools = malloc(num_sizes * sizeof(dma_buffer_pool_size_t));
    multi_pool->num_pool_sizes = num_sizes;
    
    for (size_t i = 0; i < num_sizes; i++) {
        dma_buffer_pool_size_t* pool = &multi_pool->pools[i];
        pool->buffer_size = sizes[i];
        pool->num_buffers = counts[i];
        
        // 配置緩衝區
        pool->buffers = malloc(counts[i] * sizeof(void*));
        pool->buffer_used = calloc(counts[i], sizeof(bool));
        
        for (size_t j = 0; j < counts[i]; j++) {
            pool->buffers[j] = aligned_alloc(DMA_BUFFER_ALIGNMENT, sizes[i]);
        }
    }
    
    return multi_pool;
}

void* allocate_dma_buffer_from_multi_pool(dma_multi_size_pool_t* multi_pool, 
                                         size_t required_size) {
    // 找到適當的池大小
    for (size_t i = 0; i < multi_pool->num_pool_sizes; i++) {
        dma_buffer_pool_size_t* pool = &multi_pool->pools[i];
        if (pool->buffer_size >= required_size) {
            // 在此池中找到可用緩衝區
            for (size_t j = 0; j < pool->num_buffers; j++) {
                if (!pool->buffer_used[j]) {
                    pool->buffer_used[j] = true;
                    return pool->buffers[j];
                }
            }
        }
    }
    return NULL;  // 未找到合適的緩衝區
}
```

## ⏱️ 即時 DMA 考量

### DMA 優先權管理
```c
// DMA 優先權等級
typedef enum {
    DMA_PRIORITY_LOW = 0,
    DMA_PRIORITY_MEDIUM = 1,
    DMA_PRIORITY_HIGH = 2,
    DMA_PRIORITY_CRITICAL = 3
} dma_priority_t;

typedef struct {
    uint8_t channel_id;
    dma_priority_t priority;
    uint32_t deadline_us;
    bool is_real_time;
} dma_real_time_config_t;

dma_real_time_config_t* create_real_time_dma_config(uint8_t channel, 
                                                   dma_priority_t priority,
                                                   uint32_t deadline_us) {
    dma_real_time_config_t* config = malloc(sizeof(dma_real_time_config_t));
    if (!config) return NULL;
    
    config->channel_id = channel;
    config->priority = priority;
    config->deadline_us = deadline_us;
    config->is_real_time = true;
    
    return config;
}

bool configure_real_time_dma(dma_real_time_config_t* config) {
    // 設定 DMA 通道優先權
    set_dma_channel_priority(config->channel_id, config->priority);
    
    // 配置即時操作
    if (config->is_real_time) {
        enable_dma_interrupts(config->channel_id);
        set_dma_deadline(config->channel_id, config->deadline_us);
    }
    
    return true;
}
```

### DMA 截止時間監控
```c
// 監控 DMA 傳輸截止時間
typedef struct {
    uint32_t start_time;
    uint32_t deadline;
    bool deadline_missed;
    void (*deadline_callback)(void*);
    void* callback_data;
} dma_deadline_monitor_t;

dma_deadline_monitor_t* create_dma_deadline_monitor(uint32_t deadline_us,
                                                   void (*callback)(void*),
                                                   void* data) {
    dma_deadline_monitor_t* monitor = malloc(sizeof(dma_deadline_monitor_t));
    if (!monitor) return NULL;
    
    monitor->start_time = get_current_time_us();
    monitor->deadline = deadline_us;
    monitor->deadline_missed = false;
    monitor->deadline_callback = callback;
    monitor->callback_data = data;
    
    return monitor;
}

void check_dma_deadline(dma_deadline_monitor_t* monitor) {
    uint32_t current_time = get_current_time_us();
    uint32_t elapsed = current_time - monitor->start_time;
    
    if (elapsed > monitor->deadline && !monitor->deadline_missed) {
        monitor->deadline_missed = true;
        if (monitor->deadline_callback) {
            monitor->deadline_callback(monitor->callback_data);
        }
    }
}
```

## 🚨 DMA 錯誤處理

### DMA 錯誤偵測
```c
// DMA 錯誤類型
typedef enum {
    DMA_ERROR_NONE = 0,
    DMA_ERROR_TIMEOUT,
    DMA_ERROR_BUS_ERROR,
    DMA_ERROR_ALIGNMENT,
    DMA_ERROR_MEMORY_ERROR
} dma_error_t;

typedef struct {
    dma_error_t error_type;
    uint32_t error_address;
    uint32_t transfer_count;
    uint8_t channel_id;
} dma_error_info_t;

dma_error_info_t* create_dma_error_info(void) {
    dma_error_info_t* error_info = malloc(sizeof(dma_error_info_t));
    if (error_info) {
        error_info->error_type = DMA_ERROR_NONE;
        error_info->error_address = 0;
        error_info->transfer_count = 0;
        error_info->channel_id = 0;
    }
    return error_info;
}

dma_error_t check_dma_errors(uint8_t channel_id) {
    // 檢查 DMA 錯誤暫存器
    uint32_t error_status = read_dma_error_status(channel_id);
    
    if (error_status & DMA_ERROR_TIMEOUT_MASK) {
        return DMA_ERROR_TIMEOUT;
    } else if (error_status & DMA_ERROR_BUS_ERROR_MASK) {
        return DMA_ERROR_BUS_ERROR;
    } else if (error_status & DMA_ERROR_ALIGNMENT_MASK) {
        return DMA_ERROR_ALIGNMENT;
    }
    
    return DMA_ERROR_NONE;
}

void handle_dma_error(dma_error_info_t* error_info) {
    printf("DMA Error on channel %d: ", error_info->channel_id);
    
    switch (error_info->error_type) {
        case DMA_ERROR_TIMEOUT:
            printf("Timeout error\n");
            break;
        case DMA_ERROR_BUS_ERROR:
            printf("Bus error at address 0x%08X\n", error_info->error_address);
            break;
        case DMA_ERROR_ALIGNMENT:
            printf("Alignment error\n");
            break;
        case DMA_ERROR_MEMORY_ERROR:
            printf("Memory error\n");
            break;
        default:
            printf("Unknown error\n");
            break;
    }
    
    // 實作錯誤恢復
    reset_dma_channel(error_info->channel_id);
}
```

## ⚠️ 常見陷阱

### 1. 快取一致性問題
```c
// 錯誤：未處理快取一致性
void incorrect_dma_transfer(void* buffer, size_t size) {
    // 寫入緩衝區
    memset(buffer, 0xAA, size);
    
    // 未刷新快取就開始 DMA
    start_dma_transfer(buffer, size);  // DMA 可能看不到最新資料
}

// 正確：適當的快取處理
void correct_dma_transfer(void* buffer, size_t size) {
    // 寫入緩衝區
    memset(buffer, 0xAA, size);
    
    // 在 DMA 前刷新快取
    flush_cache_for_dma(buffer, size);
    
    // 開始 DMA 傳輸
    start_dma_transfer(buffer, size);
}
```

### 2. 緩衝區對齊問題
```c
// 錯誤：未對齊的 DMA 緩衝區
void* incorrect_dma_allocation(size_t size) {
    return malloc(size);  // 可能未對齊
}

// 正確：對齊的 DMA 緩衝區
void* correct_dma_allocation(size_t size) {
    return aligned_alloc(DMA_BUFFER_ALIGNMENT, size);
}
```

### 3. 缺少錯誤處理
```c
// 錯誤：無錯誤處理
void unsafe_dma_transfer(void* buffer, size_t size) {
    start_dma_transfer(buffer, size);
    // 無錯誤檢查
}

// 正確：適當的錯誤處理
bool safe_dma_transfer(void* buffer, size_t size) {
    if (!is_dma_buffer_aligned(buffer, size)) {
        return false;
    }
    
    if (!start_dma_transfer(buffer, size)) {
        return false;
    }
    
    // 帶逾時等待完成
    return wait_for_dma_completion(DMA_TIMEOUT_MS);
}
```

## ✅ 最佳實踐

### 1. DMA 緩衝區生命週期管理
```c
// 完整的 DMA 緩衝區生命週期
typedef struct {
    void* buffer;
    size_t size;
    bool is_allocated;
    bool is_in_use;
    uint32_t allocation_time;
} dma_buffer_lifecycle_t;

dma_buffer_lifecycle_t* create_dma_buffer_lifecycle(size_t size) {
    dma_buffer_lifecycle_t* lifecycle = malloc(sizeof(dma_buffer_lifecycle_t));
    if (!lifecycle) return NULL;
    
    lifecycle->buffer = aligned_alloc(DMA_BUFFER_ALIGNMENT, size);
    lifecycle->size = size;
    lifecycle->is_allocated = (lifecycle->buffer != NULL);
    lifecycle->is_in_use = false;
    lifecycle->allocation_time = get_current_time_ms();
    
    return lifecycle;
}

void destroy_dma_buffer_lifecycle(dma_buffer_lifecycle_t* lifecycle) {
    if (lifecycle) {
        if (lifecycle->buffer) {
            free(lifecycle->buffer);
        }
        free(lifecycle);
    }
}
```

### 2. DMA 傳輸驗證
```c
// 驗證 DMA 傳輸參數
bool validate_dma_transfer(void* buffer, size_t size, uint32_t peripheral_addr) {
    // 檢查緩衝區對齊
    if (!is_dma_buffer_aligned(buffer, size)) {
        return false;
    }
    
    // 檢查緩衝區大小
    if (size == 0 || size > MAX_DMA_TRANSFER_SIZE) {
        return false;
    }
    
    // 檢查周邊地址
    if (!is_valid_peripheral_address(peripheral_addr)) {
        return false;
    }
    
    return true;
}

bool is_valid_peripheral_address(uint32_t addr) {
    // 檢查地址是否在周邊記憶體範圍內
    return (addr >= PERIPHERAL_BASE && addr < PERIPHERAL_END);
}
```

### 3. DMA 效能監控
```c
// 監控 DMA 效能
typedef struct {
    uint32_t total_transfers;
    uint32_t successful_transfers;
    uint32_t failed_transfers;
    uint32_t total_bytes_transferred;
    uint32_t average_transfer_time_us;
} dma_performance_stats_t;

dma_performance_stats_t* create_dma_performance_stats(void) {
    dma_performance_stats_t* stats = malloc(sizeof(dma_performance_stats_t));
    if (stats) {
        memset(stats, 0, sizeof(dma_performance_stats_t));
    }
    return stats;
}

void update_dma_performance_stats(dma_performance_stats_t* stats, 
                                 bool success, uint32_t bytes, 
                                 uint32_t transfer_time_us) {
    stats->total_transfers++;
    stats->total_bytes_transferred += bytes;
    
    if (success) {
        stats->successful_transfers++;
    } else {
        stats->failed_transfers++;
    }
    
    // 更新平均傳輸時間
    stats->average_transfer_time_us = 
        (stats->average_transfer_time_us * (stats->total_transfers - 1) + 
         transfer_time_us) / stats->total_transfers;
}
```

## 🎯 面試問題

### 基礎問題
1. **什麼是 DMA，為什麼它在嵌入式系統中很重要？**
   - DMA：用於高效資料傳輸的直接記憶體存取
   - 重要性：減少 CPU 負載，改善效能

2. **DMA 緩衝區的關鍵需求是什麼？**
   - 適當的對齊
   - 快取一致性
   - 實體記憶體映射
   - 大小對齊

3. **你如何處理 DMA 的快取一致性？**
   - 在 DMA 寫入前刷新快取
   - 在 DMA 讀取後無效化快取
   - 使用不可快取的記憶體

### 進階問題
1. **你如何為即時系統實作 DMA 緩衝區池？**
   - 預先配置對齊的緩衝區
   - 實作基於優先權的配置
   - 加入截止時間監控

2. **分散-聚集 DMA 有哪些挑戰？**
   - 描述符鏈管理
   - 記憶體對齊需求
   - 跨多個傳輸的錯誤處理

3. **你如何最佳化 DMA 效能？**
   - 使用適當的緩衝區大小
   - 最小化快取操作
   - 實作高效的緩衝區池化

## 📚 其他資源

### 標準與文件
- **ARM DMA 文件**：DMA 控制器規範
- **快取一致性標準**：記憶體一致性需求
- **即時 DMA**：即時系統考量

### 相關主題
- **[快取感知程式設計](./Cache_Aware_Programming.md)** - 快取最佳化
- **[記憶體池配置](./Memory_Pool_Allocation.md)** - 高效記憶體管理
- **[效能最佳化](./performance_optimization.md)** - 一般最佳化
- **[即時系統](./Real_Time_Systems.md)** - 即時考量

### 工具與函式庫
- **DMA 除錯工具**：傳輸監控
- **快取分析工具**：一致性驗證
- **效能分析器**：DMA 效能分析

---

**下一個主題：** [記憶體映射 I/O](./Memory_Mapped_IO.md) → [共享記憶體程式設計](./Shared_Memory_Programming.md)
