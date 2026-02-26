# 系統程式設計

> **掌握應用程式與作業系統之間的介面**  
> 理解 POSIX API、系統呼叫與信號處理，構建穩健的嵌入式應用程式

---

## 📋 **目錄**

- [系統程式設計基礎](#系統程式設計基礎)
- [POSIX API 與標準](#posix-api-與標準)
- [系統呼叫介面](#系統呼叫介面)
- [信號處理](#信號處理)
- [檔案與 I/O 操作](#檔案與-io-操作)
- [程序與執行緒控制](#程序與執行緒控制)
- [記憶體管理 API](#記憶體管理-api)
- [進階系統程式設計](#進階系統程式設計)

---

## 🏗️ **系統程式設計基礎**

### **什麼是系統程式設計？**

系統程式設計涉及撰寫直接與作業系統核心互動的軟體，以存取硬體資源和系統服務。與應用程式設計不同，系統程式設計在較低層級運作，需要深入理解作業系統的架構和行為。

**系統程式設計在嵌入式系統中的角色：**

- **硬體存取**：直接控制硬體資源
- **效能最佳化**：繞過高階抽象層以提升效率
- **系統整合**：與核心服務和驅動程式互動
- **資源管理**：控制記憶體、CPU 和 I/O 配置
- **即時需求**：滿足嚴格的時序和回應要求

#### **系統程式設計 vs. 應用程式設計**

理解系統程式設計與應用程式設計之間的區別對於嵌入式開發至關重要：

**應用程式設計：**
- **層級**：使用者空間，高階抽象
- **重點**：商業邏輯、使用者介面、資料處理
- **安全性**：受核心保護，無法使系統崩潰
- **效能**：中等，具有核心開銷
- **複雜度**：較低，擁有豐富的函式庫支援

**系統程式設計：**
- **層級**：核心空間或特權使用者空間
- **重點**：系統服務、硬體控制、效能
- **安全性**：可能影響系統穩定性，需要謹慎設計
- **效能**：高，可直接存取硬體
- **複雜度**：較高，需要手動管理資源

```
┌─────────────────────────────────────┐
│           應用程式層                │ ← 高階抽象
│         （使用者程式）              │   - 函式庫與框架
│                                    │   - 商業邏輯
├─────────────────────────────────────┤
│          系統程式設計               │ ← 低階系統介面
│      （系統呼叫、API）             │   - 核心服務
│                                    │   - 硬體抽象
├─────────────────────────────────────┤
│           作業系統                  │ ← 核心與驅動程式
│         （核心空間）                │   - 資源管理
│                                    │   - 硬體控制
├─────────────────────────────────────┤
│           硬體層                    │ ← 實體硬體
│     （CPU、記憶體、I/O）           │   - 原始硬體資源
└─────────────────────────────────────┘
```

#### **系統程式設計哲學**

系統程式設計遵循**效率與可靠性原則**——撰寫既高效能又穩健的程式碼，並仔細注意資源管理和錯誤處理。

**設計原則：**

- **效率**：最小化開銷並最大化效能
- **可靠性**：優雅且可預測地處理錯誤
- **可攜性**：撰寫可在不同系統上運作的程式碼
- **可維護性**：建立清晰、文件完善的介面
- **安全性**：遵循安全性最佳實務與原則

---

## 📚 **POSIX API 與標準**

### **可攜式作業系統介面**

POSIX（可攜式作業系統介面）定義了一組標準 API，確保應用程式在不同類 Unix 作業系統之間的可攜性。理解 POSIX 對於撰寫可攜式嵌入式應用程式至關重要。

#### **POSIX 標準哲學**

POSIX 遵循**可攜性原則**——定義在不同作業系統之間一致運作的介面，同時允許系統特定的最佳化和擴充。

**POSIX 設計目標：**

- **可攜性**：程式碼可在不同 POSIX 相容系統上運作
- **一致性**：相似的函式在各處以相同方式運作
- **效率**：API 為效能而設計
- **可擴充性**：系統可在維持相容性的前提下新增功能
- **向後相容性**：舊程式碼繼續運作

#### **核心 POSIX 標準**

**POSIX.1（核心服務）：**
- **檔案操作**：開啟、讀取、寫入、關閉檔案
- **程序管理**：建立、終止、等待程序
- **信號**：處理非同步事件
- **記憶體管理**：配置和管理記憶體
- **行程間通訊**：管道、訊息佇列、共享記憶體

**POSIX.1b（即時擴充）：**
- **即時排程**：基於優先權的排程策略
- **計時器**：高精度計時器和時鐘
- **同步 I/O**：阻塞式 I/O 操作
- **記憶體鎖定**：將記憶體鎖定在實體 RAM 中
- **訊息佇列**：即時訊息傳遞

**POSIX.1c（執行緒）：**
- **執行緒建立**：建立和管理執行緒
- **執行緒同步**：互斥鎖、條件變數、號誌
- **執行緒特定資料**：每個執行緒的儲存空間
- **執行緒取消**：優雅地終止執行緒
- **執行緒排程**：執行緒層級的排程控制

#### **POSIX 相容性層級**

不同系統提供不同程度的 POSIX 相容性：

```
┌─────────────────────────────────────┐
│       完全 POSIX 相容              │ ← 完整標準支援
│       （Linux、FreeBSD）           │   - 所有必要 API
│                                    │   - 可選擴充
├─────────────────────────────────────┤
│       部分 POSIX 相容              │ ← 僅核心功能
│    （macOS、Windows 子系統）       │   - 基本 API
│                                    │   - 部分擴充缺失
├─────────────────────────────────────┤
│       最低 POSIX 支援              │ ← 基本相容性
│       （嵌入式 RTOS）              │   - 僅核心 API
│                                    │   - 功能有限
└─────────────────────────────────────┘
```

---

## 🔧 **系統呼叫介面**

### **使用者空間與核心空間之間的橋樑**

系統呼叫提供使用者空間應用程式向核心請求服務的基本機制。理解系統呼叫的運作方式對於高效的系統程式設計至關重要。

#### **系統呼叫架構**

系統呼叫遵循一個明確定義的流程，涉及多個架構層級：

```
使用者應用程式
       │
       ▼
   函式庫函式（例如 printf）
       │
       ▼
   系統呼叫包裝函式
       │
       ▼
   系統呼叫編號（存於暫存器中）
       │
       ▼
   核心進入點
       │
       ▼
   系統呼叫處理器
       │
       ▼
   核心服務函式
       │
       ▼
   返回使用者空間
```

**系統呼叫流程：**

1. **使用者準備**：應用程式將系統呼叫編號和參數放入暫存器
2. **陷阱指令**：特殊指令觸發轉換到核心模式
3. **核心進入**：核心儲存使用者上下文並切換到核心模式
4. **參數驗證**：核心驗證所有參數的安全性
5. **服務執行**：核心執行請求的操作
6. **上下文恢復**：核心恢復使用者上下文並返回結果

#### **常見系統呼叫類別**

**程序管理：**
- **`fork()`**：建立新程序
- **`execve()`**：執行程式
- **`wait()`**：等待子程序
- **`exit()`**：終止程序
- **`getpid()`**：取得程序 ID

**檔案操作：**
- **`open()`**：開啟檔案或裝置
- **`read()`**：從檔案讀取資料
- **`write()`**：將資料寫入檔案
- **`close()`**：關閉檔案描述符
- **`lseek()`**：變更檔案位置

**記憶體管理：**
- **`brk()`**：變更資料區段大小
- **`mmap()`**：將檔案或裝置映射到記憶體
- **`munmap()`**：取消映射記憶體區域
- **`mprotect()`**：變更記憶體保護

**行程間通訊：**
- **`pipe()`**：建立管道用於 IPC
- **`shmget()`**：取得共享記憶體區段
- **`msgget()`**：取得訊息佇列
- **`semget()`**：取得號誌集合

#### **系統呼叫效能考量**

系統呼叫具有可能影響嵌入式系統效能的開銷：

**開銷來源：**
- **模式切換**：使用者模式到核心模式的轉換
- **上下文儲存/恢復**：儲存和恢復 CPU 暫存器
- **參數驗證**：核心驗證所有參數
- **記憶體存取**：跨使用者/核心邊界的資料存取
- **排程**：系統呼叫期間可能的上下文切換

**最佳化策略：**
- **批次操作**：將多個操作合併為單一系統呼叫
- **記憶體映射**：對大型 I/O 操作使用 `mmap()`
- **直接 I/O**：在適當時繞過核心緩衝
- **系統呼叫批次處理**：使用 `io_uring` 或類似機制
- **避免不必要的呼叫**：快取結果並最小化呼叫次數

---

## ⚡ **信號處理**

### **管理非同步事件**

信號提供核心通知程序非同步事件的機制。理解信號處理對於構建能夠回應外部事件和系統狀態的穩健嵌入式應用程式至關重要。

#### **信號哲學**

信號處理遵循**非同步事件原則**——提供一種機制，讓程序能夠回應在其正常執行流程之外發生的事件。

**信號設計目標：**

- **非同步**：處理在不可預測時間發生的事件
- **高效**：信號傳遞的最小開銷
- **彈性**：允許程序自訂信號處理
- **可靠**：確保信號傳遞給預期的程序
- **安全**：防止信號處理破壞程序狀態

#### **信號類型與分類**

**標準信號（1-31）：**
- **終止信號**：SIGTERM、SIGKILL、SIGQUIT
- **錯誤信號**：SIGSEGV、SIGBUS、SIGFPE、SIGILL
- **控制信號**：SIGINT、SIGTSTP、SIGCONT
- **警報信號**：SIGALRM、SIGVTALRM、SIGPROF
- **使用者信號**：SIGUSR1、SIGUSR2

**即時信號（32-63）：**
- **擴充範圍**：供應用程式使用的額外信號編號
- **佇列支援**：可將多個信號排入佇列
- **優先權**：比標準信號更高的優先權
- **自訂用途**：應用程式定義的用途

#### **信號處理實作**

信號處理涉及設定信號處理器和管理信號傳遞：

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>
#include <string.h>

// 用於信號處理的全域變數
volatile sig_atomic_t signal_received = 0;
volatile sig_atomic_t graceful_shutdown = 0;

// 優雅關閉的信號處理器
void signal_handler(int sig) {
    switch (sig) {
        case SIGINT:
            printf("收到 SIGINT（Ctrl+C）\n");
            graceful_shutdown = 1;
            break;
            
        case SIGTERM:
            printf("收到 SIGTERM\n");
            graceful_shutdown = 1;
            break;
            
        case SIGUSR1:
            printf("收到 SIGUSR1\n");
            signal_received = 1;
            break;
            
        case SIGALRM:
            printf("收到 SIGALRM（逾時）\n");
            signal_received = 1;
            break;
            
        default:
            printf("收到信號 %d\n", sig);
            break;
    }
}

// 設定信號處理
int setup_signal_handling(void) {
    struct sigaction sa;
    
    // 初始化信號動作結構
    memset(&sa, 0, sizeof(sa));
    sa.sa_handler = signal_handler;
    sa.sa_flags = SA_RESTART;  // 重新啟動被中斷的系統呼叫
    
    // 設定信號處理器
    if (sigaction(SIGINT, &sa, NULL) == -1) {
        perror("設定 SIGINT 處理器失敗");
        return -1;
    }
    
    if (sigaction(SIGTERM, &sa, NULL) == -1) {
        perror("設定 SIGTERM 處理器失敗");
        return -1;
    }
    
    if (sigaction(SIGUSR1, &sa, NULL) == -1) {
        perror("設定 SIGUSR1 處理器失敗");
        return -1;
    }
    
    if (sigaction(SIGALRM, &sa, NULL) == -1) {
        perror("設定 SIGALRM 處理器失敗");
        return -1;
    }
    
    printf("信號處理器設定成功\n");
    return 0;
}

// 主應用程式迴圈
int main() {
    // 設定信號處理
    if (setup_signal_handling() == -1) {
        exit(1);
    }
    
    printf("應用程式已啟動（PID：%d）\n", getpid());
    printf("傳送 SIGUSR1 觸發動作，SIGINT/SIGTERM 退出\n");
    
    // 主應用程式迴圈
    while (!graceful_shutdown) {
        // 檢查信號
        if (signal_received) {
            printf("正在處理信號...\n");
            signal_received = 0;
            
            // 執行信號特定的動作
            // 這可能涉及更新狀態、記錄日誌等
        }
        
        // 模擬應用程式工作
        sleep(1);
    }
    
    printf("已啟動優雅關閉\n");
    
    // 清理程式碼放在此處
    printf("應用程式已終止\n");
    
    return 0;
}
```

**信號處理最佳實務：**

- **保持處理器簡單**：信號處理器應該精簡且快速
- **使用 volatile 變數**：將信號旗標標記為 volatile 以確保正確存取
- **避免複雜操作**：不要在處理器中呼叫非可重入函式
- **處理所有信號**：為應用程式使用的所有信號設定處理器
- **優雅關閉**：在信號處理器中實作適當的清理

---

## 📁 **檔案與 I/O 操作**

### **管理資料輸入與輸出**

檔案與 I/O 操作構成大多數系統程式設計任務的基礎。理解如何高效地從檔案、裝置和網路連線讀取及寫入資料，對於嵌入式應用程式至關重要。

#### **I/O 哲學**

I/O 操作遵循**效率與可靠性原則**——提供快速、可靠的資料存取，同時優雅地處理錯誤並有效管理系統資源。

**I/O 設計目標：**

- **效能**：最小化 I/O 開銷並最大化吞吐量
- **可靠性**：確保資料完整性並優雅地處理錯誤
- **效率**：針對不同使用情境使用適當的 I/O 方法
- **可攜性**：在不同系統之間一致運作
- **可擴展性**：處理不同的資料大小和 I/O 模式

#### **檔案描述符管理**

檔案描述符是類 Unix 系統中 I/O 操作的基本抽象：

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <errno.h>

// 檔案描述符管理範例
int manage_file_descriptors(void) {
    int fd1, fd2, fd3;
    char buffer[1024];
    ssize_t bytes_read, bytes_written;
    
    // 開啟來源檔案以供讀取
    fd1 = open("source.txt", O_RDONLY);
    if (fd1 == -1) {
        perror("開啟 source.txt 失敗");
        return -1;
    }
    
    // 開啟目標檔案以供寫入
    fd2 = open("destination.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
    if (fd2 == -1) {
        perror("開啟 destination.txt 失敗");
        close(fd1);
        return -1;
    }
    
    // 開啟日誌檔案以供附加
    fd3 = open("operation.log", O_WRONLY | O_CREAT | O_APPEND, 0644);
    if (fd3 == -1) {
        perror("開啟 operation.log 失敗");
        close(fd1);
        close(fd2);
        return -1;
    }
    
    // 從來源複製資料到目標
    while ((bytes_read = read(fd1, buffer, sizeof(buffer))) > 0) {
        bytes_written = write(fd2, buffer, bytes_read);
        if (bytes_written == -1) {
            perror("寫入失敗");
            break;
        }
        
        // 記錄操作
        dprintf(fd3, "已複製 %zd 位元組\n", bytes_written);
    }
    
    if (bytes_read == -1) {
        perror("讀取失敗");
    }
    
    // 關閉所有檔案描述符
    close(fd1);
    close(fd2);
    close(fd3);
    
    return 0;
}
```

**檔案描述符最佳實務：**

- **檢查返回值**：始終檢查 I/O 函式的返回值
- **關閉描述符**：確保所有檔案描述符被正確關閉
- **錯誤處理**：優雅地處理 I/O 錯誤
- **資源限制**：注意系統對開啟檔案數的限制
- **描述符洩漏**：避免留下未關閉的檔案描述符

#### **進階 I/O 技術**

**記憶體映射 I/O：**
```c
#include <sys/mman.h>
#include <sys/stat.h>
#include <fcntl.h>

int memory_mapped_io_example(void) {
    int fd;
    struct stat sb;
    char *mapped_data;
    
    // 開啟檔案
    fd = open("large_file.txt", O_RDONLY);
    if (fd == -1) {
        perror("開啟檔案失敗");
        return -1;
    }
    
    // 取得檔案大小
    if (fstat(fd, &sb) == -1) {
        perror("取得檔案資訊失敗");
        close(fd);
        return -1;
    }
    
    // 將檔案映射到記憶體
    mapped_data = mmap(NULL, sb.st_size, PROT_READ, MAP_PRIVATE, fd, 0);
    if (mapped_data == MAP_FAILED) {
        perror("映射檔案失敗");
        close(fd);
        return -1;
    }
    
    // 在記憶體中處理資料
    printf("檔案已映射到記憶體，大小：%ld 位元組\n", sb.st_size);
    
    // 取消映射並關閉
    munmap(mapped_data, sb.st_size);
    close(fd);
    
    return 0;
}
```

**非阻塞 I/O：**
```c
#include <fcntl.h>

int set_nonblocking(int fd) {
    int flags = fcntl(fd, F_GETFL, 0);
    if (flags == -1) {
        perror("取得檔案旗標失敗");
        return -1;
    }
    
    flags |= O_NONBLOCK;
    if (fcntl(fd, F_SETFL, flags) == -1) {
        perror("設定非阻塞模式失敗");
        return -1;
    }
    
    return 0;
}
```

---

## 🔄 **程序與執行緒控制**

### **管理執行上下文**

程序與執行緒控制涉及建立、管理和協調多個執行上下文。這對於構建需要同時處理多個任務的嵌入式應用程式至關重要。

#### **程序控制哲學**

程序控制遵循**隔離與協調原則**——為不同任務提供獨立的執行上下文，同時實現受控的通訊和資源共享。

**程序控制目標：**

- **隔離性**：獨立的程序不能互相干擾
- **資源管理**：高效配置和共享系統資源
- **通訊**：實現受控的行程間通訊
- **排程**：協調多個程序的執行
- **安全性**：控制對系統資源的存取

#### **程序建立與管理**

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>
#include <sys/types.h>

// 程序建立與管理範例
int process_management_example(void) {
    pid_t pid1, pid2;
    int status1, status2;
    
    printf("父程序（PID：%d）啟動中\n", getpid());
    
    // 建立第一個子程序
    pid1 = fork();
    if (pid1 == -1) {
        perror("第一次 fork 失敗");
        return -1;
    }
    
    if (pid1 == 0) {
        // 第一個子程序
        printf("第一個子程序（PID：%d）已建立\n", getpid());
        
        // 模擬工作
        sleep(2);
        printf("第一個子程序完成中\n");
        exit(10);  // 以狀態碼 10 退出
    }
    
    // 建立第二個子程序
    pid2 = fork();
    if (pid2 == -1) {
        perror("第二次 fork 失敗");
        wait(&status1);  // 等待第一個子程序
        return -1;
    }
    
    if (pid2 == 0) {
        // 第二個子程序
        printf("第二個子程序（PID：%d）已建立\n", getpid());
        
        // 模擬工作
        sleep(1);
        printf("第二個子程序完成中\n");
        exit(20);  // 以狀態碼 20 退出
    }
    
    // 父程序等待兩個子程序完成
    printf("父程序正在等待子程序完成\n");
    
    waitpid(pid1, &status1, 0);
    printf("第一個子程序（PID：%d）已完成，狀態碼：%d\n", 
           pid1, WEXITSTATUS(status1));
    
    waitpid(pid2, &status2, 0);
    printf("第二個子程序（PID：%d）已完成，狀態碼：%d\n", 
           pid2, WEXITSTATUS(status2));
    
    printf("所有子程序已完成\n");
    return 0;
}
```

#### **執行緒建立與管理**

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

// 執行緒資料結構
struct thread_data {
    int thread_id;
    int work_duration;
    pthread_mutex_t *mutex;
};

// 執行緒函式
void *thread_function(void *arg) {
    struct thread_data *data = (struct thread_data *)arg;
    
    printf("執行緒 %d 啟動中\n", data->thread_id);
    
    // 模擬工作
    sleep(data->work_duration);
    
    // 臨界區段
    pthread_mutex_lock(data->mutex);
    printf("執行緒 %d 在臨界區段中\n", data->thread_id);
    sleep(1);  // 模擬臨界工作
    pthread_mutex_unlock(data->mutex);
    
    printf("執行緒 %d 完成中\n", data->thread_id);
    
    return (void *)(long)(data->thread_id * 100);
}

// 執行緒管理範例
int thread_management_example(void) {
    pthread_t threads[3];
    struct thread_data thread_data[3];
    pthread_mutex_t mutex;
    void *thread_result;
    int i;
    
    // 初始化互斥鎖
    if (pthread_mutex_init(&mutex, NULL) != 0) {
        perror("初始化互斥鎖失敗");
        return -1;
    }
    
    // 建立執行緒
    for (i = 0; i < 3; i++) {
        thread_data[i].thread_id = i;
        thread_data[i].work_duration = i + 1;
        thread_data[i].mutex = &mutex;
        
        if (pthread_create(&threads[i], NULL, thread_function, &thread_data[i]) != 0) {
            perror("建立執行緒失敗");
            return -1;
        }
    }
    
    // 等待執行緒完成
    for (i = 0; i < 3; i++) {
        if (pthread_join(threads[i], &thread_result) != 0) {
            perror("結合執行緒失敗");
            return -1;
        }
        
        printf("執行緒 %d 返回：%ld\n", i, (long)thread_result);
    }
    
    // 清理互斥鎖
    pthread_mutex_destroy(&mutex);
    
    return 0;
}
```

---

## 💾 **記憶體管理 API**

### **控制記憶體配置與存取**

記憶體管理 API 提供對記憶體配置、管理和保護方式的控制。理解這些 API 對於構建高效且可靠的嵌入式應用程式至關重要。

#### **記憶體管理哲學**

記憶體管理遵循**效率與安全原則**——提供快速、可靠的記憶體配置，同時防止記憶體相關錯誤並最佳化系統效能。

**記憶體管理目標：**

- **效率**：最小化記憶體配置開銷
- **安全性**：防止記憶體洩漏和緩衝區溢位
- **效能**：最佳化記憶體存取模式
- **彈性**：支援各種配置策略
- **可靠性**：優雅地處理配置失敗

#### **動態記憶體配置**

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>

// 帶錯誤處理的記憶體配置
void *safe_malloc(size_t size) {
    void *ptr = malloc(size);
    if (ptr == NULL) {
        fprintf(stderr, "記憶體配置失敗：%s\n", strerror(errno));
        exit(1);
    }
    return ptr;
}

// 記憶體配置範例
int memory_allocation_example(void) {
    int *int_array;
    char *string_buffer;
    struct {
        int id;
        char name[64];
        double value;
    } *data_array;
    
    // 配置整數陣列
    int_array = safe_malloc(100 * sizeof(int));
    for (int i = 0; i < 100; i++) {
        int_array[i] = i * i;
    }
    
    // 配置字串緩衝區
    string_buffer = safe_malloc(1024);
    strcpy(string_buffer, "Hello, World!");
    
    // 配置資料結構陣列
    data_array = safe_malloc(50 * sizeof(*data_array));
    for (int i = 0; i < 50; i++) {
        data_array[i].id = i;
        snprintf(data_array[i].name, sizeof(data_array[i].name), "Item_%d", i);
        data_array[i].value = i * 1.5;
    }
    
    // 使用已配置的記憶體
    printf("整數陣列[50] = %d\n", int_array[50]);
    printf("字串緩衝區：%s\n", string_buffer);
    printf("資料陣列[25]：id=%d, name=%s, value=%.1f\n",
           data_array[25].id, data_array[25].name, data_array[25].value);
    
    // 釋放已配置的記憶體
    free(int_array);
    free(string_buffer);
    free(data_array);
    
    return 0;
}
```

#### **記憶體映射與保護**

```c
#include <sys/mman.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

// 記憶體映射範例
int memory_mapping_example(void) {
    int fd;
    char *mapped_memory;
    size_t page_size = getpagesize();
    size_t map_size = page_size * 2;  // 映射 2 個分頁
    
    // 建立用於映射的暫存檔案
    fd = open("/tmp/memory_map", O_RDWR | O_CREAT | O_TRUNC, 0644);
    if (fd == -1) {
        perror("建立暫存檔案失敗");
        return -1;
    }
    
    // 將檔案擴展到所需大小
    if (ftruncate(fd, map_size) == -1) {
        perror("擴展檔案失敗");
        close(fd);
        return -1;
    }
    
    // 將檔案映射到記憶體
    mapped_memory = mmap(NULL, map_size, PROT_READ | PROT_WRITE, 
                        MAP_SHARED, fd, 0);
    if (mapped_memory == MAP_FAILED) {
        perror("映射記憶體失敗");
        close(fd);
        return -1;
    }
    
    // 將資料寫入映射記憶體
    strcpy(mapped_memory, "Hello from mapped memory!");
    printf("已寫入映射記憶體：%s\n", mapped_memory);
    
    // 將記憶體保護變更為唯讀
    if (mprotect(mapped_memory, map_size, PROT_READ) == -1) {
        perror("變更記憶體保護失敗");
    } else {
        printf("記憶體保護已變更為唯讀\n");
        
        // 嘗試寫入（這應該會失敗）
        strcpy(mapped_memory, "This should fail");
        printf("從記憶體讀取：%s\n", mapped_memory);
    }
    
    // 取消映射並清理
    munmap(mapped_memory, map_size);
    close(fd);
    unlink("/tmp/memory_map");
    
    return 0;
}
```

---

## 🚀 **進階系統程式設計**

### **超越基本系統介面**

進階系統程式設計涉及用於最佳化效能、管理複雜資源和構建穩健系統的精密技術。這些技術對於高效能嵌入式應用程式至關重要。

#### **進階 I/O 技術**

**使用 `io_uring` 的非同步 I/O：**
```c
#include <linux/io_uring.h>
#include <sys/syscall.h>
#include <unistd.h>
#include <fcntl.h>

// 簡化的 io_uring 範例（Linux 專用）
int io_uring_example(void) {
    struct io_uring ring;
    struct io_uring_sqe *sqe;
    struct io_uring_cqe *cqe;
    int fd, ret;
    
    // 初始化 io_uring
    ret = io_uring_queue_init(32, &ring, 0);
    if (ret < 0) {
        fprintf(stderr, "初始化 io_uring 失敗：%s\n", strerror(-ret));
        return -1;
    }
    
    // 開啟檔案進行 I/O
    fd = open("test_file.txt", O_RDWR | O_CREAT | O_TRUNC, 0644);
    if (fd == -1) {
        perror("開啟檔案失敗");
        io_uring_queue_exit(&ring);
        return -1;
    }
    
    // 提交寫入請求
    sqe = io_uring_get_sqe(&ring);
    if (!sqe) {
        fprintf(stderr, "取得 SQE 失敗\n");
        close(fd);
        io_uring_queue_exit(&ring);
        return -1;
    }
    
    char write_data[] = "Hello, io_uring!";
    io_uring_prep_write(sqe, fd, write_data, strlen(write_data), 0);
    
    // 提交請求
    ret = io_uring_submit(&ring);
    if (ret < 0) {
        fprintf(stderr, "提交請求失敗：%s\n", strerror(-ret));
        close(fd);
        io_uring_queue_exit(&ring);
        return -1;
    }
    
    // 等待完成
    ret = io_uring_wait_cqe(&ring, &cqe);
    if (ret < 0) {
        fprintf(stderr, "等待完成失敗：%s\n", strerror(-ret));
        close(fd);
        io_uring_queue_exit(&ring);
        return -1;
    }
    
    printf("寫入完成：%d 位元組\n", cqe->res);
    
    // 標記完成為已讀
    io_uring_cqe_seen(&ring, cqe);
    
    // 清理
    close(fd);
    io_uring_queue_exit(&ring);
    
    return 0;
}
```

#### **程序與執行緒同步**

**使用 futex 的進階同步：**
```c
#include <linux/futex.h>
#include <sys/syscall.h>
#include <unistd.h>
#include <stdint.h>

// 基於 futex 的互斥鎖實作
struct futex_mutex {
    int32_t value;
};

// futex 系統呼叫包裝函式
int futex(int *uaddr, int op, int val, const struct timespec *timeout,
          int *uaddr2, int val3) {
    return syscall(SYS_futex, uaddr, op, val, timeout, uaddr2, val3);
}

// 初始化 futex 互斥鎖
void futex_mutex_init(struct futex_mutex *mutex) {
    mutex->value = 0;
}

// 鎖定 futex 互斥鎖
void futex_mutex_lock(struct futex_mutex *mutex) {
    int32_t expected = 0;
    
    // 嘗試取得鎖
    while (!__sync_bool_compare_and_swap(&mutex->value, expected, 1)) {
        // 等待鎖變為可用
        futex(&mutex->value, FUTEX_WAIT, 1, NULL, NULL, 0);
        expected = 0;
    }
}

// 解鎖 futex 互斥鎖
void futex_mutex_unlock(struct futex_mutex *mutex) {
    int32_t expected = 1;
    
    // 釋放鎖
    if (__sync_bool_compare_and_swap(&mutex->value, expected, 0)) {
        // 喚醒等待中的執行緒
        futex(&mutex->value, FUTEX_WAKE, 1, NULL, NULL, 0);
    }
}
```

---

## 🎯 **結論**

系統程式設計為構建穩健、高效的嵌入式應用程式奠定了基礎，這些應用程式直接與作業系統互動。理解 POSIX API、系統呼叫和信號處理對於建立能夠高效管理系統資源並回應外部事件的應用程式至關重要。

**重要要點：**

- **POSIX 標準**確保跨不同系統的可攜性
- **系統呼叫**提供對核心服務的受控存取
- **信號處理**實現對非同步事件的回應
- **I/O 操作**構成資料處理的基礎
- **程序與執行緒控制**實現並行執行
- **記憶體管理 API**提供對資源配置的控制
- **進階技術**最佳化效能與可靠性

**未來之路：**

隨著嵌入式系統變得更加複雜，需要更精密的作業系統互動，系統程式設計技能的重要性只會持續增加。現代系統持續演進，提供新的 API 和技術，使更強大、更高效的嵌入式應用程式成為可能。

**請記住**：系統程式設計不僅僅是進行系統呼叫——而是理解作業系統的運作方式、如何高效管理系統資源，以及如何構建能夠可靠地與底層系統互動的應用程式。您在此培養的技能將貫穿您的嵌入式系統職業生涯，使您能夠建立穩健、高效且可維護的系統。
