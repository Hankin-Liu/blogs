# Strip and Add Debug Info for ELF file 
In this article, we describe how to strip/add debug info for ELF file.

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

## Strip Debug Info

**Step 1:** backup debug info and symbol table.

​              Instruction: objcopy --only-keep-debug test test.debug

```
[hankin@localhost test]$ ll
total 212
-rwxrwxr-x. 1 hankin hankin 110296 Jan 21 16:46 test
-rw-rw-r--. 1 hankin hankin    121 Jan 21 16:45 test.cpp
-rwxrwxr-x. 1 hankin hankin  98664 Jan 21 16:47 test.debug
```

**Step 2:** strip debug info and symbol table

​              Instruction: strip -s test

​                                 or objcopy --strip-debug test

```
[hankin@localhost test]$ ll
total 120
-rwxrwxr-x. 1 hankin hankin 14552 Jan 21 16:47 test
-rw-rw-r--. 1 hankin hankin   121 Jan 21 16:45 test.cpp
-rwxrwxr-x. 1 hankin hankin 98664 Jan 21 16:47 test.debug
```

**Step 3:** now, when we debug this ELF file using gdb, there will be no symbol and source code.

​              Instruction: (1) gdb -tui test

​                                   (2) b main

![gdb stripped elf](https://github.com/Hankin-Liu/hankin.github.io/blob/master/debug_skills/gdb_stripped.png)

From this picture, we can see that there is no source code and symbol in this ELF file, So we can't set a breakpoint.

## Add Debug Info

Now, we use the debug info file we backuped above to debug this ELF file.

**Step 1:** link debug info file to ELF file.

​              Instruction: objcopy --add-gnu-debuglink=test.debug test

**Step 2:** now, when we debug this ELF file using gdb.

​              Instruction: (1) gdb -tui test

​                                   (2) b main

​                                   (3) r

![gdb add debug elf](https://github.com/Hankin-Liu/hankin.github.io/blob/master/debug_skills/gdb_add_debug.png)

From this picture, we can see the source code and symbol in this ELF file, So we set a breakpoint successfully.

[\[Back\]](https://github.com/Hankin-Liu/hankin.github.io/blob/master/debug_skills/debug_skills.md)