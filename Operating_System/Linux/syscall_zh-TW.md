## Linux 系統呼叫機制

### 為什麼我們需要系統呼叫？

系統呼叫是使用者空間應用程式用來向作業系統核心請求服務的方法。這是因為使用者空間應用程式由於權限差異，無法直接存取作業系統核心中的資源。

- 系統呼叫基本上是使用者應用程式與作業系統核心服務互動的 API。
- 可請求的作業系統核心服務包括：裝置 I/O、程序建立、硬體存取、記憶體配置等。
- 系統呼叫產生_軟體中斷_，使 CPU 從__使用者模式__轉換到__核心模式__。每個系統呼叫都有自己的系統呼叫編號，在軟體中斷 ISR 常式中進行處理。
- 系統呼叫的存在是為了保護核心空間，使使用者空間應用程式無法直接干擾系統資源，防止惡意嘗試修改或損壞系統。

系統呼叫層級圖：

![syscall layer](../images/syscall.png)

使用者空間程序請求作業系統服務：

![syscall execution](../images/syscall_2.png)

### 以 kill() 系統呼叫為例：

- 使用者空間方法 XXX，其對應的系統呼叫層方法為 sys_XXX。例如 kill() -> sys_kill()。
- unistd.h（_/kernel/include/uapi/asm-generic/unistd.h_）包含所有系統呼叫軟體中斷編號資訊。
- 

### 參考資料

https://www.slideshare.net/VandanaSalve/introduction-to-char-device-driver

https://www.slideshare.net/garyyeh165/linux-char-device-driver 

https://www.ptt.cc/bbs/b97902HW/M.1268953601.A.BA9.html

https://www.ptt.cc/bbs/b97902HW/M.1268932130.A.0CF.html

http://gityuan.com/2016/05/21/syscall/

http://hwchiu.logdown.com/posts/1733-c-pipe

http://wiki.csie.ncku.edu.tw/embedded/ARMv8

http://linux.vbird.org/linux_basic/0440processcontrol.php

_Advanced Programming in the UNIX Environment 3rd Edition_
