# AOF  
## Key Source File  
1. aof.c  
    This source file contains the logic of AOF.   
## Main Process  
1. propagate the command  
    Source File : server.c  
	Function : void propagate(struct redisCommand *cmd, int dbid, robj **argv, int argc, int flags)  
    Called By : void call(client *c, int flags) in server.c, line:2416, refer to "Fetch Command and Run" => "Run Command" => "Read From Socket and Run" => step 7. [Fetch Command and Run](https://github.com/Hankin-Liu/hankin.github.io/blob/master/redis/Process_Command.md)  
    Note: This function will be invoked after the command has been processed.  

[\[Back\]](https://github.com/Hankin-Liu/hankin.github.io/blob/master/redis/High_Avaliablility.md)

