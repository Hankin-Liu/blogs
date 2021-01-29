# Memory Fence Flush Registers  
Refer to gcc official doc, compiler memory fence can flush registers. The following is a part of the doc.
"To ensure memory contains correct values, GCC may need to flush specific register values to memory before executing the asm. "  
[gcc doc page](https://gcc.gnu.org/onlinedocs/gcc/Extended-Asm.html#Clobbers-and-Scratch-Registers)

## Example
Operating System : Linux  
Gcc Version : 9.3.1  
Compile Command : g++ test.cpp -o test -O2 -pthread -std=c++11 -g  

```c++
  1 #include <iostream>                                                                                                             2 #include <mutex>  
  3 #include <thread>  
  4 #include <vector>  
  5 #include <unistd.h>  
  6 #include <atomic>  
  7 using namespace std;  
  8 
  9 std::mutex mtx_{};  
 10 class temp  
 11 {   
 12     public:  
 13         void run()
 14         {   
 15             while (true) {  
 16                 // if uncommented below fence, program will exit normally. Otherwise, program will go to endless loop.  
 17                 //std::atomic_thread_fence(std::memory_order_relaxed);  
 18                 if (! v.empty()) {
 19                     {   
 20                         std::lock_guard<std::mutex> temp_mtx(mtx_);
 21                         cout << "value : " << v[0] << endl;
 22                         v.clear();
 23                     }  
 24                     break;
 25                 }
 26             }
 27         }    
 28     public:  
 29         vector<uint32_t> v;
 30 };
 31 
 32 temp t;  
 33 
 34 void thread1()  
 35 {   
 36     sleep(5);  
 37     std::lock_guard<std::mutex> temp_mtx(mtx_);
 38     t.v.push_back(1);
 39 }
 40 
 41 int main()
 42 {
 43     std::thread t1(thread1);
 44     t.run();
 45     t1.join();
 46     return 0;
 47 }
```
## Analyze the assembly code

Let us use below command to disassemble the binary file and compare the above C++ code in two cases. **Case 1**: uncomment line 17. **Case 2**: comment line 17.

**Instruction**: (1) objdump -S -mi386:x86-64:intel -d test > test_comment.asm

​                       (2) uncomment line 17 and compile again

​                       (3) objdump -S -mi386:x86-64:intel -d test > test_uncomment.asm

### STL knowledge

Before analyzing the assembly code, we must aware of some knowledge of std::vector, because we use empty() in line 18 which is std::vector's member function.

Include file:  bits/stl_vector.h

position: C++ include directory 

```c++
  template<typename _Tp, typename _Alloc = std::allocator<_Tp> >
    class vector : protected _Vector_base<_Tp, _Alloc> 
    {
      ......
      bool 
      empty() const _GLIBCXX_NOEXCEPT
      { return begin() == end(); }
      ......
      iterator
      begin() _GLIBCXX_NOEXCEPT
      { return iterator(this->_M_impl._M_start); }
      ......
      iterator
      end() _GLIBCXX_NOEXCEPT
      { return iterator(this->_M_impl._M_finish); }
      ......
    public:
      _Vector_impl _M_impl;
      ......
    }
```

```c++
      struct _Vector_impl
      : public _Tp_alloc_type
      {
        pointer _M_start;
        pointer _M_finish;
        pointer _M_end_of_storage;
```

From the STL code above, we know that function empty is equal to (vector::\_M_impl.\_M_start == vector::\_M_impl.\_M_finish).

### Case 1

```
......
 211 int main()
 212 {
 213   401240:       41 54                   push   r12
 214         std::thread t1(thread1);  
 215   401242:       be c0 14 40 00          mov    esi,0x4014c0
 216 {
 217   401247:       55                      push   rbp
 218   401248:       53                      push   rbx
 219   401249:       48 83 ec 10             sub    rsp,0x10
 220         std::thread t1(thread1);  
 221   40124d:       48 8d 7c 24 08          lea    rdi,[rsp+0x8]
 222   401252:       e8 69 03 00 00          call   4015c0 <_ZNSt6threadC1IRFvvEJEvEEOT_DpOT0_>
 223                 void run()
 224   401257:       48 8b 05 ca 2f 00 00    mov    rax,QWORD PTR [rip+0x2fca]        # 404228 <t+0x8>
 225   40125e:       66 90                   xchg   ax,ax
 226                                 if (! v.empty()) {  
 227   401260:       48 3b 05 b9 2f 00 00    cmp    rax,QWORD PTR [rip+0x2fb9]        # 404220 <t>
 228   401267:       74 f7                   je     401260 <main+0x20>
 229   if (__gthread_active_p ())
 230   401269:       bb 50 11 40 00          mov    ebx,0x401150
 231   40126e:       48 85 db                test   rbx,rbx
 232   401271:       74 12                   je     401285 <main+0x45>
 233     return __gthrw_(pthread_mutex_lock) (__mutex);
 234   401273:       bf 40 42 40 00          mov    edi,0x404240
 235   401278:       e8 e3 fe ff ff          call   401160 <pthread_mutex_lock@plt>
......
```

From the assembly code, let us see what is done before checking v.empty(). let us focus on line 224 227 and 228, I will explain them line by line.

line 224: load vector v.\_M_impl.\_M_start from memory to register rax

line 227: compare register rax and vector v.\_M_impl.\_M_finish from memory

line 228: if same, go to line 227, else go to line 230

We can see that every time the comparision is between register rax and memory(vector v.\_M_impl.\_M_finish). So when thread 1 modifies this memory in line 38 in C++ code above, register rax will not be equal to the new value in memory. Then the program will go to the else branch.

### Case 2

```
>> 211 int main()  
>> 212 {  
>> 213   401240:   41 54                   push   r12
>> 214     std::thread t1(thread1);  
>> 215   401242:   be c0 14 40 00          mov    esi,0x4014c0
>> 216 {  
>> 217   401247:   55                      push   rbp
>> 218   401248:   53                      push   rbx
>> 219   401249:   48 83 ec 10             sub    rsp,0x10
>> 220     std::thread t1(thread1);  
>> 221   40124d:   48 8d 7c 24 08          lea    rdi,[rsp+0x8]
>> 222   401252:   e8 69 03 00 00          call   4015c0 <_ZNSt6threadC1IRFvvEJEvEEOT_DpOT0_>
>> 223        *  Returns true if the %vector is empty.  (Thus begin() would
>> 224        *  equal end().)
>> 225        */
>> 226       _GLIBCXX_NODISCARD bool
>> 227       empty() const _GLIBCXX_NOEXCEPT
>> 228       { return begin() == end(); }
>> 229   401257:   48 8b 15 ca 2f 00 00    mov    rdx,QWORD PTR [rip+0x2fca]        # 404228 <t+0x8>
>> 230   40125e:   48 8b 05 bb 2f 00 00    mov    rax,QWORD PTR [rip+0x2fbb]        # 404220 <t>
>> 231   401265:   0f 1f 00                nop    DWORD PTR [rax]
>> 232                 if (! v.empty()) {  
>> 233   401268:   48 39 c2                cmp    rdx,rax
>> 234   40126b:   74 fb                   je     401268 <main+0x28>
>> 235   if (__gthread_active_p ())
>> 236   40126d:   bb 50 11 40 00          mov    ebx,0x401150
>> 237   401272:   48 85 db                test   rbx,rbx
>> 238   401275:   74 12                   je     401289 <main+0x49>
>> 239     return __gthrw_(pthread_mutex_lock) (__mutex);
>> 240   401277:   bf 40 42 40 00          mov    edi,0x404240
>> 241   40127c:   e8 df fe ff ff          call   401160 <pthread_mutex_lock@plt>
```

From the assembly code, let us see what is done before checking v.empty(). let us focus on line 229 230 233 and 234, I will explain them line by line.

line 229: load vector v.\_M_impl.\_M_start from memory to register rdx

line 230: load vector v.\_M_impl.\_M_finish from memory to register rax

line 233: compare register rdx and register rax

line 234: if same, go to line 233, else go to line 236

We can see that every time the comparision is between register rdx and register rax and these two registers are not updated. So when thread 1 modifies this memory in line 38 in C++ code above, register rdx and register rax will not be updated. Then the program becomes endless loop.

Now let's debug it using gdb and prove it.

Instruction: (1) gdb -tui test

​                     (2) set disassembly-flavor intel

​                     (3) b 44

​                     (4) r

​                     (5) b	*0x40126b if v.\_M_impl.\_M_start != v.\_M_impl._M_finish

​                                       Note: 0x40126b is the address of assembly code "je     0x401268 <main+40>" in line 234 in case 2 above.

​                     (6) c

​                     (7) p	v.\_M_impl._M_finish

​                     (8) p	v.\_M_impl._M_start

![endless loop_by_O2](https://github.com/Hankin-Liu/hankin.github.io/blob/master/concurrency/endless_loopby_O2.png



[\[Back\]](https://github.com/Hankin-Liu/hankin.github.io/blob/master/concurrency/Concurrency.md)

