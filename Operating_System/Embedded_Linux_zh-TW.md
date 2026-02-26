# 嵌入式 Linux

> **為嵌入式系統量身打造的 Linux**  
> 了解 Buildroot、Yocto 以及針對資源受限裝置的自訂發行版

---

## 📋 **目錄**

- [嵌入式 Linux 基礎](#嵌入式-linux-基礎)
- [建置系統理念](#建置系統理念)
- [Buildroot 框架](#buildroot-框架)
- [Yocto 專案](#yocto-專案)
- [自訂發行版開發](#自訂發行版開發)
- [系統最佳化](#系統最佳化)
- [部署與維護](#部署與維護)

---

## 🏗️ **嵌入式 Linux 基礎**

### **什麼是嵌入式 Linux？**

嵌入式 Linux 是指專為嵌入式系統設計的 Linux 發行版——這些裝置具有有限的資源、特定的硬體需求以及專門的功能。與桌面 Linux 不同，嵌入式 Linux 著重於效率、客製化和可靠性。

**嵌入式 Linux 的特性：**

- **資源最佳化**：最小化記憶體與儲存空間佔用
- **硬體專屬**：針對目標硬體架構量身打造
- **功能導向**：僅包含必要的元件
- **可靠性**：在嚴苛環境中穩定運作
- **可維護性**：易於在現場進行更新與維護

#### **嵌入式 Linux 與桌面 Linux 的比較**

**桌面 Linux：**
- **目標**：通用運算、使用者體驗
- **大小**：數 GB 的安裝空間
- **套件**：數千個可用套件
- **更新**：頻繁的、由使用者發起的更新
- **硬體**：通用 x86/ARM 支援

**嵌入式 Linux：**
- **目標**：特定應用需求
- **大小**：數 MB 到數百 MB
- **套件**：最少的、應用專屬的套件
- **更新**：受控的、可現場更新
- **硬體**：針對目標進行最佳化

```
┌─────────────────────────────────────┐
│        嵌入式 Linux 堆疊架構        │
├─────────────────────────────────────┤
│           應用程式層                │
│        （自訂應用程式）              │
├─────────────────────────────────────┤
│           系統服務層                │
│      （初始化系統、網路服務）         │
├─────────────────────────────────────┤
│          Linux 核心                 │
│       （針對目標最佳化）             │
├─────────────────────────────────────┤
│          開機載入器                  │
│     （U-Boot、GRUB 等）             │
├─────────────────────────────────────┤
│           硬體層                    │
│        （目標專屬硬體）              │
└─────────────────────────────────────┘
```

#### **嵌入式 Linux 的設計理念**

嵌入式 Linux 遵循**專用建置原則**——建立專門針對目標應用、硬體和運行需求而設計的 Linux 發行版。

**嵌入式 Linux 設計目標：**

- **效率**：在維持功能的同時最小化資源使用
- **可靠性**：確保在目標條件下穩定運作
- **客製化**：根據特定需求量身打造系統
- **可維護性**：支援現場更新與維護
- **效能**：針對目標工作負載和硬體進行最佳化

---

## 🔧 **建置系統理念**

### **理解建置系統**

嵌入式 Linux 的建置系統自動化了建立自訂發行版的過程。它們處理相依性解析、交叉編譯和系統整合，以產生可開機的映像檔。

#### **建置系統的設計理念**

建置系統遵循**自動化與可重現性原則**——自動化建置嵌入式 Linux 系統的複雜過程，同時確保在不同環境中獲得可重現的結果。

**建置系統目標：**

- **自動化**：減少手動配置與建置步驟
- **可重現性**：確保在不同環境中建置結果一致
- **相依性管理**：處理複雜的套件相依關係
- **交叉編譯**：支援為不同架構進行建置
- **整合**：整合核心、根檔案系統和開機載入器

#### **建置系統元件**

**核心元件：**
- **套件管理**：原始碼、修補檔和配置
- **建置環境**：交叉編譯工具鏈
- **相依性解析**：套件之間的關係與衝突
- **映像檔產生**：可開機的系統映像檔
- **配置管理**：系統與套件的配置

**建置流程：**
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│    原始碼       │───▶│    建置系統      │───▶│   系統映像檔    │
│   與修補檔      │    │     處理         │    │     產生        │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│    相依性       │    │   交叉編譯      │    │    可開機       │
│     解析        │    │    工具鏈       │    │    映像檔       │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

---

## 🚀 **Buildroot 框架**

### **簡單高效的建置系統**

Buildroot 是一個輕量級的建置系統，從原始碼建立嵌入式 Linux 系統。它的設計理念是簡單與高效，非常適合小型專案和快速原型開發。

#### **Buildroot 的設計理念**

Buildroot 遵循**簡單與高效原則**——提供一個直觀的建置系統，以最小的額外負擔產生精簡、高效的嵌入式 Linux 系統。

**Buildroot 設計目標：**

- **簡單性**：易於理解和使用
- **效率**：快速建置與最小的系統負擔
- **彈性**：支援各種架構與配置
- **極簡主義**：僅包含必要的元件
- **速度**：快速的迭代與測試週期

#### **Buildroot 架構**

**核心元件：**
```
┌─────────────────────────────────────┐
│          Buildroot 結構             │
├─────────────────────────────────────┤
│          Config.in                  │
│       （套件選擇）                   │
├─────────────────────────────────────┤
│          Rules.mak                  │
│       （建置規則）                   │
├─────────────────────────────────────┤
│          Package/                   │
│       （套件定義）                   │
├─────────────────────────────────────┤
│          Board/                     │
│       （開發板配置）                  │
├─────────────────────────────────────┤
│          Configs/                   │
│       （預設配置）                   │
└─────────────────────────────────────┘
```

**Buildroot 配置：**
```bash
# 基本 Buildroot 設定
git clone https://github.com/buildroot/buildroot.git
cd buildroot

# 為目標進行配置
make defconfig
make menuconfig

# 建置系統
make

# 清除建置
make clean
make distclean
```

#### **Buildroot 套件管理**

**套件定義範例：**
```makefile
# 自訂應用程式的套件定義
################################################################################
#
# myapp
#
################################################################################

MYAPP_VERSION = 1.0.0
MYAPP_SITE = $(TOPDIR)/package/myapp/src
MYAPP_SITE_METHOD = local
MYAPP_LICENSE = GPL-2.0
MYAPP_LICENSE_FILES = LICENSE

# 建置相依性
MYAPP_DEPENDENCIES = host-pkgconf

# 建置指令
define MYAPP_BUILD_CMDS
    $(MAKE) CC=$(TARGET_CC) LD=$(TARGET_LD) -C $(@D)
endef

# 安裝指令
define MYAPP_INSTALL_TARGET_CMDS
    $(INSTALL) -D -m 0755 $(@D)/myapp $(TARGET_DIR)/usr/bin/myapp
    $(INSTALL) -D -m 0644 $(@D)/myapp.conf $(TARGET_DIR)/etc/myapp.conf
endef

$(eval $(generic-package))
```

**自訂套件整合：**
```bash
# 建立套件目錄
mkdir -p package/myapp
cp myapp.mk package/myapp/

# 加入套件選擇
echo "source package/myapp/Config.in" >> package/Config.in

# 建立 Config.in
cat > package/myapp/Config.in << EOF
config BR2_PACKAGE_MYAPP
    bool "myapp"
    help
      嵌入式系統的自訂應用程式。
      
      http://example.com/myapp
EOF
```

---

## 🏭 **Yocto 專案**

### **工業級建置系統**

Yocto 專案是一個全面的建置系統，專為工業嵌入式 Linux 開發而設計。它提供進階功能、廣泛的套件支援以及企業級的可靠性。

#### **Yocto 專案的設計理念**

Yocto 遵循**工業強度原則**——提供一個穩健、可擴展的建置系統，以全面的工具和支援滿足企業嵌入式 Linux 開發的需求。

**Yocto 設計目標：**

- **可擴展性**：支援大型、複雜的專案
- **可靠性**：企業級的建置一致性
- **彈性**：廣泛的客製化選項
- **社群**：龐大的生態系統與支援
- **標準**：遵循業界標準的建置實務

#### **Yocto 架構**

**核心元件：**
```
┌─────────────────────────────────────┐
│           Yocto 架構                │
├─────────────────────────────────────┤
│           BitBake                   │
│        （建置引擎）                  │
├─────────────────────────────────────┤
│       OpenEmbedded Core             │
│        （建置系統）                  │
├─────────────────────────────────────┤
│          Meta Layers                │
│      （配置與套件層）                │
├─────────────────────────────────────┤
│          建置輸出                   │
│    （映像檔、套件、SDK）             │
└─────────────────────────────────────┘
```

**Yocto 設定：**
```bash
# 複製 Yocto 專案
git clone -b kirkstone git://git.yoctoproject.org/poky.git
cd poky

# 載入環境設定
source oe-init-build-env

# 為目標進行配置
bitbake-layers add-layer meta-raspberrypi
bitbake-layers add-layer meta-openembedded/meta-oe

# 建置核心映像檔
bitbake core-image-minimal

# 建置自訂映像檔
bitbake my-custom-image
```

#### **Yocto 配方開發**

**基本配方範例：**
```bitbake
# 自訂應用程式的配方
DESCRIPTION = "Custom embedded application"
HOMEPAGE = "http://example.com"
LICENSE = "GPL-2.0-only"
LIC_FILES_CHKSUM = "file://LICENSE;md5=1234567890abcdef"

SRC_URI = "file://myapp-${PV}.tar.gz"
SRC_URI[sha256sum] = "1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef"

S = "${WORKDIR}/myapp-${PV}"

inherit autotools pkgconfig

# 建置相依性
DEPENDS += "pkgconfig-native"

# 執行時相依性
RDEPENDS:${PN} += "libc"

# 安裝額外檔案
do_install:append() {
    install -d ${D}${sysconfdir}/myapp
    install -m 0644 ${S}/config/myapp.conf ${D}${sysconfdir}/myapp/
    
    install -d ${D}${systemd_system_unitdir}
    install -m 0644 ${S}/systemd/myapp.service ${D}${systemd_system_unitdir}/
}

# 啟用 systemd 整合
inherit systemd
SYSTEMD_AUTO_ENABLE = "enable"
SYSTEMD_SERVICE:${PN} = "myapp.service"
```

**自訂映像檔配方：**
```bitbake
# 自訂映像檔配方
DESCRIPTION = "Custom embedded Linux image"
LICENSE = "MIT"

inherit core-image

# 包含額外套件
IMAGE_INSTALL += " \
    myapp \
    openssh \
    iperf3 \
    htop \
"

# 包含開發工具（除錯建置）
IMAGE_INSTALL:append = " \
    packagegroup-core-buildessential \
    gdb \
    strace \
"

# 配置映像檔功能
IMAGE_FEATURES += " \
    ssh-server-openssh \
    tools-debug \
    tools-profile \
"

# 設定 root 密碼
ROOTFS_POSTPROCESS_COMMAND += "set_root_passwd;"
set_root_passwd() {
    sed -i 's/^root:[^:]*:/root:\$6\$rounds=5000\$salt\$hash:/' ${IMAGE_ROOTFS}/etc/shadow
}
```

---

## 🛠️ **自訂發行版開發**

### **建置量身打造的 Linux 系統**

自訂發行版開發涉及建立專門為目標應用設計的 Linux 系統。這需要了解系統需求、元件選擇和整合。

#### **自訂發行版的設計理念**

自訂發行版遵循**應用專屬原則**——設計完美匹配目標應用需求、限制和運行環境的 Linux 系統。

**自訂發行版目標：**

- **應用契合**：完美匹配目標應用
- **資源最佳化**：高效使用可用資源
- **可靠性**：在目標環境中穩定運作
- **可維護性**：易於更新和維護
- **安全性**：適合目標使用場景的安全防護

#### **系統元件選擇**

**必要元件：**
```bash
# 最小系統元件
- Linux 核心（針對目標最佳化）
- 初始化系統（systemd、busybox 或自訂）
- 核心函式庫（glibc、uclibc 或 musl）
- 基本工具（coreutils、busybox）
- 裝置管理（udev、mdev）
- 網路堆疊（如有需要）
- 應用專屬套件
```

**元件選擇標準：**
```bash
# 容量考量
- 記憶體佔用
- 儲存空間需求
- 開機時間影響

# 功能需求
- 應用程式相依性
- 硬體支援需求
- 網路需求

# 維護考量
- 更新機制
- 安全性更新
- 長期支援
```

#### **自訂初始化系統**

**Systemd 配置：**
```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=自訂應用程式
After=network.target
Wants=network.target

[Service]
Type=simple
User=myapp
Group=myapp
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/myapp
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

**Busybox 初始化配置：**
```bash
# /etc/inittab
::sysinit:/etc/init.d/rcS
::respawn:/sbin/getty -L ttyS0 115200 vt100
::respawn:/opt/myapp/myapp
::shutdown:/etc/init.d/rcK
```

---

## ⚡ **系統最佳化**

### **針對嵌入式用途進行最佳化**

系統最佳化涉及降低資源使用、提升效能，以及確保在嵌入式硬體的限制條件下可靠運作。

#### **最佳化理念**

系統最佳化遵循**效率與可靠性原則**——在最小化資源使用和功耗的同時，最大化系統效能和可靠性。

**最佳化目標：**

- **記憶體效率**：最小化 RAM 使用量
- **儲存最佳化**：減少磁碟/快閃記憶體使用量
- **效能**：針對目標工作負載進行最佳化
- **功耗效率**：最小化功率消耗
- **可靠性**：確保穩定運作

#### **核心最佳化**

**核心配置：**
```bash
# 必要的核心選項
CONFIG_EMBEDDED=y
CONFIG_EXPERT=y
CONFIG_SLOB=y                    # 簡單記憶體配置器
CONFIG_BLK_DEV_INITRD=y         # 初始 RAM 磁碟
CONFIG_DEVTMPFS=y               # 裝置檔案系統
CONFIG_DEVTMPFS_MOUNT=y         # 自動掛載 devtmpfs

# 停用不必要的功能
# CONFIG_DEBUG_FS is not set
# CONFIG_DEBUG_KERNEL is not set
# CONFIG_DEBUG_INFO is not set
# CONFIG_DEBUG_MEMORY_INIT is not set

# 僅啟用所需的驅動程式
CONFIG_SERIAL_8250=y
CONFIG_SERIAL_8250_CONSOLE=y
CONFIG_MMC=y
CONFIG_MMC_BLOCK=y
```

**核心大小縮減：**
```bash
# 移除未使用的驅動程式
# CONFIG_SOUND is not set
# CONFIG_VIDEO_DEV is not set
# CONFIG_INPUT is not set

# 針對大小進行最佳化
CONFIG_CC_OPTIMIZE_FOR_SIZE=y
CONFIG_KERNEL_GZIP=y
CONFIG_MODULES=n
```

#### **根檔案系統最佳化**

**檔案系統選擇：**
```bash
# 輕量級檔案系統
- squashfs：唯讀、壓縮
- ext4：日誌式、效能良好
- f2fs：快閃記憶體最佳化
- jffs2：快閃記憶體檔案系統
- ubifs：進階快閃記憶體檔案系統
```

**函式庫最佳化：**
```bash
# 使用輕量級替代方案
- 以 musl libc 取代 glibc
- 以 busybox 取代 coreutils
- 以 dropbear 取代 openssh
- 以 mdev 取代 udev
```

**套件最佳化：**
```bash
# 移除不必要的套件
- 開發工具
- 文件
- 語系資料（僅保留必要的）
- 除錯符號
- 未使用的函式庫
```

---

## 🚀 **部署與維護**

### **嵌入式系統的部署與維護**

部署與維護涉及建立可靠的更新機制、監控系統健康狀態，以及確保在現場的長期運作。

#### **部署理念**

部署遵循**可靠性與可維護性原則**——確保系統能夠可靠地部署，並在整個運作生命週期中有效維護。

**部署目標：**

- **可靠性**：一致的部署成功率
- **效率**：快速的部署流程
- **驗證**：驗證系統完整性
- **回滾**：能夠還原變更
- **監控**：追蹤系統健康狀態

#### **更新機制**

**OTA（空中下載）更新：**
```bash
# 更新腳本範例
#!/bin/sh

# 下載更新
wget -O /tmp/update.tar.gz http://update.server/update.tar.gz

# 驗證校驗碼
echo "expected_checksum /tmp/update.tar.gz" | sha256sum -c

# 解壓更新
tar -xzf /tmp/update.tar.gz -C /tmp/update

# 備份目前系統
cp -r /opt/myapp /opt/myapp.backup

# 安裝更新
cp -r /tmp/update/* /opt/myapp/

# 重新啟動應用程式
systemctl restart myapp

# 清理
rm -rf /tmp/update*
```

**雙分割區更新：**
```bash
# 雙分割區更新腳本
#!/bin/sh

# 判斷目前分割區
CURRENT_PART=$(cat /proc/cmdline | grep -o "root=/dev/mmcblk0p[12]")

if echo $CURRENT_PART | grep -q "p1"; then
    UPDATE_PART="/dev/mmcblk0p2"
    NEXT_PART="p2"
else
    UPDATE_PART="/dev/mmcblk0p1"
    NEXT_PART="p1"
fi

# 格式化更新分割區
mkfs.ext4 $UPDATE_PART

# 掛載並複製檔案
mount $UPDATE_PART /mnt/update
cp -r /opt/myapp/* /mnt/update/

# 更新開機載入器以從新分割區開機
fw_setenv bootpart $NEXT_PART

# 重新開機至新分割區
reboot
```

#### **系統監控**

**健康監控：**
```bash
#!/bin/sh

# 系統健康檢查腳本
while true; do
    # 檢查記憶體使用量
    MEM_USAGE=$(free | grep Mem | awk '{print $3/$2 * 100.0}')
    if [ $(echo "$MEM_USAGE > 90" | bc) -eq 1 ]; then
        logger "警告：記憶體使用量過高：${MEM_USAGE}%"
    fi
    
    # 檢查磁碟使用量
    DISK_USAGE=$(df / | tail -1 | awk '{print $5}' | sed 's/%//')
    if [ $DISK_USAGE -gt 90 ]; then
        logger "警告：磁碟使用量過高：${DISK_USAGE}%"
    fi
    
    # 檢查應用程式狀態
    if ! pgrep myapp > /dev/null; then
        logger "錯誤：應用程式未執行，正在重新啟動"
        systemctl restart myapp
    fi
    
    sleep 60
done
```

**日誌管理：**
```bash
# 日誌輪替配置
cat > /etc/logrotate.d/myapp << EOF
/var/log/myapp/*.log {
    daily
    missingok
    rotate 7
    compress
    notifempty
    create 644 myapp myapp
    postrotate
        systemctl reload myapp
    endscript
}
EOF
```

---

## 🎯 **結論**

嵌入式 Linux 為建立客製化、高效的嵌入式應用 Linux 系統提供了強大的能力。了解 Buildroot、Yocto 和自訂發行版開發對於建置可靠的嵌入式系統至關重要。

**重點摘要：**

- **嵌入式 Linux 著重於效率和客製化**，針對特定應用進行設計
- **Buildroot 提供簡單、高效的建置**，適合小型專案
- **Yocto 提供工業級功能**，適合複雜的企業專案
- **自訂發行版能夠完美契合應用需求**並進行最佳化
- **系統最佳化對於資源受限的裝置至關重要**
- **部署與維護**確保長期可靠運作

**未來展望：**

隨著嵌入式系統變得更加複雜且互聯，嵌入式 Linux 技能的重要性只會持續增加。現代系統不斷演進，提供新的最佳化技術和部署策略。

**請記住**：嵌入式 Linux 不僅僅是在嵌入式裝置上執行 Linux——它是關於建立完美匹配您應用需求的專用系統，同時高效利用可用資源。您在此培養的技能將使您能夠建立穩健、高效且可維護的嵌入式 Linux 系統。
