
## 目錄
1.  使用者空間
2.  核心空間
3.  虛擬記憶體
4.  檔案系統
5.  排程
6.  中斷
    1. 什麼是中斷？
        中斷是由硬體或軟體向處理器發出的訊號，表示有一個需要立即處理的事件。

        中斷分為兩種類型：軟體中斷和硬體中斷。

        硬體中斷（外部中斷）可以分為兩類：
        可遮罩中斷和不可遮罩中斷。

        可遮罩中斷是可以被阻擋的中斷，通常由不太重要的周邊裝置（如印表機）所發出。不可遮罩中斷則必須由作業系統處理，例如磁碟讀取錯誤。

    2.  中斷操作期間會發生什麼事？
        每當中斷發生時，控制器會完成當前指令的執行，然後開始執行中斷服務常式（ISR）或中斷處理程式。ISR 告訴處理器或控制器在中斷發生時應該做什麼。中斷可以是硬體中斷或軟體中斷。

        ![alt text](http://hi.csdn.net/attachment/200910/18/10307_1255838664t2Or.jpg "中斷分類")

    3. 中斷是如何實作的？
        1. 上半部／下半部
        2. tasklet

        實用連結：

        [CPU 中斷機制](https://blog.csdn.net/qq_36894974/article/details/79172603)
        [中斷、例外與系統呼叫](http://www.cse.iitm.ac.in/~chester/courses/15o_os/slides/5_Interrupts.pdf)
7.  系統呼叫
8.  行程間通訊
9.  多行程／多執行緒
10. RTOS


## 參考資料

[Linux 核心中的中斷](https://linux-kernel-labs.github.io/refs/heads/master/lectures/interrupts.html#:~:text=In%20Linux%20the%20interrupt%20handling,interrupt%20and%20the%20interrupt%20controller.)

[作業系統運作基礎](http://faculty.salina.k-state.edu/tim/ossg/Introduction/OSworking.html)

[ARM cortex-M 中斷](https://www.youtube.com/watch?v=uFBNf7F3l60&list=PLRJhV4hUhIymmp5CCeIFPyxbknsdcXCc8&index=9&ab_channel=EmbeddedSystemswithARMCortex-MMicrocontrollersinAssemblyLanguageandC)
