# Implementing Shellcode on 64 bits Centos7 (20200620)

* Typically, we first write a C program that calls a shell, then we disassemble the program to get the shellcode. This note is based on this [blog](http://blog.nsfocus.net/easy-implement-shellcode-xiangjie/).

## Function *execve()*

* **\#include <unistd.h>**

* The declaration is: 

  ```c
  int execve(const char *pathname, char * const argv[], char * const envp[]);
  ```

  * **pathname** is the path to the executable.
  * **argv** is an array that contains command-line arguments of the executed program. **BY CONVENTION**, argv[0] = filename, argv[argc] = NULL.
  * **envp** is the environment of the new program??? Terminated by NULL.

* To observe the **execve**, we need to compile it with **-static** flag.

  * To do so, we need to install the static version of `glibc` and `libstdc++`. 
  * `$ sudo yum -y install glibc-static libstdc++-static`
  * Check this [blog](http://rnowling.github.io/software/engineering/2015/05/05/static-compilation-dependencies.html).

## Basic of GDB

* To use GDB, we have to use **-g** flag when compiling our program.
* To get assembly code output, use **-S** flag. e.g., `gcc -S -o test.s test.c`. Or, if you only have **executable file**, use this command `objdump -d exec-name` 
* To debug a program with GDB, type in `gdb program-name`.
  * use `disassemble function-name` or `disas function-name` to check the assembly code of a function.
  * use `break function-name: line-number` to set a break point.
  * use `x /10c address` to output the content of a specific memory address.
* To show the content of the register:
  * `info register` shows the contents of all registers.
  * `info register register-name` shows a specific register.
* To debug:
  * **s / si**: step / step instruction. Execute until next step with stepping into functions.
  * **n / ni**: next / next instruction. Execute until next step without stepping into functions.
  * **run**: start running the program.
  * **start**: start from the beginning. After execute one step, stop.
* If GDB does not allow you to disassemble a memory address (i.e., **No function contains specified address.**), you can examine the address as instruction by `(gdb) x/i address`.

## Syscall on Centos7

* 64 bit Centos 7 use `syscall` assembly command to invoke an OS system-call handler at privilege level 0.
* The arguments are putted in these register (in order): **%rdi, %rsi, %rdx, %rcx, %r8 and %r9**. And the number of the syscall has to be passed in register **%rax**.
* For more detail, check this [blog](https://blog.packagecloud.io/eng/2016/04/05/the-definitive-guide-to-linux-system-calls/#syscallsysret).

## Nasm

* Linux version of Masm.

* To push a 64 bit number into stack, you need to do so via a register. e.g., 

  ```assembly
  mov rax, 0xACEACEACACEACEAC 
  push rax
  ```

* To include **debug** information in Nasm, use **-g** and **-F dwarf**.

  * `nasm -f elf64 -g -F dwarf hello.asm` 

## Write the shellcode

* assembly code for starting a shell on 64 bit Centos 7 is [here](./shellcode.asm).
* Now, we can `objdump -d shellcode` to get the hexadecimal machine code for our shellcode. (**Watch out for \x00** as it is the null char may be treated as the end of a string.)
* Finally we have a small sample [program](./hex_shell.c) for shellcode. But, REMEMBER, most OS have make the stack unexecutable to protect the system for buffer overflow. So, to make our code execute, we need to use **-z execstack** flag to make the stack executable. `gcc -z execstack -o hex_shell hex_shell.c`.

## Next step

* The next step is to put this shellcode into a buffer overflow scenario to perform attack.

## P.S.

* An interesting [website](https://www.rapidtables.com/convert/number/hex-to-ascii.html) to transform a series of hexadecimal into ASCII char.

