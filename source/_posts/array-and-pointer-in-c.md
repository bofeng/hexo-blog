---
title: 'Array and Pointer in C'
date: 2020-04-05 11:01:28
tags: [c]
published: true
hideInList: false
feature: 
isTop: false
---
### Array is and is not a pointer
Is array a pointer in C? seems we can treat it as a pointer when we access its element. But sometimes Array has different behavior from pointer.

Take this for example:
```c
int main() {
    int arr[10] = {0};
    printf("size: %ld", sizeof(arr));
    return 0;
}
```

If `arr` here is a pointer, then seems this should print out the size of a pointer (4 or 8, depends on the Operating System), but if we compile this code, it prints out 40, so clearly it is `sizeof(int) * 10`.

I like the quote here:
> The first step to learning C is understanding that pointers and arrays are the same thing. The second step is understanding that pointers and arrays are different.

### Pass array as parameter to function
Ok, from this `sizeof` at least we know they are different, but how do we explain this:
```c
#include <stdio.h>

void func(int arr[]) {
    size_t size = sizeof(arr);
    printf("Inside func: %ld\n", size);
}

int main() {
    int arr[10] = {0};
    size_t size = sizeof(arr);
    printf("Inside main: %ld\n", size);
    func(arr);
    return 0;
}
```

We passed `arr` as parameter to function `func`, if we compile and run this program, it prints out:
```
Inside main: 40
Inside func: 8
```

Well, although we set the parameter type to be `arr[]` in function `func`, but inside the function, `arr` behaves like a pointer now (it prints 8, instead of 40). So what's the magic here? we call the same `sizeof` function, why inside main function it prints out 40, and inside `func`, it prints 8?

This problem bothered me for a while, I decide to check the generate assembly code to see what's the difference for these 2 `sizeof` call.

```bash
$ gcc -S -fverbose-asm main.c
```

A file `main.s` is generated, open this file, first let's check the `sizeof` inside main function:
```assembly
# main.c:10:     int arr[10] = {0};
    movq    $0, -48(%rbp)   #, arr
    movq    $0, -40(%rbp)   #, arr
    movq    $0, -32(%rbp)   #, arr
    movq    $0, -24(%rbp)   #, arr
    movq    $0, -16(%rbp)   #, arr
# main.c:11:     size_t size = sizeof(arr);
    movq    $40, -56(%rbp)  #, size
```

Here the `movq` is move 8 bytes command. We can see after we defined `arr`, it first moved 8 bytes for 5 times from the `arr` address, then for our `sizeof` call, the compiler immediately set the size to be 40. Ha, so this size would be already set during the compile time, not at run time! The compiler knows `arr` here is an array's base address, it knows the array's size.

But the `sizeof` call in `func` function gave us 8, its assembly code:
```assembly
    movq    %rdi, -24(%rbp) # arr, arr
# main.c:5:     size_t size = sizeof(arr);
    movq    $8, -8(%rbp)    #, size
```
We can see here, compiler directly put 8 to the `size` variable. The reason is when we pass an array to a function, it will be downgraded to a pointer, so although we write the funcion to be `func(int arr[])`, when we pass the `arr` to this function, compiler will treat it as `func(int* arr)`.

But it seems weird to me: the compiler should be definitely smart enough to track the passed parameter and can tell it is an array. The only reasonable explanation I found is the "historical" reason:
> The only purpose of this rule is to maintain backwards compatibility with historical compilers that did not support passing aggregate values as function arguments.

But anyway, if the C standard says so, ok, at leaset we know when pass array as parameter, it is a pointer. So inside the function, if we want to know how many elements that array contains, we cann't use `sizeof(arr)/sizeof(arr[0])` any more, because `sizeof(arr`) will just give us the size of a pointer.

So in the `func` function, how do we tell the array's size? An obvious way is we can pass another length paramter to tell this `func` the array length.
```c
void func(int arr[], int len) {
   // ...
}
```
Or we use the array's address as the paramter :
```c
void func2(int (*arr)[10]) {
    size_t size = sizeof(*arr);
    printf("Inside func2: %ld\n", size);
}
```
If in our `main` function, we call this `func2` as `func2(&arr);`, it will print `Inside func2: 40`.

For our `func2` function, we write a fixed length 10 there. If we want the same trick works for variable length array, we could define the function like this:
```c
void func3(int n, int (*arr)[n]) {
    size_t size = sizeof(*arr);
    printf("Inside func3: %ld\n", size);
}
```
Then call it like `func3(10, &arr)`, it will also print out the size to be 40.

### Variable Length Array  (VLA) 

In the above example, we defined a fixed length array, and when compiler sees `sizeof`, it immidiately know the size. But what if the array's length is determined at the run time? Say if we have code:
```c
int main() {}
    int len = 0;
    scanf("%d", &len);
    int arr2[len];
    size = sizeof(arr2);
    printf("arr2 size: %ld\n", size);
    return 0;
}
```
Here we take the length from user input, can compiler output the right array size in this case? or it will just treat the array as a pointer now?

If we try it: we type 10 as the length, it prints arr2 size: 40; we type 20 as the length, it prints arr2 size: 80. So although the size determined during run time, the compiler still has a way to tell use the correct size.

If we check the generated assembly code, this time the compiler generated much more than the fixed length:
```assembly
# main.c:17:     int arr2[len];
    movl    -124(%rbp), %ecx    # len, len.0_19
# main.c:17:     int arr2[len];
    movslq  %ecx, %rax  # len.0_19, _1
    subq    $1, %rax    #, _2
    movq    %rax, -112(%rbp)    # _2, D.2335
    movslq  %ecx, %rax  # len.0_19, _4
    movq    %rax, %r14  # _4, _5
    movl    $0, %r15d   #, _5
    movslq  %ecx, %rax  # len.0_19, _9
    movq    %rax, %r12  # _9, _10
    movl    $0, %r13d   #, _10
    movslq  %ecx, %rax  # len.0_19, _12
    leaq    0(,%rax,4), %rdx    #, _24
    movl    $16, %eax   #, tmp122
    subq    $1, %rax    #, tmp103
    addq    %rdx, %rax  # _24, tmp104
    movl    $16, %edi   #, tmp123
    movl    $0, %edx    #, tmp107
    divq    %rdi    # tmp123
    imulq   $16, %rax, %rax #, tmp106, tmp108
    movq    %rax, %rdx  # tmp108, tmp110
    andq    $-4096, %rdx    #, tmp110
    movq    %rsp, %rsi  #, tmp111
    subq    %rdx, %rsi  # tmp110, tmp111
    movq    %rsi, %rdx  # tmp111, tmp111
.L3:
    cmpq    %rdx, %rsp  # tmp111,
    je  .L4 #,
    subq    $4096, %rsp #,
    orq $0, 4088(%rsp)  #,
    jmp .L3 #
.L4:
    movq    %rax, %rdx  # tmp108, tmp112
    andl    $4095, %edx #, tmp112
    subq    %rdx, %rsp  # tmp112,
    movq    %rax, %rdx  # tmp108, tmp113
    andl    $4095, %edx #, tmp113
    testq   %rdx, %rdx  # tmp113
    je  .L5 #,
    andl    $4095, %eax #, tmp114
    subq    $8, %rax    #, tmp114
    addq    %rsp, %rax  #, tmp115
    orq $0, (%rax)  #,
.L5:
    movq    %rsp, %rax  #, tmp109
    addq    $3, %rax    #, tmp116
    shrq    $2, %rax    #, tmp117
    salq    $2, %rax    #, tmp118
    movq    %rax, -104(%rbp)    # tmp118, arr2.1
# main.c:18:     size = sizeof(arr2);
    movslq  %ecx, %rax  # len.0_19, _14
# main.c:18:     size = sizeof(arr2);
    salq    $2, %rax    #, tmp119
    movq    %rax, -120(%rbp)    # tmp119, size
```

We won't dive into details for this code (to see the explanation, you can [check it here](https://stackoverflow.com/questions/10078283/how-sizeofarray-works-at-runtime)), but we can see for VLA, `sizeof` can still correctly evaluate the array size in run time..

### malloc and free

Another question is, we know if we allocate some memory using `malloc`, and when we free the memory with the `free` function, we never pass a second parameter to tell the size (how many memory needs to be free'ed), how does the compiler know that?

The magic is behind the `malloc` function, for example, when we call the `p = malloc(10)` function, although it returns us the address for 10 bytes, under the hood, it will allocate more to keep some meta data like the size. For example, a simplest method could be, it allocate 4 bytes + 10 bytes where the first 4 bytes is used to save the size 10, then it returns the memory address starts from the 10 bytes. Later when we call the `free(p)` function, `free` can first use p - 4 bytes to read what the size is, then do a proper delete.

### Takeaways
1. array and pointer are different
2. when pass an array as parameter to function, it decays to pointer
3. we can pass a array pointer as parameter to keep it behaves as an array instead of a pointer
4. even array's length is determined during the run time, sizeof can still print out correct size
5. malloc function will allocate more to save the size info such that later the free function can get the correct size to free

### Reference:
1. [https://www.quora.com/Why-doesn%E2%80%99t-sizeof-a-sizeof-a-0-work-for-an-array-passed-as-a-parameter](https://www.quora.com/Why-doesn%E2%80%99t-sizeof-a-sizeof-a-0-work-for-an-array-passed-as-a-parameter)
2. [https://stackoverflow.com/questions/851958/where-do-malloc-and-free-store-allocated-sizes-and-addresses](https://stackoverflow.com/questions/851958/where-do-malloc-and-free-store-allocated-sizes-and-addresses)
3. [https://stackoverflow.com/questions/10078283/how-sizeofarray-works-at-runtime](https://stackoverflow.com/questions/10078283/how-sizeofarray-works-at-runtime)



