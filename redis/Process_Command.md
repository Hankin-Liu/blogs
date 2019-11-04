# Run Command
## Key Source File
1. networking.c  
    This file contains functions to process client requirements.  
2. server.c  
    This file contains functions to process commands.
## Read From Socket and Run
1. read data from socket and process data  
    Source File : networking.c  
    Function : void readQueryFromClient(aeEventLoop *el, int fd, void *privdata, int mask)  
    Called By : event loop，line:1518  
2. process input data depend on the role(MASTER or SLAVE)  
    Source File : networking.c  
    Function : void processInputBufferAndReplicate(client *c)  
    Called By : step 1，line:1587  
    Note: if role is not Master, it only feed replication log and feed its slaves by calling void replicationFeedSlavesFromMasterStream(list *slaves, char *buf, size_t buflen), line:1511  
3. wrapper for processing data  
    Source File : networking.c  
    Function : void processInputBuffer(client *c)  
    Called By : step 2，line:1505(MASTER), line:1508(SLAVE)  
4. parse input command and save result into argc and argv of client object  
    (1) inline protocol  
            Source File : networking.c  
            Function : int processInlineBuffer(client *c)  
            Called By : step 3, line:1458  
    (2) RESP protocol  
            Source File : networking.c  
            Function : int processMultibulkBuffer(client *c)  
            Called By : step 3, line:1460  
5. process command  
    Source File : server.c  
    Function : int processCommand(client *c)  
    Called By : step 3,，line:1470  
6. look for command  
    Source File : server.c  
    Function : struct redisCommand *lookupCommand(sds name)  
    Called By : step 5,，line:2560  
7. execute command  
    Source File : server.c  
    Function : void call(client *c, int flags)  
    Called By : step 5，line:2733  
    Note : proc is a function pointer of command execution, it means run the command. Line:2439, c->cmd->proc(c);  
  

[\[Back\]](https://github.com/Hankin-Liu/hankin.github.io/blob/master/redis/Redis_Analysis.md)









