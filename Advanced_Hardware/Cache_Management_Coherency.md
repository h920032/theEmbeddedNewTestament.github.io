# Cache Management and Coherency

> **Optimizing Memory Access and Maintaining Data Consistency**  
> Understanding cache management principles and coherency protocols for high-performance embedded systems

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

Cache management and coherency are critical concepts for optimizing memory access performance in embedded systems. Embedded engineers care about these topics because they directly impact system performance, power consumption, and real-time behavior. Cache coherency ensures that multiple processor cores see consistent data views, preventing bugs that could cause system failures. In automotive systems, cache coherency is essential for ensuring that safety-critical functions across multiple cores always operate on the most current sensor data.

## 🔍 Deep Dive

### 🚀 **Cache Fundamentals**

#### **What is Cache Memory?**

Cache memory is a small, high-speed memory system that stores frequently accessed data and instructions, positioned between the CPU and main memory. It acts as a buffer to reduce the average time to access data from main memory, significantly improving system performance by exploiting temporal and spatial locality in program behavior.

#### **The Philosophy of Cache Memory**

Cache memory represents a fundamental optimization philosophy in computer architecture:

**Performance Philosophy:**
- **Memory Hierarchy**: Optimize memory access through hierarchical organization
- **Locality Exploitation**: Exploit temporal and spatial locality in programs
- **Access Time Reduction**: Reduce average memory access time
- **Bandwidth Optimization**: Optimize memory bandwidth utilization

**System Architecture Philosophy:**
Cache enables more sophisticated system architectures:
- **Performance Scaling**: Scale performance with memory access patterns
- **Power Efficiency**: Reduce power consumption through faster access
- **Cost Optimization**: Balance performance with cost constraints
- **Scalability**: Scale cache performance with system requirements

#### **Cache Functions and Responsibilities**

Modern cache systems perform multiple critical functions:

**Primary Functions:**
- **Data Storage**: Store frequently accessed data and instructions
- **Access Acceleration**: Accelerate memory access operations
- **Bandwidth Management**: Manage memory bandwidth efficiently
- **Performance Optimization**: Optimize overall system performance

**Secondary Functions:**
- **Power Management**: Reduce power consumption through faster access
- **Memory Protection**: Provide memory protection capabilities
- **Coherency Maintenance**: Maintain data coherency across system
- **Performance Monitoring**: Monitor cache performance metrics

### **Cache vs. Main Memory: Understanding the Trade-offs**

Understanding the relationship between cache and main memory is fundamental:

#### **Cache Characteristics**

Cache memory has specific characteristics:

**Cache Advantages:**
- **High Speed**: Much faster access than main memory
- **Low Latency**: Low access latency
- **High Bandwidth**: High bandwidth for cache hits
- **Power Efficiency**: Lower power consumption per access

**Cache Disadvantages:**
- **Limited Capacity**: Limited storage capacity
- **Higher Cost**: Higher cost per bit than main memory
- **Complexity**: Complex management and coherency requirements
- **Unpredictability**: Variable access times (hits vs. misses)

#### **Main Memory Characteristics**

Main memory has different characteristics:

**Main Memory Advantages:**
- **Large Capacity**: Large storage capacity
- **Lower Cost**: Lower cost per bit
- **Persistence**: Data persists across power cycles
- **Simplicity**: Simpler management requirements

**Main Memory Disadvantages:**
- **Lower Speed**: Slower access than cache
- **Higher Latency**: Higher access latency
- **Lower Bandwidth**: Lower bandwidth than cache
- **Higher Power**: Higher power consumption per access

### 🏗️ **Cache Architecture and Organization**

#### **Cache Organization Philosophy**

Cache organization determines performance characteristics and management complexity:

#### **Cache Mapping Strategies**

Different mapping strategies serve different performance goals:

**Direct Mapped Cache:**
- **Simple Design**: Simple hardware implementation
- **Fast Access**: Fast cache access time
- **Low Cost**: Lower hardware cost
- **Conflict Misses**: Higher conflict miss rate

**Set Associative Cache:**
- **Balanced Design**: Balance between performance and complexity
- **Reduced Conflicts**: Reduced conflict miss rate
- **Moderate Cost**: Moderate hardware cost
- **Flexible Mapping**: Flexible data placement

**Fully Associative Cache:**
- **Maximum Flexibility**: Maximum flexibility in data placement
- **Lowest Miss Rate**: Lowest miss rate for given capacity
- **Highest Cost**: Highest hardware cost
- **Complex Implementation**: Complex hardware implementation

#### **Cache Line Organization**

Cache line organization affects performance and efficiency:

**Line Size Considerations:**
- **Spatial Locality**: Larger lines better exploit spatial locality
- **Transfer Efficiency**: Larger lines improve memory transfer efficiency
- **Conflict Misses**: Larger lines may increase conflict misses
- **Memory Bandwidth**: Larger lines require more memory bandwidth

**Line Structure:**
- **Data Field**: Store actual data or instructions
- **Tag Field**: Store address information for identification
- **Status Bits**: Store cache line status information
- **Metadata**: Store additional management information

### **Cache Hierarchy Design**

Cache hierarchy design optimizes overall system performance:

#### **Multi-Level Cache Philosophy**

Multi-level caches provide performance and cost optimization:

**Level 1 (L1) Cache:**
- **Fastest Access**: Fastest access time
- **Smallest Size**: Smallest capacity
- **Highest Cost**: Highest cost per bit
- **CPU Integration**: Integrated with CPU

**Level 2 (L2) Cache:**
- **Medium Access**: Medium access time
- **Medium Size**: Medium capacity
- **Medium Cost**: Medium cost per bit
- **External Integration**: External to CPU

**Level 3 (L3) Cache:**
- **Slower Access**: Slower access time
- **Larger Size**: Larger capacity
- **Lower Cost**: Lower cost per bit
- **Shared Access**: Shared between multiple cores

#### **Cache Coherency Architecture**

Cache coherency architecture ensures data consistency:

**Snooping Protocol:**
- **Bus Monitoring**: Monitor bus transactions
- **Coherency Actions**: Take coherency actions as needed
- **Simple Implementation**: Simple hardware implementation
- **Scalability Limits**: Limited scalability with bus bandwidth

**Directory-Based Protocol:**
- **Centralized Directory**: Centralized coherency directory
- **Scalable Design**: Scalable to many cores
- **Complex Implementation**: More complex hardware implementation
- **Higher Latency**: Higher coherency operation latency

### 🎯 **Cache Management Strategies**

#### **Replacement Policy Philosophy**

Replacement policies determine which cache lines to evict:

#### **Replacement Algorithm Fundamentals**

Different replacement algorithms serve different performance goals:

**Least Recently Used (LRU):**
- **Temporal Locality**: Exploit temporal locality
- **Predictable Behavior**: Predictable replacement behavior
- **Hardware Complexity**: Complex hardware implementation
- **Performance**: Good performance for most workloads

**First In, First Out (FIFO):**
- **Simple Implementation**: Simple hardware implementation
- **Predictable Behavior**: Predictable replacement behavior
- **Limited Locality**: Limited temporal locality exploitation
- **Performance**: Moderate performance for most workloads

**Random Replacement:**
- **Simple Implementation**: Simple hardware implementation
- **Unpredictable Behavior**: Unpredictable replacement behavior
- **No Locality**: No temporal locality exploitation
- **Performance**: Variable performance depending on workload

#### **Advanced Replacement Policies**

Advanced policies optimize for specific workloads:

**Adaptive Replacement:**
- **Workload Adaptation**: Adapt to workload characteristics
- **Performance Optimization**: Optimize for specific workloads
- **Complex Implementation**: Complex hardware implementation
- **Dynamic Behavior**: Dynamic replacement behavior

**Application-Specific Policies:**
- **Workload Knowledge**: Use workload knowledge for optimization
- **Specialized Optimization**: Optimize for specific applications
- **Custom Implementation**: Custom hardware implementation
- **Maximum Performance**: Maximum performance for target workloads

### **Write Policy Management**

Write policies determine how cache updates are handled:

#### **Write-Through vs. Write-Back**

Different write policies serve different consistency requirements:

**Write-Through Policy:**
- **Immediate Consistency**: Immediate consistency with main memory
- **Simple Implementation**: Simple hardware implementation
- **Higher Bandwidth**: Higher memory bandwidth requirements
- **Lower Performance**: Lower cache performance

**Write-Back Policy:**
- **Deferred Consistency**: Deferred consistency with main memory
- **Complex Implementation**: Complex hardware implementation
- **Lower Bandwidth**: Lower memory bandwidth requirements
- **Higher Performance**: Higher cache performance

#### **Write Allocation Strategies**

Write allocation strategies affect cache performance:

**Write Allocate:**
- **Cache Loading**: Load cache line on write miss
- **Spatial Locality**: Better spatial locality exploitation
- **Higher Bandwidth**: Higher memory bandwidth requirements
- **Better Performance**: Better performance for most workloads

**No-Write Allocate:**
- **Direct Write**: Write directly to main memory
- **Lower Bandwidth**: Lower memory bandwidth requirements
- **Reduced Conflicts**: Reduced cache conflicts
- **Variable Performance**: Variable performance depending on workload

### 🔄 **Cache Coherency Protocols**

#### **Coherency Protocol Philosophy**

Cache coherency protocols ensure data consistency across the system:

#### **Coherency Problem Understanding**

Understanding coherency problems is fundamental:

**Read-Write Coherency:**
- **Multiple Readers**: Multiple cores reading same data
- **Single Writer**: Single core writing to data
- **Consistency Requirements**: Ensure consistent data view
- **Performance Impact**: Coherency operations impact performance

**Write-Write Coherency:**
- **Multiple Writers**: Multiple cores writing to same data
- **Sequential Consistency**: Ensure sequential consistency
- **Ordering Requirements**: Maintain write ordering
- **Complexity**: Complex ordering requirements

#### **Protocol Categories**

Different protocol categories serve different system requirements:

**MESI Protocol:**
- **Four States**: Modified, Exclusive, Shared, Invalid states
- **Simple Implementation**: Simple hardware implementation
- **Good Performance**: Good performance for most workloads
- **Limited Scalability**: Limited scalability with core count

**MOESI Protocol:**
- **Five States**: Modified, Owned, Exclusive, Shared, Invalid states
- **Enhanced Performance**: Enhanced performance for some workloads
- **Complex Implementation**: More complex hardware implementation
- **Better Scalability**: Better scalability than MESI

### **Protocol Implementation Details**

Protocol implementation affects performance and complexity:

#### **State Transitions**

State transitions implement coherency protocols:

**Transition Triggers:**
- **CPU Operations**: CPU read and write operations
- **Bus Transactions**: Bus snooping operations
- **Coherency Actions**: Coherency protocol actions
- **External Requests**: External coherency requests

**Transition Actions:**
- **State Changes**: Change cache line state
- **Data Transfers**: Transfer data between caches
- **Invalidation**: Invalidate cache lines
- **Update Propagation**: Propagate updates to other caches

#### **Performance Optimization**

Performance optimization improves coherency efficiency:

**Reduced Coherency Traffic:**
- **Efficient Protocols**: Use efficient coherency protocols
- **Smart Invalidation**: Smart invalidation strategies
- **Update Coalescing**: Coalesce multiple updates
- **Bandwidth Optimization**: Optimize coherency bandwidth usage

**Latency Reduction:**
- **Fast Coherency**: Fast coherency operations
- **Parallel Processing**: Parallel coherency processing
- **Pipelined Operations**: Pipelined coherency operations
- **Optimized Paths**: Optimized coherency operation paths

### ⚡ **Cache Performance Optimization**

#### **Performance Optimization Philosophy**

Cache performance optimization balances multiple objectives:

#### **Hit Rate Optimization**

Hit rate optimization improves cache effectiveness:

**Capacity Optimization:**
- **Optimal Size**: Choose optimal cache size
- **Workload Analysis**: Analyze workload characteristics
- **Size Scaling**: Scale cache size with workload
- **Cost Balance**: Balance performance with cost

**Associativity Optimization:**
- **Conflict Reduction**: Reduce conflict misses
- **Associativity Scaling**: Scale associativity with workload
- **Hardware Cost**: Consider hardware cost implications
- **Performance Impact**: Evaluate performance impact

#### **Miss Rate Reduction**

Miss rate reduction improves overall performance:

**Compulsory Misses:**
- **First Access**: First access to data
- **Prefetching**: Use prefetching to reduce misses
- **Initialization**: Optimize data initialization
- **Workload Optimization**: Optimize workload patterns

**Capacity Misses:**
- **Cache Size**: Increase cache size
- **Data Locality**: Improve data locality
- **Memory Management**: Optimize memory management
- **Workload Analysis**: Analyze workload requirements

**Conflict Misses:**
- **Associativity**: Increase cache associativity
- **Data Placement**: Optimize data placement
- **Hash Functions**: Optimize hash functions
- **Conflict Avoidance**: Avoid conflict patterns

### **Advanced Optimization Techniques**

Advanced techniques provide sophisticated optimization:

#### **Prefetching Strategies**

Prefetching reduces compulsory misses:

**Hardware Prefetching:**
- **Pattern Detection**: Detect access patterns
- **Automatic Prefetching**: Automatic data prefetching
- **Performance Impact**: Evaluate performance impact
- **Bandwidth Usage**: Consider bandwidth usage

**Software Prefetching:**
- **Explicit Prefetching**: Explicit prefetch instructions
- **Compiler Optimization**: Compiler-generated prefetching
- **Performance Tuning**: Performance tuning for specific workloads
- **Portability**: Consider portability implications

#### **Cache Partitioning**

Cache partitioning optimizes for specific workloads:

**Static Partitioning:**
- **Fixed Allocation**: Fixed cache allocation
- **Workload Optimization**: Optimize for specific workloads
- **Predictable Performance**: Predictable performance characteristics
- **Limited Flexibility**: Limited flexibility for dynamic workloads

**Dynamic Partitioning:**
- **Adaptive Allocation**: Adaptive cache allocation
- **Workload Adaptation**: Adapt to changing workloads
- **Performance Optimization**: Optimize for current workload
- **Complex Implementation**: Complex hardware implementation

### 🏢 **Multi-Level Cache Systems**

#### **Multi-Level Cache Philosophy**

Multi-level caches provide performance and cost optimization:

#### **Hierarchy Design Principles**

Hierarchy design optimizes overall system performance:

**Performance Optimization:**
- **Access Time**: Minimize average access time
- **Bandwidth Utilization**: Optimize bandwidth utilization
- **Miss Rate**: Minimize overall miss rate
- **Cost Efficiency**: Optimize cost-performance ratio

**Scalability Considerations:**
- **Core Scaling**: Scale with increasing core count
- **Memory Scaling**: Scale with increasing memory size
- **Bandwidth Scaling**: Scale with increasing bandwidth
- **Power Scaling**: Scale with power constraints

#### **Level Interaction**

Level interaction affects overall performance:

**Inclusive Caches:**
- **Data Duplication**: Duplicate data across levels
- **Simple Management**: Simple cache management
- **Higher Bandwidth**: Higher bandwidth requirements
- **Consistent View**: Consistent view across levels

**Exclusive Caches:**
- **No Duplication**: No data duplication across levels
- **Complex Management**: Complex cache management
- **Lower Bandwidth**: Lower bandwidth requirements
- **Efficient Use**: Efficient use of cache capacity

### **Advanced Multi-Level Features**

Advanced features provide sophisticated capabilities:

#### **Unified vs. Split Caches**

Different cache organizations serve different purposes:

**Unified Caches:**
- **Shared Capacity**: Shared capacity for data and instructions
- **Flexible Allocation**: Flexible capacity allocation
- **Simplified Management**: Simplified cache management
- **Capacity Efficiency**: Efficient capacity utilization

**Split Caches:**
- **Specialized Optimization**: Specialized optimization for data and instructions
- **Independent Management**: Independent cache management
- **Performance Optimization**: Optimize for specific access patterns
- **Complex Implementation**: More complex hardware implementation

#### **Cache Coherency Across Levels**

Coherency across levels ensures data consistency:

**Level Coherency:**
- **Cross-Level Consistency**: Maintain consistency across levels
- **Protocol Coordination**: Coordinate coherency protocols
- **Performance Impact**: Consider performance impact
- **Complexity Management**: Manage implementation complexity

### 💻 **Cache-Aware Programming**

#### **Programming Philosophy**

Cache-aware programming optimizes for cache behavior:

#### **Memory Access Patterns**

Memory access patterns affect cache performance:

**Spatial Locality:**
- **Sequential Access**: Sequential memory access
- **Array Traversal**: Optimize array traversal patterns
- **Data Structure Layout**: Optimize data structure layout
- **Memory Alignment**: Consider memory alignment

**Temporal Locality:**
- **Repeated Access**: Repeated access to same data
- **Loop Optimization**: Optimize loop structures
- **Data Reuse**: Maximize data reuse
- **Cache-Friendly Algorithms**: Use cache-friendly algorithms

#### **Data Structure Optimization**

Data structure optimization improves cache performance:

**Cache Line Alignment:**
- **Line Boundaries**: Align to cache line boundaries
- **Padding Optimization**: Optimize padding for cache lines
- **Structure Size**: Consider cache line size in structure design
- **Access Patterns**: Optimize for expected access patterns

**Memory Layout:**
- **Contiguous Storage**: Use contiguous memory storage
- **Stride Optimization**: Optimize memory access strides
- **Cache Blocking**: Use cache blocking techniques
- **Memory Pooling**: Use memory pooling for efficiency

### **Advanced Programming Techniques**

Advanced techniques provide sophisticated optimization:

#### **Compiler Optimization**

Compiler optimization improves cache performance:

**Automatic Optimization:**
- **Loop Optimization**: Automatic loop optimization
- **Memory Access**: Optimize memory access patterns
- **Data Layout**: Optimize data layout
- **Performance Tuning**: Automatic performance tuning

**Profile-Guided Optimization:**
- **Workload Profiling**: Profile actual workload behavior
- **Optimization Targeting**: Target optimization to specific workloads
- **Performance Measurement**: Measure optimization effectiveness
- **Iterative Improvement**: Iterative optimization improvement

#### **Runtime Optimization**

Runtime optimization adapts to changing conditions:

**Adaptive Algorithms:**
- **Workload Adaptation**: Adapt to changing workloads
- **Performance Monitoring**: Monitor runtime performance
- **Dynamic Adjustment**: Dynamically adjust algorithms
- **Optimization Selection**: Select optimal algorithms at runtime

**Memory Management:**
- **Dynamic Allocation**: Dynamic memory allocation
- **Cache-Aware Allocation**: Cache-aware memory allocation
- **Memory Pooling**: Runtime memory pooling
- **Garbage Collection**: Cache-aware garbage collection

### Common Pitfalls & Misconceptions

<Callout>
**Pitfall: Ignoring Cache Coherency in Multi-Core Systems**
Many developers assume that writing to memory automatically updates all cores, but without proper cache coherency protocols, cores can see stale data, leading to subtle bugs that are difficult to reproduce.

**Misconception: Bigger Cache Always Means Better Performance**
While larger caches can improve hit rates, they also increase access latency and power consumption. The optimal cache size depends on the specific workload and system constraints.
</Callout>

### Real Debugging Story

In a multi-core automotive control system, the team was experiencing intermittent sensor reading errors that only occurred under specific timing conditions. Traditional debugging couldn't reproduce the issue consistently. When they analyzed the cache coherency behavior, they discovered that one core was reading stale sensor data from its local cache while another core had updated the sensor value. The issue was resolved by implementing proper cache invalidation protocols and using memory barriers to ensure data consistency across cores.

### Performance vs. Resource Trade-offs

| Cache Feature | Performance Impact | Power Consumption | Hardware Complexity |
|---------------|-------------------|-------------------|-------------------|
| **Larger Cache Size** | Higher hit rates | Higher power usage | Moderate complexity |
| **Higher Associativity** | Lower conflict misses | Higher power usage | Higher complexity |
| **Write-Back Policy** | Better performance | Lower bandwidth usage | Higher complexity |
| **Cache Coherency** | Data consistency | Higher power usage | High complexity |

**What embedded interviewers want to hear is** that you understand the fundamental trade-offs in cache design, that you can analyze cache performance issues in multi-core systems, and that you know how to optimize code for cache behavior while considering power and real-time constraints.

## 💼 Interview Focus

### Classic Embedded Interview Questions

1. **"How do you handle cache coherency issues in multi-core embedded systems?"**
2. **"What's the difference between write-through and write-back cache policies?"**
3. **"How would you optimize code for cache performance?"**
4. **"What are the trade-offs between different cache mapping strategies?"**
5. **"How do you debug cache-related performance issues?"**

### Model Answer Starters

1. **"For cache coherency in multi-core systems, I ensure proper use of memory barriers and cache invalidation protocols, and I'm careful about shared data access patterns..."**
2. **"Write-through immediately updates main memory but requires more bandwidth, while write-back defers updates to reduce bandwidth but requires more complex coherency management..."**
3. **"I optimize for cache performance by improving spatial and temporal locality through better data structure layout and access patterns..."

### Trap Alerts

- **Trap**: Assuming cache coherency is automatic in multi-core systems
- **Trap**: Optimizing for cache size without considering power consumption
- **Trap**: Ignoring cache behavior when designing real-time systems

## 🧪 Practice

<Quiz>
**Question**: In a multi-core embedded system, what happens if Core A writes to a memory location while Core B reads from the same location without proper cache coherency?

A) Core B always sees the updated value
B) Core B might see stale data from its local cache
C) The system crashes immediately
D) Both cores get the same value automatically

**Answer**: B) Core B might see stale data from its local cache. Without proper cache coherency protocols, each core maintains its own cache copy, and Core B might read an outdated value from its local cache while Core A has updated the value in its cache.
</Quiz>

### Coding Task
Implement a cache-friendly matrix multiplication algorithm:

```c
// Implement cache-optimized matrix multiplication
void matrix_multiply_cache_friendly(int* A, int* B, int* C, int N);

// Your tasks:
// 1. Implement the algorithm with cache blocking
// 2. Optimize for spatial locality
// 3. Consider cache line size in your implementation
// 4. Measure performance improvement over naive implementation
// 5. Analyze cache miss rates using profiling tools
```

### Debugging Scenario
Your multi-core embedded system is experiencing intermittent data corruption that only occurs under high load. The issue seems related to cache behavior. How would you approach debugging this problem?

### System Design Question
Design a cache architecture for a real-time embedded system that must meet strict timing requirements while supporting multiple cores and minimizing power consumption.

## 🏭 Real-World Tie-In

### In Embedded Development
At ARM, cache design is critical for their processor cores used in billions of embedded devices. The team optimizes cache architectures for different market segments, from power-efficient IoT devices to high-performance automotive systems, ensuring optimal balance of performance, power, and cost.

### On the Production Line
In semiconductor manufacturing, cache testing is essential for ensuring processor reliability. Companies like Intel and AMD use sophisticated cache testing methodologies to verify cache coherency and performance across millions of processor cores, preventing field failures.

### In the Industry
The automotive industry relies heavily on cache coherency for safety-critical systems. Companies like Bosch and Continental use cache-aware design principles to ensure that multiple processor cores in vehicle control systems always operate on consistent sensor and control data, preventing safety issues.

## ✅ Checklist

<Checklist>
- [ ] Understand cache hierarchy and memory organization
- [ ] Know the differences between cache mapping strategies
- [ ] Understand cache coherency protocols (MESI, MOESI)
- [ ] Be able to analyze cache performance issues
- [ ] Know how to optimize code for cache behavior
- [ ] Understand the trade-offs in cache design
- [ ] Be able to debug cache-related problems in multi-core systems
- [ ] Know how to handle cache coherency in real-time systems
</Checklist>

## 📚 Extra Resources

### Recommended Reading

- **"Computer Architecture: A Quantitative Approach" by Hennessy & Patterson** - Comprehensive cache architecture coverage
- **"Memory Systems: Cache, DRAM, Disk" by Bruce Jacob** - Detailed memory system analysis
- **"High Performance Computing" by David Kirk** - Performance optimization techniques

### Online Resources

- **Cache Simulators** - Tools for cache behavior analysis
- **ARM Cache Documentation** - Official cache architecture guides
- **Intel Cache Optimization Manual** - Performance tuning guides

### Practice Exercises

1. **Implement cache blocking** - Optimize matrix operations
2. **Analyze cache miss patterns** - Use profiling tools to understand cache behavior
3. **Design cache-friendly data structures** - Optimize for spatial and temporal locality
4. **Debug cache coherency issues** - Practice with multi-core cache problems

---

**Next Topic**: [Memory Protection Units](./Memory_Protection_Units.md) → [Hardware Accelerators](./Hardware_Accelerators.md)
