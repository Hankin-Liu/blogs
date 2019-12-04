# Memory Fence Flush Registers  
Refer to gcc official doc, compiler memory fence can flush registers. The following is a part of the doc.
"To ensure memory contains correct values, GCC may need to flush specific register values to memory before executing the asm. "  
[gcc doc page](https://gcc.gnu.org/onlinedocs/gcc/Extended-Asm.html#Clobbers-and-Scratch-Registers)
