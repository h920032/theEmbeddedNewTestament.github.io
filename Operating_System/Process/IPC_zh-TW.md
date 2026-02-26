## 行程間通訊

#### **行程間通訊的方法**
行程間通訊（IPC）是一組介面，通常透過程式設計使程式能夠在一系列程序之間進行通訊。這使得在作業系統中能夠同時執行多個程式。以下是 IPC 的方法：

- 管道（同一程序）
    - 這只允許資料單向流動。類似於單工系統（鍵盤）。來自輸出的資料通常會被緩衝，直到輸入程序接收它，且必須具有相同的來源。
    - [說明與範例](https://www.tutorialspoint.com/inter_process_communication/inter_process_communication_pipes.htm)

- 具名管道（不同程序）
    - 這是具有特定名稱的管道，可以在沒有共同程序來源的程序中使用。例如 FIFO，其中寫入管道的詳細資訊會先被命名。
    - [說明與範例](https://www.tutorialspoint.com/inter_process_communication/inter_process_communication_named_pipes.htm)

- 訊息佇列
    - 這允許透過單一佇列或多個訊息佇列在程序之間傳遞訊息。這由系統核心管理，這些訊息透過 API 進行協調。
    - [說明與範例](https://www.tutorialspoint.com/inter_process_communication/inter_process_communication_message_queues.htm)

- 號誌
    - 這用於解決與同步相關的問題並避免競爭條件。這些是大於或等於 0 的整數值。
    - [說明與範例](https://www.tutorialspoint.com/inter_process_communication/inter_process_communication_semaphores.htm)

- 共享記憶體
    - 這允許透過定義的記憶體區域來交換資料。在資料可以存取共享記憶體之前，必須先取得號誌值。
    - [說明與範例](https://www.tutorialspoint.com/inter_process_communication/inter_process_communication_semaphores.htm)

- 套接字
    - 這種方法主要用於在網路上進行客戶端和伺服器之間的通訊。它允許建立標準連線，且不依賴特定電腦和作業系統。

- 訊號
    - [說明與範例](https://www.tutorialspoint.com/inter_process_communication/inter_process_communication_signals.htm)

- 記憶體映射
    - [說明與範例](https://www.tutorialspoint.com/inter_process_communication/inter_process_communication_memory_mapping.htm)

#### 詳細說明
- [共享位址空間](https://courses.engr.illinois.edu/cs241/sp2012/lectures/29-IPC.pdf)（共享記憶體、記憶體映射檔案）
- [由作業系統從一個位址空間傳輸訊息到另一個位址空間](https://courses.engr.illinois.edu/cs241/sp2012/lectures/30-IPC.pdf)（檔案、管道與 FIFO）
- [一些 POSIX 範例](https://courses.engr.illinois.edu/cs241/fa2010/ppt/31-IPC.pdf)


#### 實作
[Brendan 的多工教學](https://wiki.osdev.org/Brendan%27s_Multi-tasking_Tutorial)

本教學將描述一種為使用「每個任務一個核心堆疊」的核心實作多工和任務切換的方法。它的設計允許讀者逐步實作功能完整的排程器（同時避免常見的陷阱），並可在進入下一步之前進行測試。
