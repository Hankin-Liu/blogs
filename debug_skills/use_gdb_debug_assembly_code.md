# Debug Assembly Code Using Gdb 
In this article, we describe how to use gdb to debug assembly code.

## Example code
```c++
#include <iostream>
using namespace std;

uint64_t func2(uint64_t p1, uint64_t p2, uint64_t p3, uint64_t p4, uint64_t p5, uint64_t p6)
{
    static uint64_t i = 0;
    i += p1 + p2 + p3 + p4 + p5 + p6;
    cout << i << endl;
    return i;
}

uint64_t func3(uint64_t p1, uint64_t p2, uint64_t p3, uint64_t p4)
{
    static uint64_t i = 0;
    i += p1 + p2 + p3 + p4;
    cout << i << endl;
    return i;
}

void func1()
{
    uint64_t p1 = 0xFFFFFFFFFFF1;
    uint64_t p2 = 0xFFFFFFFFFFF2;
    uint64_t p3 = 0xFFFFFFFFFFF3;
    uint64_t p4 = 0xFFFFFFFFFFF4;
    uint64_t p5 = 0xFFFFFFFFFFF5;
    uint64_t p6 = 0xFFFFFFFFFFF6;
    uint64_t r2 = func2(p1, p2, p3, p4, p5, p6);
    uint64_t r3 = func3(p1, p2, p3, p4);
    cout << r2 + r3 << endl;
}

int main()
{
    func1();
    return 0;
}
```
Operating System : Linux  
Gcc Version : 9.3.1  
Compile Command : g++ test.cpp -o test -O2 -g  

## Debug Steps

**Step 1:** use gdb to open the binary file.

​              Instruction: gdb -tui test

**Step 2:** display assembly code

​              Instruction: (1) set disassembly-flavor intel

​                                    (2) layout split

![layout split](https://github.com/Hankin-Liu/hankin.github.io/blob/master/debug_skills/layout_split.png)

**Step 3:** set a breakpoint on function func1, then run this program.

​              Instruction: (1) b *func1

​                                   (2) r

![check_parameter_from_register](https://github.com/Hankin-Liu/hankin.github.io/blob/master/debug_skills/check_parameter_from_register.png)

From this picture, we can see that the assembly code is moving parameter into register in order to call function func2. The relationship between parameter and registers are shown below:

| parameter            | register |
| -------------------- | -------- |
| The 1st parameter p1 | rdi      |
| the 2nd parameter P2 | rsi      |
| the 3rd parameter P3 | rdx      |
| the 4th parameter P4 | rcx      |
| the 5th parameter P5 | r8       |
| the 6th parameter P6 | r9       |

**Step 4:** create a breakpoint at line 16 of func2 and check the return value of func2

​             Instruction: (1) b 16

​                                   (2) ni

​                                   (3) p/x $rax

​                                   (4) p/x i

![check_return_value_from_register](https://github.com/Hankin-Liu/hankin.github.io/blob/master/debug_skills/check_return_value_from_register.png)

From this picture, we can see that the assembly code is moving the return value of func2 into register in order to call function func2. The relationship between function's return value and registers are shown below:

| function's return value | register |
| ----------------------- | -------- |
| return value            | rax      |

[\[Back\]](https://github.com/Hankin-Liu/hankin.github.io/blob/master/debug_skills/debug_skills.md)