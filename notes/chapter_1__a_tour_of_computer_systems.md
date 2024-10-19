# Chapter 1: A Tour of Computer Systems
A Computer System consists of Hardware and Systems Software that work together to run application programs. Specific implementations of systems change over time, but the underlying concepts do not. All computer systems have similar hardware and software components that perform similar functions.

## Information is Bits + Context
```c
// Figure 1: the hello world program
#include <stdio.h>

int main() {
    printf("hello, world\n");
    return 0;
}
```
Refer [figure_1__hello_world.c](/programs/chapter_1/figure_1__hello_world.c)

It's ASCII representation

```
cat csapp_notes/programs/chapter_1/figure_1__hello_world.c | od -An -vtu1

           47  47  32  70 105 103 117 114 101  32  49  58  32 116 104 101
           32 104 101 108 108 111  32 119 111 114 108 100  32 112 114 111
          103 114  97 109  10  35 105 110  99 108 117 100 101  32  60 115
          116 100 105 111  46 104  62  10  10 105 110 116  32 109  97 105
          110  40  41  32 123  10  32  32  32  32 112 114 105 110 116 102
           40  34 104 101 108 108 111  44  32 119 111 114 108 100  92 110
           34  41  59  10  32  32  32  32 114 101 116 117 114 110  32  48
           59  10 125
```
It's hexadecimal representation
```
hexdump -C csapp_notes/programs/chapter_1/figure_1__hello_world.c

00000000  2f 2f 20 46 69 67 75 72  65 20 31 3a 20 74 68 65  |// Figure 1: the|
00000010  20 68 65 6c 6c 6f 20 77  6f 72 6c 64 20 70 72 6f  | hello world pro|
00000020  67 72 61 6d 0a 23 69 6e  63 6c 75 64 65 20 3c 73  |gram.#include <s|
00000030  74 64 69 6f 2e 68 3e 0a  0a 69 6e 74 20 6d 61 69  |tdio.h>..int mai|
00000040  6e 28 29 20 7b 0a 20 20  20 20 70 72 69 6e 74 66  |n() {.    printf|
00000050  28 22 68 65 6c 6c 6f 2c  20 77 6f 72 6c 64 5c 6e  |("hello, world\n|
00000060  22 29 3b 0a 20 20 20 20  72 65 74 75 72 6e 20 30  |");.    return 0|
00000070  3b 0a 7d                                          |;.}|
00000073
```

The [hello_world.c](/programs/chapter_1/figure_1__hello_world.c) program is stored as a sequence of bytes. Each byte has an integer value that corresponds to some character. For example, the first two bytes have the integer value of `47` and hexadecimal value of `2F`, which corresponds to the first two characters `//` in the first line. Notice that each line is terminated by the invisible _newline_ character, which is represented by integer `10` or hexadecimal `0A`.

The above two representations of [hello_world.c](/programs/chapter_1/figure_1__hello_world.c) program illustrate a fundamental idea: All information in a system -- including disk files, programs stored in memory and data transferred across a network -- is represented as a bunch of bits. The only thing that distinguishes different data objects is the context in which we view them. For example, in different contexts, the same sequence of bytes might represent an integer, floating-point number, character string or machine instruction. 

## Programs are Translated by other Programs into different forms
The hello program begins life as a high level C Program because it can be read and understood by human beings in that form. However, in order to run [hello.c](/programs/chapter_1/figure_1__hello_world.c) on the system, the individual C statements must be translated by other programs into a sequence of low-level _machine language_ instructions. These instructions are then packaged into a form called an _executable object program_ or _executable object file_ and stored as a binary disk file. 

On a Unix system, the translation from source file to object file is performed by a _compiler driver_:
![Image](/diagrams/chapter_1/figure_3__compilation_system.png "The Compilation System")

```bash
$ gcc -o hello hello.c  # from the book
$ gcc -o hello figure_1__hello_world.c  # on my machine

$ ls programs/chapter_1/
figure_1__hello_world.c hello
```
Here the GCC compiler driver reads the source file [figure_1__hello_world.c](/programs/chapter_1/figure_1__hello_world.c) and translates it into an executable object file `hello`. This transformation is performed in the sequence of 4 phases shown in the above diagram. The programs that perform the four phases (_preprocessor_, _compiler_, _assembler_ and _linker_) are known collectively as the _compilation system_.

> [!NOTE]
> 
> The above paragraph is a summarization from the book. I ran this command:
> ```bash
> $ cd programs/chapter_1/
> $ gcc -save-temps -c -o figure_1__hello_world.o figure_1__hello_world.c
> ```
> Which led to creation of these files
> ```bash
> $ ls
> figure_1__hello_world.bc figure_1__hello_world.c  
> figure_1__hello_world.i  figure_1__hello_world.s  
> figure_1__hello_world.o
> ```
> That's why the mix-up between the names `hello` and `figure_1__hello_world`

- _Preprocessing phase:_ The preprocessor (cpp) modifies the original C program according to the directives that begin with `#` character. For example `#include<stdio.h>` command in line 1 of [hello.c](/programs/chapter_1/figure_1__hello_world.c) tells the preprocessor to read the contents of the system header file `stdio.h` and insert it directly into the program text. The result is another C program, typically with the `.i` suffix.

- _Compilation phase:_ The compiler (cc1) translates the text file [hello.i](/programs/chapter_1/figure_1__hello_world.i) into the text file [hello.s](/programs/chapter_1/figure_1__hello_world.s) which contains an _assembly-language program_. This program includes the following definition of function `main` (copied from the book, actual `.s` file generated on my M1 macbook was different):
    ```asm
    main: 
        subq    $8, %rsp
        movl    $.LCO, %edi
        call    puts
        movl    $0, %eax
        addq    $8, %rsp
        ret
    ```
    Each of the lines describes one low-level machine-language instruction in a textual form. Assembly language is useful because it provides a common output language for different compilers for different high-level languages. For example, C compilers and Fortran compilers both generate output files in the same assembly language.

- _Assembly phase:_ Next, the assembler (as) translates [hello.s](/programs/chapter_1/figure_1__hello_world.s) into machine-language instructions, packages them in a form known as _relocatable object program_, and stores the result in the object file [hello.o](/programs/chapter_1/figure_1__hello_world.o). And as evident, viewing [hello.o](/programs/chapter_1/figure_1__hello_world.o) in a text editor makes it appear gibberish, because it's a binary file.

- _Linking phase:_ the `printf` function called by the [hello.c](/programs/chapter_1/figure_1__hello_world.c) resides in a separate pre-compiled object file `printf.o` and is a part of _Standard C Library_ provided by every C compiler, which must somehow be merged with our [hello.o](/programs/chapter_1/figure_1__hello_world.o) program. The Linker (ld) handles this merging. The output of Linker is an executable object file (or simply _executable_) that is ready to be loaded into memory and executed by the system.

## It pays to understand how Compilation Systems work
For simple programs, we can rely on the compilation system to produce correct and efficient machine code. However there are some important reasons why programmers need to understand how compilation systems work:

- _Optimizing program performance:_ Modern compilers are sophisticated tools that usually produce good code. However, in order to make good coding decisions in our C programs, we do need a basic understanding of machine-level code and how the compiler translates different C statements into machine code. For example is a `switch` statement always more efficient than a sequence of `if-else` statements? Are pointer references more efficient than array indexes?

- _Understanding link-time errors:_ Some of the most perplexing programming errors are related to the operation of the linker, especially when you are trying to build large software systems. For example, what does it mean when the linker reports that it cannot resolve a reference? What is the difference between a static variable and a global variable?

- _Avoiding security holes:_ For many years, _buffer overflow vulnerabilities_ have accounted for many of the security holes in network and Internet servers. These vulnerabilities exist because too few programmers understand the need to carefully restrict the quantity and forms of data they accept from untrusted sources. A first step in learning secure programming is to understand the consequences of the way data and control information are stored on the program stack.

## Processors Read and Interpret Instructions Stored in Memory
(coming soon)