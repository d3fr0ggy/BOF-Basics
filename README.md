# BOF-Basics
Learning material for completely newcomers in the field of BOFs 

# Buffer
A buffer in actual is a temporary memory that is allocated to a particular program to retrieve and store the data whenever it requires. 
Memory i.e. the buffer in this case is always limited and can store a finite amount of data. More specifically, the ones who are coding the program know how much data will be required by the program. That memory is known as buffer.

# Buffer Overflow 
This occurs when the buffer limit is exceeded i.e. when we put more data than what was actually expected by the buffer. This happens in programs when we try to write data into the buffer and we overrun the actual buffer size. So, the extra data overwrites the adjacent memory locations.

Suppose, we have an array in C which can be initialized as above.

``` C
char array[10];
```
Now, we have defined implicitly that this array will be storing 10 data members. Suppose it to be integers from 0-9. When we will try to add more than 10 value and will see what happens when we do it so!

Consider the following program which takes 10 values from the user on runtime and copies them into the array.

```C
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
int main(int argc, char *argv[]) {

        char array[10];

        strcpy(array, argv[1]);

        printf("Data : %s", array);

        return 0;
}
```
On Linux, we can utilize the gcc compiler to compile this code. Once compiled we will run the executable and will supply it values 0 through 9. We can see that our program has successfully accepted the values and have printed them on the screen.

![alt text](https://github.com/d3fr0ggy/BOF-Basics/blob/master/images/1.png)

Now, we will supply more data to the program which it is expecting. 

![alt text](https://github.com/d3fr0ggy/BOF-Basics/blob/master/images/2.png)

We can see that we have achieved "Segmentation Fault". This means that the buffer was overflowed.

GCC or GNU C Compiler has many buit-in functions by default which prevents buffer overflow vulnerabilties. While we are learning we will turn off those checks for our learnings.

# Types of Buffer Overflows
There are mainly two types of "Buffer Overflows" named as above.

1. Stack-Based Buffer Overflow
2. Heap-Based Buffer OVerflow

For the sake of basics, will only be looking onto finding and exploiting "Stack-Based Buffer Overflows".

# Security Protections Against Buffer Overflows 
The security mechanism which have been added to Windows by Microsoft are as above.

1. DEP - Data Execution Prevention
2. ASLR - Address Space Layout Randomization

## DEP - Data Execution Prevention
Programs require memory in order to run and execute. DEP is a security feature which helps protect the system memory and the memory reserved for the authorized apps to not be used unethically by any other program. DEP keeps an eye on the memory management of the programs and as soon a particular program tries to execute instructions from antoher memory location. It is stopped and the user is notified. 

In technical words, DEP enables system to set some memory locations to be non-executable. This works by setting non-execution flag on some pages like default heap, stack and memory pools. As soon as the program tries to execute instruction from these protected data pages. Is is stopped to do such activities. 

Consider that two applications are running at the same time. The OS has allocated different memory regions for both application. Suppose we have total of 20 bytes of memory. The 1st application gets the first 10 bytes and the 2nd application gets the remaining 10 bytes. So, both have different address spaces and are running in different memory locations. 

Now application 1 can not execute anything from the memory which has been allocated to the 2nd application and similarly the 2nd application can not execute anything from the memory which has been allocated to the 1st application. They solely have to complete their execution while being in their own memory limits. 

If applications do not follow the process DEP mechanism will be called and the applications will be terminated.

<b>Reference - </b> https://docs.microsoft.com/en-us/windows/win32/memory/data-execution-prevention

## ASLR - Address Space Layout Randomization
Address Space Layout Randomization is also a security feature which has been implemented by the Microsoft in order to get rid of "Buffer Overflow" vulnerabilities. ASLR as the name suggests "Space" and "Randomization" are the key words. On the memory there are different buffers (memory regions) where data is required to be stored. Those locations have their particular addresses. What ASLR does is it randomized those address of memory locations so that it becomes hard to guess which part of the program sits on which location. This happens automatically when the program is run. 

In technical words, when the program is executed it is allowed a specific process region in which it has to complete its exectution. The program needs different regios to put its data i.e. stack, heap, libraries etc. So, ASLR randomized the addresses of stack, heap and libraries positions in the allocated program's memory region. 

Let's understand it with an example.

We know that instructions are executeed one after an other. Consider the following two lines.

```C
1 - strcpy(array, argv[1]);
2 - printf("Data : %s", array);
```
We know that these are two instruction. At first, the 1st instruction will be executed and then the 2nd instruction will be executed. Suppose we wrote a complete program and ran it on Windows machine. Now the OS has allocated 60-blocks of memory (automatically) for the execution of these 2 lines. But we know that only 2 blocks are required to complete the execution of these instructions i.e. 1-60 blocks, so 1st and 2nd block will be used.

Now, what happens due to ASLR is that these two instructions are put far away from each other. Suppose the first instruction is kept in 1st memory block and the other (WE DON'T KNOW) but it has been added to randomized memory location. So if we keep our exploit code on second line. We don't know the address where it has been shifted to by the OS, so we won't be able to call it because we have to use the EIP (Instruction Pointer).

<b>Reference - </b> https://en.wikipedia.org/wiki/Address_space_layout_randomization


# Finding Buffer Overflows
There are 2 most commonly used techniques for finding "Buffer OVerflows". 

1. Fuzzing 
2. Debugging

## 1. Fuzzing
This is the most easiest way to find "Buffer Overflows". In fuzzing we try to send bigger chunks of data in order to crash the program. Once the program crashes we try to find the exact part where the program gets crashed and then we try to use it for our advantage.

There are few tools available online for fuzzing as well as some comes pre-built in Kali Linux. Meanwhile we can also write our own scripts to perform fuzzing.

___
Now we will try to find the "Buffer Overflow" vulnerability in one of our own written programs. But before that we need to recompile our code to remove the basic "Buffer Overflow" security mechanism which is implemented while we compile our program with GCC. So we will turn those protections off so that we can exploit this vulnerability. 
___

We will be using the same program but we will limit the array size to 5 to keep it as small and simple as possible.

```C
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
int main(int argc, char *argv[]) {

        char array[5];

        strcpy(array, argv[1]);

        printf("Data : %s", array);

        return 0;
}
```
Now, we will compile this program with the default options set with GCC.

![alt text](https://github.com/d3fr0ggy/BOF-Basics/blob/master/images/3.png)

We can see that we were able to compile the program. Also we know that the program can only hold 5 values but in this case it is accepting more. But if we move way more further we get the segmentation fault. It is because the GCC by default set different flags which prohibit the buffers to overflow.
___
Before moving any further, we need to discuss the STACKS here. What they are and how we are getting the STACK BASED BUFFER OVERFLOWS.
___
## STACKS
Stack is a data structure based on LIFO (Last In First Out). Which means that the last value to be added on the stack will be the first to remove or release. 
```
Consider a glass of water. What happens when you pour water into it. It starts to fill up from the bottom and when to try to drink the water it gets released from the top of the glass to the bottom and at the end the glass is empty. 
```
In similar way the stacks work. Stacks only have two basic operation i.e. PUSH and POP. PUSH is used to add something on the stack which starts to be placed from the bottom and the data can be pushed until the stack is full. Meanwhile POP is used to remove the values from the STACK and it POPs the values from top to bottom. Values can be popped until there are no more values left on the stack.

![alt text](https://github.com/d3fr0ggy/BOF-Basics/blob/master/images/s1.png)

<b>a - </b> In figure (a) we can see that we have an empty stack. <br>
<b>b - </b> In figure (b) we pushed character A on the stack which was placed on the bottom. <br>
<b>c - </b> In Figure (c) we can see that character A was popped out of stack leaving the stack empty.

Similarly in the image we can see that one by one 3 characters were pushed onto the stack and one by one they were popped out of the stack leaving the stack empty.

![alt text](https://github.com/d3fr0ggy/BOF-Basics/blob/master/images/s2.png)

Now we will take a look onto how this program is actually running on the system and how we are getting the "Segmentation Fault".

![alt text](https://github.com/d3fr0ggy/BOF-Basics/blob/master/images/s3.png)


## 2. Debugging
Debugging is the process of finding and locating errors in computer programs. There are different tools available online by using which we can perform debugging and can look for potential "Buffer OVerflows".

