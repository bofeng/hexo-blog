---
title: 'Why there is no "gets" function in C standard library?'
date: 2020-04-18 16:39:49
tags: [c,linux,gcc]
published: true
hideInList: false
feature: 
isTop: false
---
If you are new to learn C program, you may find there are many functions like "getc", "putc", "getchar", "putchar", "fgets", "fputs" : every "put" function comes with a corresponding "get" function. But there is a "puts" function while no "gets" function. Why?

Well, the short anwer is, the "gets" function was there before in C89 standard, then it got deprecated in C99 and removed in C11. But let's see why it got removed.

First let's check the function's declaration:
```c
char* gets(char* str)
```

Basically we pass a pre-allocated "str" buffer to this function, `gets` will get user's input and save it into this buffer. So we can write some code like this:
```c
char buffer[20] = {0};
gets(buffer);
printf("Your input is: %s\n", buffer);
```
If we use `gcc` to compile the code, we can see the error something like this:
> main.c:(.text+0x164): warning: the 'gets' function is dangerous and should not be used.

We can see using this function is **dangerous**, but if we ignore this warning and go ahead run our program: if we type "hello world" as our input, the program can correctly put this string out.

So why is it dangerous? In our program we allocated a char array which has 20 slots, and our input "hello world" which are 11 characters, will be saved inside this array from position 0. Our "hello world" case works well, but now imagine what if our input has more than 20 characters, then we won't have enough space to save all of these characters:

```bash
$ ./main
hello world hello world hello world
Your input is: hello world hello world hello world
*** stack smashing detected ***: <unknown> terminated
Aborted (core dumped)
```

In this case, since we don't have enough space to save user's input, the program crashed.

Sometimes this may cause bug that harder to find, suppose we have code like this:
```c
#include <stdio.h>

int main() {
    char c = 'c';
    char arr[10] = {0};
    printf("before gets: variable c is %c\n", c);
    gets(arr);
    printf("after gets: variable c is %c\n", c);
    return 0;
}
```
Before we compile and run the code, guess what the output would be if in the `gets` step, we type "hello world".

Your guess might be the program will crash since "hello world" in total are 11 characters (including the white-space in the middel) but our allocated `arr` only have 10 slots. Well, you are not wrong with the **modern compiler**, by that I mean, if we compile the code and run it:

```bash
$ gcc main.c -o main
main.c:(.text+0x51): warning: the 'gets' function is dangerous and should not be used.
$ ./main
before gets: variable c is c
hello world
after gets: variable c is c
*** stack smashing detected ***: <unknown> terminated
Aborted (core dumped)
```

But in the old compiler, you might won't have this crashed. The modern `gcc` will compile with a default "stack-protector" turned on. Let's try to turn it off then run again:
```bash
$ gcc main.c -fno-stack-protector -o main
main.c:(.text+0x42): warning: the 'gets' function is dangerous and should not be used.
$ ./main
before gets: variable c is c
hello world
after gets: variable c is d
```
Ah, this time the program doesn't crash. But our variable 'c' which is declared before `arr` has been "magically" changed to the character 'd', which is the last letter in "world".

To understand this, we need to know how the program allocate spaces in stack. Most of the operating system will allocate stack from high address to low address. The stack in our program will be something like this:
![](/post-images/1587244468586.png) 

When we input "hello world", program will try to save this string from `arr[0]`, like this:
![](/post-images/1587244621171.png)

So technically we still have 11 slots, but the last one is not for our array, it is the space for our variable `c`. When the program saved the whole string, it put the last letter 'd' in our `char c` slot - that's why after input our varaible `c` has been modified to `d` ! This kind bug is quite dangerous and not easy to find!

### The solution
The alternative function we should use is `fgets`:
```c
char* fgets(char *string, int length, FILE * stream);
```
So we can change our code to:
```c
#include <stdio.h>

int main() {
    char c = 'c';
    char arr[10] = {0};
    printf("before gets: variable c is %c\n", c);
    fgets(arr, 10, stdin);
    printf("after gets: variable c is %c\n", c);
    return 0;
}
```

Use `gcc` to compile and run it:
```bash
$ gcc main.c -o main
$ ./main
before gets: variable c is c
hello world
after gets: variable c is c
```
Two things changed here: 1) when compile, there is no warning info printed out 2) although our "hello world" contains 11 letters, our program didn't crash anymore and our variable `c` hadn't been overwritten.


### Reference
* [stack smashing detected](https://stackoverflow.com/questions/1345670/stack-smashing-detected)
* [gets is deprecated in c99](https://en.wikipedia.org/wiki/C_file_input/output#gets)
* [fgets function](https://en.wikibooks.org/wiki/C_Programming/stdio.h/fgets)


