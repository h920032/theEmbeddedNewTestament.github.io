# DMA Programming

> **Direct Memory Access for System Performance**  
> Understanding DMA principles and programming for efficient data transfer in embedded systems

---

## 📋 **Table of Contents**

- [🎯 Quick Cap](#quick-cap) - What is this and why do interviewers care?
- [🔍 Deep Dive](#deep-dive) - Technical details you need to know
- [💼 Interview Focus](#interview-focus) - Common questions and how to answer them
- [🧪 Practice](#practice) - Test your knowledge with problems and scenarios
- [🏭 Real-World Tie-In](#real-world-tie-in) - How this applies in actual embedded jobs
- [✅ Checklist](#checklist) - Are you ready for interviews on this topic?
- [📚 Extra Resources](#extra-resources) - Where to learn more

---

## 🎯 Quick Cap

Direct Memory Access (DMA) is a hardware mechanism that allows data transfer between memory and peripherals without CPU involvement, significantly improving system performance. Embedded engineers care about DMA because it enables high-bandwidth data operations while freeing the CPU for other tasks, making it essential for real-time systems, high-performance applications, and power-sensitive designs. In automotive systems, DMA is critical for handling high-speed sensor data, audio processing, and communication protocols without impacting real-time control loops.

## 🔍 Deep Dive

### 🔄 **DMA Fundamentals**

#### **What is DMA?**

Direct Memory Access (DMA) is a hardware mechanism that allows data to be transferred between memory and peripherals without involving the CPU. DMA controllers handle data transfers autonomously, freeing the CPU to perform other tasks and significantly improving system performance for high-bandwidth data operations.

#### **The Philosophy of DMA**

DMA represents a fundamental shift in system architecture philosophy:

**Performance Philosophy:**
- **CPU Offloading**: Free CPU from repetitive data transfer tasks
- **Parallel Operation**: Enable CPU and DMA to operate simultaneously
- **Bandwidth Optimization**: Maximize data transfer bandwidth
- **Efficiency Improvement**: Improve overall system efficiency

**System Architecture Philosophy:**
DMA enables more sophisticated system architectures:
- **Resource Separation**: Separate data transfer from data processing
- **Pipeline Operation**: Enable pipeline processing of data
- **Real-Time Performance**: Improve real-time system performance
- **Scalability**: Scale data transfer with system requirements

#### **DMA Functions and Responsibilities**

Modern DMA systems perform multiple critical functions:

**Primary Functions:**
- **Data Transfer**: Transfer data between memory and peripherals
- **CPU Offloading**: Free CPU from data transfer operations
- **Bandwidth Management**: Manage data transfer bandwidth
- **Synchronization**: Synchronize data transfer with system operation

**Secondary Functions:**
- **Memory Management**: Manage memory access patterns
- **Peripheral Control**: Control peripheral data transfer
- **Error Handling**: Handle data transfer errors
- **Performance Monitoring**: Monitor data transfer performance

### **DMA vs. CPU Transfer: Understanding the Trade-offs**

Understanding when to use DMA versus CPU transfer is fundamental:

#### **CPU Transfer Characteristics**

CPU-based transfers have specific characteristics:

**CPU Transfer Advantages:**
- **Simple Implementation**: Simple to implement and debug
- **Flexible Control**: Full control over transfer process
- **Data Processing**: Can process data during transfer
- **Error Handling**: Immediate error detection and handling

**CPU Transfer Disadvantages:**
- **CPU Overhead**: CPU cannot perform other tasks during transfer
- **Limited Bandwidth**: Limited by CPU memory access bandwidth
- **Power Consumption**: Higher power consumption during transfers
- **Scalability**: Limited scalability with transfer size

#### **DMA Transfer Characteristics**

DMA transfers have different characteristics:

**DMA Transfer Advantages:**
- **CPU Offloading**: CPU free for other tasks during transfer
- **High Bandwidth**: Can achieve maximum memory bandwidth
- **Power Efficiency**: Lower power consumption during transfers
- **Scalability**: Scales well with transfer size

**DMA Transfer Disadvantages:**
- **Complexity**: More complex to implement and debug
- **Setup Overhead**: Setup time required before transfer
- **Limited Control**: Less control over transfer process
- **Resource Requirements**: Requires dedicated DMA hardware

### 🏗️ **DMA Architecture and Operation**

#### **DMA Controller Architecture**

DMA controllers have specific architectural features:

#### **Basic DMA Controller Structure**

DMA controllers consist of several key components:

**Control Registers:**
- **Configuration Registers**: Configure transfer parameters
- **Status Registers**: Monitor transfer status
- **Control Registers**: Control transfer operation
- **Interrupt Registers**: Configure interrupt behavior

**Data Paths:**
- **Address Registers**: Source and destination addresses
- **Count Registers**: Transfer count and remaining count
- **Data Buffers**: Temporary data storage
- **Bus Interfaces**: Memory and peripheral bus interfaces

**Control Logic:**
- **Transfer Control**: Control transfer sequence
- **Arbitration**: Arbitrate between multiple DMA requests
- **Timing Control**: Control transfer timing
- **Error Detection**: Detect and handle transfer errors

#### **DMA Controller Types**

Different DMA controller types serve different applications:

**Single-Channel DMA:**
- **Simple Design**: Single transfer channel
- **Basic Functionality**: Basic transfer capabilities
- **Cost Effective**: Lower cost implementation
- **Limited Performance**: Limited performance capabilities

**Multi-Channel DMA:**
- **Multiple Channels**: Multiple independent transfer channels
- **Advanced Features**: Advanced transfer features
- **High Performance**: High-performance capabilities
- **Complex Design**: More complex design and operation

**Scatter-Gather DMA:**
- **Scatter-Gather**: Support for non-contiguous memory transfers
- **Memory Management**: Advanced memory management
- **High Efficiency**: High efficiency for complex transfers
- **Advanced Control**: Advanced transfer control

### **DMA Operation Principles**

Understanding how DMA operates is fundamental to effective use:

#### **Transfer Sequence**

DMA transfers follow a specific sequence:

**Initialization Phase:**
1. **Configuration**: Configure DMA controller parameters
2. **Address Setup**: Set source and destination addresses
3. **Count Setup**: Set transfer count
4. **Channel Enable**: Enable DMA channel

**Transfer Phase:**
1. **Request Detection**: Detect DMA transfer request
2. **Bus Acquisition**: Acquire memory and peripheral buses
3. **Data Transfer**: Transfer data between source and destination
4. **Address Update**: Update addresses for next transfer
5. **Count Update**: Update remaining transfer count

**Completion Phase:**
1. **Transfer Complete**: Detect transfer completion
2. **Interrupt Generation**: Generate completion interrupt
3. **Status Update**: Update transfer status
4. **Channel Disable**: Disable DMA channel

#### **Bus Arbitration and Control**

DMA controllers must coordinate with other system components:

**Bus Arbitration:**
- **Priority System**: Priority-based bus access
- **Fairness**: Ensure fair access to system resources
- **Efficiency**: Maximize bus utilization
- **Conflict Resolution**: Resolve access conflicts

**Memory Access Control:**
- **Address Validation**: Validate memory addresses
- **Access Rights**: Check memory access rights
- **Cache Coherency**: Maintain cache coherency
- **Memory Protection**: Enforce memory protection

### 🔀 **DMA Transfer Modes**

#### **Transfer Mode Philosophy**

Different transfer modes serve different application requirements:

#### **Single Transfer Mode**

Single transfer mode transfers one data unit per request:

**Single Transfer Characteristics:**
- **One Unit Per Request**: Transfer one data unit per DMA request
- **Simple Operation**: Simple transfer operation
- **Low Latency**: Low latency for individual transfers
- **Limited Throughput**: Limited overall throughput

**Single Transfer Applications:**
- **Low-Frequency Transfers**: Low-frequency data transfers
- **Real-Time Systems**: Real-time systems with strict timing
- **Simple Peripherals**: Simple peripheral interfaces
- **Debugging**: Debugging and testing scenarios

#### **Burst Transfer Mode**

Burst transfer mode transfers multiple data units per request:

**Burst Transfer Characteristics:**
- **Multiple Units Per Request**: Transfer multiple data units per request
- **High Throughput**: High overall transfer throughput
- **Efficient Memory Access**: Efficient memory access patterns
- **Higher Latency**: Higher latency for individual transfers

**Burst Transfer Applications:**
- **High-Bandwidth Transfers**: High-bandwidth data transfers
- **Bulk Data Operations**: Bulk data operations
- **Streaming Applications**: Streaming data applications
- **Performance-Critical Systems**: Performance-critical systems

### **Advanced Transfer Modes**

Advanced transfer modes provide sophisticated capabilities:

#### **Scatter-Gather Mode**

Scatter-gather mode supports non-contiguous memory transfers:

**Scatter-Gather Characteristics:**
- **Non-Contiguous Memory**: Support for non-contiguous memory regions
- **Descriptor Lists**: Use descriptor lists for transfer control
- **Flexible Addressing**: Flexible addressing capabilities
- **Complex Transfers**: Support for complex transfer patterns

**Scatter-Gather Applications:**
- **Memory Management**: Advanced memory management systems
- **File Systems**: File system data transfers
- **Network Processing**: Network packet processing
- **Multimedia Applications**: Multimedia data processing

#### **Linked List Mode**

Linked list mode uses linked descriptors for complex transfers:

**Linked List Characteristics:**
- **Descriptor Linking**: Link multiple transfer descriptors
- **Dynamic Control**: Dynamic transfer control
- **Complex Sequences**: Support for complex transfer sequences
- **Memory Efficiency**: Memory-efficient descriptor storage

**Linked List Applications:**
- **Complex Protocols**: Complex communication protocols
- **Data Processing Pipelines**: Data processing pipelines
- **Real-Time Systems**: Real-time systems with complex requirements
- **High-Performance Computing**: High-performance computing applications

### 💻 **DMA Programming Models**

#### **Programming Model Philosophy**

Different programming models serve different development approaches:

#### **Register-Based Programming**

Register-based programming provides direct hardware control:

**Register-Based Characteristics:**
- **Direct Control**: Direct control over hardware registers
- **Low Overhead**: Low software overhead
- **High Performance**: High performance operation
- **Complex Implementation**: Complex implementation requirements

**Register-Based Applications:**
- **Performance-Critical Systems**: Performance-critical systems
- **Real-Time Systems**: Real-time systems with strict timing
- **Low-Level Control**: Low-level hardware control
- **Custom Implementations**: Custom DMA implementations

#### **Driver-Based Programming**

Driver-based programming provides abstraction and portability:

**Driver-Based Characteristics:**
- **Hardware Abstraction**: Hardware abstraction layer
- **Portability**: Portable across different hardware
- **Ease of Use**: Easier to implement and maintain
- **Performance Overhead**: Some performance overhead

**Driver-Based Applications:**
- **General-Purpose Systems**: General-purpose embedded systems
- **Portable Applications**: Portable applications
- **Rapid Development**: Rapid development scenarios
- **Maintenance**: Long-term maintenance scenarios

### **Programming Interface Design**

Programming interface design affects ease of use and performance:

#### **Synchronous Programming Interface**

Synchronous interfaces block until transfer completion:

**Synchronous Characteristics:**
- **Blocking Operation**: Block until transfer completion
- **Simple Programming**: Simple programming model
- **Deterministic Behavior**: Deterministic behavior
- **Limited Concurrency**: Limited concurrency capabilities

**Synchronous Applications:**
- **Simple Transfers**: Simple data transfer scenarios
- **Sequential Processing**: Sequential data processing
- **Debugging**: Debugging and testing scenarios
- **Learning**: Learning and educational purposes

#### **Asynchronous Programming Interface**

Asynchronous interfaces use callbacks or events:

**Asynchronous Characteristics:**
- **Non-Blocking Operation**: Non-blocking operation
- **Event-Driven**: Event-driven programming model
- **High Concurrency**: High concurrency capabilities
- **Complex Programming**: More complex programming model

**Asynchronous Applications:**
- **High-Performance Systems**: High-performance systems
- **Real-Time Systems**: Real-time systems
- **Multitasking**: Multitasking environments
- **Event-Driven Systems**: Event-driven systems

### ⚙️ **DMA Configuration and Control**

#### **Configuration Philosophy**

DMA configuration determines transfer behavior and performance:

#### **Transfer Parameter Configuration**

Transfer parameters control transfer behavior:

**Address Configuration:**
- **Source Address**: Source memory or peripheral address
- **Destination Address**: Destination memory or peripheral address
- **Address Increment**: Address increment mode and value
- **Address Wrapping**: Address wrapping behavior

**Transfer Configuration:**
- **Transfer Size**: Size of each data unit
- **Transfer Count**: Total number of transfers
- **Transfer Direction**: Transfer direction (memory to peripheral, etc.)
- **Transfer Priority**: Transfer priority level

**Timing Configuration:**
- **Transfer Rate**: Transfer rate and timing
- **Burst Length**: Burst transfer length
- **Wait States**: Wait states for slow peripherals
- **Timing Control**: Precise timing control

#### **Channel Configuration**

Channel configuration controls DMA channel behavior:

**Channel Selection:**
- **Channel Assignment**: Assign channels to specific peripherals
- **Channel Priority**: Set channel priority levels
- **Channel Sharing**: Share channels between peripherals
- **Channel Reservation**: Reserve channels for critical transfers

**Channel Control:**
- **Channel Enable**: Enable or disable channels
- **Channel Reset**: Reset channel state
- **Channel Status**: Monitor channel status
- **Channel Error Handling**: Handle channel errors

### **Dynamic Configuration**

Dynamic configuration enables runtime adaptation:

#### **Runtime Parameter Changes**

Runtime parameter changes enable adaptive operation:

**Parameter Updates:**
- **Address Updates**: Update transfer addresses
- **Count Updates**: Update transfer counts
- **Mode Changes**: Change transfer modes
- **Priority Changes**: Change transfer priorities

**Configuration Switching:**
- **Profile Switching**: Switch between configuration profiles
- **Mode Switching**: Switch between transfer modes
- **Parameter Optimization**: Optimize parameters based on conditions
- **Adaptive Control**: Adaptive control based on system state

### 🔔 **DMA Interrupts and Synchronization**

#### **Interrupt Philosophy**

Interrupts provide synchronization and error handling capabilities:

#### **Interrupt Types and Usage**

Different interrupt types serve different purposes:

**Completion Interrupts:**
- **Transfer Complete**: Transfer completion notification
- **Block Complete**: Block transfer completion
- **Channel Complete**: Channel completion notification
- **System Complete**: System-level completion notification

**Error Interrupts:**
- **Transfer Errors**: Transfer error notifications
- **Configuration Errors**: Configuration error notifications
- **Hardware Errors**: Hardware error notifications
- **Timeout Errors**: Timeout error notifications

**Status Interrupts:**
- **Status Changes**: Status change notifications
- **Threshold Reached**: Threshold reached notifications
- **Condition Met**: Condition met notifications
- **State Changes**: State change notifications

#### **Interrupt Handling**

Interrupt handling affects system performance and reliability:

**Interrupt Service Routines:**
- **Quick Processing**: Process interrupts quickly
- **Minimal Processing**: Minimal processing in ISR
- **Error Handling**: Handle errors appropriately
- **Status Updates**: Update system status

**Interrupt Priorities:**
- **Priority Assignment**: Assign appropriate priorities
- **Priority Management**: Manage interrupt priorities
- **Priority Inversion**: Avoid priority inversion
- **Real-Time Requirements**: Meet real-time requirements

### **Synchronization Mechanisms**

Synchronization mechanisms coordinate DMA with other system components:

#### **Software Synchronization**

Software synchronization provides programmatic control:

**Polling:**
- **Status Polling**: Poll DMA status registers
- **Completion Polling**: Poll for transfer completion
- **Error Polling**: Poll for error conditions
- **Performance Impact**: Consider performance impact

**Event Flags:**
- **Event Setting**: Set event flags on completion
- **Event Waiting**: Wait for event flags
- **Event Clearing**: Clear event flags
- **Event Management**: Manage event flag lifecycle

#### **Hardware Synchronization**

Hardware synchronization provides automatic coordination:

**Handshake Signals:**
- **Request Signals**: DMA request signals
- **Acknowledge Signals**: DMA acknowledge signals
- **Ready Signals**: Peripheral ready signals
- **Busy Signals**: Peripheral busy signals

**Hardware Triggers:**
- **Timer Triggers**: Timer-based triggers
- **External Triggers**: External signal triggers
- **Conditional Triggers**: Condition-based triggers
- **Sequential Triggers**: Sequential operation triggers

### 🚀 **Advanced DMA Features**

#### **Advanced Feature Philosophy**

Advanced features enable sophisticated DMA applications:

#### **Memory Management Features**

Memory management features optimize memory usage:

**Memory Protection:**
- **Access Control**: Control memory access rights
- **Boundary Checking**: Check memory boundaries
- **Protection Violations**: Handle protection violations
- **Security**: Enhance system security

**Memory Optimization:**
- **Cache Management**: Manage cache coherency
- **Memory Alignment**: Optimize memory alignment
- **Prefetching**: Implement data prefetching
- **Memory Pooling**: Use memory pools for efficiency

#### **Performance Enhancement Features**

Performance enhancement features improve transfer efficiency:

**Transfer Optimization:**
- **Burst Optimization**: Optimize burst transfers
- **Pipeline Operation**: Implement pipeline operation
- **Parallel Transfers**: Enable parallel transfers
- **Transfer Chaining**: Chain multiple transfers

**Bandwidth Management:**
- **Bandwidth Allocation**: Allocate bandwidth between channels
- **Quality of Service**: Implement quality of service
- **Dynamic Adjustment**: Dynamically adjust bandwidth
- **Performance Monitoring**: Monitor transfer performance

### **Specialized DMA Features**

Specialized features address specific application requirements:

#### **Real-Time Features**

Real-time features support real-time applications:

**Timing Control:**
- **Precise Timing**: Precise transfer timing control
- **Deadline Management**: Manage transfer deadlines
- **Jitter Control**: Control transfer jitter
- **Synchronization**: Synchronize with external events

**Predictability:**
- **Deterministic Behavior**: Ensure deterministic behavior
- **Worst-Case Analysis**: Support worst-case analysis
- **Real-Time Guarantees**: Provide real-time guarantees
- **Performance Bounds**: Establish performance bounds

#### **Security Features**

Security features enhance system security:

**Access Control:**
- **Permission Checking**: Check access permissions
- **Secure Channels**: Implement secure channels
- **Isolation**: Isolate different DMA channels
- **Audit Trails**: Maintain audit trails

**Data Protection:**
- **Encryption**: Support data encryption
- **Integrity Checking**: Check data integrity
- **Secure Storage**: Secure storage of sensitive data
- **Tamper Detection**: Detect tampering attempts

### Common Pitfalls & Misconceptions

<Callout>
**Pitfall: Not Considering DMA Setup Overhead**
Many developers assume DMA is always faster than CPU transfers, but for small transfers, the DMA setup overhead can exceed the transfer time, making CPU transfers more efficient.

**Misconception: DMA Transfers are Always Asynchronous**
While DMA transfers can be asynchronous, they can also be configured for synchronous operation. The programming model depends on the specific DMA controller and application requirements.
</Callout>

### Real Debugging Story

In a high-performance audio processing system, the team was experiencing audio dropouts and timing issues. Traditional debugging showed that the CPU was spending too much time handling audio data transfers. When they implemented DMA for the audio interface, they discovered that the DMA controller was configured for single transfers instead of burst transfers, causing excessive bus overhead. The issue was resolved by configuring the DMA for burst transfers with appropriate burst lengths, which significantly improved audio performance and eliminated dropouts.

### Performance vs. Resource Trade-offs

| DMA Feature | Performance Impact | Complexity Impact | Resource Usage |
|-------------|-------------------|-------------------|----------------|
| **Burst Transfers** | Higher throughput | Moderate complexity | Higher bus usage |
| **Scatter-Gather** | Better memory efficiency | Higher complexity | More memory for descriptors |
| **Multiple Channels** | Parallel transfers | Higher complexity | More hardware resources |
| **Interrupt-Driven** | Better CPU efficiency | Higher complexity | More interrupt overhead |

**What embedded interviewers want to hear is** that you understand when to use DMA vs. CPU transfers, that you can configure DMA for optimal performance, and that you know how to handle DMA interrupts and synchronization in real-time systems.

## 💼 Interview Focus

### Classic Embedded Interview Questions

1. **"When would you choose DMA over CPU-based transfers?"**
2. **"How do you configure DMA for optimal performance?"**
3. **"What are the trade-offs between different DMA transfer modes?"**
4. **"How do you handle DMA interrupts in a real-time system?"**
5. **"What are the challenges of DMA in multi-core systems?"**

### Model Answer Starters

1. **"I choose DMA when I need high-bandwidth transfers, want to free the CPU for other tasks, or have large data blocks to transfer. For small transfers, CPU transfers can be more efficient due to DMA setup overhead..."**
2. **"For optimal DMA performance, I configure burst transfers with appropriate burst lengths, use scatter-gather for non-contiguous memory, and ensure proper memory alignment..."**
3. **"Different transfer modes serve different needs: single transfers for real-time systems, burst transfers for high bandwidth, and scatter-gather for complex memory patterns..."

### Trap Alerts

- **Trap**: Assuming DMA is always faster than CPU transfers
- **Trap**: Not considering DMA setup overhead for small transfers
- **Trap**: Ignoring cache coherency issues in DMA transfers

## 🧪 Practice

<Quiz>
**Question**: For which type of data transfer would DMA be most beneficial?

A) Small, frequent transfers of 1-2 bytes
B) Large, infrequent transfers of 1KB blocks
C) Medium transfers with complex data processing
D) All of the above

**Answer**: B) Large, infrequent transfers of 1KB blocks. DMA is most beneficial for large transfers where the setup overhead is amortized over the transfer size. Small transfers often have higher overhead than the actual transfer time, making CPU transfers more efficient.
</Quiz>

### Coding Task
Implement a DMA-based circular buffer:

```c
// Implement a DMA circular buffer for audio data
typedef struct {
    uint8_t* buffer;
    uint32_t size;
    uint32_t head;
    uint32_t tail;
    uint32_t count;
    DMA_HandleTypeDef* dma_handle;
} dma_circular_buffer_t;

// Your tasks:
// 1. Implement DMA circular buffer initialization
// 2. Add DMA transfer configuration for audio data
// 3. Implement interrupt handling for transfer completion
// 4. Add buffer management functions (read/write)
// 5. Test with different transfer sizes and frequencies
```

### Debugging Scenario
Your embedded system is experiencing intermittent data corruption in DMA transfers. The corruption seems to occur more frequently under high load conditions. How would you investigate and resolve this DMA issue?

### System Design Question
Design a DMA system for a multi-core embedded processor that must handle high-speed sensor data, audio processing, and communication protocols while maintaining real-time performance guarantees.

## 🏭 Real-World Tie-In

### In Embedded Development
At Qualcomm, DMA is critical for their mobile processors handling high-speed data streams from cameras, audio interfaces, and wireless communications. The team uses sophisticated DMA controllers with scatter-gather capabilities to efficiently manage complex data flows while minimizing CPU overhead and power consumption.

### On the Production Line
In automotive manufacturing, DMA is essential for handling high-speed sensor data from multiple sources. Companies like Bosch and Continental use DMA controllers to process sensor data from radar, lidar, and camera systems in real-time, ensuring that safety-critical control systems receive data without delay.

### In the Industry
The telecommunications industry relies heavily on DMA for network packet processing. Companies like Cisco and Ericsson use advanced DMA systems with multiple channels and scatter-gather capabilities to handle high-speed network traffic, ensuring low-latency packet processing for 5G and fiber networks.

## ✅ Checklist

<Checklist>
- [ ] Understand DMA fundamentals and when to use DMA vs. CPU transfers
- [ ] Know how to configure DMA controllers for different transfer modes
- [ ] Understand DMA interrupt handling and synchronization
- [ ] Be able to implement DMA in embedded systems
- [ ] Know how to optimize DMA performance for specific applications
- [ ] Understand DMA challenges in multi-core systems
- [ ] Be able to debug DMA-related issues
- [ ] Know how to handle cache coherency in DMA transfers
</Checklist>

## 📚 Extra Resources

### Recommended Reading

- **"Embedded Systems Design" by various authors** - Comprehensive DMA implementation
- **"Computer Architecture" by various authors** - Memory hierarchy and DMA principles
- **"Real-Time Systems" by various authors** - DMA in real-time applications

### Online Resources

- **DMA Simulators** - Tools for DMA simulation and analysis
- **Manufacturer Documentation** - DMA controller specifications and examples
- **Code Examples** - Open-source DMA implementations and libraries

### Practice Exercises

1. **DMA configuration** - Configure DMA for different transfer modes
2. **Interrupt handling** - Implement DMA interrupt service routines
3. **Performance optimization** - Optimize DMA for specific applications
4. **Debugging practice** - Debug common DMA issues and problems

---

**Next Topic**: [Cache Management and Coherency](./Cache_Management_Coherency.md) → [Memory Protection Units](./Memory_Protection_Units.md)
