# VE280 2022FA RC2

[TOC]

## L3: Developing Programs

### Compilation Process
Compilation process in Linux contains three parts:
- **Preprocessing**: The codes with `#` starts will be implemented.
- **Compiler**: Compiles the `.c`/`.cpp` file into object code. 
  - The `.c`/`.cpp` files will be compiled to `.o`.
- **Linker**: Links object files into an executable. 
  - `.o` files will be linked to an executable file.

### Use g++ to Compile Multiple Resources
Suppose you have three sources files `class.cpp`, `function.cpp` and `main.cpp` in your directory. The simplest way to compile them is:\
`g++ -o [name] class.cpp function.cpp main.cpp`,
where `[name]` can be replaced by any name you want.

However, the above command is an integrated one. The actually compile process should be:

`g++ -c class.cpp`,

`g++ -c function.cpp`,

`g++ -c main.cpp`.

The three commands implement the "compile" part, which compile the source files into `.o` files. In this situation, three files `class.o`, `function.o` and `main.o` will be created in the current directory. Then:

`g++ -o [name] class.o function.o main.o`

will do the "link" part. Finally, an executable file named "[name]" will be created. Notice that you do not need to do the "preprocessing" part by command.

The **benefit** of dividing the compile process apart is that if the project is very large and only a small fraction of the codes are changed, we do not have to compile them again. We only have to recompile the files which are not change and we will save a lot of time. 
### Header File and Header Guard

Another commonly used source file is **header file**, which is usually used to contain the function declarations and class declarations. However, you **don't** need to add the header files in the compiling commands. This is because the header files all already **included** in the preprocessing part.

A very usual problem when we develop a large project is that the header files are included many times. For example, in the previous case, if we write `#include "class.h"` in both `main.cpp` and `function.cpp`, the header file `class.h` is included twice. This may cause multiple definitions of the classes and functions defined in the header file, which will lead to tough problems.

The **header guard** is used to avoid this situation.
```c++
#ifndef CLASS_H
#define CLASS_H

CODE BODY...

#endif
```

`#ifndef VAR` is a conditional diretive. Like the conditional statement in c++, the directive will test whether VAR is defined in the environment. If it is not, the following codes will be implemented until the `#endif`. Different from the conditional statement, `#ifndef` ("n" means not) and `#ifdef` are always implemented in the preprocessing part.

What will happen is the `class.h` is included for the first time? The `#ifndef CLASS_H` will return true and the environmental variable `CLASS_H` will be defined. Then the body codes will be implemented until the `#endif`.

For the second time when the `class.h` is included, since the variable `CLASS_H` already exists in the environment, `#ifndef CLASS_H` will return false and the body codes will not be implemented twice.

You should **always** write the header guard when you write your own header files.

### Makefile
Since the compiling process contains many commands, it is a better way to write these commands together in a file. The file used to help you compile your codes is called: **Makefile**.

Also, we use the example to write a makefile:
```
all: main

main: main.o class.o function.o
    g++ -o main main.o class.o function.o

class.o: class.cpp
    g++ -c class.cpp

main.o: main.cpp
    g++ -c main.cpp

function.o: function.cpp
    g++ -c function.cpp

clean:
    rm -f main *.o
```

If there exists a makefile in the current directory and we type the command "**make**", the computer will implement the first line of the makefile automatically. The "**:**" means "**need**".

The `all` needs the file `main`, then the makefile will go to the part to create `main`. The `main` needs `main.o`, `class.o` and `function.o` and `main` is created by the command `g++ -o main main.o class.o function.o`. Instead of implement the command, the dependent files will be created firstly by their own commands, like `class.o` needs `class.cpp` and is created `g++ -c class.cpp`. Notice that you should always switch the line when you write the command to create a file and there is always a `<Tab>` before the command.

Besides, you can type "**make <target>**" to use some other options of your makefile. For example, if you type `make clean`, the command `rm -f main *.o` will be implemented and all the `*.o` files and the executable file `main` will be removed.

## L4 Review of C++ Basics
### Basice Concepts
Commonly used built-in data types:
`int`, `double`,`float` ,`char`, `string`.

Input and output by "flow":
`cout<<"hello world"<<endl`, `cin>>[variables]`

Operators:
- Arithmetic: `+`,`-`,`*`,`/`
- Comparison: `>=`, `==`
- `x++` or `++x`
- **Flow**: `>>`,`<<`

Flow of controls:
- Branch: if/else, switch/case
- Loop: while, for

### **lvalue** and **rvalue** 
lvalue: An expression which may appear as **either the left-hand or right-hand side** of an assignment
rvalue: An expression which may appear on the **right- but not left-hand** side of an assignment
- Common lvalue: local variables, return type of "++x", *ptr, ptr[index].
- Common rvalue: constant, (x+y), return type of "x++" .

### Fucntion declaration and definition
Declaration: should appear before the function is called.
- Syntax:
```c++
Return_Type Function_Name(Parameter_List);
//comment
```

Definition: can appear after the function is called.
- Syntax:
```c++
Return_Type Function_Name(Parameter_List)
{
    //function body
}
//comment
```

### Reference
We can define a variable as a **reference** of an **existing** variable. For example:
```c++
int a = 1;
int &b = a;
```
where `b` is called the reference of `a`. Reference is just like the pointer, which means if we change the value of `b`, the value of `a` will also be changed. We can only define the reference of a variable, not a constant. Notice that we can not rebind a reference to another variable.

We can also pass the value by reference to a function, like:
```c++
void f(int &a){
    a*=2;
}
```
If we call the function `f(b)`, the function will define `a` as the reference of `b`. If `a` is changed in the function, the value of `b` will also be changed. You should notice that is `b` is the name of an array and `f` is written as:
```c++
void f(int a[])
```
The total array is passed by reference to the function and will be changed by the function.

### Pointers
- Some functions of pointer can be replaced by reference.
- Still very important in the dynamic memory allocation.

### Structs
- A set of variables.
- What is the total memory of a stuct variable?
- How to declare and define a struct? How to create a struct variable?
- How to access a struct pointer's member variables?

## Exercises
Write a struct Complex in file `complex.h`, which contains two `int` called `real` and `imag`. 

Write 2 functions. 
`Complex complexAdd(Complex a, Complex b)`, which return the addition of two complex numbers.
`void complexIncre(Complex &a)`, which add the real and imaginary part of `a` by 1.

You should write the function declaration and definition apart, in header file and cpp file respectivley. Remember to write header guard.

Then, write `main.cpp` which can scan the four integers by user and form 2 complex variables `a` and `b`. Then, output `complexAdd(a, b)` and `complexIncre(a)`. For example, the user input "1 2 3 4", the output should be "3+6i 2+3i". 

Then, write a makefile and obtain the executable file.
