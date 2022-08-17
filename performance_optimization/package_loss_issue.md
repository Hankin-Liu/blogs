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
![netstat_ringbuff_full](https://github.com/Hankin-Liu/blogs/blob/master/performance_optimization/resources/package_loss/netstat_ring_buffer_full.png)  
(2) ifconfig  
Command: ifconfig  
![ifconfig_ringbuff_full](https://github.com/Hankin-Liu/blogs/blob/master/performance_optimization/resources/package_loss/ifconfig_ring_buffer_full.png)  
(3) sar  
Command: sar -n EDEV 1  
![sar_ringbuff_full](https://github.com/Hankin-Liu/blogs/blob/master/performance_optimization/resources/package_loss/sar_ring_buffer_full.png)  
### 2. NIC fifo is full
In this scenario, network data can not be stored into NIC fifo, then the package will be dropped.  
Let's locate this issue using below tools：  
(1) netstat  
Command: netstat -i  
![netstat_fifo_full](https://github.com/Hankin-Liu/blogs/blob/master/performance_optimization/resources/package_loss/netstat_fifo_full.png)  
(2) ifconfig  
Command: ifconfig
![ifconfig_fifo_full](https://github.com/Hankin-Liu/blogs/blob/master/performance_optimization/resources/package_loss/ifconfig_fifo_full.png)  
(3) sar  
Command: sar -n EDEV 1  
![sar_fifo_full](https://github.com/Hankin-Liu/blogs/blob/master/performance_optimization/resources/package_loss/sar_fifo_full.png)  
## Lost by Linux Kernel
In softIRQ handler, Linux kernel(NIC driver) moves data of ring buffers to destination CPU's backlog buffer. Each CPU has its own backlog buffer. This is happened before Linux network stack processes the data.   
Then packages go into network stack. After processing, kernel copies packages into socket buffers in relative according to the protocal type(TCP,UDP).
### 1. backlog buffers are full
In this scenario, Linux kernel(NIC driver) can not store packages into backlog buffer. Then packages will be dropped.  
Let's locate this issue using below command:  
Command: cat /proc/net/softnet_stat  
![backlog_full](https://github.com/Hankin-Liu/blogs/blob/master/performance_optimization/resources/package_loss/backlog_full.png)  
### 2. socket buffers are full
We talk about UDP's socket buffera are full, because TCP has congestion control.
In this scenario, Linux kernel network stack can not put packages into socket buffers. Then packages will be dropped.  
Let's locate this issue using below tools：  
(1) netstat  
Command: netstat -s -u  
![netstat_socket_buffer_full](https://github.com/Hankin-Liu/blogs/blob/master/performance_optimization/resources/package_loss/netstat_socket_buffer_full.png)
(2) sar  
Command: sar -n UDP,UDP6 1  
Note: This command can fetch the dropped udp package count per second which could not be delivered for reasons other than the lack of an application at the destination port.  
![sar_socket_buffer_full](https://github.com/Hankin-Liu/blogs/blob/master/performance_optimization/resources/package_loss/sar_socket_buffer_full.png)
## more powerful tools
If you need more issue details such as call stacks. The following tools are very useful.  
(1) dropwatch  
Command: dropwatch -l kas  
         start  
Note: This tool can show the kernel function name and address where drop packages.  
![dropwatch](https://github.com/Hankin-Liu/blogs/blob/master/performance_optimization/resources/package_loss/dropwatch.png)  
(2) perf  
Command: perf record -e 'skb:kfree_skb' -g -a  
         perf script  
Note: skb:kfree_skb is a tracepoint which indicates package loss.  
(3) [func_args](https://github.com/Hankin-Liu/bpf_perf_tools.git)    
Command: ./func_args kfree_skb -s -args ':' -Ti 1   
Note: func_args is implemented by BPF. Use this tool to trace kernel function 'kfree_skb' which indicates package loss.  
Example:  
>[root@fedora bpf_perf_tools]# ./func_args kfree_skb -s -args ':' -Ti 1  
>Tracing 1 functions for "b'\^kfree_skb$'"... Hit Ctrl-C to end.  
>10:10:42  
>FUNCTION                                THREAD_ID GLOBAL_INDEX  
>b'kfree_skb'                                    1            2  
>KERNEL STACK:   
>&emsp;&emsp;&emsp;&emsp;b'kfree_skb+0x1'  
>&emsp;&emsp;&emsp;&emsp;b'audit_log_end+0x64'  
>&emsp;&emsp;&emsp;&emsp;b'audit_receive_msg+0x6ac'  
>&emsp;&emsp;&emsp;&emsp;b'audit_receive+0x85'  
>&emsp;&emsp;&emsp;&emsp;b'netlink_unicast+0x211'  
>&emsp;&emsp;&emsp;&emsp;b'netlink_sendmsg+0x23f'  
>&emsp;&emsp;&emsp;&emsp;b'sock_sendmsg+0x5e'  
>&emsp;&emsp;&emsp;&emsp;b'__sys_sendto+0xf0'  
>&emsp;&emsp;&emsp;&emsp;b'__x64_sys_sendto+0x20'  
>&emsp;&emsp;&emsp;&emsp;b'do_syscall_64+0x3b'  
>&emsp;&emsp;&emsp;&emsp;b'entry_SYSCALL_64_after_hwframe+0x44'  
>USER STACK:   
>&emsp;&emsp;&emsp;&emsp;b'sendto+0x74'  
>Command: b'systemd'  
>FUNCTION                                THREAD_ID GLOBAL_INDEX  
>b'kfree_skb'                                    1            1  
>KERNEL STACK:   
>&emsp;&emsp;&emsp;&emsp;b'kfree_skb+0x1'  
>&emsp;&emsp;&emsp;&emsp;b'audit_log_end+0x64'  
>&emsp;&emsp;&emsp;&emsp;b'audit_receive_msg+0x6ac'  
>&emsp;&emsp;&emsp;&emsp;b'audit_receive+0x85'  
>&emsp;&emsp;&emsp;&emsp;b'netlink_unicast+0x211'  
>&emsp;&emsp;&emsp;&emsp;b'netlink_sendmsg+0x23f'  
>&emsp;&emsp;&emsp;&emsp;b'sock_sendmsg+0x5e'  
>&emsp;&emsp;&emsp;&emsp;b'__sys_sendto+0xf0'  
>&emsp;&emsp;&emsp;&emsp;b'__x64_sys_sendto+0x20'  
>&emsp;&emsp;&emsp;&emsp;b'do_syscall_64+0x3b'  
>&emsp;&emsp;&emsp;&emsp;b'entry_SYSCALL_64_after_hwframe+0x44'  
>USER STACK:   
>&emsp;&emsp;&emsp;&emsp;b'sendto+0x74'  
>Command: b'systemd'  
>  
>10:10:43  
>  
>10:10:44  
>  
>10:10:45  
>  
>10:10:46  
>  
>10:10:47  
>  
>10:10:48  
>  
>10:10:49  
>  
>10:10:50  
>  
>10:10:51  
>  
>10:10:52  
>  
>10:10:53  
>  
>10:10:54  
>  
>10:10:55  
>FUNCTION                                THREAD_ID GLOBAL_INDEX  
>b'kfree_skb'                                  809            3  
>KERNEL STACK:   
>&emsp;&emsp;&emsp;&emsp;b'kfree_skb+0x1'  
>&emsp;&emsp;&emsp;&emsp;b'inet6_destroy_sock+0x38'  
>&emsp;&emsp;&emsp;&emsp;b'inet_csk_destroy_sock+0x4d'  
>&emsp;&emsp;&emsp;&emsp;b'__tcp_close+0x32b'  
>&emsp;&emsp;&emsp;&emsp;b'tcp_close+0x20'  
>&emsp;&emsp;&emsp;&emsp;b'inet_release+0x3f'  
>&emsp;&emsp;&emsp;&emsp;b'__sock_release+0x3d'  
>&emsp;&emsp;&emsp;&emsp;b'sock_close+0x11'  
>&emsp;&emsp;&emsp;&emsp;b'__fput+0x94'  
>&emsp;&emsp;&emsp;&emsp;b'task_work_run+0x5c'  
>&emsp;&emsp;&emsp;&emsp;b'exit_to_user_mode_prepare+0x229'  
>&emsp;&emsp;&emsp;&emsp;b'syscall_exit_to_user_mode+0x18'  
>&emsp;&emsp;&emsp;&emsp;b'do_syscall_64+0x48'  
>&emsp;&emsp;&emsp;&emsp;b'entry_SYSCALL_64_after_hwframe+0x44'  
>USER STACK:   
>&emsp;&emsp;&emsp;&emsp;b'close+0x3b'  
>Command: b'NetworkManager'  
>FUNCTION                                THREAD_ID GLOBAL_INDEX  
>b'kfree_skb'                                  809            2  
>KERNEL STACK:   
>&emsp;&emsp;&emsp;&emsp;b'kfree_skb+0x1'  
>&emsp;&emsp;&emsp;&emsp;b'inet6_destroy_sock+0x47'  
>&emsp;&emsp;&emsp;&emsp;b'sk_common_release+0x1d'  
>&emsp;&emsp;&emsp;&emsp;b'inet_release+0x3f'  
>&emsp;&emsp;&emsp;&emsp;b'__sock_release+0x3d'  
>&emsp;&emsp;&emsp;&emsp;b'sock_close+0x11'  
>&emsp;&emsp;&emsp;&emsp;b'__fput+0x94'  
>&emsp;&emsp;&emsp;&emsp;b'task_work_run+0x5c'  
>&emsp;&emsp;&emsp;&emsp;b'exit_to_user_mode_prepare+0x229'  
>&emsp;&emsp;&emsp;&emsp;b'syscall_exit_to_user_mode+0x18'  
>&emsp;&emsp;&emsp;&emsp;b'do_syscall_64+0x48'  
>&emsp;&emsp;&emsp;&emsp;b'entry_SYSCALL_64_after_hwframe+0x44'  
>USER STACK:   
>&emsp;&emsp;&emsp;&emsp;b'close+0x3b'  
>Command: b'NetworkManager'  
>FUNCTION                                THREAD_ID GLOBAL_INDEX  
>b'kfree_skb'                                  809            4  
>KERNEL STACK:   
>&emsp;&emsp;&emsp;&emsp;b'kfree_skb+0x1'  
>&emsp;&emsp;&emsp;&emsp;b'inet6_destroy_sock+0x47'  
>&emsp;&emsp;&emsp;&emsp;b'inet_csk_destroy_sock+0x4d'  
>&emsp;&emsp;&emsp;&emsp;b'__tcp_close+0x32b'  
>&emsp;&emsp;&emsp;&emsp;b'tcp_close+0x20'  
>&emsp;&emsp;&emsp;&emsp;b'inet_release+0x3f'  
>&emsp;&emsp;&emsp;&emsp;b'__sock_release+0x3d'  
>&emsp;&emsp;&emsp;&emsp;b'sock_close+0x11'  
>&emsp;&emsp;&emsp;&emsp;b'__fput+0x94'  
>&emsp;&emsp;&emsp;&emsp;b'task_work_run+0x5c'  
>&emsp;&emsp;&emsp;&emsp;b'exit_to_user_mode_prepare+0x229'  
>&emsp;&emsp;&emsp;&emsp;b'syscall_exit_to_user_mode+0x18'  
>&emsp;&emsp;&emsp;&emsp;b'do_syscall_64+0x48'  
>&emsp;&emsp;&emsp;&emsp;b'entry_SYSCALL_64_after_hwframe+0x44'  
>USER STACK:   
>&emsp;&emsp;&emsp;&emsp;b'close+0x3b'  
>Command: b'NetworkManager'  
>FUNCTION                                THREAD_ID GLOBAL_INDEX  
>b'kfree_skb'                                  809            6  
>KERNEL STACK:   
>&emsp;&emsp;&emsp;&emsp;b'kfree_skb+0x1'  
>&emsp;&emsp;&emsp;&emsp;b'inet6_destroy_sock+0x47'  
>&emsp;&emsp;&emsp;&emsp;b'sk_common_release+0x1d'  
>&emsp;&emsp;&emsp;&emsp;b'inet_release+0x3f'  
>&emsp;&emsp;&emsp;&emsp;b'__sock_release+0x3d'  
>&emsp;&emsp;&emsp;&emsp;b'sock_close+0x11'  
>&emsp;&emsp;&emsp;&emsp;b'__fput+0x94'  
>&emsp;&emsp;&emsp;&emsp;b'task_work_run+0x5c'  
>&emsp;&emsp;&emsp;&emsp;b'exit_to_user_mode_prepare+0x229'  
>&emsp;&emsp;&emsp;&emsp;b'syscall_exit_to_user_mode+0x18'  
>&emsp;&emsp;&emsp;&emsp;b'do_syscall_64+0x48'  
>&emsp;&emsp;&emsp;&emsp;b'entry_SYSCALL_64_after_hwframe+0x44'  
>USER STACK:   
>&emsp;&emsp;&emsp;&emsp;b'close+0x3b'  
>Command: b'NetworkManager'  
>FUNCTION                                THREAD_ID GLOBAL_INDEX  
>b'kfree_skb'                                  809            5  
>KERNEL STACK:   
>&emsp;&emsp;&emsp;&emsp;b'kfree_skb+0x1'  
>&emsp;&emsp;&emsp;&emsp;b'inet6_destroy_sock+0x38'  
>&emsp;&emsp;&emsp;&emsp;b'sk_common_release+0x1d'  
>&emsp;&emsp;&emsp;&emsp;b'inet_release+0x3f'  
>&emsp;&emsp;&emsp;&emsp;b'__sock_release+0x3d'  
>&emsp;&emsp;&emsp;&emsp;b'sock_close+0x11'  
>&emsp;&emsp;&emsp;&emsp;b'__fput+0x94'  
>&emsp;&emsp;&emsp;&emsp;b'task_work_run+0x5c'  
>&emsp;&emsp;&emsp;&emsp;b'exit_to_user_mode_prepare+0x229'  
>&emsp;&emsp;&emsp;&emsp;b'syscall_exit_to_user_mode+0x18'  
>&emsp;&emsp;&emsp;&emsp;b'do_syscall_64+0x48'  
>&emsp;&emsp;&emsp;&emsp;b'entry_SYSCALL_64_after_hwframe+0x44'  
>USER STACK:   
>&emsp;&emsp;&emsp;&emsp;b'close+0x3b'  
>Command: b'NetworkManager'  
>FUNCTION                                THREAD_ID GLOBAL_INDEX  
>b'kfree_skb'                                  809            1  
>KERNEL STACK:   
>&emsp;&emsp;&emsp;&emsp;b'kfree_skb+0x1'  
>&emsp;&emsp;&emsp;&emsp;b'inet6_destroy_sock+0x38'  
>&emsp;&emsp;&emsp;&emsp;b'sk_common_release+0x1d'  
>&emsp;&emsp;&emsp;&emsp;b'inet_release+0x3f'  
>&emsp;&emsp;&emsp;&emsp;b'__sock_release+0x3d'  
>&emsp;&emsp;&emsp;&emsp;b'sock_close+0x11'  
>&emsp;&emsp;&emsp;&emsp;b'__fput+0x94'  
>&emsp;&emsp;&emsp;&emsp;b'task_work_run+0x5c'  
>&emsp;&emsp;&emsp;&emsp;b'exit_to_user_mode_prepare+0x229'  
>&emsp;&emsp;&emsp;&emsp;b'syscall_exit_to_user_mode+0x18'  
>&emsp;&emsp;&emsp;&emsp;b'do_syscall_64+0x48'  
>&emsp;&emsp;&emsp;&emsp;b'entry_SYSCALL_64_after_hwframe+0x44'  
>USER STACK:   
>&emsp;&emsp;&emsp;&emsp;b'close+0x3b'  
>Command: b'NetworkManager'  
>  
>10:10:56  
>FUNCTION                                THREAD_ID GLOBAL_INDEX  
>b'kfree_skb'                                    0            1  
>KERNEL STACK:   
>&emsp;&emsp;&emsp;&emsp;b'kfree_skb+0x1'  
>&emsp;&emsp;&emsp;&emsp;b'tcp_v4_rcv+0x5d'  
>&emsp;&emsp;&emsp;&emsp;b'ip_protocol_deliver_rcu+0x2b'  
>&emsp;&emsp;&emsp;&emsp;b'ip_local_deliver+0x107'  
>&emsp;&emsp;&emsp;&emsp;b'__netif_receive_skb_core.constprop.0+0x629'  
>&emsp;&emsp;&emsp;&emsp;b'__netif_receive_skb_list_core+0x126'  
>&emsp;&emsp;&emsp;&emsp;b'netif_receive_skb_list_internal+0x1b6'  
>&emsp;&emsp;&emsp;&emsp;b'napi_complete_done+0x6f'  
>&emsp;&emsp;&emsp;&emsp;b'e1000_clean+0x27a'  
>&emsp;&emsp;&emsp;&emsp;b'__napi_poll+0x2c'  
>&emsp;&emsp;&emsp;&emsp;b'net_rx_action+0x21a'  
>&emsp;&emsp;&emsp;&emsp;b'__softirqentry_text_start+0xfc'  
>&emsp;&emsp;&emsp;&emsp;b'__irq_exit_rcu+0xe8'  
>&emsp;&emsp;&emsp;&emsp;b'common_interrupt+0xb8'  
>&emsp;&emsp;&emsp;&emsp;b'asm_common_interrupt+0x1e'  
>&emsp;&emsp;&emsp;&emsp;b'native_safe_halt+0xb'  
>&emsp;&emsp;&emsp;&emsp;b'default_idle+0xa'  
>&emsp;&emsp;&emsp;&emsp;b'default_idle_call+0x33'  
>&emsp;&emsp;&emsp;&emsp;b'do_idle+0x1f1'  
>&emsp;&emsp;&emsp;&emsp;b'cpu_startup_entry+0x19'  
>&emsp;&emsp;&emsp;&emsp;b'secondary_startup_64_no_verify+0xc2'  
>USER STACK:   
>Command: b'swapper/3'
 

[\[Back\]](https://github.com/Hankin-Liu/hankin.github.io/blob/master/performance_optimization/performance_optimization.md)
