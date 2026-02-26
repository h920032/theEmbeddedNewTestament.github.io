## Linux 開機程序

![Linux-boot-process](../images/linux-boot-process.png)

### 1. BIOS

* BIOS 代表基本輸入／輸出系統（Basic Input/Output System）
* 執行一些系統完整性檢查
* 搜尋、載入並執行開機載入器程式。
* 它在軟碟、光碟機或硬碟中尋找開機載入器。你可以在 BIOS 啟動期間按下一個按鍵（通常是 F12 或 F2，但取決於你的系統）來更改開機順序。
* 一旦開機載入器程式被偵測到並載入記憶體中，BIOS 就將控制權交給它。

簡單來說，BIOS 載入並執行 MBR 開機載入器。

### 2. MBR

* MBR 代表主開機紀錄（Master Boot Record）。
* 它位於可開機磁碟的第一個磁區。通常是 /dev/hda 或 /dev/sda
* MBR 大小不到 512 位元組。它有三個組成部分：1) 前 446 位元組為主要開機載入器資訊 2) 接下來 64 位元組為分割區表資訊 3) 最後 2 位元組為 MBR 驗證檢查。
* 它包含關於 GRUB（或舊系統中的 LILO）的資訊。

簡單來說，MBR 載入並執行 GRUB 開機載入器。

### 3. GRUB
```shell
#boot=/dev/sda
default=0
timeout=5
splashimage=(hd0,0)/boot/grub/splash.xpm.gz
hiddenmenu
title CentOS (2.6.18-194.el5PAE)
          root (hd0,0)
          kernel /boot/vmlinuz-2.6.18-194.el5PAE ro root=LABEL=/
          initrd /boot/initrd-2.6.18-194.el5PAE.img
```
* GRUB 代表統一開機載入器（Grand Unified Bootloader）。
* 如果你的系統安裝了多個核心映像，你可以選擇要執行哪一個。
* GRUB 顯示啟動畫面，等待幾秒鐘，如果你沒有輸入任何內容，它會載入 GRUB 組態檔中指定的預設核心映像。
* GRUB 了解檔案系統（較舊的 Linux 載入器 LILO 不了解檔案系統）。
* GRUB 組態檔是 /boot/grub/grub.conf（/etc/grub.conf 是指向此檔案的連結）。以下是 CentOS 的範例 grub.conf。
* 如你從上述資訊所注意到的，它包含核心和 initrd 映像。

簡單來說，GRUB 只是載入並執行核心和 initrd 映像。

### 4. 核心（Kernel）

* 按照 grub.conf 中 "root=" 指定的方式掛載根檔案系統
* 核心執行 /sbin/init 程式
* 由於 init 是 Linux 核心執行的第一個程式，它的程序 ID（PID）為 1。執行 'ps -ef | grep init' 並檢查 PID。
* initrd 代表初始 RAM 磁碟（Initial RAM Disk）。
* initrd 被核心用作臨時根檔案系統，直到核心開機完成並掛載真正的根檔案系統為止。它還包含編譯在內的必要驅動程式，幫助它存取硬碟分割區和其他硬體。

### 5. Init

* 查看 /etc/inittab 檔案以決定 Linux 執行級別。
* 以下是可用的執行級別
```
    0 – 關機（halt）
    1 – 單一使用者模式
    2 – 多使用者模式，不含 NFS
    3 – 完整多使用者模式
    4 – 未使用
    5 – X11
    6 – 重新開機（reboot）
```
* Init 從 /etc/inittab 識別預設的初始級別，並使用它來載入所有適當的程式。
* 在你的系統上執行 'grep initdefault /etc/inittab' 以識別預設的執行級別
* 如果你想惹麻煩，可以將預設執行級別設為 0 或 6。既然你知道 0 和 6 代表什麼，你大概不會這麼做。
* 通常你會將預設執行級別設為 3 或 5。

### 6. 執行級別程式

* 當 Linux 系統開機時，你可能會看到各種服務正在啟動。例如，它可能顯示 "starting sendmail …. OK"。這些就是執行級別程式，從你的執行級別所定義的執行級別目錄中執行。
* 根據你的預設 init 級別設定，系統將從以下其中一個目錄執行程式。
```
執行級別 0 – /etc/rc.d/rc0.d/
執行級別 1 – /etc/rc.d/rc1.d/
執行級別 2 – /etc/rc.d/rc2.d/
執行級別 3 – /etc/rc.d/rc3.d/
執行級別 4 – /etc/rc.d/rc4.d/
執行級別 5 – /etc/rc.d/rc5.d/
執行級別 6 – /etc/rc.d/rc6.d/
```
* 請注意，在 /etc 下也有這些目錄的符號連結。所以 /etc/rc0.d 連結到 /etc/rc.d/rc0.d。
* 在 /etc/rc.d/rc*.d/ 目錄下，你會看到以 S 和 K 開頭的程式。
* 以 S 開頭的程式在啟動時使用。S 代表啟動（startup）。
* 以 K 開頭的程式在關機時使用。K 代表終止（kill）。
* 程式名稱中 S 和 K 旁邊有數字。這些是程式應該啟動或終止的順序編號。
* 例如，S12syslog 是啟動 syslog 守護程序，其順序編號為 12。S80sendmail 是啟動 sendmail 守護程序，其順序編號為 80。所以 syslog 程式會在 sendmail 之前啟動。

以上就是 Linux 開機程序中發生的事情。

## [Linux 開機程序的所有階段](https://github.com/nu11secur1ty/All-Stages-of-Linux-Booting-Process-)

這個 GitHub 儲存庫提供了更多關於 Linux 開機程序的詳細資訊。

## 參考資料

https://leetcode.com/discuss/interview-question/124638/what-happens-in-the-background-from-the-time-you-press-the-Power-button-until-the-Linux-login-prompt-appears/
