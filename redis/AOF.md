# AOF  
## Key Source File  
1. aof.c  
    This source file contains the logic of AOF.   
## Main Process  
1. propagate the command  
    Source File : server.c  
	Function : void propagate(struct redisCommand *cmd, int dbid, robj **argv, int argc, int flags)  
    Called By : void call(client *c, int flags) in server.c, line:2504, refer to "Fetch Command and Run" => "Run Command" => "Read From Socket and Run" => step 7. [Fetch Command and Run](https://github.com/Hankin-Liu/hankin.github.io/blob/master/redis/Process_Command.md)  
    Note: This function will be invoked after the command has been processed.  
2. main logic of AOF  
    Source File : aof.c  
	Function : void feedAppendOnlyFile(struct redisCommand *cmd, int dictid, robj **argv, int argc)  
    Called By : step 1, line:2321  
3. translate command to RESP protocol 
    Source File : aof.c  
	Function : sds catAppendOnlyGenericCommand(sds dst, int argc, robj **argv)  
    Called By : step 2, line:627  
4. cat command which has been translated to server's buffer(global variable)  
    Source File : sds.c  
	Function : sds sdscatlen(sds s, const void *t, size_t len)  
    Called By : step 2, line:634   
5. Waiting for flushing server's buffer data to disk  
    Source File : aof.c  
	Function : void flushAppendOnlyFile(int force)
	Called By : (1) line:1403(server.c) in function void before_sleep(sturct aeEventLoop *eventLoop), this function has registered into eventLoop(server.c line 4198) and will be called before sleep.  
                (2) line:1301,1309(server.c) in function int serverCron(struct aeEventLoop *eventLoop, long long id, void *clientData), this function will be called by timer, and it is registered at line:2128(server.c) in function void initServer(void)  


[\[Back\]](https://github.com/Hankin-Liu/hankin.github.io/blob/master/redis/High_Avaliablility.md)

