# Change Source Directory in Gdb 
In this article, we describe how to change source directory when debug using gdb.

## Example code
```c++
#include <iostream>

int main()
{
    std::string h_w{"Hello World"};
    std::cout << h_w << std::endl;
    return 0;
}
```
Operating System : Linux  
Gcc Version : 9.3.1  
Compile Command : g++ test.cpp -o test -O2 -g  
Directory: /home/test

## change directory for source and bin file

**Step 1:** rename the source directory's name.

​              Instruction: mv /home/test /home/test1

```
[root@localhost test1]# ll
total 112
-rwxr-xr-x. 1 root   root   110264 Jan 27 09:14 test
-rw-rw-r--. 1 hankin hankin    121 Jan 27 09:04 test.cpp
```

**Step 2:** change directory for the binary file

​              Instruction: mv test /usr/local/bin

```
[root@localhost test1]# ll /usr/local/bin/test
-rwxr-xr-x. 1 root root 110264 Jan 27 09:14 /usr/local/bin/test
```

**Step 3:** now, when we debug this ELF file using gdb, there will be no source code.

​              Instruction: (1) cd /usr/local/bin

​                                   (2) gdb test

​                                   (3) info sources

```
[root@localhost bin]# gdb test
GNU gdb (GDB) Red Hat Enterprise Linux 7.6.1-119.el7
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-redhat-linux-gnu".
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>...
Reading symbols from /usr/local/bin/test...done.
(gdb) l
1	test.cpp: No such file or directory.
(gdb) info sources
Source files for which symbols have been read in:

/home/test/test.cpp, /opt/rh/devtoolset-9/root/usr/include/c++/9/bits/ostream_insert.h, /usr/include/wctype.h, /usr/include/errno.h, 
......
```

From this output, we can't see the source code and gdb is searching for the source code "/home/test/test.cpp", but we have changed the source directory to /home/test1, so gdb can't find it.

## Set source path in gdb

**Step 1:** change the source path to the right position.

​              Instruction: (1) set substitute-path /home/test /home/test1

​                                   (2) l

```
(gdb) set substitute-path /home/test /home/test1
(gdb) l
1	#include <iostream>
2	
3	int main()
4	{
5	    std::string h_w{"Hello World"};
6	    std::cout << h_w << std::endl;
7	    return 0;
8	}
(gdb)
```

From this output, we can see the source  code successfully.

**Step 2:** check the source setting in gdb.

​              Instruction: info sources

```
(gdb) info sources
Source files for which symbols have been read in:

/home/test1/test.cpp, /opt/rh/devtoolset-9/root/usr/include/c++/9/bits/ostream_insert.h, /usr/include/wctype.h, /usr/include/errno.h, 
......
```

From this output, we can see that the source code's directory has changed to "/home/test1".

[\[Back\]](https://github.com/Hankin-Liu/hankin.github.io/blob/master/debug_skills/debug_skills.md)
