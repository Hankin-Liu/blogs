# Program profiling
In this article, we describe how to use code profilers to identify performance bottlenecks.

## Example code
```c++
#include <iostream>
using namespace std;
void func2()
{
    static int i = 0;
    cout << ++i << endl;
}
void func3()
{
    static int i = 0;
    cout << ++i << endl;
}
void func1()
{
    for (;;) {
        for (int j = 0; j < 3; ++j) {
            func2();
        }
        func3();
    }
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

## valgrind + kcachegrind  
The first tool is valgrind, let us use it to profile our code which is shown above.  
**Step 1:** Run program using valgrind  
        Instruction: valgrind --tool=callgrind --trace-children=yes --dump-instr=yes --collect-jumps=yes --collect-systime=no --branch-sim=yes --simulate-cache=yes ./test >/dev/null 2>&1  
**Step 2:** After running for 10 seconds, use CTRL + C to exit this program. Now you can find a callgrind.out file in your working directory. For example:   
             [hankin@localhost test]$ ls  
             callgrind.out.115774  test  test.cpp  
**Step 3:** Use kcachegrind to open and analyze this profile file.  
        Instruction: kcachegrind  
![valgrind+kcachegrind](https://github.com/Hankin-Liu/blogs/blob/master/performance_optimization/valgrind.png)  

From the picture, we can see that func1 calls func2 and func3, the time consuming proportion of func2 is 74.82%, and the time consuming proportion of  func3 is 24.78%. So the func2 is the biggest performance bottleneck.

## Gperftools + kcachegrind

The second tool is Gperftools, let us use it to profile our code which is shown above.  
**Step 1:** Run program with Gperftools' lib  
             Instruction: LD_PRELOAD="/usr/local/lib/libprofiler.so" CPUPROFILE=prof.out CPUPROFILESIGNAL=12 CPUPROFILE_FREQUENCY=10000 ./test 2>&1 > /dev/null  
**Step 2:** send signal 12 to this process to start profiling  
             Instruction: kill -12 PID  

​             Note: PID is the pid of this program.  
**Step 3:** After 10 seconds,  send signal 12 to this process to stop profiling, Now you can find a file named prof.out.0 in your working directory. For example  
​             [hankin@localhost test]$ ls  

​             prof.out.0  test  test.cpp

**Step 4:** use pprof to open the profile file

​             Instruction: pprof test prof.out.0

**Step 5:** generate callgrind file. After running below instruction, a file named mycallgind.out will be generated in your working directory.

​             Instruction: callgrind mycallgrind.out

**Step 6:** use kcachegrind to open and analyze file mycallgrind.out

​             Instruction: kcachegrind 

![gperftools+kcachegrind](https://github.com/Hankin-Liu/blogs/blob/master/performance_optimization/gperftools.png)  

From the picture, we can see that func1 calls func2 and func3 and std::ostream::flush, the time consuming proportion of func2 is 22.52% + 51.52%, and the time consuming proportion of  func3 is 7.54% + 17.85%. So the func2 is the biggest performance bottleneck.

There is a little difference between the result of valgrind and gperftools. In the result of gperftools, func1 calls std::ostream::flush, but in the result of valgrind, it doesn't happen. I guess program running on valgrind maybe has a little difference because valgrind provides a virtual environment for programs to run.  

[\[Back\]](https://github.com/Hankin-Liu/hankin.github.io/blob/master/performance_optimization/performance_optimization.md)
