# Microsoft Hacking
I am a Security Researcher and this is a collection of Microsoft security information, exploits, and tools for informational use only. This repo is meant to improve my knowledge and skills on Microsoft security mechanisms and will hopefully help others as well. This repo is meant for informational purposes only and will likely not be maintained. 

# Hacking Basics
A compiled program is broken into five segments: text, data, bss, heap, and stack.
![image](https://user-images.githubusercontent.com/19600892/155240452-e614b195-38d5-4908-81cd-7c51f1f28958.png)

The text segment is where the instructions of the program are located. When a program begins executing, the RIP (the register that points to the currently executing instruction), is set to the first machine language instruction in the text segment. The processor than follows a execution loop as it steps through the instructions:

1. Reads the instruction that RIP is pointing to
2. Adds the byte length of the instruction to RIP
3. Executes the instruction that was read in step 1
4. Goes back to step 1
You cannot write to the text segment of memory. Any attempt to do so will result in the program being killed. The text memory segment is of fixed size.

The data section is used to store initialized global and static variables.
The bss section is used to store uninitialized global and static variables.
Both these sections of memory are writable although they are fixed in size. Global and static variables are able to persist — regardless of function context — because they are stored in their own memory segments.

The heap section of memory is directly under the programmers control. In C, programmers can use the function malloc() to dynamically allocate memory on the heap. The heap is not of fixed size and can grow larger or smaller. The growth of the heap “moves downward toward higher memory addresses” (Hacking: The Art of Exploitation, Jon Erickson). 

The stack section of memory is used to store local function variables and context during function calls. The stack is not of fixed size. When a function is called, "that function will have its own set of passed variables, and the function’s code will be at a different memory location in the text segment. Since the context and the RIP must change when a function is called, the stack is used to remember all of the passed variables, the location the RIP should return to after the function is finished, and all the local variables used by that function,” (Erickson).

# Stack Exploitation
Stack exploitation is often referred to as 'smashing the stack' because it is possible to write past the end of an array on the stack and overwrite the return pointer, thereby smashing the stack and controlling where the program returns to. This is often done by storing more data into a variable on the stack than is actually allowed. Below is an example of this:
```
void function(char *str) {
  char buffer[16];
  strcpy(buffer,str);
}
void main() {
  char large_string[256];
  int i;
  
  for( i = 0; i < 255; i++)
    large_string[i] = 'A';
    
   function(large_string);
}
```
The above code will use strcpy to copy a string of 256 bytes into a buffer that only has a size of 16 bytes which will result in writing past the end of the buffer which will clober the stack and overwrite anything before it on the stack. This will allow you to overwrite the return pointer that tells the CPU which address to return to once the function is completed. You can then control where the CPU will jump to once the function is complete which should give you control over the computer's execution.

# Heap Exploitation


# OS Hacking Basics

# Microsoft Security Mitigations and Bypasses
## Stack Canaries 
![image](https://user-images.githubusercontent.com/19600892/155240686-5c18d6af-742d-456b-91b5-5dea4c19770e.png)

Stack Canaries are used as a way to detect if the stack if being smashed and to abort the program so the potential attacker cannot gain control of the computer's execution. A stack canary is a value placed on the stack just before (a higher memory address as the stack grows down) any local variables in the function get added to the stack. The Stack Canary is also in front of the Stack Frame Pointer and the Return Pointer so that neither of these values can be overwritten without overwriting the stack canary as well. The Stack Canary should be generated at program runtime and will stay the same for the entire program execution.

Stack Canaries were introduced in Visual Studio 2003 and turned on by default in Visual Studio 2005. Microsoft’s /GS protection, uses a random canary, although they call it cookie or security cookie. This random canary is initialized in a function called security init cookie() which you can find in the file seccinit.c in
crt sources that come with Microsoft’s Visual Studio .NET. This function is called once, every time the program is started.
For more information look here https://docs.microsoft.com/en-us/cpp/build/reference/gs-buffer-security-check?view=msvc-160#gs-buffers

## Stack Canary Bypass
There are multiple ways to defeat a Stack Canary. The first way is to leak the value from the stack and once you are able to smash the stack you just need to determine where the stack canary is and then overwrite it with the same value and the stack smashing will no longer be detected. This method won't work against a random XOR'd stack canary.

In some occasions, brute forcing the canary may be successful. When using a random canary on a 32 bit system, the canary will have 24 bits of entropy. This is due to the 8 bits (1 byte) being used for the NULL byte. 2^24 possibilities of randomisation makes 16.777.216 possible canary values. In a local privilege escalation exploit, 16 million guesses could well be within the bounds of a brute force attack.

On 64 bit systems, that entropy increases to 2^56 or 7.20 * 10^16 possibilities. This would be less feasible.

However, our guessing can be steered a bit. If we guess the canary byte by byte, we will be able to discern when we have guessed the right value. This is possible because an incorrect guess for a byte will generate a stack smashing error, where a correct byte guess will yield no such error. The maximum number (worst case) of guesses will remain 2^24, but the average number will decrease.

The canary can also be avoided if the attacker can overwrite a buffer and then cause an exception to occur within the same function. On Windows systems, this would trigger the structured exception handler (SEH) to intervene to solve the exception. If the SEH pointer is also overwritten, this can also lead to control over the instruction pointer. The SEH routine will not check canary values before handing execution to the handler.

# Fuzzers

# Additional Links
[Frame Pointer Overwrite](http://phrack.org/issues/55/8.html#article)
[Defeating StackGuard, StackShield, and Stack Cookies](https://www.cs.purdue.edu/homes/xyzhang/spring07/Papers/defeat-stackguard.pdf)

