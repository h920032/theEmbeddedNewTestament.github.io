# 即時 Linux

> **Linux 系統中的確定性時序**  
> 了解 PREEMPT_RT、Xenomai 以及嵌入式應用的即時擴展

---

## 📋 **目錄**

- [即時基礎概念](#即時基礎概念)
- [Linux 即時擴展](#linux-即時擴展)
- [PREEMPT_RT 補丁](#preempt_rt-補丁)
- [Xenomai 框架](#xenomai-框架)
- [即時程式設計](#即時程式設計)
- [效能與延遲](#效能與延遲)
- [即時最佳實踐](#即時最佳實踐)

---

## 🏗️ **即時基礎概念**

### **什麼是即時運算？**

即時運算涉及必須在嚴格的時序約束內回應事件的系統。在嵌入式系統中，即時能力對於工業控制、汽車系統和醫療設備等應用至關重要。

**即時系統特性：**

- **確定性回應**：可預測的時序行為
- **有界延遲**：最大回應時間保證
- **優先權排程**：關鍵任務獲得即時處理
- **資源管理**：有效分配系統資源
- **容錯能力**：優雅地處理時序違規

#### **即時運算與通用運算的比較**

**通用系統：**
- **目標**：最大化吞吐量和公平性
- **排程**：分時、輪詢
- **延遲**：可變、不可預測
- **使用場景**：桌面應用程式、伺服器

**即時系統：**
- **目標**：持續滿足時序截止期限
- **排程**：基於優先權、截止期限驅動
- **延遲**：有界、可預測
- **使用場景**：控制系統、安全關鍵應用

```
┌─────────────────────────────────────┐
│          即時需求                    │
├─────────────────────────────────────┤
│  硬即時：                           │
│  • 必須滿足截止期限                  │
│  • 錯過則系統失效                    │
│  • 範例：安全氣囊部署               │
├─────────────────────────────────────┤
│  軟即時：                           │
│  • 應該滿足截止期限                  │
│  • 錯過則效能降低                    │
│  • 範例：影片串流                    │
├─────────────────────────────────────┤
│  穩固即時：                         │
│  • 必須滿足截止期限                  │
│  • 可接受偶爾錯過                    │
│  • 範例：資料記錄                    │
└─────────────────────────────────────┘
```

---

## 🔧 **Linux 即時擴展**

### **為即時應用擴展 Linux**

標準 Linux 並非為即時應用而設計。已開發出各種擴展和補丁來新增即時功能，同時保持 Linux 的通用功能。

#### **即時擴展理念**

即時 Linux 擴展遵循**最小修改原則**——以最少的核心修改來新增即時功能，確保相容性和可維護性。

**擴展設計目標：**

- **相容性**：與現有 Linux 應用程式協同工作
- **效能**：即時操作的最小額外負擔
- **可靠性**：在即時負載下維持系統穩定
- **可維護性**：易於與核心更新整合
- **彈性**：支援多樣的即時需求

#### **可用的即時解決方案**

**1. PREEMPT_RT 補丁：**
- **類型**：核心補丁
- **方法**：修改 Linux 核心以支援搶占
- **優點**：原生 Linux，良好的相容性
- **缺點**：整合複雜，維護負擔

**2. Xenomai 框架：**
- **類型**：雙核心架構
- **方法**：與 Linux 並行的協同核心
- **優點**：優秀的即時效能，成熟
- **缺點**：獨立的 API，學習曲線

**3. RTAI（即時應用介面）：**
- **類型**：雙核心架構
- **方法**：與 Linux 並行的協同核心
- **優點**：高效能，學術背景支持
- **缺點**：社群支援有限

---

## ⚡ **PREEMPT_RT 補丁**

### **使 Linux 可搶占**

PREEMPT_RT 補丁將 Linux 轉變為完全可搶占的核心，允許高優先權的即時任務中斷核心操作。

#### **PREEMPT_RT 理念**

PREEMPT_RT 遵循**核心搶占原則**——使 Linux 核心完全可搶占，以降低最壞情況延遲並改善即時回應能力。

**PREEMPT_RT 目標：**

- **降低延遲**：最小化最壞情況回應時間
- **核心搶占**：允許即時任務搶占核心
- **優先權繼承**：防止優先權反轉
- **鎖轉換**：將自旋鎖轉換為互斥鎖
- **中斷執行緒化**：在核心執行緒中處理中斷

#### **PREEMPT_RT 實作**

**核心配置：**
```bash
# 在核心配置中啟用 PREEMPT_RT
CONFIG_PREEMPT_RT=y
CONFIG_PREEMPT=y
CONFIG_PREEMPT_COUNT=y
CONFIG_DEBUG_PREEMPT=y
```

**鎖轉換：**
```c
// 標準 Linux：自旋鎖
spinlock_t lock;
spin_lock(&lock);
// 臨界區段
spin_unlock(&lock);

// PREEMPT_RT：互斥鎖
struct rt_mutex lock;
rt_mutex_lock(&lock);
// 臨界區段
rt_mutex_unlock(&lock);
```

**中斷執行緒化：**
```c
// 標準 Linux：中斷處理程式
irqreturn_t irq_handler(int irq, void *dev_id) {
    // 立即處理中斷
    return IRQ_HANDLED;
}

// PREEMPT_RT：執行緒化中斷
irqreturn_t irq_handler(int irq, void *dev_id) {
    // 排程工作至稍後處理
    return IRQ_WAKE_THREAD;
}

irqreturn_t irq_thread(int irq, void *dev_id) {
    // 在執行緒上下文中處理中斷
    return IRQ_HANDLED;
}
```

---

## 🚀 **Xenomai 框架**

### **雙核心即時解決方案**

Xenomai 提供協同核心架構，即時核心與 Linux 並行運行，在保持 Linux 相容性的同時提供優秀的即時效能。

#### **Xenomai 理念**

Xenomai 遵循**雙核心原則**——分離即時和通用核心，以在兩個領域都達到最佳效能。

**Xenomai 設計目標：**

- **即時效能**：次微秒等級的延遲
- **Linux 相容性**：運行標準 Linux 應用程式
- **API 彈性**：多種即時 API
- **資源共享**：核心間的高效共享
- **開發支援**：豐富的開發工具

#### **Xenomai 架構**

```
┌─────────────────────────────────────┐
│         使用者應用程式               │
├─────────────────────────────────────┤
│      即時應用程式                    │
│      （Xenomai API）                │
├─────────────────────────────────────┤
│         Linux 應用程式              │
│      （POSIX、系統呼叫）            │
├─────────────────────────────────────┤
│         Xenomai 協同核心            │
│      （即時排程器）                  │
├─────────────────────────────────────┤
│         Linux 核心                  │
│      （通用）                       │
├─────────────────────────────────────┤
│         硬體層                      │
└─────────────────────────────────────┘
```

**Xenomai API：**

**1. 原生 API：**
```c
#include <native/task.h>
#include <native/timer.h>

RT_TASK task_desc;
RT_TIMER timer_desc;

void real_time_task(void *arg) {
    // 即時任務程式碼
    rt_task_sleep(1000000);  // 1ms 休眠
}

int main() {
    // 建立即時任務
    rt_task_create(&task_desc, "rt_task", 0, 99, T_CPU(0));
    rt_task_start(&task_desc, &real_time_task, NULL);
    
    // 啟動 Xenomai
    rt_task_shutdown();
    return 0;
}
```

**2. POSIX API：**
```c
#include <pthread.h>
#include <sched.h>
#include <time.h>

void *real_time_thread(void *arg) {
    struct timespec ts;
    clock_gettime(CLOCK_MONOTONIC, &ts);
    
    // 即時執行緒程式碼
    ts.tv_nsec += 1000000;  // 1ms
    clock_nanosleep(CLOCK_MONOTONIC, TIMER_ABSTIME, &ts, NULL);
    
    return NULL;
}

int main() {
    pthread_t thread;
    struct sched_param param;
    
    param.sched_priority = 99;
    pthread_create(&thread, NULL, real_time_thread, NULL);
    pthread_setschedparam(thread, SCHED_FIFO, &param);
    
    pthread_join(thread, NULL);
    return 0;
}
```

---

## ⏱️ **即時程式設計**

### **撰寫即時應用程式**

即時程式設計需要仔細注意時序、資源管理和系統行為。理解即時程式設計原則對於建構可靠的即時應用程式至關重要。

#### **即時程式設計理念**

即時程式設計遵循**確定性執行原則**——透過精心設計、資源管理和系統理解來確保可預測的時序行為。

**即時程式設計目標：**

- **可預測的時序**：一致的回應時間
- **資源管理**：有效使用系統資源
- **錯誤處理**：優雅地處理時序違規
- **測試**：全面的時序驗證
- **文件**：清楚的時序需求與約束

#### **即時任務設計**

**任務結構：**
```c
#include <native/task.h>
#include <native/timer.h>
#include <native/mutex.h>

RT_TASK task_desc;
RT_MUTEX mutex_desc;
RT_TIMER timer_desc;

// 即時任務函式
void real_time_task(void *arg) {
    RTIME start_time, current_time;
    int iteration = 0;
    
    // 將任務設定為週期模式
    rt_task_set_periodic(NULL, TM_NOW, 1000000);  // 1ms 週期
    
    while (1) {
        start_time = rt_timer_read();
        
        // 取得互斥鎖
        rt_mutex_acquire(&mutex_desc, TM_INFINITE);
        
        // 臨界區段
        // 執行即時操作
        
        // 釋放互斥鎖
        rt_mutex_release(&mutex_desc);
        
        // 等待下一個週期
        rt_task_wait_period(NULL);
        
        // 檢查時序
        current_time = rt_timer_read();
        if (current_time - start_time > 500000) {  // 500μs
            rt_printf("時序違規：%lld ns\n", 
                     current_time - start_time);
        }
        
        iteration++;
    }
}

// 主函式
int main() {
    // 初始化 Xenomai
    rt_print_auto_init(1);
    
    // 建立互斥鎖
    rt_mutex_create(&mutex_desc, "rt_mutex");
    
    // 建立即時任務
    rt_task_create(&task_desc, "rt_task", 0, 99, T_CPU(0));
    rt_task_start(&task_desc, &real_time_task, NULL);
    
    // 等待使用者輸入
    printf("按 Enter 鍵退出\n");
    getchar();
    
    // 清理
    rt_task_delete(&task_desc);
    rt_mutex_delete(&mutex_desc);
    
    return 0;
}
```

**優先權管理：**
```c
#include <native/task.h>
#include <native/sem.h>

RT_TASK high_priority_task;
RT_TASK low_priority_task;
RT_SEM semaphore;

// 高優先權任務
void high_priority_handler(void *arg) {
    while (1) {
        // 等待號誌
        rt_sem_p(&semaphore, TM_INFINITE);
        
        // 處理高優先權事件
        rt_printf("高優先權任務正在執行\n");
        
        // 模擬工作
        rt_task_sleep(100000);  // 100μs
        
        rt_task_wait_period(NULL);
    }
}

// 低優先權任務
void low_priority_handler(void *arg) {
    while (1) {
        // 執行背景工作
        rt_printf("低優先權任務正在執行\n");
        
        // 模擬工作
        rt_task_sleep(1000000);  // 1ms
        
        rt_task_wait_period(NULL);
    }
}

int main() {
    // 建立號誌
    rt_sem_create(&semaphore, "rt_sem", 0, S_FIFO);
    
    // 建立高優先權任務
    rt_task_create(&high_priority_task, "high_task", 0, 99, T_CPU(0));
    rt_task_set_periodic(&high_priority_task, TM_NOW, 1000000);
    rt_task_start(&high_priority_task, &high_priority_handler, NULL);
    
    // 建立低優先權任務
    rt_task_create(&low_priority_task, "low_task", 0, 50, T_CPU(0));
    rt_task_set_periodic(&low_priority_task, TM_NOW, 5000000);
    rt_task_start(&low_priority_task, &low_priority_handler, NULL);
    
    // 定期發送訊號給高優先權任務
    while (1) {
        rt_sem_v(&semaphore);
        rt_task_sleep(2000000);  // 2ms
    }
    
    return 0;
}
```

---

## 📊 **效能與延遲**

### **測量即時效能**

即時效能透過延遲、抖動和吞吐量來衡量。了解如何測量和最佳化這些指標對於建構高效能即時系統至關重要。

#### **效能指標**

**延遲指標：**
- **回應時間**：從事件到回應的時間
- **中斷延遲**：從中斷到處理程式的時間
- **排程延遲**：從就緒到執行的時間
- **上下文切換**：任務切換所需時間

**抖動指標：**
- **時序抖動**：回應時間的變異
- **週期抖動**：任務週期的變異
- **執行抖動**：執行時間的變異

#### **延遲測量**

**中斷延遲測量：**
```c
#include <native/task.h>
#include <native/timer.h>
#include <native/irq.h>

RT_TASK measurement_task;
RT_TIMER measurement_timer;
volatile RTIME interrupt_time, task_time;

// 中斷處理程式
void irq_handler(int irq, void *dev_id) {
    interrupt_time = rt_timer_read();
}

// 測量任務
void measurement_handler(void *arg) {
    RTIME latency;
    
    while (1) {
        // 等待計時器中斷
        rt_task_sleep(1000000);  // 1ms
        
        // 計算延遲
        latency = task_time - interrupt_time;
        
        rt_printf("中斷延遲：%lld ns\n", latency);
        
        rt_task_wait_period(NULL);
    }
}

// 計時器回呼
void timer_callback(void *arg) {
    task_time = rt_timer_read();
}

int main() {
    // 建立測量任務
    rt_task_create(&measurement_task, "measure", 0, 99, T_CPU(0));
    rt_task_set_periodic(&measurement_task, TM_NOW, 1000000);
    rt_task_start(&measurement_task, &measurement_handler, NULL);
    
    // 建立計時器
    rt_timer_create(&measurement_timer, "measure_timer", 
                   TM_NOW, 1000000, TM_PERIODIC, &timer_callback, NULL);
    
    // 等待使用者輸入
    printf("按 Enter 鍵退出\n");
    getchar();
    
    // 清理
    rt_task_delete(&measurement_task);
    rt_timer_delete(&measurement_timer);
    
    return 0;
}
```

---

## 🛡️ **即時最佳實踐**

### **建構可靠的即時系統**

即時系統需要精心的設計和實作以確保可靠運作。遵循最佳實踐對於建構健壯的即時應用程式至關重要。

#### **設計原則**

**1. 最小化中斷處理：**
```c
// 良好：最小化中斷處理程式
irqreturn_t irq_handler(int irq, void *dev_id) {
    // 僅執行必要操作
    schedule_work(&deferred_work);
    return IRQ_HANDLED;
}

// 不良：複雜的中斷處理程式
irqreturn_t irq_handler(int irq, void *dev_id) {
    // 在中斷上下文中進行複雜處理
    process_data();
    update_display();
    send_network_packet();
    return IRQ_HANDLED;
}
```

**2. 使用適當的排程：**
```c
// 良好：即時排程
struct sched_param param;
param.sched_priority = 99;
pthread_setschedparam(thread, SCHED_FIFO, &param);

// 不良：預設排程
// 使用 SCHED_OTHER（分時排程）
```

**3. 資源管理：**
```c
// 良好：預先分配資源
static char buffer[1024];
static RT_MUTEX buffer_mutex;

// 不良：在即時上下文中動態分配
char *buffer = malloc(1024);  // 可能阻塞
```

#### **測試與驗證**

**延遲測試：**
```c
#include <native/task.h>
#include <native/timer.h>
#include <native/mutex.h>

RT_TASK test_task;
RT_MUTEX test_mutex;
RT_TIMER test_timer;

volatile RTIME max_latency = 0;
volatile RTIME min_latency = 1000000000;
volatile int test_count = 0;

void latency_test(void *arg) {
    RTIME start_time, end_time, latency;
    
    while (test_count < 10000) {
        start_time = rt_timer_read();
        
        // 取得互斥鎖
        rt_mutex_acquire(&test_mutex, TM_INFINITE);
        
        // 模擬工作
        rt_task_sleep(1000);  // 1μs
        
        // 釋放互斥鎖
        rt_mutex_release(&test_mutex);
        
        end_time = rt_timer_read();
        latency = end_time - start_time;
        
        // 更新統計資料
        if (latency > max_latency) max_latency = latency;
        if (latency < min_latency) min_latency = latency;
        
        test_count++;
        
        rt_task_wait_period(NULL);
    }
    
    rt_printf("延遲測試完成\n");
    rt_printf("最小延遲：%lld ns\n", min_latency);
    rt_printf("最大延遲：%lld ns\n", max_latency);
}

int main() {
    // 建立測試任務
    rt_task_create(&test_task, "test", 0, 99, T_CPU(0));
    rt_task_set_periodic(&test_task, TM_NOW, 1000000);
    rt_task_start(&test_task, &latency_test, NULL);
    
    // 等待完成
    while (test_count < 10000) {
        rt_task_sleep(100000);  // 100ms
    }
    
    rt_task_delete(&test_task);
    return 0;
}
```

---

## 🎯 **結論**

即時 Linux 為建構確定性嵌入式系統提供了強大的能力。理解 PREEMPT_RT、Xenomai 和即時程式設計原則對於建立可靠的即時應用程式至關重要。

**重點摘要：**

- **即時系統需要確定性時序**和有界延遲
- **PREEMPT_RT 使 Linux 可搶占**以降低延遲
- **Xenomai 提供雙核心架構**以達到優秀的即時效能
- **即時程式設計需要精心設計**和資源管理
- **效能測量與測試**對於驗證至關重要
- **最佳實踐確保可靠運作**在即時約束下

**未來展望：**

隨著嵌入式系統變得更加複雜且即時需求日益嚴格，即時 Linux 技能的重要性只會持續增加。現代系統持續演進，提供新的即時功能和最佳化技術。

**請記住**：即時 Linux 不僅僅是在即時環境中運行 Linux——而是理解如何設計、實作和驗證滿足嚴格時序要求的系統。您在此培養的技能將使您能夠建立健壯、可靠且高效能的即時嵌入式系統。
