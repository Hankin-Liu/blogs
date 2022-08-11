# Package Loss Issue
In this article, we will describe how to locate package loss issue.

## Lost by NIC
NIC driver has applied for ring buffers to store data of network. DMA writes network data into ring buffers, then a hardware IRQ will be generated to notify Linux kernel. Linux kernel will read the data from ring buffers in softIRQ handler.  
In the process above, there are 2 scenarios can lead to package loss which are shown blow:  
### 1. ring buffers are full
In this scenario, DMA can not copy data into ring buffers, then the packages will be dropped.  
Let's locate this issue using below tools：  
(1) netstat  
Command: netstat -i  
![netstat_ringbuff_full](https://github.com/Hankin-Liu/blogs/blob/master/performance_optimization/netstat_ring_buffer_full.png)
(2) ifconfig  
Command: ifconfig
![ifconfig_ringbuff_full](https://github.com/Hankin-Liu/blogs/blob/master/performance_optimization/ifconfig_ring_buffer_full.png)
### 2. NIC fifo is full
In this scenario, network data can not be stored into NIC fifo, then the package will be dropped.  
Let's locate this issue using below tools：  
(1) netstat  
Command: netstat -i  
![netstat_fifo_full](https://github.com/Hankin-Liu/blogs/blob/master/performance_optimization/netstat_fifo_full.png)
(2) ifconfig  
Command: ifconfig
![ifconfig_fifo_full](https://github.com/Hankin-Liu/blogs/blob/master/performance_optimization/ifconfig_fifo_full.png)  
(3) sar  
Command:   

[\[Back\]](https://github.com/Hankin-Liu/hankin.github.io/blob/master/performance_optimization/performance_optimization.md)