# Net
## Key Source File
1. anet.h anet.c  
    These source files contains some interfaces to encapsulate socket operation, such as create socket, listen, read, write and so forth.  
2. networking.c  
    This file contains functions to process client requirements.  
## Main Network Process
1. main function  
    Source File : server.c    
	Function : int main(int argc, char **argv)  
2. init server  
    Source File : server.c  
    Function : void initServer(void)  
    Called By : step 1，line:4052  
3. create listen socket  
    (1) tcp socket  
        Source File : server.c  
        Function : int listenToPort(int port, int *fds, int *count)  
        Called By : step 2，line:2057  
    (2) unix domain socket  
        Source File : anet.c  
        Function : int anetUnixServer(char *err, char *path, mode_t perm, int backlog)  
        Called By : step 2，line:2063  
4. add listen socket into event loop and register callback function for connecting event  
    Source File : ae.c  
    Function : aeEventLoop *aeCreateEventLoop(int setsize)  
    Note : register a callback function into event loop, when client connects, event loop will call this function. (1) TCP Callback Function : void acceptTcpHandler(aeEventLoop *el, int fd, void *privdata, int mask), which is in file networking.c.(2) Unix Domain Socket Callback Function : void acceptUnixHandler(aeEventLoop *el, int fd, void *privdata, int mask) , which is in file networking.c.  
    Called By : step 2，line: 2136，2144  
5. event loop starts to loop  
    Source File : ae.c  
    Function : void aeMain(aeEventLoop *eventLoop)  
    Called By : step 1，line:4200  
## Process Client’s Connection
When TCP client connects, callback function acceptTcpHandler will be called. When unix domain socket client connects, callback function acceptUnixHandler will be called. They contain below steps(key steps, not all steps).  
1. accept connections  
    (1) TCP client  
        Source File : anet.c  
        Function : int anetTcpAccept(char *err, int s, char *ip, size_t ip_len, int *port)  
        Called By : networking.c line:742, in function acceptTcpHandler  
    (2) Unix domain socket client  
        Source File : anet.c  
        Function : int anetUnixAccept(char *err, int s)  
        Called By : networking.c line:761, in function acceptUnixHandler  
2. register command handler for client  
    Source File : networking.c  
    Function : static void acceptCommonHandler(int fd, int flags, char *ip)  
    Called By : step 1，line:750,769  
3. create client object  
    Source File : networking.c  
    Function : client *createClient(int fd)  
    Called By : step 2，line:666  
4. add client fd into event loop and register callback function for client’s command  
    Source File : ae.c  
    Function : aeEventLoop *aeCreateEventLoop(int setsize)  
    Note : register a callback function into event loop, when client’s command comes, event loop will call this function. Callback Function : void readQueryFromClient(aeEventLoop *el, int fd, void *privdata, int mask), which is in file networking.c.  
    Called By : step 3，line: 98  

[\[Back\]](https://github.com/Hankin-Liu/hankin.github.io/blob/master/redis/Redis_Analysis.md)









