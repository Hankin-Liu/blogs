# Memory Fence Flush Registers  
Refer to gcc official doc, compiler memory fence can flush registers. The following is a part of the doc.
"To ensure memory contains correct values, GCC may need to flush specific register values to memory before executing the asm. "  
[gcc doc page](https://gcc.gnu.org/onlinedocs/gcc/Extended-Asm.html#Clobbers-and-Scratch-Registers)

## Example
Operating System : Linux  
Gcc Version : 4.8.5  
Compile Command : g++ test.cpp -o test -O2 -pthread -std=c++11 -g  

```c++
#include <iostream>  
#include <mutex>  
#include <thread>  
#include <vector>  
#include <unistd.h>  
#include <atomic>  
using namespace std;  

class temp  
{  
    public:  
	    void run()  
		{  
		    while (true) {  
			    // if uncommented below fence, program will exit normally. Otherwise, program will go to endless loop.  
			    //std::atomic_thread_fence(std::memory_order_relaxed);  
				if (! v.empty()) {  
				    {  
					    std::lock_guard<std::mutex> temp_mtx(mtx_);  
						cout << "value : " << v[0] << endl;  
						v.clear();  
					}  
					break;  
				}  
			}  
		}  
	public:  
	    std::mutex_;  
		vector<uint32_t> v;  
};  

temp t;  

void thread1()  
{  
    sleep(5);  
	std::lock_guard<std::mutex> temp_mtx(mtx_);  
	t.v.push_back(1);  
}  

int main()  
{  
    std::thread t1(thread1);  
	t.run();  
	t1.join();  
	return 0;  
}  
