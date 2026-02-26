# 記憶體保護

## 📋 目錄
- [概述](#-概述)
- [記憶體保護單元 (MPU)](#-記憶體保護單元-mpu)
- [記憶體管理單元 (MMU)](#-記憶體管理單元-mmu)
- [ARM TrustZone](#-arm-trustzone)
- [記憶體存取控制](#-記憶體存取控制)
- [堆疊保護](#-堆疊保護)
- [堆積保護](#-堆積保護)
- [程式碼保護](#-程式碼保護)
- [即時保護](#-即時保護)
- [常見陷阱](#-常見陷阱)
- [最佳實踐](#-最佳實踐)
- [面試問題](#-面試問題)
- [額外資源](#-額外資源)

## 🎯 概述

記憶體保護機制可防止對記憶體區域的未授權存取，確保系統安全性、穩定性和可靠性。在嵌入式系統中，記憶體保護對於隔離任務、防止緩衝區溢位和維護系統完整性至關重要。

## 🛡️ 記憶體保護單元 (MPU)

### MPU 組態設定
```c
// MPU 區域組態
typedef struct {
    uint32_t base_address;
    uint32_t size;
    uint32_t access_permissions;
    uint32_t attributes;
} mpu_region_t;

#define MPU_REGION_PRIVILEGED_RW    0x00000006
#define MPU_REGION_USER_RW          0x00000003
#define MPU_REGION_PRIVILEGED_RO    0x00000002
#define MPU_REGION_USER_RO          0x00000001
#define MPU_REGION_NO_ACCESS        0x00000000

// 設定 MPU 區域
void configure_mpu_region(uint8_t region, mpu_region_t* config) {
    // 停用 MPU 區域
    MPU->RNR = region;
    MPU->CTRL &= ~MPU_CTRL_ENABLE_Msk;
    
    // 設定區域
    MPU->RBAR = config->base_address | region;
    MPU->RASR = config->size | config->access_permissions | config->attributes;
    
    // 啟用 MPU
    MPU->CTRL |= MPU_CTRL_ENABLE_Msk;
    __DSB();
    __ISB();
}
```

### 記憶體區域保護
```c
// 保護不同的記憶體區域
void setup_memory_protection(void) {
    mpu_region_t regions[] = {
        // Flash 記憶體 - 使用者唯讀
        {
            .base_address = 0x08000000,
            .size = MPU_REGION_SIZE_1MB,
            .access_permissions = MPU_REGION_USER_RO,
            .attributes = MPU_REGION_CACHEABLE
        },
        // SRAM - 使用者可讀寫
        {
            .base_address = 0x20000000,
            .size = MPU_REGION_SIZE_128KB,
            .access_permissions = MPU_REGION_USER_RW,
            .attributes = MPU_REGION_CACHEABLE
        },
        // 周邊裝置 - 僅特權模式
        {
            .base_address = 0x40000000,
            .size = MPU_REGION_SIZE_1MB,
            .access_permissions = MPU_REGION_PRIVILEGED_RW,
            .attributes = MPU_REGION_DEVICE
        }
    };
    
    // 設定所有區域
    for (int i = 0; i < sizeof(regions)/sizeof(regions[0]); i++) {
        configure_mpu_region(i, &regions[i]);
    }
}
```

### 任務隔離
```c
// 使用 MPU 隔離任務
typedef struct {
    uint32_t stack_start;
    uint32_t stack_size;
    uint32_t data_start;
    uint32_t data_size;
    uint8_t priority;
} task_memory_config_t;

void isolate_task_memory(uint8_t task_id, task_memory_config_t* config) {
    mpu_region_t task_region = {
        .base_address = config->stack_start,
        .size = config->stack_size,
        .access_permissions = MPU_REGION_USER_RW,
        .attributes = MPU_REGION_CACHEABLE
    };
    
    // 為任務設定 MPU 區域
    configure_mpu_region(task_id, &task_region);
}

// 切換任務上下文並帶記憶體保護
void switch_task_context(uint8_t task_id) {
    // 儲存當前任務上下文
    save_current_context();
    
    // 為新任務設定 MPU
    task_memory_config_t* new_task_config = get_task_config(task_id);
    isolate_task_memory(task_id, new_task_config);
    
    // 恢復新任務上下文
    restore_task_context(task_id);
}
```

## 🏗️ 記憶體管理單元 (MMU)

### MMU 組態設定
```c
// MMU 頁表項目
typedef struct {
    uint32_t physical_address;
    uint32_t virtual_address;
    uint32_t page_size;
    uint32_t permissions;
    uint32_t attributes;
} mmu_page_entry_t;

#define MMU_PAGE_4KB     0x00000000
#define MMU_PAGE_64KB    0x00000001
#define MMU_PAGE_1MB     0x00000002
#define MMU_PAGE_16MB    0x00000003

#define MMU_PERM_NONE    0x00000000
#define MMU_PERM_RO      0x00000001
#define MMU_PERM_RW      0x00000003

// 設定 MMU 分頁
void configure_mmu_page(mmu_page_entry_t* entry) {
    // 計算頁表項目
    uint32_t pte = (entry->physical_address & 0xFFFFF000) |
                   (entry->page_size << 2) |
                   (entry->permissions << 1) |
                   0x00000001;  // 有效位元
    
    // 寫入頁表
    uint32_t* page_table = get_page_table_base();
    uint32_t page_index = entry->virtual_address >> 12;
    page_table[page_index] = pte;
    
    // 無效化 TLB
    __asm volatile ("mcr p15, 0, %0, c8, c7, 0" : : "r" (0));
}
```

### 虛擬記憶體映射
```c
// 將虛擬記憶體映射到實體記憶體
void map_virtual_memory(uint32_t virtual_addr, uint32_t physical_addr, 
                       uint32_t size, uint32_t permissions) {
    mmu_page_entry_t entry = {
        .physical_address = physical_addr,
        .virtual_address = virtual_addr,
        .page_size = MMU_PAGE_4KB,
        .permissions = permissions,
        .attributes = 0
    };
    
    // 映射每一頁
    for (uint32_t offset = 0; offset < size; offset += 4096) {
        entry.virtual_address = virtual_addr + offset;
        entry.physical_address = physical_addr + offset;
        configure_mmu_page(&entry);
    }
}

// 設定虛擬記憶體配置
void setup_virtual_memory(void) {
    // 映射核心空間
    map_virtual_memory(0xC0000000, 0x08000000, 0x1000000, MMU_PERM_RO);
    
    // 映射使用者空間
    map_virtual_memory(0x00000000, 0x20000000, 0x100000, MMU_PERM_RW);
    
    // 映射裝置空間
    map_virtual_memory(0x40000000, 0x40000000, 0x1000000, MMU_PERM_RW);
}
```

## 🔒 ARM TrustZone

### TrustZone 組態設定
```c
// TrustZone 安全狀態
typedef enum {
    SECURE_STATE = 0,
    NON_SECURE_STATE = 1
} trustzone_state_t;

// 設定 TrustZone
void configure_trustzone(void) {
    // 設定安全和非安全區域
    SAU->CTRL = SAU_CTRL_ENABLE_Msk;
    
    // 區域 0：安全 Flash
    SAU->RNR = 0;
    SAU->RBAR = 0x08000000;  // Flash 基址
    SAU->RLAR = 0x080FFFFF | SAU_RLAR_ENABLE_Msk;
    
    // 區域 1：非安全 RAM
    SAU->RNR = 1;
    SAU->RBAR = 0x20000000;  // RAM 基址
    SAU->RLAR = 0x2001FFFF | SAU_RLAR_ENABLE_Msk;
    
    // 啟用 TrustZone
    SCB->AIRCR = SCB_AIRCR_VECTKEY_Msk | SCB_AIRCR_PRIVDEFENA_Msk;
}

// 切換到安全狀態
void enter_secure_state(void) {
    __asm volatile (
        "smc #0\n"
        : : : "r0", "r1", "r2", "r3", "r12", "lr"
    );
}

// 切換到非安全狀態
void enter_non_secure_state(void) {
    __asm volatile (
        "bxns lr\n"
    );
}
```

### 安全服務介面
```c
// 安全服務呼叫
typedef struct {
    uint32_t service_id;
    uint32_t parameters[4];
    uint32_t return_value;
} secure_service_t;

uint32_t call_secure_service(uint32_t service_id, uint32_t* params) {
    secure_service_t service = {
        .service_id = service_id,
        .parameters = {params[0], params[1], params[2], params[3]}
    };
    
    // 呼叫安全服務
    __asm volatile (
        "mov r0, %0\n"
        "smc #1\n"
        : : "r" (&service) : "r0", "r1", "r2", "r3"
    );
    
    return service.return_value;
}

// 安全服務範例
uint32_t secure_crypto_service(uint32_t operation, uint32_t* data) {
    uint32_t params[4] = {operation, (uint32_t)data, 0, 0};
    return call_secure_service(0x1001, params);
}
```

## 🔐 記憶體存取控制

### 存取權限檢查
```c
// 記憶體存取權限
typedef enum {
    ACCESS_NONE = 0,
    ACCESS_READ = 1,
    ACCESS_WRITE = 2,
    ACCESS_EXECUTE = 4,
    ACCESS_ALL = 7
} memory_access_t;

typedef struct {
    uint32_t start_address;
    uint32_t end_address;
    memory_access_t permissions;
    bool is_secure;
} memory_region_t;

// 檢查記憶體存取權限
bool check_memory_access(uint32_t address, memory_access_t access) {
    memory_region_t* regions = get_memory_regions();
    int region_count = get_region_count();
    
    for (int i = 0; i < region_count; i++) {
        if (address >= regions[i].start_address && 
            address <= regions[i].end_address) {
            
            // 檢查當前狀態是否符合區域安全性
            bool current_secure = is_secure_state();
            if (regions[i].is_secure != current_secure) {
                return false;  // 安全狀態不符
            }
            
            // 檢查權限
            return (regions[i].permissions & access) == access;
        }
    }
    
    return false;  // 位址不在任何區域中
}

// 記憶體存取包裝器
void* secure_memory_access(uint32_t address, memory_access_t access) {
    if (!check_memory_access(address, access)) {
        // 觸發記憶體保護錯誤
        trigger_memory_fault(address, access);
        return NULL;
    }
    
    return (void*)address;
}
```

### 緩衝區溢位保護
```c
// 帶邊界檢查的受保護緩衝區
typedef struct {
    uint8_t* data;
    size_t size;
    uint8_t* guard_zone;
    size_t guard_size;
} protected_buffer_t;

protected_buffer_t* create_protected_buffer(size_t size) {
    protected_buffer_t* buffer = malloc(sizeof(protected_buffer_t));
    if (!buffer) return NULL;
    
    // 配置保護區域和資料
    size_t guard_size = 64;  // 64 位元組保護區域
    buffer->guard_size = guard_size;
    buffer->size = size;
    
    buffer->guard_zone = malloc(guard_size * 2 + size);
    if (!buffer->guard_zone) {
        free(buffer);
        return NULL;
    }
    
    buffer->data = buffer->guard_zone + guard_size;
    
    // 初始化保護區域
    memset(buffer->guard_zone, 0xAA, guard_size);
    memset(buffer->data + size, 0xAA, guard_size);
    
    return buffer;
}

bool check_buffer_integrity(protected_buffer_t* buffer) {
    // 檢查保護區域
    for (size_t i = 0; i < buffer->guard_size; i++) {
        if (buffer->guard_zone[i] != 0xAA) {
            return false;  // 下方保護區域已損壞
        }
        if (buffer->data[buffer->size + i] != 0xAA) {
            return false;  // 上方保護區域已損壞
        }
    }
    return true;
}

void destroy_protected_buffer(protected_buffer_t* buffer) {
    if (buffer) {
        if (!check_buffer_integrity(buffer)) {
            printf("警告：偵測到緩衝區溢位！\n");
        }
        free(buffer->guard_zone);
        free(buffer);
    }
}
```

## 🛡️ 堆疊保護

### 堆疊金絲雀值實作
```c
// 用於溢位偵測的堆疊金絲雀值
typedef struct {
    uint32_t canary;
    uint8_t data[];
} stack_frame_t;

// 產生隨機金絲雀值
uint32_t generate_canary(void) {
    // 如果可用，使用硬體隨機數產生器
    return 0xDEADBEEF;  // 示範用
}

// 建立受保護的堆疊框架
stack_frame_t* create_protected_stack_frame(size_t data_size) {
    stack_frame_t* frame = malloc(sizeof(stack_frame_t) + data_size);
    if (!frame) return NULL;
    
    frame->canary = generate_canary();
    return frame;
}

// 檢查堆疊框架完整性
bool check_stack_integrity(stack_frame_t* frame) {
    return frame->canary == generate_canary();
}

// 帶堆疊保護的函式
void function_with_stack_protection(void) {
    stack_frame_t* frame = create_protected_stack_frame(100);
    if (!frame) return;
    
    // 使用 frame->data...
    
    if (!check_stack_integrity(frame)) {
        printf("偵測到堆疊溢位！\n");
        // 處理溢位
    }
    
    free(frame);
}
```

### 堆疊指標驗證
```c
// 驗證堆疊指標邊界
typedef struct {
    uint32_t stack_start;
    uint32_t stack_end;
    uint32_t current_sp;
} stack_validator_t;

bool validate_stack_pointer(stack_validator_t* validator) {
    uint32_t sp;
    __asm volatile ("mov %0, sp" : "=r" (sp));
    
    validator->current_sp = sp;
    
    return (sp >= validator->stack_start && sp <= validator->stack_end);
}

// 堆疊溢位偵測
void check_stack_overflow(stack_validator_t* validator) {
    if (!validate_stack_pointer(validator)) {
        printf("堆疊指標超出邊界！\n");
        // 處理堆疊溢位
        handle_stack_overflow();
    }
}
```

## 🗄️ 堆積保護

### 堆積損壞偵測
```c
// 受保護的堆積配置
typedef struct {
    uint32_t magic_start;
    size_t size;
    uint8_t data[];
} protected_heap_block_t;

#define HEAP_MAGIC_START 0xDEADBEEF
#define HEAP_MAGIC_END   0xCAFEBABE

void* protected_malloc(size_t size) {
    protected_heap_block_t* block = malloc(sizeof(protected_heap_block_t) + 
                                         size + sizeof(uint32_t));
    if (!block) return NULL;
    
    block->magic_start = HEAP_MAGIC_START;
    block->size = size;
    
    // 加入結尾魔術數字
    uint32_t* end_magic = (uint32_t*)(block->data + size);
    *end_magic = HEAP_MAGIC_END;
    
    return block->data;
}

bool check_heap_block_integrity(void* ptr) {
    protected_heap_block_t* block = (protected_heap_block_t*)((char*)ptr - 
                                                             offsetof(protected_heap_block_t, data));
    
    // 檢查起始魔術數字
    if (block->magic_start != HEAP_MAGIC_START) {
        return false;
    }
    
    // 檢查結尾魔術數字
    uint32_t* end_magic = (uint32_t*)(block->data + block->size);
    if (*end_magic != HEAP_MAGIC_END) {
        return false;
    }
    
    return true;
}

void protected_free(void* ptr) {
    if (!check_heap_block_integrity(ptr)) {
        printf("偵測到堆積損壞！\n");
        // 處理損壞
        return;
    }
    
    protected_heap_block_t* block = (protected_heap_block_t*)((char*)ptr - 
                                                             offsetof(protected_heap_block_t, data));
    free(block);
}
```

## 🔒 程式碼保護

### 程式碼完整性檢查
```c
// 程式碼完整性驗證
typedef struct {
    uint32_t start_address;
    uint32_t size;
    uint32_t checksum;
} code_region_t;

uint32_t calculate_checksum(uint32_t start, uint32_t size) {
    uint32_t checksum = 0;
    uint32_t* ptr = (uint32_t*)start;
    
    for (uint32_t i = 0; i < size / 4; i++) {
        checksum ^= ptr[i];
    }
    
    return checksum;
}

bool verify_code_integrity(code_region_t* region) {
    uint32_t current_checksum = calculate_checksum(region->start_address, 
                                                  region->size);
    return current_checksum == region->checksum;
}

// 驗證所有程式碼區域
void verify_all_code_regions(void) {
    code_region_t* regions = get_code_regions();
    int region_count = get_code_region_count();
    
    for (int i = 0; i < region_count; i++) {
        if (!verify_code_integrity(&regions[i])) {
            printf("區域 %d 發生程式碼完整性違規！\n", i);
            // 處理程式碼損壞
            handle_code_corruption();
        }
    }
}
```

### 僅執行記憶體
```c
// 設定僅執行記憶體區域
void configure_execute_only_memory(uint32_t start, uint32_t size) {
    mpu_region_t region = {
        .base_address = start,
        .size = size,
        .access_permissions = MPU_REGION_EXECUTE_ONLY,
        .attributes = MPU_REGION_CACHEABLE
    };
    
    configure_mpu_region(0, &region);
}

// 防止程式碼被讀取
void protect_code_from_reading(uint32_t code_start, uint32_t code_size) {
    // 設定 MPU 使程式碼為僅執行
    configure_execute_only_memory(code_start, code_size);
}
```

## ⏱️ 即時保護

### 記憶體保護錯誤處理常式
```c
// 記憶體保護錯誤處理常式
void memory_protection_fault_handler(void) {
    uint32_t fault_address;
    uint32_t fault_type;
    
    // 讀取錯誤資訊
    __asm volatile (
        "mrc p15, 0, %0, c6, c0, 0\n"  // 讀取 FAR
        "mrc p15, 0, %1, c5, c0, 0\n"  // 讀取 FSR
        : "=r" (fault_address), "=r" (fault_type)
    );
    
    printf("記憶體保護錯誤！\n");
    printf("位址：0x%08X\n", fault_address);
    printf("類型：0x%08X\n", fault_type);
    
    // 根據類型處理錯誤
    if (fault_type & 0x1) {
        // 資料中止
        handle_data_abort(fault_address);
    } else {
        // 指令中止
        handle_instruction_abort(fault_address);
    }
}

void handle_data_abort(uint32_t address) {
    // 檢查是否為可恢復的錯誤
    if (is_recoverable_fault(address)) {
        // 嘗試恢復
        attempt_fault_recovery(address);
    } else {
        // 系統重置或錯誤處理
        system_reset();
    }
}
```

### 受保護的臨界區段
```c
// 臨界區段的記憶體保護
typedef struct {
    uint32_t original_permissions;
    bool protection_enabled;
} critical_section_protection_t;

void enter_critical_section_protection(critical_section_protection_t* protection) {
    // 儲存當前記憶體權限
    protection->original_permissions = get_current_memory_permissions();
    
    // 啟用嚴格記憶體保護
    enable_strict_memory_protection();
    protection->protection_enabled = true;
}

void exit_critical_section_protection(critical_section_protection_t* protection) {
    if (protection->protection_enabled) {
        // 恢復原始權限
        set_memory_permissions(protection->original_permissions);
        protection->protection_enabled = false;
    }
}

// 使用範例
void critical_function(void) {
    critical_section_protection_t protection;
    
    enter_critical_section_protection(&protection);
    
    // 帶有增強保護的臨界程式碼
    perform_critical_operation();
    
    exit_critical_section_protection(&protection);
}
```

## ⚠️ 常見陷阱

### 1. 不正確的 MPU 組態
```c
// 錯誤：重疊的 MPU 區域
void incorrect_mpu_config(void) {
    mpu_region_t region1 = {0x20000000, 0x10000, MPU_REGION_USER_RW, 0};
    mpu_region_t region2 = {0x20008000, 0x10000, MPU_REGION_USER_RW, 0};
    // 區域重疊 - 未定義行為
    
    configure_mpu_region(0, &region1);
    configure_mpu_region(1, &region2);
}

// 正確：不重疊的區域
void correct_mpu_config(void) {
    mpu_region_t region1 = {0x20000000, 0x8000, MPU_REGION_USER_RW, 0};
    mpu_region_t region2 = {0x20008000, 0x8000, MPU_REGION_USER_RW, 0};
    // 區域不重疊
    
    configure_mpu_region(0, &region1);
    configure_mpu_region(1, &region2);
}
```

### 2. 忽略記憶體對齊
```c
// 錯誤：未對齊的記憶體存取
void unaligned_access(void) {
    uint8_t buffer[100];
    uint32_t* ptr = (uint32_t*)(buffer + 1);  // 未對齊的指標
    *ptr = 0x12345678;  // 在某些架構上可能導致錯誤
}

// 正確：對齊的記憶體存取
void aligned_access(void) {
    uint8_t buffer[100];
    uint32_t* ptr = (uint32_t*)((uintptr_t)(buffer + 1) & ~0x3);  // 對齊
    *ptr = 0x12345678;
}
```

### 3. 未處理保護錯誤
```c
// 錯誤：沒有錯誤處理
void unprotected_function(void) {
    uint8_t* ptr = (uint8_t*)0x40000000;  // 周邊裝置位址
    *ptr = 0xFF;  // 可能導致保護錯誤
    // 沒有錯誤處理常式 - 系統可能當機
}

// 正確：適當的錯誤處理
void protected_function(void) {
    // 安裝錯誤處理常式
    install_memory_fault_handler();
    
    uint8_t* ptr = (uint8_t*)0x40000000;
    *ptr = 0xFF;  // 錯誤將被優雅地處理
}
```

## ✅ 最佳實踐

### 1. 記憶體保護組態設定
```c
// 在啟動時設定記憶體保護
void initialize_memory_protection(void) {
    // 設定期間停用 MPU
    MPU->CTRL = 0;
    
    // 設定區域
    setup_memory_regions();
    
    // 啟用 MPU
    MPU->CTRL = MPU_CTRL_ENABLE_Msk;
    
    // 啟用錯誤處理常式
    SCB->SHCSR |= SCB_SHCSR_MEMFAULTENA_Msk;
    
    // 設定錯誤處理常式
    NVIC_SetPriority(MemoryManagement_IRQn, 0);
    NVIC_EnableIRQ(MemoryManagement_IRQn);
}
```

### 2. 記憶體保護監控
```c
// 監控記憶體保護違規
typedef struct {
    uint32_t fault_count;
    uint32_t last_fault_address;
    uint32_t last_fault_type;
} memory_protection_monitor_t;

static memory_protection_monitor_t protection_monitor = {0};

void update_protection_monitor(uint32_t address, uint32_t type) {
    protection_monitor.fault_count++;
    protection_monitor.last_fault_address = address;
    protection_monitor.last_fault_type = type;
    
    if (protection_monitor.fault_count > 10) {
        printf("警告：記憶體保護錯誤率過高\n");
        // 實作恢復或關閉
    }
}
```

### 3. 安全記憶體管理
```c
// 安全記憶體配置
void* secure_malloc(size_t size) {
    // 檢查在當前安全狀態下是否允許配置
    if (!is_allocation_allowed()) {
        return NULL;
    }
    
    void* ptr = malloc(size);
    if (ptr) {
        // 如果在安全狀態下，將記憶體標記為安全
        if (is_secure_state()) {
            mark_memory_secure(ptr, size);
        }
    }
    
    return ptr;
}

bool is_allocation_allowed(void) {
    // 檢查當前安全狀態和權限
    return true;  // 實作取決於系統
}

void mark_memory_secure(void* ptr, size_t size) {
    // 在 MPU/MMU 中將記憶體區域標記為安全
    // 實作取決於硬體
}
```

## 🎯 面試問題

### 基礎問題
1. **什麼是記憶體保護，為什麼在嵌入式系統中很重要？**
   - 記憶體保護防止對記憶體區域的未授權存取
   - 對安全性、穩定性和任務隔離至關重要

2. **MPU 和 MMU 之間有什麼區別？**
   - MPU：簡單、固定區域、無虛擬記憶體
   - MMU：複雜、靈活、支援虛擬記憶體

3. **ARM TrustZone 如何提供安全性？**
   - 分隔安全和非安全世界
   - 提供基於硬體的安全隔離

### 進階問題
1. **如何在即時系統中實作記憶體保護？**
   - 使用 MPU 進行簡單保護
   - 為不同任務設定區域
   - 高效地處理保護錯誤

2. **不同記憶體保護機制之間的權衡是什麼？**
   - MPU：簡單但靈活性有限
   - MMU：複雜但功能強大
   - TrustZone：安全但增加複雜性

3. **如何偵測和處理記憶體保護違規？**
   - 安裝錯誤處理常式
   - 監控錯誤率
   - 實作恢復機制

## 📚 額外資源

### 標準與文件
- **ARM 架構參考手冊**：MPU/MMU 規格
- **ARM TrustZone 文件**：安全功能
- **C 標準**：記憶體安全要求

### 相關主題
- **[堆疊溢位預防](./Stack_Overflow_Prevention.md)** - 堆疊保護
- **[記憶體池配置](./Memory_Pool_Allocation.md)** - 安全的記憶體管理
- **[快取感知程式設計](./Cache_Aware_Programming.md)** - 記憶體存取模式
- **[嵌入式安全](./embedded_security.md)** - 安全考量

### 工具與函式庫
- **ARM TrustZone**：硬體安全功能
- **MPU/MMU 設定工具**：記憶體保護設定
- **記憶體保護監控器**：執行時期保護驗證

---

**下一主題：** [快取感知程式設計](./Cache_Aware_Programming.md) → [DMA 緩衝區管理](./DMA_Buffer_Management.md)
