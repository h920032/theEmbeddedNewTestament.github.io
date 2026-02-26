# 嵌入式系統的網路協定

設計聯網嵌入式裝置需要在確定性行為、小記憶體佔用量及有限的 CPU 時脈週期之間取得平衡，同時還要使用可互通的協定。本指南著重於 IPv4/IPv6、ICMP/ARP/ND、UDP/TCP 以及應用層 IoT 協定的實務與生產導向面向。

---

## 概念 → 為什麼重要 → 最小範例 → 動手試試 → 重點整理

**概念**：嵌入式系統中的網路協定是在管理有限資源的同時維持可靠通訊。與桌上型系統擁有充裕的記憶體和處理能力不同，嵌入式裝置必須在功能性、效能與資源限制之間仔細取得平衡。

**為什麼重要**：網路連線能力對現代嵌入式系統至關重要，但傳統的網路堆疊可能會壓垮受限裝置。了解如何配置輕量級堆疊、管理記憶體池以及實作高效率的協定，對於建置可靠的聯網裝置至關重要。

**最小範例**：一個簡單的 UDP 回應伺服器，展示記憶體池管理和零拷貝緩衝區處理。

**動手試試**：實作一個具有連線池的基本 TCP 客戶端，並觀察不同負載條件下的記憶體使用模式。

**重點整理**：嵌入式系統中的網路協定需要謹慎的資源管理、深思熟慮的配置，以及理解功能性與資源限制之間的取捨。

---

## 目標
- 了解嵌入式環境中的 TCP/IP 堆疊
- 選擇並配置輕量級堆疊（例如 lwIP）
- 實作穩健的 UDP/TCP 客戶端/伺服器
- 整合 IoT 應用協定（MQTT/CoAP）
- 使用結構化的檢查清單診斷問題

---

## TCP/IP 堆疊概述

### 網路堆疊模型與嵌入式考量
**OSI（7 層）**模型提供了理論框架，但嵌入式系統通常基於實用考量實作 **TCP/IP（4 層）**模型。這種簡化減少了記憶體開銷和處理複雜度，同時維持完整的網路功能。

**為什麼嵌入式選擇 TCP/IP？**
- **記憶體效率**：較少的層代表更小的程式碼佔用量
- **處理開銷**：減少堆疊遍歷時間
- **業界標準**：與現有基礎設施直接相容
- **可擴充性**：可依需求輕鬆新增/移除功能

**鏈結層選擇與取捨**
- **乙太網路（10/100/1000 Mbps）**：高頻寬、複雜 PHY、需要磁性元件
- **Wi‑Fi（802.11a/b/g/n/ax）**：無線便利性、功耗、干擾
- **PPP**：簡單的點對點、最小開銷、僅限序列鏈路
- **LTE Cat-x**：行動網路連線、高功耗、訂閱費用
- **802.15.4（Zigbee/Thread）**：低功耗、網狀網路、有限頻寬

**網際層設計決策**
IPv4 與 IPv6 是嵌入式系統的關鍵選擇：
- **IPv4**：較小的標頭（20-60 位元組）、NAT 複雜性、位址耗盡
- **IPv6**：固定標頭（40 位元組）、不需要 NAT、較大位址空間，但鄰居發現更複雜

**傳輸層選擇標準**
- **UDP**：當你需要訊息邊界、低延遲，且能在應用層處理可靠性時
- **TCP**：當你需要保證傳遞、排序和內建的流量控制時

### 記憶體配置考量
了解記憶體分配對於每個位元組都很寶貴的嵌入式系統至關重要。lwIP 堆疊提供廣泛的配置選項，以在功能性與記憶體限制之間取得平衡。

**記憶體池哲學**
嵌入式系統使用預先分配的記憶體池來取代動態分配，以達到：
- 消除堆積碎片化
- 提供可預測的記憶體使用
- 實現最壞情況執行時間分析
- 防止網路風暴期間的記憶體耗盡

**關鍵配置參數說明**
- **MEM_SIZE**：用於封包緩衝區和控制結構的通用記憶體
- **MEMP_NUM_TCP_PCB**：每個 TCP 連線消耗約 200-400 位元組
- **TCP_MSS**：必須與鏈路 MTU 對齊以避免碎片化
- **PBUF_POOL_SIZE**：決定同時可以有多少封包在傳輸中

```c
// 典型的 lwIP 記憶體池配置
#define MEM_SIZE                    (20*1024)        // 20KB 通用記憶體
#define MEMP_NUM_TCP_PCB          20                 // 最大 TCP 連線數
#define MEMP_NUM_TCP_PCB_LISTEN   10                 // 最大監聽通訊端數
#define MEMP_NUM_TCP_SEG          32                 // 最大傳輸中 TCP 區段數
#define TCP_MSS                   1460               // 最大區段大小（乙太網路 MTU 1500 - IP 20 - TCP 20）
#define TCP_SND_BUF               (4*TCP_MSS)       // 4 個區段的傳送緩衝區
#define TCP_WND                   (4*TCP_MSS)       // 4 個區段的接收視窗
#define PBUF_POOL_SIZE            24                 // 封包緩衝區池
#define PBUF_POOL_BUFSIZE         1520               // 緩衝區大小（乙太網路 MTU - 14）
```

---

## 定址、名稱解析與配置

### 位址指派策略及其影響
**靜態 IP 配置**
- **優點**：可預測的定址、無 DHCP 依賴、更快的開機時間
- **缺點**：手動配置、部署複雜性、IP 衝突
- **使用案例**：關鍵基礎設施、固定安裝、開發環境

**DHCPv4 動態指派**
- **優點**：自動配置、集中管理、避免衝突
- **缺點**：開機延遲、依賴 DHCP 伺服器、租約續期複雜性
- **使用案例**：消費性裝置、辦公環境、動態部署

**IPv6 SLAAC（無狀態位址自動配置）**
- **優點**：不需要伺服器、自動配置、全域定址
- **缺點**：較大的標頭、更複雜的鄰居發現、安全性考量
- **使用案例**：IoT 部署、現代網路、僅 IPv6 環境

**NAT 穿越考量**
NAT 為嵌入式裝置帶來挑戰：
- **對外連線**：通常無問題
- **對內連線**：需要連接埠轉發或 UPnP
- **P2P 通訊**：需要 STUN/TURN 伺服器或打洞技術
- **建議**：盡可能設計為客戶端發起的連線

### ARP 與鄰居發現實作
**位址解析協定（ARP）**
ARP 在 IPv4 網路中將 IP 位址對應到 MAC 位址。了解 ARP 行為對於以下方面至關重要：
- 網路故障排除
- 安全性分析（ARP 欺騙偵測）
- 效能最佳化（ARP 快取）

**IPv6 中的鄰居發現（ND）**
IPv6 以 ND 取代 ARP，提供：
- 位址解析
- 路由器發現
- 重複位址偵測
- 重新導向訊息

**ARP 表管理策略**
嵌入式系統需要高效率的 ARP 表管理：
- **固定大小表格**：可預測的記憶體使用，但擴充性有限
- **LRU 淘汰**：自動清理，但可能影響效能
- **基於逾時的清理**：記憶體效率高，但需要計時器管理

```c
// 嵌入式系統的自訂 ARP 表管理
typedef struct {
    uint32_t ip_addr;
    uint8_t mac_addr[6];
    uint32_t timestamp;
    uint8_t state;  // ARP_STATE_EMPTY、ARP_STATE_PENDING、ARP_STATE_STABLE
} arp_entry_t;

#define ARP_TABLE_SIZE 16
static arp_entry_t arp_table[ARP_TABLE_SIZE];

// 具有逾時和重試的 ARP 請求
err_t arp_request_with_retry(struct netif *netif, const ip4_addr_t *ipaddr) {
    err_t err = arp_request(netif, ipaddr);
    if (err == ERR_OK) {
        // 啟動重試計時器
        sys_timeout(ARP_TIMEOUT_MS, arp_retry_timeout, netif);
    }
    return err;
}
```

---

## 嵌入式網路堆疊

### lwIP 配置深入探討
**配置哲學**
lwIP 提供廣泛的配置選項，必須為嵌入式系統仔細調校。目標是僅啟用所需的功能，同時維持系統穩定性。

**功能選擇標準**
- **LWIP_IPV6**：僅在需要 IPv6 連線時啟用
- **LWIP_DNS**：若僅使用靜態 IP 位址則停用
- **LWIP_DHCP**：為動態配置啟用
- **LWIP_AUTOIP**：在生產環境中停用（鏈路本地位址衝突）
- **LWIP_NETCONN 與 LWIP_SOCKET**：根據 API 偏好和記憶體限制選擇

**記憶體池調校策略**
記憶體池必須根據以下因素調整大小：
- **預期連線數量**
- **封包大小和突發模式**
- **可用系統記憶體**
- **效能需求**

```c
// lwIP 配置標頭（lwipopts.h）
#define LWIP_IPV4                  1
#define LWIP_IPV6                  0  // 不需要時停用
#define LWIP_DNS                   1
#define LWIP_DHCP                  1
#define LWIP_AUTOIP                0  // 生產環境停用
#define LWIP_NETIF_HOSTNAME        1
#define LWIP_NETCONN               0  // 使用原始 API 以獲得更好的控制
#define LWIP_SOCKET                0  // 若使用原始 API 則停用通訊端 API
#define LWIP_STATS                 1
#define LWIP_DEBUG                 0  // 開發時啟用

// 針對特定使用案例的記憶體池調校
#define MEMP_NUM_UDP_PCB           8
#define MEMP_NUM_TCP_PCB           4
#define MEMP_NUM_TCP_SEG           16
#define MEMP_NUM_NETBUF            8
#define MEMP_NUM_NETCONN           0
#define MEMP_NUM_TCPIP_MSG_API     8
#define MEMP_NUM_TCPIP_MSG_INPKT   8

// 嵌入式 TCP 調校
#define TCP_TMR_INTERVAL           250  // 250ms 計時器間隔
#define TCP_MSL                    (60*1000/TCP_TMR_INTERVAL)  // 60 秒 MSL
#define TCP_FIN_WAIT_TIMEOUT      (2*TCP_MSL)
#define TCP_SYNMAXRTX              6
#define TCP_DEFAULT_LISTEN_BACKLOG 1
```

**執行緒模型考量**
lwIP 可以在不同的執行緒模型下運作：
- **單執行緒**：所有網路操作在主迴圈中
- **多執行緒**：專用網路執行緒搭配訊息傳遞
- **中斷驅動**：在中斷上下文中處理網路

**效能影響**
- **單執行緒**：簡單，但可能阻塞應用程式
- **多執行緒**：回應性佳，但需要同步機制
- **中斷驅動**：最快，但錯誤處理複雜

### 零拷貝緩衝區管理
**為什麼零拷貝很重要**
傳統網路堆疊會多次複製資料：
1. 驅動程式接收封包 → 核心緩衝區
2. 核心緩衝區 → 使用者緩衝區
3. 使用者緩衝區 → 應用程式

每次複製都會消耗 CPU 時脈週期和記憶體頻寬。零拷貝消除了這些複製動作。

**DMA 與快取考量**
具有 DMA 和快取的現代 MCU 需要謹慎的緩衝區管理：
- **快取一致性**：確保 DMA 和 CPU 看到一致的資料
- **緩衝區對齊**：對齊到快取行大小以獲得最佳效能
- **不可快取區域**：將 DMA 緩衝區映射以避免快取問題

```c
// 具有快取維護的 DMA 安全緩衝區分配
typedef struct {
    uint8_t *buffer;
    uint32_t size;
    uint32_t flags;
} dma_buffer_t;

dma_buffer_t* allocate_dma_buffer(uint32_t size) {
    dma_buffer_t *buf = malloc(sizeof(dma_buffer_t));
    if (buf) {
        // 對齊到快取行大小（ARM Cortex-M7 為 32 位元組）
        buf->buffer = aligned_alloc(32, size);
        buf->size = size;
        buf->flags = DMA_BUFFER_FLAG_CACHEABLE;
        
        // 確保快取一致性
        SCB_CleanInvalidateDCache_by_Addr((uint32_t*)buf->buffer, size);
    }
    return buf;
}
```

---

## 嵌入式系統中的 UDP

### 何時使用 UDP
**UDP 對嵌入式系統的優勢**
- **低開銷**：無連線建立、最小標頭
- **訊息邊界**：天然適合命令/回應協定
- **多播支援**：高效率的群組通訊
- **即時友善**：無重傳延遲

**UDP 的挑戰與緩解措施**
- **無可靠性**：必須在應用層實作
- **無排序**：若需要則需要序號
- **無流量控制**：突發流量需要速率限制
- **無壅塞控制**：若不小心可能壓垮網路

**可靠 UDP 的設計模式**
**序號與確認**
每個 UDP 訊息應包含：
- 用於排序的序號
- 用於逾時計算的時間戳記
- 用於完整性驗證的校驗和
- 用於協定狀態機的訊息類型

**重試策略**
- **指數退避**：防止網路風暴
- **抖動**：避免同步重試
- **最大重試次數**：防止無限迴圈
- **逾時計算**：基於網路特性

```c
// 具有序號和 ACK 的可靠 UDP
typedef struct {
    uint32_t seq_num;
    uint32_t ack_num;
    uint16_t length;
    uint16_t checksum;
    uint8_t flags;
    uint8_t data[];
} reliable_udp_header_t;

#define UDP_FLAG_ACK    0x01
#define UDP_FLAG_NACK   0x02
#define UDP_FLAG_RETRY  0x04

typedef struct {
    uint32_t seq_num;
    uint32_t timestamp;
    uint8_t retry_count;
    uint8_t data[UDP_MAX_PAYLOAD];
} udp_packet_t;

// UDP 可靠性層
err_t udp_send_reliable(struct udp_pcb *pcb, const void *data, u16_t len,
                       const ip_addr_t *addr, u16_t port) {
    static uint32_t seq_counter = 0;
    udp_packet_t *packet = malloc(sizeof(udp_packet_t) + len);
    
    packet->seq_num = seq_counter++;
    packet->timestamp = sys_now();
    packet->retry_count = 0;
    memcpy(packet->data, data, len);
    
    // 加入重傳佇列
    add_to_retry_queue(packet);
    
    return udp_send(pcb, packet, sizeof(udp_packet_t) + len);
}
```

### UDP 多播實作
**多播與廣播**
- **廣播**：到達所有裝置、浪費頻寬、僅限本地網路
- **多播**：僅到達有興趣的裝置、高效率、可跨越網路邊界

**IGMP（網際網路群組管理協定）**
IGMP 允許主機加入/離開多播群組：
- **加入訊息**：主機宣告對群組的興趣
- **離開訊息**：主機宣告離開群組
- **查詢訊息**：路由器檢查群組成員資格

**多播位址範圍**
- **224.0.0.0/4**：保留給多播
- **224.0.0.0/24**：本地網路控制
- **224.0.1.0/24**：網際網路間控制
- **232.0.0.0/8**：來源特定多播
- **233.0.0.0/8**：GLOP 定址

```c
// 使用 IGMP 加入多播群組
err_t udp_join_multicast_group(const ip4_addr_t *multicast_addr, 
                               const ip4_addr_t *netif_addr) {
    err_t err = igmp_joingroup(netif_addr, multicast_addr);
    if (err == ERR_OK) {
        // 配置網路介面以支援多播
        struct netif *netif = ip4_route_src(multicast_addr);
        if (netif) {
            netif->flags |= NETIF_FLAG_IGMP;
        }
    }
    return err;
}
```

---

## 嵌入式系統中的 TCP

### TCP 連線管理
**連線池設計**
嵌入式系統通常需要高效率地管理多個 TCP 連線：
- **預先分配的連線**：避免動態分配開銷
- **連線重用**：減少建立/拆除成本
- **負載平衡**：將連線分散到多個伺服器
- **容錯移轉**：自動切換到備援伺服器

**連線生命週期管理**
每個 TCP 連線會經歷多個狀態：
1. **CLOSED**：不存在連線
2. **LISTEN**：伺服器等待連線
3. **SYN_SENT**：客戶端發送連線請求
4. **SYN_RECEIVED**：伺服器收到連線請求
5. **ESTABLISHED**：連線處於活動狀態
6. **FIN_WAIT_1**：應用程式關閉連線
7. **FIN_WAIT_2**：等待遠端關閉
8. **CLOSE_WAIT**：遠端已關閉，等待應用程式
9. **LAST_ACK**：等待最終確認
10. **TIME_WAIT**：連線已關閉，等待清理

**保活配置**
TCP 保活偵測失效連線：
- **閒置時間**：發送探測前等待多久
- **探測間隔**：探測之間的時間
- **探測次數**：宣告失效前的探測次數
- **考量**：功耗、網路開銷、誤判

```c
// 具有保活和逾時的 TCP 連線
err_t tcp_connect_with_keepalive(const ip_addr_t *ipaddr, u16_t port) {
    tcp_connection_t *conn = tcp_connection_acquire();
    if (!conn) return ERR_MEM;
    
    conn->pcb = tcp_new();
    if (!conn->pcb) {
        conn->in_use = 0;
        return ERR_MEM;
    }
    
    // 設定保活參數
    tcp_keepalive(conn->pcb, 1, 60, 3); // 啟用，60 秒閒置，3 次探測
    
    // 設定回呼函式
    tcp_arg(conn->pcb, conn);
    tcp_recv(conn->pcb, tcp_recv_callback);
    tcp_sent(conn->pcb, tcp_sent_callback);
    tcp_err(conn->pcb, tcp_err_callback);
    
    // 連線
    return tcp_connect(conn->pcb, ipaddr, port, tcp_connected_callback);
}
```

### TCP 流量控制與視窗管理
**流量控制基礎**
TCP 使用滑動視窗機制進行流量控制：
- **通告視窗**：接收端告訴傳送端可以接受多少資料
- **壅塞視窗**：傳送端對網路容量的估計
- **有效視窗**：通告視窗和壅塞視窗的最小值

**視窗管理策略**
- **靜態視窗**：簡單，但效率低
- **動態視窗**：適應網路狀況，但複雜
- **視窗縮放**：處理高頻寬、高延遲的網路

**Nagle 演算法與延遲**
Nagle 演算法透過合併小封包來減少網路開銷：
- **啟用**：更好的吞吐量，更高的延遲
- **停用**：更低的延遲，更多的封包
- **決策因素**：封包大小、延遲需求、網路特性

```c
// 自訂 TCP 視窗管理
typedef struct {
    uint16_t advertised_window;
    uint16_t effective_window;
    uint16_t congestion_window;
    uint16_t slow_start_threshold;
    uint8_t dup_ack_count;
} tcp_window_state_t;

void tcp_window_update(struct tcp_pcb *pcb, tcp_window_state_t *state) {
    // 根據可用緩衝區空間更新通告視窗
    uint16_t available_space = tcp_sndbuf(pcb);
    state->advertised_window = available_space;
    
    // 套用流量控制
    if (available_space < TCP_MIN_WINDOW) {
        // 暫停傳送
        tcp_output(pcb);
    }
}
```

---

## IoT 應用協定

### MQTT（訊息佇列遙測傳輸）
**MQTT 架構與概念**
MQTT 是一種專為受限裝置設計的發布/訂閱訊息協定：
- **代理伺服器**：中央訊息路由器（可以是雲端或本地）
- **客戶端**：發布或訂閱主題的裝置
- **主題**：階層式訊息路由（例如 "sensors/temperature/room1"）
- **QoS 等級**：0（至多一次）、1（至少一次）、2（恰好一次）

**MQTT 用於嵌入式系統**
**優勢**
- 輕量級協定（最小 2 位元組標頭）
- 適合間歇性連線
- 內建遺囑訊息
- 為新訂閱者保留訊息

**挑戰**
- 需要持久的代理伺服器連線
- QoS 2 對資源受限裝置的複雜性
- 大型部署的主題設計複雜性

**實作考量**
- **持久連線階段**：減少重新連線開銷
- **主題設計**：為可擴充性和安全性做規劃
- **QoS 選擇**：在可靠性與開銷之間取得平衡
- **保活**：在回應性與功耗之間取得平衡

```c
// MQTT 客戶端狀態機
typedef enum {
    MQTT_STATE_DISCONNECTED,
    MQTT_STATE_CONNECTING,
    MQTT_STATE_CONNECTED,
    MQTT_STATE_PUBLISHING,
    MQTT_STATE_SUBSCRIBING
} mqtt_state_t;

typedef struct {
    mqtt_state_t state;
    uint16_t packet_id;
    uint32_t keepalive_interval;
    uint32_t last_activity;
    struct tcp_pcb *tcp_pcb;
    mqtt_message_callback_t message_callback;
} mqtt_client_t;
```

### CoAP（受限應用協定）
**CoAP 設計哲學**
CoAP 將類似 HTTP 的語意帶入受限網路：
- **RESTful**：以資源為導向的設計
- **基於 UDP**：比 HTTP 更低的開銷
- **二進位格式**：高效率的編碼
- **可靠傳輸**：內建重傳機制

**CoAP 的嵌入式功能**
- **可確認訊息**：透過 ACK 實現可靠傳遞
- **不可確認訊息**：非關鍵資料的發後即忘
- **可觀察資源**：伺服器推送通知
- **區塊傳輸**：大型資源處理

**CoAP 與 HTTP 的取捨**
- **CoAP**：更低的開銷、基於 UDP、內建可靠性
- **HTTP**：更熟悉、基於 TCP、廣泛的工具支援

---

## 效能調校與最佳化

### 記憶體池最佳化
**池大小調整策略**
記憶體池必須根據以下因素調整大小：
- **流量模式**：突發性與穩態
- **封包大小**：MTU 和應用需求
- **連線數量**：並行連線數
- **效能需求**：延遲與吞吐量

**碎片化預防**
- **固定大小池**：消除碎片化
- **可變大小池**：更高效率，但更複雜
- **混合方式**：常見大小使用固定池，其他使用可變池

```c
// 網路緩衝區的自訂記憶體池
typedef struct {
    uint8_t *pool_start;
    uint8_t *pool_end;
    uint32_t pool_size;
    uint32_t used_blocks;
    uint32_t total_blocks;
    uint32_t block_size;
    uint8_t *free_list;
} network_pool_t;

network_pool_t* create_network_pool(uint32_t block_size, uint32_t num_blocks) {
    network_pool_t *pool = malloc(sizeof(network_pool_t));
    if (pool) {
        pool->block_size = block_size;
        pool->total_blocks = num_blocks;
        pool->pool_size = block_size * num_blocks;
        
        // 分配對齊的記憶體
        pool->pool_start = aligned_alloc(32, pool->pool_size);
        pool->pool_end = pool->pool_start + pool->pool_size;
        
        // 初始化空閒清單
        pool->free_list = pool->pool_start;
        for (uint32_t i = 0; i < num_blocks - 1; i++) {
            *(uint32_t*)(pool->pool_start + i * block_size) = 
                (uint32_t)(pool->pool_start + (i + 1) * block_size);
        }
        *(uint32_t*)(pool->pool_start + (num_blocks - 1) * block_size) = 0;
        
        pool->used_blocks = 0;
    }
    return pool;
}
```

### 中斷合併配置
**中斷合併理論**
中斷合併透過批次處理中斷來減少 CPU 開銷：
- **封包計數閾值**：在 N 個封包後產生中斷
- **時間閾值**：在 T 微秒後產生中斷
- **取捨**：更低延遲與更高 CPU 效率

**配置指南**
- **低延遲應用**：使用封包計數閾值
- **高吞吐量應用**：使用時間閾值
- **平衡方式**：結合兩種閾值

```c
// 乙太網路中斷合併設定
typedef struct {
    uint32_t rx_coal_pkt;
    uint32_t rx_coal_time;
    uint32_t tx_coal_pkt;
    uint32_t tx_coal_time;
} eth_coal_config_t;

err_t eth_set_interrupt_coalescing(eth_coal_config_t *config) {
    // 配置接收合併
    ETH->MACCR |= ETH_MACCR_IPC; // 啟用中斷合併
    
    // 設定封包計數閾值
    ETH->MACFCR = (config->rx_coal_pkt << ETH_MACFCR_RXCOAL_Pos) |
                  (config->rx_coal_time << ETH_MACFCR_RXCOAL_TIME_Pos);
    
    // 設定時間閾值（以 64ns 為單位）
    uint32_t time_threshold = config->rx_coal_time * 15625; // 轉換為 64ns 單位
    ETH->MACFCR |= (time_threshold << ETH_MACFCR_RXCOAL_TIME_Pos);
    
    return ERR_OK;
}
```

---

## 診斷與故障排除

### 網路統計資料收集
**要測量什麼**
- **介面統計**：封包、位元組、錯誤、丟棄
- **協定統計**：TCP 連線、重傳、逾時
- **記憶體統計**：分配、碎片化、峰值使用量
- **效能指標**：延遲、吞吐量、抖動

**統計資料收集策略**
- **即時監控**：對問題的立即可見性
- **歷史趨勢**：識別模式和容量規劃
- **警報**：主動問題偵測
- **容量規劃**：資源分配決策

```c
// 完整的網路統計資料
typedef struct {
    // 介面統計
    uint32_t rx_packets;
    uint32_t tx_packets;
    uint32_t rx_bytes;
    uint32_t tx_bytes;
    uint32_t rx_errors;
    uint32_t tx_errors;
    uint32_t rx_dropped;
    uint32_t tx_dropped;
    
    // TCP 統計
    uint32_t tcp_connections;
    uint32_t tcp_retransmissions;
    uint32_t tcp_timeouts;
    uint32_t tcp_keepalive_probes;
    
    // UDP 統計
    uint32_t udp_packets_sent;
    uint32_t udp_packets_received;
    uint32_t udp_checksum_errors;
    
    // 記憶體統計
    uint32_t mem_allocated;
    uint32_t mem_peak;
    uint32_t mem_fragments;
} network_stats_t;
```

### 進階封包擷取
**擷取策略**
- **選擇性擷取**：專注於特定流量模式
- **基於時間的擷取**：在問題期間擷取
- **基於觸發的擷取**：在特定條件發生時擷取
- **關聯擷取**：同時從多個來源擷取

**分析技術**
- **協定分析**：解碼應用層協定
- **時序分析**：測量延遲和抖動
- **模式識別**：識別異常和趨勢
- **統計分析**：量化效能特性

---

## 生產就緒與部署

### 健康監控與看門狗
**監控策略**
- **心跳監控**：定期健康檢查
- **效能監控**：延遲、吞吐量、錯誤率
- **資源監控**：記憶體、CPU、網路使用率
- **環境監控**：溫度、電源、連線狀態

**恢復機制**
- **自動恢復**：無需介入的自我修復
- **優雅降級**：在壓力下減少功能
- **容錯移轉**：切換到備援系統
- **重設與重啟**：最後手段的恢復

```c
// 網路健康監控
typedef struct {
    uint32_t last_heartbeat;
    uint32_t heartbeat_interval;
    uint32_t missed_heartbeats;
    uint8_t healthy;
} network_health_t;

void network_health_check(network_health_t *health) {
    uint32_t current_time = sys_now();
    
    if (current_time - health->last_heartbeat > health->heartbeat_interval) {
        health->missed_heartbeats++;
        
        if (health->missed_heartbeats > MAX_MISSED_HEARTBEATS) {
            health->healthy = 0;
            // 觸發網路恢復程序
            network_recovery_procedure();
        }
    }
}
```

### 配置管理
**配置哲學**
- **預設值**：常見場景的合理預設
- **驗證**：驗證配置參數
- **持久化**：將配置儲存在非揮發性記憶體中
- **更新機制**：遠端配置更新

**配置驗證**
- **範圍檢查**：確保參數在有效範圍內
- **依賴性檢查**：驗證相關參數的一致性
- **衝突偵測**：識別衝突的配置
- **效能影響**：評估配置對效能的影響

本增強版提供了概念解釋、實務見解和技術實作細節之間更好的平衡，嵌入式工程師可以用來理解和實作穩健的網路解決方案。

---

## 引導式實驗

### 實驗 1：記憶體池分析
**目標**：了解記憶體池如何影響網路效能和穩定性。

**設定**：使用不同的記憶體池大小配置 lwIP，並觀察負載下的行為。

**步驟**：
1. 從最小記憶體池開始（MEMP_NUM_TCP_PCB = 2、PBUF_POOL_SIZE = 8）
2. 使用 10 個並行連線執行 TCP 壓力測試
3. 監控記憶體使用量和連線失敗
4. 逐漸增加池大小直到穩定運行
5. 記錄最小可行配置

**預期結果**：了解記憶體分配與網路穩定性之間的關係。

### 實驗 2：TCP 連線池
**目標**：實作並測試連線池以改善效能。

**設定**：建立一個維護預先分配連線池的 TCP 客戶端。

**步驟**：
1. 實作一個可配置大小的連線池
2. 新增連線健康檢查和自動重新連線
3. 使用不同的連線數量和故障場景進行測試
4. 測量有無連線池的連線建立時間
5. 分析記憶體使用模式

**預期結果**：減少連線開銷並改善可靠性。

### 實驗 3：網路效能分析
**目標**：分析網路效能並識別瓶頸。

**設定**：實作完整的網路統計資料收集和分析。

**步驟**：
1. 在關鍵網路操作中新增統計資料收集
2. 實作具有可配置閾值的效能監控
3. 建立即時網路健康監控儀表板
4. 在各種網路狀況下測試（高延遲、封包遺失）
5. 分析效能模式並相應最佳化

**預期結果**：資料驅動的網路最佳化和主動問題偵測。

---

## 自我檢核

### 理解檢核
- [ ] 你能解釋為什麼嵌入式系統使用記憶體池而非動態分配嗎？
- [ ] 你了解嵌入式應用中 UDP 和 TCP 之間的取捨嗎？
- [ ] 你能為特定使用案例配置 lwIP 記憶體池嗎？
- [ ] 你知道如何使用序號和 ACK 實作可靠 UDP 嗎？
- [ ] 你能解釋連線池的優點和挑戰嗎？

### 應用檢核
- [ ] 你能設計一個在可靠性與資源限制之間取得平衡的網路協定嗎？
- [ ] 你知道如何為嵌入式系統在 IPv4 和 IPv6 之間做選擇嗎？
- [ ] 你能為網路操作實作高效率的緩衝區管理嗎？
- [ ] 你了解如何配置中斷合併以獲得最佳效能嗎？
- [ ] 你能設計一個網路健康監控系統嗎？

### 分析檢核
- [ ] 你能分析網路效能資料以識別瓶頸嗎？
- [ ] 你了解記憶體配置與網路穩定性之間的關係嗎？
- [ ] 你能評估不同網路堆疊配置之間的取捨嗎？
- [ ] 你知道如何排除嵌入式系統中常見的網路問題嗎？
- [ ] 你能評估不同網路配置的安全性影響嗎？

---

## 交叉連結

### 相關主題
- **[記憶體管理](./../Embedded_C/Memory_Management.md)**：了解網路緩衝區的記憶體分配策略
- **[即時系統](./../Real_Time_Systems/FreeRTOS_Basics.md)**：將網路操作與即時約束整合
- **[通訊協定](./../Communication_Protocols/UART_Protocol.md)**：了解協定設計原則
- **[系統整合](./../System_Integration/Build_Systems.md)**：建置和配置網路堆疊

### 延伸閱讀
- **lwIP 文件**：官方 lwIP 使用手冊和 API 參考
- **TCP/IP Illustrated**：TCP/IP 協定內部的深入探討
- **嵌入式網路程式設計**：嵌入式系統網路程式設計實務指南
- **網路效能分析**：網路最佳化的工具和技術

### 產業標準
- **RFC 791**：網際網路協定（IPv4）
- **RFC 2460**：網際網路協定第 6 版（IPv6）
- **RFC 793**：傳輸控制協定（TCP）
- **RFC 768**：使用者資料報協定（UDP）
- **MQTT 3.1.1**：MQTT 協定規範
- **RFC 7252**：受限應用協定（CoAP）
