# High Availability  
Redis has several ways to achieve high availability. The details are shown below.  
## AOF  
AOF means append only file. After client's instructions have processed, redis will record them into the end of append only file. If redis crashes, new redis process can load this file, and run instruction one by one in order to restore the data into memory.   
[AOF Detail](https://github.com/Hankin-Liu/hankin.github.io/blob/master/redis/AOF.md)  

[\[Back\]](https://github.com/Hankin-Liu/hankin.github.io/blob/master/redis/Redis_Analysis.md)

