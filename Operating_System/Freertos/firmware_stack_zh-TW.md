## FreeRTOS 韌體堆疊

### 韌體堆疊層次圖

![韌體堆疊](../images/FreeRTOS_firmware_stack.png)

- 使用者程式碼能夠存取相同的 FreeRTOS API，不受底層硬體移植實作的影響。
- FreeRTOS __不會__阻止__使用者程式碼__使用供應商提供的驅動程式、CMSIS 或直接操作硬體暫存器。
