# Event
Before reading this article, you must know what is IO multiplexing mechanism and the specific IO multiplexing mechanism such as epoll, select.
## Key Source File
1. ae.h ae.c  
    These source files contain some abstract interfaces in order to operate different IO multiplexing mechanism in different operating system. For example, epoll in Linux, Event ports in Solaris, kqueue in FreeBSD and so forth.  
2. ae_epoll.c ae_evport.c ae_kqueue.c ae_select.c  
    These source files contain specific IO multiplexing mechanism.  
## Abstract Interfaces
### Create Event Loop
    eEventLoop *aeCreateEventLoop(int setsize);
### Destroy Event Loop
    void aeDeleteEventLoop(aeEventLoop *eventLoop);
### Add Event Into Event Loop
    int aeCreateFileEvent(aeEventLoop *eventLoop, int fd, int mask, aeFileProc *proc, void *clientData);
### Delete Event from Event Loop
    void aeDeleteFileEvent(aeEventLoop *eventLoop, int fd, int mask);
### Stop Looping
    void aeStop(aeEventLoop *eventLoop);
### Create Timer Event In Event Loop
    long long aeCreateTimeEvent(aeEventLoop *eventLoop, long long milliseconds,
        aeTimeProc *proc, void *clientData,
        aeEventFinalizerProc *finalizerProc);
### Delete Timer Event From Event Loop
    int aeDeleteTimeEvent(aeEventLoop *eventLoop, long long id);
### Wait For Events And Process Events
    int aeProcessEvents(aeEventLoop *eventLoop, int flags);
### Main Entrance For Event Loop
    void aeMain(aeEventLoop *eventLoop);
### Wait For Event Or Timeout
    int aeWait(int fd, int mask, long long milliseconds);
### Set Callback Before Looping
    void aeSetBeforeSleepProc(aeEventLoop *eventLoop, aeBeforeSleepProc *beforesleep);
### Set Callback After Looping
    void aeSetAfterSleepProc(aeEventLoop *eventLoop, aeBeforeSleepProc *aftersleep);
## Epoll Interfaces
### Create Epoll
    static int aeApiCreate(aeEventLoop *eventLoop)
### Destroy Epoll
    static void aeApiFree(aeEventLoop *eventLoop)
### Add Event Into Epoll
    static int aeApiAddEvent(aeEventLoop *eventLoop, int fd, int mask)
### Delete Event From Epoll
    static void aeApiDelEvent(aeEventLoop *eventLoop, int fd, int delmask)
### Wait For Events Or Timeout
    static int aeApiPoll(aeEventLoop *eventLoop, struct timeval *tvp)  
	[Back](https://github.com/Hankin-Liu/hankin.github.io/blob/master/redis/Redis_Analysis.md)

