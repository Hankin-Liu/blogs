# Package Loss Issue
In this article, we will describe how to locate package loss issue.

## Lost by NIC
NIC has its own memory(fifo). Network data is stored in this memory first. Then DMA moves the data to ring buffers.  
NIC driver has applied for ring buffers to store data of network. WHen DMA writes network data into ring buffers, a hardware IRQ will be generated to notify Linux kernel. Linux kernel will read the data from ring buffers in softIRQ handler.  
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
(3) sar  
Command: sar -n EDEV 1  
![sar_ringbuff_full](https://github.com/Hankin-Liu/blogs/blob/master/performance_optimization/sar_ring_buffer_full.png)  
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
Command: sar -n EDEV 1  
![sar_fifo_full](https://github.com/Hankin-Liu/blogs/blob/master/performance_optimization/sar_fifo_full.png)  
## Lost by Linux Kernel
In softIRQ handler, Linux kernel(NIC driver) moves data of ring buffers to destination CPU's backlog buffer. Each CPU has its own backlog buffer. This is happened before Linux network stack processes the data.   
Then packages go into network stack. After processing, kernel copies packages into socket buffers in relative according to the protocal type(TCP,UDP).
### 1. backlog buffers are full
In this scenario, linux kernel(NIC driver) can not store packages into backlog buffer. Then packages will be dropped.  
Let's locate this issue using below command:  
Command: cat /proc/net/softnet_stat  
![backlog_full](https://github.com/Hankin-Liu/blogs/blob/master/performance_optimization/backlog_full.png)  

[\[Back\]](https://github.com/Hankin-Liu/hankin.github.io/blob/master/performance_optimization/performance_optimization.md)