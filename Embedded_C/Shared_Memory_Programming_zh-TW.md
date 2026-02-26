# 共享記憶體程式設計

## 📋 目錄
- [概述](#-概述)
- [共享記憶體基礎](#-共享記憶體基礎)
- [記憶體同步](#-記憶體同步)
- [無鎖程式設計](#-無鎖程式設計)
- [記憶體屏障](#-記憶體屏障)
- [快取一致性](#-快取一致性)
- [多核心通訊](#-多核心通訊)
- [即時性考量](#-即時性考量)
- [常見陷阱](#-常見陷阱)
- [最佳實踐](#-最佳實踐)
- [面試問題](#-面試問題)
- [額外資源](#-額外資源)

## 🎯 概述

共享記憶體程式設計使多個核心或處理程序能夠透過共用記憶體區域進行通訊。在嵌入式系統中，這對於高效的核間通訊、資料共享和平行處理至關重要，同時需要維護資料一致性並避免競爭條件。

## 🔧 共享記憶體基礎

### 共享記憶體結構
```c
// 基本共享記憶體結構
typedef struct {
    volatile uint32_t flag;
    uint32_t data[100];
    volatile uint32_t read_index;
    volatile uint32_t write_index;
    uint32_t buffer_size;
} __attribute__((aligned(64))) shared_buffer_t;

// 共享記憶體區域定義
typedef struct {
    shared_buffer_t* buffer;
    uint32_t core_id;
    bool is_initialized;
} shared_memory_context_t;

// 初始化共享記憶體
shared_memory_context_t* create_shared_memory_context(uint32_t core_id) {
    shared_memory_context_t* context = malloc(sizeof(shared_memory_context_t));
    if (!context) return NULL;
    
    // 分配共享記憶體（實際上，這會是一個特定的記憶體區域）
    context->buffer = (shared_buffer_t*)SHARED_MEMORY_BASE_ADDRESS;
    context->core_id = core_id;
    context->is_initialized = false;
    
    return context;
}

void initialize_shared_buffer(shared_buffer_t* buffer) {
    if (!buffer) return;
    
    buffer->flag = 0;
    buffer->read_index = 0;
    buffer->write_index = 0;
    buffer->buffer_size = 100;
    
    // 清除資料緩衝區
    memset(buffer->data, 0, sizeof(buffer->data));
}
```

### 記憶體區域映射
```c
// 共享存取的記憶體區域映射
typedef struct {
    void* virtual_address;
    uint32_t physical_address;
    size_t size;
    uint32_t permissions;
    bool is_shared;
} memory_region_t;

memory_region_t* map_shared_memory_region(uint32_t physical_addr, 
                                         size_t size, 
                                         uint32_t permissions) {
    memory_region_t* region = malloc(sizeof(memory_region_t));
    if (!region) return NULL;
    
    // 將實體記憶體映射到虛擬位址
    region->physical_address = physical_addr;
    region->virtual_address = map_physical_memory(physical_addr, size, permissions);
    region->size = size;
    region->permissions = permissions;
    region->is_shared = true;
    
    return region;
}

void* map_physical_memory(uint32_t physical_addr, size_t size, uint32_t permissions) {
    // 平台特定的記憶體映射
    // 這會使用 MMU 或記憶體映射函式
    return (void*)physical_addr;  // 簡化示範用
}

void unmap_shared_memory_region(memory_region_t* region) {
    if (region && region->virtual_address) {
        unmap_physical_memory(region->virtual_address, region->size);
        free(region);
    }
}
```

## 🔒 記憶體同步

### 基於互斥鎖的同步
```c
// 共享記憶體互斥鎖
typedef struct {
    volatile uint32_t lock;
    uint32_t owner_core;
    uint32_t lock_count;
} __attribute__((aligned(64))) shared_mutex_t;

// 初始化共享互斥鎖
void init_shared_mutex(shared_mutex_t* mutex) {
    mutex->lock = 0;
    mutex->owner_core = 0xFFFFFFFF;
    mutex->lock_count = 0;
}

// 取得共享互斥鎖
bool acquire_shared_mutex(shared_mutex_t* mutex, uint32_t core_id) {
    uint32_t expected = 0;
    uint32_t desired = 1;
    
    // 使用比較並交換嘗試取得鎖
    if (__sync_bool_compare_and_swap(&mutex->lock, expected, desired)) {
        mutex->owner_core = core_id;
        mutex->lock_count++;
        return true;
    }
    
    return false;
}

// 釋放共享互斥鎖
void release_shared_mutex(shared_mutex_t* mutex, uint32_t core_id) {
    if (mutex->owner_core == core_id) {
        mutex->lock_count--;
        if (mutex->lock_count == 0) {
            mutex->owner_core = 0xFFFFFFFF;
            mutex->lock = 0;
        }
    }
}

// 使用共享緩衝區的範例
bool write_to_shared_buffer(shared_buffer_t* buffer, 
                          shared_mutex_t* mutex, 
                          uint32_t data, 
                          uint32_t core_id) {
    if (!acquire_shared_mutex(mutex, core_id)) {
        return false;
    }
    
    // 將資料寫入共享緩衝區
    if (buffer->write_index < buffer->buffer_size) {
        buffer->data[buffer->write_index] = data;
        buffer->write_index++;
        release_shared_mutex(mutex, core_id);
        return true;
    }
    
    release_shared_mutex(mutex, core_id);
    return false;
}
```

### 基於號誌的同步
```c
// 共享號誌結構
typedef struct {
    volatile uint32_t count;
    volatile uint32_t max_count;
    uint32_t waiting_cores;
} __attribute__((aligned(64))) shared_semaphore_t;

// 初始化共享號誌
void init_shared_semaphore(shared_semaphore_t* sem, uint32_t initial_count, uint32_t max_count) {
    sem->count = initial_count;
    sem->max_count = max_count;
    sem->waiting_cores = 0;
}

// 等待共享號誌
bool wait_shared_semaphore(shared_semaphore_t* sem, uint32_t core_id) {
    uint32_t expected;
    uint32_t desired;
    
    do {
        expected = sem->count;
        if (expected == 0) {
            return false;  // 號誌不可用
        }
        desired = expected - 1;
    } while (!__sync_bool_compare_and_swap(&sem->count, expected, desired));
    
    return true;
}

// 發出共享號誌信號
bool signal_shared_semaphore(shared_semaphore_t* sem) {
    uint32_t expected;
    uint32_t desired;
    
    do {
        expected = sem->count;
        if (expected >= sem->max_count) {
            return false;  // 號誌已達最大值
        }
        desired = expected + 1;
    } while (!__sync_bool_compare_and_swap(&sem->count, expected, desired));
    
    return true;
}
```

## 🔄 無鎖程式設計

### 無鎖佇列實作
```c
// 無鎖佇列結構
typedef struct {
    volatile uint32_t head;
    volatile uint32_t tail;
    uint32_t size;
    uint32_t* data;
} __attribute__((aligned(64))) lock_free_queue_t;

lock_free_queue_t* create_lock_free_queue(uint32_t size) {
    lock_free_queue_t* queue = malloc(sizeof(lock_free_queue_t));
    if (!queue) return NULL;
    
    queue->data = aligned_alloc(64, size * sizeof(uint32_t));
    if (!queue->data) {
        free(queue);
        return NULL;
    }
    
    queue->head = 0;
    queue->tail = 0;
    queue->size = size;
    
    return queue;
}

// 入隊操作
bool lock_free_enqueue(lock_free_queue_t* queue, uint32_t value) {
    uint32_t tail = queue->tail;
    uint32_t next_tail = (tail + 1) % queue->size;
    
    if (next_tail == queue->head) {
        return false;  // 佇列已滿
    }
    
    queue->data[tail] = value;
    queue->tail = next_tail;
    return true;
}

// 出隊操作
bool lock_free_dequeue(lock_free_queue_t* queue, uint32_t* value) {
    uint32_t head = queue->head;
    
    if (head == queue->tail) {
        return false;  // 佇列為空
    }
    
    *value = queue->data[head];
    queue->head = (head + 1) % queue->size;
    return true;
}
```

### 無鎖堆疊實作
```c
// 無鎖堆疊結構
typedef struct node {
    uint32_t data;
    struct node* next;
} lock_free_stack_node_t;

typedef struct {
    volatile lock_free_stack_node_t* top;
} lock_free_stack_t;

lock_free_stack_t* create_lock_free_stack(void) {
    lock_free_stack_t* stack = malloc(sizeof(lock_free_stack_t));
    if (stack) {
        stack->top = NULL;
    }
    return stack;
}

// 推入操作
bool lock_free_push(lock_free_stack_t* stack, uint32_t value) {
    lock_free_stack_node_t* new_node = malloc(sizeof(lock_free_stack_node_t));
    if (!new_node) return false;
    
    new_node->data = value;
    
    lock_free_stack_node_t* old_top;
    do {
        old_top = stack->top;
        new_node->next = old_top;
    } while (!__sync_bool_compare_and_swap(&stack->top, old_top, new_node));
    
    return true;
}

// 彈出操作
bool lock_free_pop(lock_free_stack_t* stack, uint32_t* value) {
    lock_free_stack_node_t* old_top;
    lock_free_stack_node_t* new_top;
    
    do {
        old_top = stack->top;
        if (!old_top) {
            return false;  // 堆疊為空
        }
        new_top = old_top->next;
    } while (!__sync_bool_compare_and_swap(&stack->top, old_top, new_top));
    
    *value = old_top->data;
    free(old_top);
    return true;
}
```

## 🚧 記憶體屏障

### 記憶體屏障類型
```c
// 記憶體屏障函式
void full_memory_barrier(void) {
    __sync_synchronize();  // 完整記憶體屏障
}

void read_memory_barrier(void) {
    __asm volatile ("dmb ishld" : : : "memory");  // 讀取屏障
}

void write_memory_barrier(void) {
    __asm volatile ("dmb ishst" : : : "memory");  // 寫入屏障
}

// 編譯器屏障
void compiler_barrier(void) {
    __asm volatile ("" : : : "memory");
}

// 在共享記憶體中使用記憶體屏障
typedef struct {
    volatile uint32_t data;
    volatile uint32_t sequence;
} __attribute__((aligned(64))) barrier_protected_data_t;

void write_with_barrier(barrier_protected_data_t* shared_data, uint32_t new_data) {
    shared_data->data = new_data;
    write_memory_barrier();  // 確保寫入可見
    shared_data->sequence++;
    full_memory_barrier();   // 確保序號更新可見
}

uint32_t read_with_barrier(barrier_protected_data_t* shared_data) {
    uint32_t sequence = shared_data->sequence;
    read_memory_barrier();   // 確保讀取最新序號
    uint32_t data = shared_data->data;
    full_memory_barrier();   // 確保在序號之後讀取
    return data;
}
```

### 帶屏障的原子操作
```c
// 帶記憶體屏障的原子操作
uint32_t atomic_exchange_with_barrier(volatile uint32_t* ptr, uint32_t value) {
    uint32_t result = __sync_lock_test_and_set(ptr, value);
    full_memory_barrier();
    return result;
}

uint32_t atomic_compare_exchange_with_barrier(volatile uint32_t* ptr, 
                                             uint32_t expected, 
                                             uint32_t desired) {
    uint32_t result = __sync_val_compare_and_swap(ptr, expected, desired);
    full_memory_barrier();
    return result;
}

// 範例：帶屏障的原子計數器
typedef struct {
    volatile uint32_t counter;
    volatile uint32_t version;
} __attribute__((aligned(64))) atomic_counter_t;

void atomic_increment_with_barrier(atomic_counter_t* counter) {
    uint32_t old_value;
    uint32_t new_value;
    
    do {
        old_value = counter->counter;
        new_value = old_value + 1;
    } while (!atomic_compare_exchange_with_barrier(&counter->counter, 
                                                  old_value, new_value));
    
    counter->version++;
    write_memory_barrier();
}
```

## 🔄 快取一致性

### 快取行對齊
```c
// 快取行對齊的共享資料
#define CACHE_LINE_SIZE 64

typedef struct {
    uint32_t core1_data;
    char padding1[CACHE_LINE_SIZE - 4];
    uint32_t core2_data;
    char padding2[CACHE_LINE_SIZE - 4];
    uint32_t core3_data;
    char padding3[CACHE_LINE_SIZE - 4];
    uint32_t core4_data;
    char padding4[CACHE_LINE_SIZE - 4];
} __attribute__((aligned(CACHE_LINE_SIZE))) cache_aligned_shared_data_t;

// 防止偽共享
typedef struct {
    uint32_t frequently_accessed;
    char padding[CACHE_LINE_SIZE - 4];
} __attribute__((aligned(CACHE_LINE_SIZE))) isolated_shared_data_t;

// 快取感知的共享記憶體配置
void* allocate_cache_aligned_shared_memory(size_t size) {
    size_t aligned_size = ((size + CACHE_LINE_SIZE - 1) / CACHE_LINE_SIZE) * CACHE_LINE_SIZE;
    return aligned_alloc(CACHE_LINE_SIZE, aligned_size);
}
```

### 快取刷新與無效化
```c
// 共享記憶體的快取操作
void flush_cache_for_shared_write(void* address, size_t size) {
    __builtin___clear_cache((char*)address, (char*)address + size);
}

void invalidate_cache_for_shared_read(void* address, size_t size) {
    __builtin___clear_cache((char*)address, (char*)address + size);
}

// 帶快取管理的共享記憶體
typedef struct {
    void* data;
    size_t size;
    bool needs_flush;
    bool needs_invalidate;
} cache_managed_shared_memory_t;

cache_managed_shared_memory_t* create_cache_managed_shared_memory(size_t size) {
    cache_managed_shared_memory_t* shared_mem = malloc(sizeof(cache_managed_shared_memory_t));
    if (!shared_mem) return NULL;
    
    shared_mem->data = allocate_cache_aligned_shared_memory(size);
    shared_mem->size = size;
    shared_mem->needs_flush = true;
    shared_mem->needs_invalidate = true;
    
    return shared_mem;
}

void prepare_shared_memory_for_write(cache_managed_shared_memory_t* shared_mem) {
    if (shared_mem->needs_flush) {
        flush_cache_for_shared_write(shared_mem->data, shared_mem->size);
    }
}

void prepare_shared_memory_for_read(cache_managed_shared_memory_t* shared_mem) {
    if (shared_mem->needs_invalidate) {
        invalidate_cache_for_shared_read(shared_mem->data, shared_mem->size);
    }
}
```

## 🔄 多核心通訊

### 核間通訊
```c
// 核間通訊結構
typedef struct {
    volatile uint32_t message;
    volatile uint32_t sender_core;
    volatile uint32_t receiver_core;
    volatile uint32_t sequence;
} __attribute__((aligned(64))) inter_core_message_t;

typedef struct {
    inter_core_message_t messages[MAX_CORES];
    volatile uint32_t message_count;
} inter_core_communication_t;

// 發送訊息到另一個核心
bool send_message_to_core(inter_core_communication_t* comm, 
                         uint32_t target_core, 
                         uint32_t message, 
                         uint32_t sender_core) {
    if (target_core >= MAX_CORES) return false;
    
    inter_core_message_t* msg = &comm->messages[target_core];
    msg->message = message;
    msg->sender_core = sender_core;
    msg->receiver_core = target_core;
    msg->sequence++;
    
    // 在目標核心觸發中斷
    trigger_core_interrupt(target_core);
    
    return true;
}

// 從另一個核心接收訊息
bool receive_message_from_core(inter_core_communication_t* comm, 
                              uint32_t core_id, 
                              uint32_t* message, 
                              uint32_t* sender_core) {
    inter_core_message_t* msg = &comm->messages[core_id];
    
    if (msg->receiver_core == core_id && msg->message != 0) {
        *message = msg->message;
        *sender_core = msg->sender_core;
        msg->message = 0;  // 清除訊息
        return true;
    }
    
    return false;
}
```

### 共享記憶體環形緩衝區
```c
// 用於核間通訊的環形緩衝區
typedef struct {
    volatile uint32_t head;
    volatile uint32_t tail;
    volatile uint32_t size;
    volatile uint32_t* buffer;
} __attribute__((aligned(64))) shared_ring_buffer_t;

shared_ring_buffer_t* create_shared_ring_buffer(uint32_t size) {
    shared_ring_buffer_t* ring_buffer = malloc(sizeof(shared_ring_buffer_t));
    if (!ring_buffer) return NULL;
    
    ring_buffer->buffer = aligned_alloc(64, size * sizeof(uint32_t));
    if (!ring_buffer->buffer) {
        free(ring_buffer);
        return NULL;
    }
    
    ring_buffer->head = 0;
    ring_buffer->tail = 0;
    ring_buffer->size = size;
    
    return ring_buffer;
}

// 生產者函式（由一個核心呼叫）
bool ring_buffer_produce(shared_ring_buffer_t* ring_buffer, uint32_t data) {
    uint32_t next_head = (ring_buffer->head + 1) % ring_buffer->size;
    
    if (next_head == ring_buffer->tail) {
        return false;  // 緩衝區已滿
    }
    
    ring_buffer->buffer[ring_buffer->head] = data;
    ring_buffer->head = next_head;
    return true;
}

// 消費者函式（由另一個核心呼叫）
bool ring_buffer_consume(shared_ring_buffer_t* ring_buffer, uint32_t* data) {
    if (ring_buffer->head == ring_buffer->tail) {
        return false;  // 緩衝區為空
    }
    
    *data = ring_buffer->buffer[ring_buffer->tail];
    ring_buffer->tail = (ring_buffer->tail + 1) % ring_buffer->size;
    return true;
}
```

## ⏱️ 即時性考量

### 即時共享記憶體
```c
// 帶截止時間的即時共享記憶體
typedef struct {
    volatile uint32_t data;
    volatile uint32_t timestamp;
    volatile uint32_t deadline;
    volatile uint32_t core_id;
} __attribute__((aligned(64))) real_time_shared_data_t;

// 即時共享記憶體存取
bool real_time_shared_write(real_time_shared_data_t* shared_data, 
                           uint32_t data, 
                           uint32_t deadline, 
                           uint32_t core_id) {
    uint32_t current_time = get_system_tick_count();
    
    if (current_time > deadline) {
        return false;  // 錯過截止時間
    }
    
    shared_data->data = data;
    shared_data->timestamp = current_time;
    shared_data->deadline = deadline;
    shared_data->core_id = core_id;
    
    return true;
}

bool real_time_shared_read(real_time_shared_data_t* shared_data, 
                          uint32_t* data, 
                          uint32_t* timestamp) {
    uint32_t current_time = get_system_tick_count();
    
    if (current_time > shared_data->deadline) {
        return false;  // 資料已過期
    }
    
    *data = shared_data->data;
    *timestamp = shared_data->timestamp;
    
    return true;
}
```

### 基於優先權的共享記憶體
```c
// 基於優先權的共享記憶體存取
typedef enum {
    PRIORITY_LOW = 0,
    PRIORITY_MEDIUM = 1,
    PRIORITY_HIGH = 2,
    PRIORITY_CRITICAL = 3
} shared_memory_priority_t;

typedef struct {
    volatile uint32_t data;
    volatile shared_memory_priority_t priority;
    volatile uint32_t access_count;
} __attribute__((aligned(64))) priority_shared_data_t;

bool priority_shared_write(priority_shared_data_t* shared_data, 
                          uint32_t data, 
                          shared_memory_priority_t priority) {
    // 檢查是否可以搶佔當前存取
    if (priority > shared_data->priority) {
        shared_data->data = data;
        shared_data->priority = priority;
        shared_data->access_count++;
        return true;
    }
    
    return false;  // 較低優先權的存取被阻擋
}
```

## ⚠️ 常見陷阱

### 1. 競爭條件
```c
// 錯誤：共享記憶體存取中的競爭條件
void incorrect_shared_access(volatile uint32_t* shared_counter) {
    uint32_t value = *shared_counter;  // 讀取
    value++;                           // 修改
    *shared_counter = value;           // 寫入
    // 讀取和寫入之間存在競爭條件
}

// 正確：原子操作
void correct_shared_access(volatile uint32_t* shared_counter) {
    __sync_fetch_and_add(shared_counter, 1);  // 原子遞增
}
```

### 2. 偽共享
```c
// 錯誤：核心之間的偽共享
typedef struct {
    uint32_t core1_counter;  // 可能與 core2_counter 在同一快取行
    uint32_t core2_counter;
} incorrect_shared_counters_t;

// 正確：快取行分隔
typedef struct {
    uint32_t core1_counter;
    char padding1[60];  // 填充到下一快取行
    uint32_t core2_counter;
    char padding2[60];
} __attribute__((aligned(64))) correct_shared_counters_t;
```

### 3. 遺失的記憶體屏障
```c
// 錯誤：缺少記憶體屏障
void incorrect_shared_write(volatile uint32_t* data, volatile uint32_t* flag) {
    *data = 0x12345678;
    *flag = 1;  // 可能與資料寫入重新排序
}

// 正確：使用記憶體屏障
void correct_shared_write(volatile uint32_t* data, volatile uint32_t* flag) {
    *data = 0x12345678;
    write_memory_barrier();  // 確保資料寫入可見
    *flag = 1;
}
```

## ✅ 最佳實踐

### 1. 共享記憶體設計模式
```c
// 生產者-消費者模式搭配共享記憶體
typedef struct {
    volatile uint32_t* buffer;
    volatile uint32_t head;
    volatile uint32_t tail;
    volatile uint32_t size;
    shared_mutex_t* mutex;
} producer_consumer_shared_t;

producer_consumer_shared_t* create_producer_consumer_shared(uint32_t buffer_size) {
    producer_consumer_shared_t* pc = malloc(sizeof(producer_consumer_shared_t));
    if (!pc) return NULL;
    
    pc->buffer = aligned_alloc(64, buffer_size * sizeof(uint32_t));
    pc->head = 0;
    pc->tail = 0;
    pc->size = buffer_size;
    pc->mutex = malloc(sizeof(shared_mutex_t));
    
    if (pc->buffer && pc->mutex) {
        init_shared_mutex(pc->mutex);
        return pc;
    }
    
    // 失敗時清理
    free(pc->buffer);
    free(pc->mutex);
    free(pc);
    return NULL;
}

bool producer_put(producer_consumer_shared_t* pc, uint32_t data, uint32_t core_id) {
    if (!acquire_shared_mutex(pc->mutex, core_id)) {
        return false;
    }
    
    uint32_t next_head = (pc->head + 1) % pc->size;
    if (next_head == pc->tail) {
        release_shared_mutex(pc->mutex, core_id);
        return false;  // 緩衝區已滿
    }
    
    pc->buffer[pc->head] = data;
    pc->head = next_head;
    
    release_shared_mutex(pc->mutex, core_id);
    return true;
}

bool consumer_get(producer_consumer_shared_t* pc, uint32_t* data, uint32_t core_id) {
    if (!acquire_shared_mutex(pc->mutex, core_id)) {
        return false;
    }
    
    if (pc->head == pc->tail) {
        release_shared_mutex(pc->mutex, core_id);
        return false;  // 緩衝區為空
    }
    
    *data = pc->buffer[pc->tail];
    pc->tail = (pc->tail + 1) % pc->size;
    
    release_shared_mutex(pc->mutex, core_id);
    return true;
}
```

### 2. 共享記憶體驗證
```c
// 驗證共享記憶體存取
bool validate_shared_memory_access(void* address, size_t size) {
    // 檢查對齊
    if ((uintptr_t)address % 64 != 0) {
        return false;
    }
    
    // 檢查大小對齊
    if (size % 64 != 0) {
        return false;
    }
    
    // 檢查位址是否在共享記憶體區域內
    if ((uintptr_t)address < SHARED_MEMORY_BASE || 
        (uintptr_t)address >= SHARED_MEMORY_END) {
        return false;
    }
    
    return true;
}

// 安全的共享記憶體配置
void* safe_shared_memory_alloc(size_t size) {
    size_t aligned_size = ((size + 63) / 64) * 64;  // 對齊到 64 位元組
    
    void* ptr = aligned_alloc(64, aligned_size);
    if (!ptr) return NULL;
    
    if (!validate_shared_memory_access(ptr, aligned_size)) {
        free(ptr);
        return NULL;
    }
    
    return ptr;
}
```

### 3. 共享記憶體監控
```c
// 監控共享記憶體使用情況
typedef struct {
    uint32_t access_count;
    uint32_t conflict_count;
    uint32_t last_access_time;
    uint32_t last_access_core;
} shared_memory_monitor_t;

shared_memory_monitor_t* create_shared_memory_monitor(void) {
    shared_memory_monitor_t* monitor = malloc(sizeof(shared_memory_monitor_t));
    if (monitor) {
        monitor->access_count = 0;
        monitor->conflict_count = 0;
        monitor->last_access_time = 0;
        monitor->last_access_core = 0;
    }
    return monitor;
}

void update_shared_memory_monitor(shared_memory_monitor_t* monitor, 
                                 uint32_t core_id, 
                                 bool conflict) {
    monitor->access_count++;
    monitor->last_access_time = get_system_tick_count();
    monitor->last_access_core = core_id;
    
    if (conflict) {
        monitor->conflict_count++;
    }
}
```

## 🎯 面試問題

### 基礎問題
1. **什麼是共享記憶體程式設計，為什麼它很重要？**
   - 使多個核心/處理程序能夠透過共用記憶體進行通訊
   - 對高效的核間通訊和資料共享至關重要

2. **共享記憶體程式設計的主要挑戰是什麼？**
   - 競爭條件
   - 快取一致性問題
   - 記憶體排序要求
   - 偽共享

3. **如何防止共享記憶體中的競爭條件？**
   - 使用原子操作
   - 實作適當的同步機制（互斥鎖、號誌）
   - 使用記憶體屏障
   - 設計無鎖資料結構

### 進階問題
1. **如何實作無鎖資料結構？**
   - 使用原子比較並交換操作
   - 針對單寫入者或多讀取者場景設計
   - 實作適當的記憶體排序

2. **基於鎖和無鎖程式設計之間的權衡是什麼？**
   - 基於鎖：較簡單，但可能導致競爭
   - 無鎖：較佳效能，但更複雜

3. **如何針對即時系統優化共享記憶體？**
   - 最小化存取延遲
   - 使用適當的記憶體屏障
   - 實作截止時間感知的存取模式

## 📚 額外資源

### 標準與文件
- **C11 標準**：原子操作與記憶體排序
- **ARM 架構參考手冊**：記憶體排序與屏障
- **即時系統**：RTOS 中的共享記憶體

### 相關主題
- **[快取感知程式設計](./Cache_Aware_Programming.md)** - 快取考量
- **[多核心程式設計](./Multi_core_Programming.md)** - 多核心考量
- **[即時系統](./Real_Time_Systems.md)** - 即時共享記憶體
- **[效能優化](./performance_optimization.md)** - 共享記憶體優化

### 工具與函式庫
- **記憶體分析工具**：共享記憶體除錯
- **效能分析器**：共享記憶體效能分析
- **競爭條件偵測器**：ThreadSanitizer、Helgrind

---

**下一主題：** [即時系統](./Real_Time_Systems.md) → [作業系統](./Operating_Systems.md)
