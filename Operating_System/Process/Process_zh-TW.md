## 作業系統中的程序管理

[***Linux 程序***](https://www.bogotobogo.com/Linux/linux_process_and_signals.php)

這篇來自 [bogotobogo.com](bogotobogo.com) 的部落格詳細描述了 Linux 作業系統中的程序。強烈推薦。

### 程序內容

[***程序、位址空間與上下文切換***](http://www.cse.iitm.ac.in/~chester/courses/15o_os/slides/6_Processes.pdf)

來自 IIT 的深入投影片，關於程序的介紹。

### 程序控制區塊 (PCB)

### 上下文切換

上下文切換的額外開銷
- 影響上下文切換時間的直接因素
  - 計時器中斷延遲
  - 儲存/恢復上下文
  - 尋找下一個要執行的程序
- 間接因素
  - TLB 需要重新載入
  - 快取區域性喪失（因此產生更多快取未命中）
  - 處理器管線清空

## 資源

[NYU CS 2250 程序管理課程教材](https://cs.nyu.edu/~gottlieb/courses/2000s/2000-01-spring/os/chapters/chapter-2.html)

[Utexas CS372 投影片](https://www.cs.utexas.edu/~lorenzo/corsi/cs372/03F/notes/9-9.pdf)

[作業系統筆記：程序管理](https://applied-programming.github.io/Operating-Systems-Notes/2-Process-Management/)

[作業系統學習指南：程序管理](http://faculty.salina.k-state.edu/tim/ossg/Process/process.html)

[作業系統程序管理](https://www.studytonight.com/operating-system/operating-system-processes)

[程序如何運作](https://www.usna.edu/Users/cs/crabbe/SI411/current/processes/processes.html)

[北佛羅里達大學投影片](https://www.unf.edu/public/cop4610/ree/Notes/PPT/PPT8E/CH%2003%20-OS8e.pdf)

[程序、位址空間與上下文切換 IIT Madras 投影片](http://www.cse.iitm.ac.in/~chester/courses/15o_os/slides/6_Processes.pdf)

[UIUC CS 241 高品質課程教材](https://courses.engr.illinois.edu/cs241/sp2012/) 
