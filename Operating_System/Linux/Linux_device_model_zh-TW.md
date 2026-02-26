## Linux 裝置模型

### 裝置模型的一小部分

![Small piece](../images/small_piece_device_model.png)

### 架構圖

![Architecture](../images/linux_device_model_architecture.jpg)

### 範例

#### I2C 充電器範例

![Example1](../images/linux_device_model_example.png)

![Example2](../images/linux_device_model_example_2.png)

![Example3](../images/linux_device_model_example_3.png)

#### 誰呼叫 i2c_new_device？

![Example4](../images/linux_device_model_example_4.png)

### 裝置樹 dtsi 檔案

![dtsi](../images/dtsi_example.png)

![dtsi2](../images/dtsi_example_2.png)

![dtsi_parse](../images/dtsi_parse.png)

裝置節點建立流程：

    of_platform_populate
        -> of_platform_bus_create
            -> of_platform_device_create_pdata
                -> of_device_add
		            ->device_add
                        ->bus_probe_device


### 參考資料

_Linux Device Drivers, 3rd Edition_

http://www.slideshare.net/jserv/linux-discovery

http://m.blog.chinaunix.net/uid-29955651-id-5095220.html

http://www.cs.fsu.edu/~baker/devices/notes/ch14.html#(1)
