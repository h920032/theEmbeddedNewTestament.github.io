# 多執行緒

> **Linux 應用程式中的並行執行**  
> 理解 pthread 程式設計、執行緒同步以及嵌入式系統的並行程式設計

---

## 📋 **目錄**

- [執行緒基礎](#執行緒基礎)
- [POSIX 執行緒 (pthread)](#posix-執行緒-pthread)
- [執行緒建立與管理](#執行緒建立與管理)
- [執行緒同步](#執行緒同步)
- [執行緒通訊](#執行緒通訊)
- [執行緒安全與最佳實踐](#執行緒安全與最佳實踐)
- [進階執行緒技術](#進階執行緒技術)
- [效能與除錯](#效能與除錯)

---

## 🏗️ **執行緒基礎**

### **什麼是執行緒？**

執行緒是行程中的輕量級執行單元，它們共享相同的記憶體空間和資源。與擁有獨立記憶體空間的行程不同，執行緒可以直接存取共享資料，使其成為嵌入式系統中並行程式設計的理想選擇。

**執行緒抽象概念：**

- **輕量級**：建立和銷毀的速度遠快於行程
- **共享記憶體**：行程中的所有執行緒共享相同的位址空間
- **並行執行**：多個執行緒可以同時運行
- **資源共享**：執行緒共享檔案描述子、記憶體和其他資源
- **高效通訊**：直接存取共享資料結構

#### **執行緒 vs. 行程：理解差異**

理解何時使用執行緒與行程對於高效的嵌入式程式設計至關重要：

**行程：**
- **記憶體空間**：每個行程擁有獨立的虛擬位址空間
- **資源隔離**：資源完全隔離
- **通訊**：需要 IPC 機制（管道、socket 等）
- **額外開銷**：較高的建立和上下文切換開銷
- **使用場景**：獨立應用程式、安全隔離

**執行緒：**
- **記憶體空間**：行程內共享位址空間
- **資源共享**：直接存取共享記憶體和資源
- **通訊**：直接存取共享資料結構
- **額外開銷**：較低的建立和上下文切換開銷
- **使用場景**：應用程式內的並行任務、效能最佳化

```
┌─────────────────────────────────────┐
│         行程位址空間                │
├─────────────────────────────────────┤
│         程式碼區段                  │ ← 所有執行緒共享
│         （程式指令）                │
├─────────────────────────────────────┤
│         資料區段                    │ ← 所有執行緒共享
│         （全域變數）                │
├─────────────────────────────────────┤
│         堆積區                      │ ← 所有執行緒共享
│         （動態配置）                │
├─────────────────────────────────────┤
│         堆疊區段                    │ ← 每個執行緒獨立
│  ┌──────────┬──────────┬──────────┐ │
│  │執行緒 1  │執行緒 2  │執行緒 3  │ │
│  │  堆疊    │  堆疊    │  堆疊    │ │
│  └──────────┴──────────┴──────────┘ │
└─────────────────────────────────────┘
```

#### **執行緒設計哲學**

執行緒遵循**並行執行原則**——在維持資料一致性和系統穩定性的同時，實現多個任務的同時執行。

**執行緒設計目標：**

- **並行性**：實現多個任務的平行執行
- **效率**：最小化執行緒建立和管理的開銷
- **安全性**：確保對共享資源的執行緒安全存取
- **效能**：針對多核心系統進行最佳化
- **簡潔性**：提供清晰且直觀的程式設計模型

---

## 🔧 **POSIX 執行緒 (pthread)**

### **標準執行緒介面**

POSIX 執行緒（pthread）是類 Unix 系統（包括 Linux）的標準執行緒介面。它提供了一套完整的函式，用於執行緒的建立、管理和同步。

#### **POSIX 執行緒設計哲學**

POSIX 執行緒遵循**可攜性與一致性原則**——提供在不同系統上一致運作的標準介面，同時維持高效能和靈活性。

**pthread 設計目標：**

- **可攜性**：在符合 POSIX 標準的系統上一致運作
- **效能**：執行緒操作的最小開銷
- **靈活性**：支援各種執行緒模型和模式
- **可靠性**：強健的錯誤處理和資源管理
- **標準符合性**：遵循 POSIX.1c 執行緒標準

#### **核心 pthread 元件**

**執行緒管理：**
- **`pthread_create()`**：建立新執行緒
- **`pthread_join()`**：等待執行緒完成
- **`pthread_detach()`**：使執行緒自動清理
- **`pthread_exit()`**：終止執行緒執行

**同步機制：**
- **`pthread_mutex_t`**：互斥鎖
- **`pthread_cond_t`**：條件變數
- **`pthread_sem_t`**：號誌
- **`pthread_rwlock_t`**：讀寫鎖

**執行緒屬性：**
- **`pthread_attr_t`**：執行緒建立屬性
- **堆疊大小和位置**：控制執行緒記憶體使用
- **排程策略**：設定執行緒排程參數
- **分離狀態**：控制執行緒清理行為

---

## 🔄 **執行緒建立與管理**

### **建立與管理執行緒生命週期**

執行緒建立和管理涉及設定執行緒、控制其執行以及確保適當的清理。理解這些操作對於建構強健的多執行緒應用程式至關重要。

#### **執行緒建立哲學**

執行緒建立遵循**受控初始化原則**——以符合其預期目的和系統需求的特定屬性和行為來建立執行緒。

**執行緒建立目標：**

- **效率**：最小化執行緒建立的開銷
- **控制**：為每個執行緒設定適當的屬性
- **資源管理**：控制執行緒記憶體和排程
- **錯誤處理**：優雅地處理建立失敗
- **清理**：確保適當的資源清理

#### **基本執行緒建立**

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>
#include <string.h>

// 執行緒函式簽名：void *function_name(void *arg)
void *thread_function(void *arg) {
    int thread_id = *(int *)arg;
    
    printf("執行緒 %d 啟動中\n", thread_id);
    
    // 模擬工作
    for (int i = 0; i < 5; i++) {
        printf("執行緒 %d：第 %d 次迭代\n", thread_id, i);
        sleep(1);
    }
    
    printf("執行緒 %d 完成中\n", thread_id);
    
    // 回傳值（可由 join 的執行緒使用）
    int *result = malloc(sizeof(int));
    *result = thread_id * 100;
    return result;
}

// 基本執行緒建立範例
int basic_thread_creation(void) {
    pthread_t threads[3];
    int thread_ids[3];
    void *thread_results[3];
    int i;
    
    printf("主執行緒啟動中\n");
    
    // 建立執行緒
    for (i = 0; i < 3; i++) {
        thread_ids[i] = i;
        
        if (pthread_create(&threads[i], NULL, thread_function, &thread_ids[i]) != 0) {
            perror("建立執行緒失敗");
            return -1;
        }
        
        printf("已建立執行緒 %d\n", i);
    }
    
    // 等待所有執行緒完成
    for (i = 0; i < 3; i++) {
        if (pthread_join(threads[i], &thread_results[i]) != 0) {
            perror("join 執行緒失敗");
            return -1;
        }
        
        printf("執行緒 %d 回傳：%d\n", i, *(int *)thread_results[i]);
        free(thread_results[i]);  // 釋放回傳的記憶體
    }
    
    printf("所有執行緒已完成\n");
    return 0;
}
```

#### **執行緒屬性與控制**

```c
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

// 具有屬性的執行緒函式
void *attributed_thread_function(void *arg) {
    pthread_t self = pthread_self();
    
    // 取得執行緒屬性
    int policy;
    struct sched_param param;
    
    if (pthread_getschedparam(self, &policy, &param) == 0) {
        printf("執行緒 %lu：排程策略=%d，優先權=%d\n", 
               (unsigned long)self, policy, param.sched_priority);
    }
    
    // 模擬工作
    sleep(2);
    
    return NULL;
}

// 使用自訂屬性建立執行緒
int attributed_thread_creation(void) {
    pthread_t thread;
    pthread_attr_t attr;
    struct sched_param param;
    
    // 初始化執行緒屬性
    if (pthread_attr_init(&attr) != 0) {
        perror("初始化執行緒屬性失敗");
        return -1;
    }
    
    // 設定堆疊大小（1 MB）
    if (pthread_attr_setstacksize(&attr, 1024 * 1024) != 0) {
        perror("設定堆疊大小失敗");
        pthread_attr_destroy(&attr);
        return -1;
    }
    
    // 設定排程策略為 SCHED_FIFO（即時排程）
    if (pthread_attr_setschedpolicy(&attr, SCHED_FIFO) != 0) {
        perror("設定排程策略失敗");
        pthread_attr_destroy(&attr);
        return -1;
    }
    
    // 設定優先權
    param.sched_priority = 50;
    if (pthread_attr_setschedparam(&attr, &param) != 0) {
        perror("設定排程參數失敗");
        pthread_attr_destroy(&attr);
        return -1;
    }
    
    // 設定分離狀態為已分離（自動清理）
    if (pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED) != 0) {
        perror("設定分離狀態失敗");
        pthread_attr_destroy(&attr);
        return -1;
    }
    
    // 使用屬性建立執行緒
    if (pthread_create(&thread, &attr, attributed_thread_function, NULL) != 0) {
        perror("建立執行緒失敗");
        pthread_attr_destroy(&attr);
        return -1;
    }
    
    printf("已建立具有自訂屬性的分離執行緒\n");
    
    // 清理屬性
    pthread_attr_destroy(&attr);
    
    // 等待一段時間讓執行緒完成（因為它是分離的）
    sleep(3);
    
    return 0;
}
```

#### **執行緒生命週期管理**

```c
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>

// 全域執行緒控制
volatile int thread_running = 1;
pthread_t worker_thread;

// 工作執行緒函式
void *worker_thread_function(void *arg) {
    printf("工作執行緒啟動中\n");
    
    while (thread_running) {
        printf("工作執行緒：處理中...\n");
        sleep(1);
    }
    
    printf("工作執行緒：收到關閉訊號\n");
    return NULL;
}

// 優雅關閉的訊號處理器
void signal_handler(int sig) {
    if (sig == SIGINT || sig == SIGTERM) {
        printf("收到關閉訊號\n");
        thread_running = 0;
    }
}

// 執行緒生命週期管理範例
int thread_lifecycle_example(void) {
    void *thread_result;
    
    // 設定訊號處理
    signal(SIGINT, signal_handler);
    signal(SIGTERM, signal_handler);
    
    printf("建立工作執行緒中\n");
    
    // 建立工作執行緒
    if (pthread_create(&worker_thread, NULL, worker_thread_function, NULL) != 0) {
        perror("建立工作執行緒失敗");
        return -1;
    }
    
    printf("工作執行緒已建立。按 Ctrl+C 停止\n");
    
    // 等待關閉訊號
    while (thread_running) {
        sleep(1);
    }
    
    printf("等待工作執行緒完成中\n");
    
    // 等待執行緒完成
    if (pthread_join(worker_thread, &thread_result) != 0) {
        perror("join 工作執行緒失敗");
        return -1;
    }
    
    printf("工作執行緒已完成\n");
    return 0;
}
```

---

## 🔒 **執行緒同步**

### **協調對共享資源的存取**

執行緒同步對於防止競爭條件和確保多個執行緒存取共享資源時的資料一致性至關重要。理解同步機制對於建構可靠的多執行緒應用程式極為關鍵。

#### **同步設計哲學**

執行緒同步遵循**安全與效能原則**——確保對共享資源的執行緒安全存取，同時最小化效能開銷並避免死結。

**同步目標：**

- **安全性**：防止競爭條件和資料損壞
- **效能**：最小化同步開銷
- **死結預防**：避免循環等待條件
- **可擴展性**：支援不斷增加的執行緒數量
- **簡潔性**：提供清晰且直觀的同步原語

#### **基於互斥鎖的同步**

互斥鎖提供互斥存取，確保同一時間只有一個執行緒可以存取臨界區段：

```c
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

// 共享資料結構
struct shared_data {
    int counter;
    pthread_mutex_t mutex;
    char buffer[256];
};

// 使用互斥鎖的執行緒函式
void *mutex_thread_function(void *arg) {
    struct shared_data *data = (struct shared_data *)arg;
    int thread_id = *(int *)arg;
    
    for (int i = 0; i < 10; i++) {
        // 取得互斥鎖
        if (pthread_mutex_lock(&data->mutex) != 0) {
            perror("鎖定互斥鎖失敗");
            return NULL;
        }
        
        // 臨界區段
        data->counter++;
        snprintf(data->buffer, sizeof(data->buffer), 
                "執行緒 %d 將計數器更新為 %d", thread_id, data->counter);
        printf("%s\n", data->buffer);
        
        // 在臨界區段中模擬工作
        usleep(100000);  // 100 毫秒
        
        // 釋放互斥鎖
        if (pthread_mutex_unlock(&data->mutex) != 0) {
            perror("解鎖互斥鎖失敗");
            return NULL;
        }
        
        // 非臨界區段工作
        usleep(50000);  // 50 毫秒
    }
    
    return NULL;
}

// 互斥鎖同步範例
int mutex_synchronization_example(void) {
    pthread_t threads[3];
    int thread_ids[3];
    struct shared_data shared_data = {0};
    
    // 初始化互斥鎖
    if (pthread_mutex_init(&shared_data.mutex, NULL) != 0) {
        perror("初始化互斥鎖失敗");
        return -1;
    }
    
    printf("開始互斥鎖同步範例\n");
    printf("初始計數器值：%d\n", shared_data.counter);
    
    // 建立執行緒
    for (int i = 0; i < 3; i++) {
        thread_ids[i] = i;
        if (pthread_create(&threads[i], NULL, mutex_thread_function, &shared_data) != 0) {
            perror("建立執行緒失敗");
            return -1;
        }
    }
    
    // 等待所有執行緒完成
    for (int i = 0; i < 3; i++) {
        if (pthread_join(threads[i], NULL) != 0) {
            perror("join 執行緒失敗");
            return -1;
        }
    }
    
    printf("最終計數器值：%d\n", shared_data.counter);
    
    // 清理互斥鎖
    pthread_mutex_destroy(&shared_data.mutex);
    
    return 0;
}
```

#### **條件變數同步**

條件變數允許執行緒等待特定條件成立：

```c
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

// 生產者-消費者資料結構
struct producer_consumer {
    int buffer[10];
    int count;
    int head;
    int tail;
    pthread_mutex_t mutex;
    pthread_cond_t not_empty;
    pthread_cond_t not_full;
    int shutdown;
};

// 生產者執行緒函式
void *producer_function(void *arg) {
    struct producer_consumer *pc = (struct producer_consumer *)arg;
    int item = 0;
    
    while (!pc->shutdown) {
        // 取得互斥鎖
        pthread_mutex_lock(&pc->mutex);
        
        // 當緩衝區滿時等待
        while (pc->count == 10 && !pc->shutdown) {
            printf("生產者：緩衝區已滿，等待中...\n");
            pthread_cond_wait(&pc->not_full, &pc->mutex);
        }
        
        if (pc->shutdown) {
            pthread_mutex_unlock(&pc->mutex);
            break;
        }
        
        // 將項目加入緩衝區
        pc->buffer[pc->tail] = item;
        pc->tail = (pc->tail + 1) % 10;
        pc->count++;
        
        printf("生產者：加入項目 %d，數量 = %d\n", item, pc->count);
        
        // 通知消費者緩衝區非空
        pthread_cond_signal(&pc->not_empty);
        
        // 釋放互斥鎖
        pthread_mutex_unlock(&pc->mutex);
        
        item++;
        usleep(200000);  // 200 毫秒
    }
    
    printf("生產者：正在關閉\n");
    return NULL;
}

// 消費者執行緒函式
void *consumer_function(void *arg) {
    struct producer_consumer *pc = (struct producer_consumer *)arg;
    
    while (!pc->shutdown) {
        // 取得互斥鎖
        pthread_mutex_lock(&pc->mutex);
        
        // 當緩衝區為空時等待
        while (pc->count == 0 && !pc->shutdown) {
            printf("消費者：緩衝區為空，等待中...\n");
            pthread_cond_wait(&pc->not_empty, &pc->mutex);
        }
        
        if (pc->shutdown && pc->count == 0) {
            pthread_mutex_unlock(&pc->mutex);
            break;
        }
        
        // 從緩衝區移除項目
        int item = pc->buffer[pc->head];
        pc->head = (pc->head + 1) % 10;
        pc->count--;
        
        printf("消費者：移除項目 %d，數量 = %d\n", item, pc->count);
        
        // 通知生產者緩衝區非滿
        pthread_cond_signal(&pc->not_full);
        
        // 釋放互斥鎖
        pthread_mutex_unlock(&pc->mutex);
        
        usleep(300000);  // 300 毫秒
    }
    
    printf("消費者：正在關閉\n");
    return NULL;
}

// 條件變數範例
int condition_variable_example(void) {
    pthread_t producer_thread, consumer_thread;
    struct producer_consumer pc = {0};
    
    // 初始化同步原語
    if (pthread_mutex_init(&pc.mutex, NULL) != 0 ||
        pthread_cond_init(&pc.not_empty, NULL) != 0 ||
        pthread_cond_init(&pc.not_full, NULL) != 0) {
        perror("初始化同步原語失敗");
        return -1;
    }
    
    printf("開始生產者-消費者範例\n");
    
    // 建立生產者和消費者執行緒
    if (pthread_create(&producer_thread, NULL, producer_function, &pc) != 0 ||
        pthread_create(&consumer_thread, NULL, consumer_function, &pc) != 0) {
        perror("建立執行緒失敗");
        return -1;
    }
    
    // 讓執行緒運行一段時間
    sleep(5);
    
    // 發送關閉訊號
    pthread_mutex_lock(&pc.mutex);
    pc.shutdown = 1;
    pthread_cond_broadcast(&pc.not_empty);
    pthread_cond_broadcast(&pc.not_full);
    pthread_mutex_unlock(&pc.mutex);
    
    // 等待執行緒完成
    pthread_join(producer_thread, NULL);
    pthread_join(consumer_thread, NULL);
    
    printf("生產者-消費者範例完成\n");
    
    // 清理
    pthread_mutex_destroy(&pc.mutex);
    pthread_cond_destroy(&pc.not_empty);
    pthread_cond_destroy(&pc.not_full);
    
    return 0;
}
```

---

## 📡 **執行緒通訊**

### **在執行緒之間共享資料**

執行緒通訊涉及在執行緒之間共享資料和協調活動。理解安全的通訊模式對於建構可靠的多執行緒應用程式至關重要。

#### **執行緒通訊哲學**

執行緒通訊遵循**安全共享原則**——在維持資料一致性和防止競爭條件的同時，實現執行緒之間的高效資料共享。

**通訊目標：**

- **效率**：最小化資料共享的開銷
- **安全性**：確保對共享資料的執行緒安全存取
- **清晰性**：使通訊模式易於理解
- **效能**：針對常見通訊模式進行最佳化
- **可靠性**：優雅地處理通訊失敗

#### **共享記憶體通訊**

```c
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

// 共享資料結構
struct shared_memory {
    int data[100];
    int data_count;
    pthread_mutex_t mutex;
    pthread_cond_t data_ready;
    int shutdown;
};

// 寫入者執行緒函式
void *writer_function(void *arg) {
    struct shared_memory *shared = (struct shared_memory *)arg;
    
    for (int i = 0; i < 50; i++) {
        pthread_mutex_lock(&shared->mutex);
        
        // 將資料加入共享記憶體
        shared->data[shared->data_count] = i * 2;
        shared->data_count++;
        
        printf("寫入者：加入資料 %d，數量 = %d\n", i * 2, shared->data_count);
        
        // 通知資料已就緒
        pthread_cond_signal(&shared->data_ready);
        
        pthread_mutex_unlock(&shared->mutex);
        
        usleep(100000);  // 100 毫秒
    }
    
    // 發送關閉訊號
    pthread_mutex_lock(&shared->mutex);
    shared->shutdown = 1;
    pthread_cond_broadcast(&shared->data_ready);
    pthread_mutex_unlock(&shared->mutex);
    
    printf("寫入者：已完成\n");
    return NULL;
}

// 讀取者執行緒函式
void *reader_function(void *arg) {
    struct shared_memory *shared = (struct shared_memory *)arg;
    int last_read = 0;
    
    while (!shared->shutdown || last_read < shared->data_count) {
        pthread_mutex_lock(&shared->mutex);
        
        // 等待新資料
        while (last_read >= shared->data_count && !shared->shutdown) {
            printf("讀取者：等待資料中...\n");
            pthread_cond_wait(&shared->data_ready, &shared->mutex);
        }
        
        // 讀取可用資料
        while (last_read < shared->data_count) {
            printf("讀取者：讀取資料 %d\n", shared->data[last_read]);
            last_read++;
        }
        
        pthread_mutex_unlock(&shared->mutex);
        
        usleep(150000);  // 150 毫秒
    }
    
    printf("讀取者：已完成\n");
    return NULL;
}

// 共享記憶體通訊範例
int shared_memory_communication_example(void) {
    pthread_t writer_thread, reader_thread;
    struct shared_memory shared = {0};
    
    // 初始化同步原語
    if (pthread_mutex_init(&shared.mutex, NULL) != 0 ||
        pthread_cond_init(&shared.data_ready, NULL) != 0) {
        perror("初始化同步原語失敗");
        return -1;
    }
    
    printf("開始共享記憶體通訊範例\n");
    
    // 建立寫入者和讀取者執行緒
    if (pthread_create(&writer_thread, NULL, writer_function, &shared) != 0 ||
        pthread_create(&reader_thread, NULL, reader_function, &shared) != 0) {
        perror("建立執行緒失敗");
        return -1;
    }
    
    // 等待執行緒完成
    pthread_join(writer_thread, NULL);
    pthread_join(reader_thread, NULL);
    
    printf("共享記憶體通訊範例完成\n");
    printf("總共處理的資料項目數：%d\n", shared.data_count);
    
    // 清理
    pthread_mutex_destroy(&shared.mutex);
    pthread_cond_destroy(&shared.data_ready);
    
    return 0;
}
```

---

## 🛡️ **執行緒安全與最佳實踐**

### **建構可靠的多執行緒應用程式**

執行緒安全涉及設計可以被多個執行緒同時安全執行的程式碼。遵循最佳實踐對於建構可靠且可維護的多執行緒應用程式至關重要。

#### **執行緒安全哲學**

執行緒安全遵循**防禦性程式設計原則**——假設程式碼將被多個執行緒執行，並設計它以安全地處理並行存取。

**執行緒安全目標：**

- **正確性**：確保在所有執行緒交錯情況下的正確行為
- **效能**：最小化同步開銷
- **可維護性**：使執行緒安全的程式碼易於理解和修改
- **可靠性**：防止競爭條件和死結
- **可擴展性**：支援不斷增加的執行緒數量

#### **執行緒安全模式**

**不可變資料：**
```c
// 執行緒安全的不可變資料結構
struct immutable_config {
    const int max_threads;
    const char *log_level;
    const double timeout;
};

// 全域不可變組態
static const struct immutable_config config = {
    .max_threads = 10,
    .log_level = "INFO",
    .timeout = 5.0
};

// 使用不可變資料的執行緒函式
void *thread_function(void *arg) {
    printf("執行緒使用組態：max_threads=%d，log_level=%s，timeout=%.1f\n",
           config.max_threads, config.log_level, config.timeout);
    
    // 不需要同步——資料是不可變的
    return NULL;
}
```

**執行緒區域儲存：**
```c
#include <pthread.h>

// 執行緒區域儲存鍵
static pthread_key_t thread_local_key;

// 執行緒區域資料結構
struct thread_local_data {
    int thread_id;
    char thread_name[64];
    int local_counter;
};

// 執行緒區域資料的解構函式
void thread_local_destructor(void *value) {
    struct thread_local_data *data = (struct thread_local_data *)value;
    printf("清理執行緒 %d 的區域資料\n", data->thread_id);
    free(data);
}

// 初始化執行緒區域儲存
int init_thread_local_storage(void) {
    if (pthread_key_create(&thread_local_key, thread_local_destructor) != 0) {
        perror("建立執行緒區域鍵失敗");
        return -1;
    }
    return 0;
}

// 取得執行緒區域資料（若不存在則建立）
struct thread_local_data *get_thread_local_data(void) {
    struct thread_local_data *data = pthread_getspecific(thread_local_key);
    
    if (data == NULL) {
        // 建立新的執行緒區域資料
        data = malloc(sizeof(struct thread_local_data));
        if (data == NULL) {
            perror("配置執行緒區域資料失敗");
            return NULL;
        }
        
        data->thread_id = (int)(long)pthread_self();
        snprintf(data->thread_name, sizeof(data->thread_name), "Thread_%d", data->thread_id);
        data->local_counter = 0;
        
        pthread_setspecific(thread_local_key, data);
    }
    
    return data;
}
```

#### **執行緒安全最佳實踐**

**1. 最小化共享狀態：**
```c
// 良好：最小化共享狀態
struct thread_safe_counter {
    pthread_mutex_t mutex;
    int count;
};

// 不良：過多的共享狀態
struct thread_unsafe_counter {
    int count;
    char description[256];
    double last_update;
    int update_count;
    // ... 更多欄位
};
```

**2. 使用適當的同步機制：**
```c
// 良好：對複雜臨界區段使用互斥鎖
pthread_mutex_lock(&mutex);
// 對共享資料的複雜操作
pthread_mutex_unlock(&mutex);

// 良好：對簡單操作使用原子操作
__sync_fetch_and_add(&counter, 1);

// 不良：沒有同步機制
shared_variable++;  // 競爭條件！
```

**3. 避免死結：**
```c
// 良好：一致的鎖定順序
void safe_operation(struct data *data1, struct data *data2) {
    if (data1 < data2) {
        pthread_mutex_lock(&data1->mutex);
        pthread_mutex_lock(&data2->mutex);
    } else {
        pthread_mutex_lock(&data2->mutex);
        pthread_mutex_lock(&data1->mutex);
    }
    
    // 臨界區段
    
    pthread_mutex_unlock(&data1->mutex);
    pthread_mutex_unlock(&data2->mutex);
}

// 不良：可能產生死結
void unsafe_operation(struct data *data1, struct data *data2) {
    pthread_mutex_lock(&data1->mutex);
    pthread_mutex_lock(&data2->mutex);  // 可能死結！
    
    // 臨界區段
    
    pthread_mutex_unlock(&data2->mutex);
    pthread_mutex_unlock(&data1->mutex);
}
```

---

## 🚀 **進階執行緒技術**

### **超越基本執行緒**

進階執行緒技術涉及用於高效能多執行緒應用程式的精密模式和最佳化。這些技術對於建構可擴展的嵌入式系統至關重要。

#### **執行緒池**

執行緒池管理固定數量的工作執行緒以高效處理任務：

```c
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

// 任務結構
struct task {
    void (*function)(void *);
    void *arg;
    struct task *next;
};

// 執行緒池結構
struct thread_pool {
    pthread_t *threads;
    int thread_count;
    struct task *task_queue;
    pthread_mutex_t queue_mutex;
    pthread_cond_t queue_condition;
    int shutdown;
    int active_threads;
};

// 建立執行緒池
struct thread_pool *thread_pool_create(int thread_count) {
    struct thread_pool *pool = malloc(sizeof(struct thread_pool));
    if (!pool) return NULL;
    
    pool->threads = malloc(thread_count * sizeof(pthread_t));
    if (!pool->threads) {
        free(pool);
        return NULL;
    }
    
    pool->thread_count = thread_count;
    pool->task_queue = NULL;
    pool->shutdown = 0;
    pool->active_threads = 0;
    
    if (pthread_mutex_init(&pool->queue_mutex, NULL) != 0 ||
        pthread_cond_init(&pool->queue_condition, NULL) != 0) {
        free(pool->threads);
        free(pool);
        return NULL;
    }
    
    // 建立工作執行緒
    for (int i = 0; i < thread_count; i++) {
        if (pthread_create(&pool->threads[i], NULL, worker_thread_function, pool) != 0) {
            thread_pool_destroy(pool);
            return NULL;
        }
    }
    
    return pool;
}

// 工作執行緒函式
void *worker_thread_function(void *arg) {
    struct thread_pool *pool = (struct thread_pool *)arg;
    
    while (1) {
        pthread_mutex_lock(&pool->queue_mutex);
        
        // 等待任務
        while (pool->task_queue == NULL && !pool->shutdown) {
            pthread_cond_wait(&pool->queue_condition, &pool->queue_mutex);
        }
        
        if (pool->shutdown && pool->task_queue == NULL) {
            pthread_mutex_unlock(&pool->queue_mutex);
            break;
        }
        
        // 從佇列取得任務
        struct task *task = pool->task_queue;
        pool->task_queue = task->next;
        
        pthread_mutex_unlock(&pool->queue_mutex);
        
        // 執行任務
        task->function(task->arg);
        free(task);
    }
    
    return NULL;
}

// 提交任務到執行緒池
int thread_pool_submit(struct thread_pool *pool, void (*function)(void *), void *arg) {
    struct task *task = malloc(sizeof(struct task));
    if (!task) return -1;
    
    task->function = function;
    task->arg = arg;
    task->next = NULL;
    
    pthread_mutex_lock(&pool->queue_mutex);
    
    // 將任務加入佇列
    if (pool->task_queue == NULL) {
        pool->task_queue = task;
    } else {
        struct task *current = pool->task_queue;
        while (current->next != NULL) {
            current = current->next;
        }
        current->next = task;
    }
    
    pthread_cond_signal(&pool->queue_condition);
    pthread_mutex_unlock(&pool->queue_mutex);
    
    return 0;
}
```

#### **無鎖程式設計**

無鎖程式設計使用原子操作來實現同步，而不使用鎖：

```c
#include <stdint.h>
#include <stdatomic.h>

// 無鎖堆疊實作
struct lock_free_stack_node {
    void *data;
    struct lock_free_stack_node *next;
};

struct lock_free_stack {
    _Atomic(struct lock_free_stack_node *) head;
};

// 推入操作（無鎖）
void lock_free_stack_push(struct lock_free_stack *stack, void *data) {
    struct lock_free_stack_node *new_node = malloc(sizeof(struct lock_free_stack_node));
    new_node->data = data;
    
    struct lock_free_stack_node *old_head;
    do {
        old_head = atomic_load(&stack->head);
        new_node->next = old_head;
    } while (!atomic_compare_exchange_weak(&stack->head, &old_head, new_node));
}

// 彈出操作（無鎖）
void *lock_free_stack_pop(struct lock_free_stack *stack) {
    struct lock_free_stack_node *old_head;
    struct lock_free_stack_node *new_head;
    
    do {
        old_head = atomic_load(&stack->head);
        if (old_head == NULL) return NULL;
        
        new_head = old_head->next;
    } while (!atomic_compare_exchange_weak(&stack->head, &old_head, new_head));
    
    void *data = old_head->data;
    free(old_head);
    return data;
}
```

---

## 📊 **效能與除錯**

### **最佳化與疑難排解多執行緒應用程式**

效能最佳化和除錯對於建構高效的多執行緒應用程式至關重要。理解如何測量效能和識別問題對於嵌入式開發不可或缺。

#### **效能測量**

**執行緒效能分析：**
```c
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/time.h>

// 效能測量結構
struct thread_performance {
    pthread_t thread_id;
    struct timeval start_time;
    struct timeval end_time;
    int work_count;
};

// 取得當前時間（微秒）
long long get_time_us(void) {
    struct timeval tv;
    gettimeofday(&tv, NULL);
    return tv.tv_sec * 1000000LL + tv.tv_usec;
}

// 具有效能測量的執行緒函式
void *performance_thread_function(void *arg) {
    struct thread_performance *perf = (struct thread_performance *)arg;
    
    gettimeofday(&perf->start_time, NULL);
    
    // 模擬工作
    for (int i = 0; i < 1000000; i++) {
        perf->work_count++;
        // 一些計算
        volatile int x = i * i;
        (void)x;  // 抑制未使用變數警告
    }
    
    gettimeofday(&perf->end_time, NULL);
    
    return NULL;
}

// 效能測量範例
int performance_measurement_example(void) {
    const int thread_count = 4;
    pthread_t threads[thread_count];
    struct thread_performance perf[thread_count];
    
    printf("使用 %d 個執行緒開始效能測量\n", thread_count);
    
    // 建立執行緒
    for (int i = 0; i < thread_count; i++) {
        perf[i].work_count = 0;
        if (pthread_create(&threads[i], NULL, performance_thread_function, &perf[i]) != 0) {
            perror("建立執行緒失敗");
            return -1;
        }
    }
    
    // 等待執行緒完成
    for (int i = 0; i < thread_count; i++) {
        pthread_join(threads[i], NULL);
    }
    
    // 計算並顯示效能指標
    long long total_time = 0;
    int total_work = 0;
    
    for (int i = 0; i < thread_count; i++) {
        long long start_us = perf[i].start_time.tv_sec * 1000000LL + perf[i].start_time.tv_usec;
        long long end_us = perf[i].end_time.tv_sec * 1000000LL + perf[i].end_time.tv_usec;
        long long duration = end_us - start_us;
        
        printf("執行緒 %d：%lld 微秒，%d 個工作項目，%.2f 工作/微秒\n",
               i, duration, perf[i].work_count, (double)perf[i].work_count / duration);
        
        total_time += duration;
        total_work += perf[i].work_count;
    }
    
    printf("總計：%lld 微秒，%d 個工作項目，%.2f 工作/微秒\n",
           total_time, total_work, (double)total_work / total_time);
    
    return 0;
}
```

#### **常見執行緒問題與解決方案**

**1. 競爭條件：**
```c
// 問題：競爭條件
int counter = 0;

void *unsafe_thread_function(void *arg) {
    for (int i = 0; i < 1000; i++) {
        counter++;  // 競爭條件！
    }
    return NULL;
}

// 解決方案：使用互斥鎖
pthread_mutex_t counter_mutex = PTHREAD_MUTEX_INITIALIZER;

void *safe_thread_function(void *arg) {
    for (int i = 0; i < 1000; i++) {
        pthread_mutex_lock(&counter_mutex);
        counter++;
        pthread_mutex_unlock(&counter_mutex);
    }
    return NULL;
}
```

**2. 死結：**
```c
// 問題：可能的死結
void unsafe_function(pthread_mutex_t *mutex1, pthread_mutex_t *mutex2) {
    pthread_mutex_lock(mutex1);
    pthread_mutex_lock(mutex2);  // 可能死結！
    
    // 臨界區段
    
    pthread_mutex_unlock(mutex2);
    pthread_mutex_unlock(mutex1);
}

// 解決方案：一致的鎖定順序
void safe_function(pthread_mutex_t *mutex1, pthread_mutex_t *mutex2) {
    if (mutex1 < mutex2) {
        pthread_mutex_lock(mutex1);
        pthread_mutex_lock(mutex2);
    } else {
        pthread_mutex_lock(mutex2);
        pthread_mutex_lock(mutex1);
    }
    
    // 臨界區段
    
    pthread_mutex_unlock(mutex1);
    pthread_mutex_unlock(mutex2);
}
```

---

## 🎯 **結論**

多執行緒為在 Linux 中建構並行應用程式提供了強大的機制。理解 pthread 程式設計、執行緒同步和最佳實踐對於建立可靠且高效的嵌入式系統至關重要。

**重點摘要：**

- **執行緒在單一行程內提供輕量級並行性**
- **POSIX 執行緒（pthread）**提供跨系統的標準介面
- **適當的同步機制**可防止競爭條件並確保資料一致性
- **執行緒安全**需要謹慎的設計和防禦性程式設計
- **進階技術**如執行緒池和無鎖程式設計可最佳化效能
- **效能測量和除錯**對於最佳化至關重要

**未來展望：**

隨著嵌入式系統變得越來越複雜，多核心處理器成為標準配備，多執行緒技能的重要性只會持續增加。現代系統不斷演進，提供新的執行緒原語和最佳化技術，使更強大且高效的並行應用程式成為可能。

**請記住**：多執行緒不僅僅是建立執行緒——它是關於理解如何協調並行執行、安全地管理共享資源，以及建構能夠高效利用多個 CPU 核心的應用程式。您在此所培養的技能將貫穿您的嵌入式系統職涯，使您能夠建立強健、高效且可擴展的系統。
