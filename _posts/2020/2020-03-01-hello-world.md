---
layout: post
author: HackingPheasant
title: Hello, World!
excerpt_separator: <!--more-->
categories: [blog]
tags: [programming, logic]
---

`Hello, World!`

With every programming language you learn, the first program you create is a Hello World program which prints the words "Hello, World!" to the console.<!--more--> An example in C is shown below.

```c
#include <stdio.h>

int main () {
    // Will print "Hello, World" to console
    printf ("Hello, World!\n");
    return 0;
}
```
It may not look like much but there is a lot to unpack here, lets go line by line.

## A line by line look at the hello world example
For this exercise we will be looking at C, a compiled language (Instead of an interpreted language such as Python). Now lets go through the above example code line by line!

### Line 1:
```c
#include <stdio.h>
```
This line tells the preprocessor to replace the line `#include <stdio.h>` with the content found in in the file called stdio.h,  which declares function prototypes to be used by what ever code includes it.

**Note:** 
- \<> Angled brackets tells the preprocessor to search for source files in the system directory
- \"\" Quotation mark tells the preprocessor to search for source files in the local or project specific paths

So now that we know that `#include <stdio.h>` is now replaced with content found in the file, now we can imagine the code looks like the following:
```c
int printf(const char *format, ...);

int main () {
    // Will print "Hello, World" to console
    printf ("Hello, World!\n");
    return 0;
}
```
**Note:** The `...` above indicates a variable-length number of arguments

*But what are function prototypes and where is the  body of code for the printf function?*

Looking at the code above, the printf is a function prototype because it defines only the functions interface (what data goes in and what comes out) and omits the functions body (Actual code/logic). This is useful, mainly for compilers as it allows it to go through the program once and know which functions are in scope at any given time, and allowing the implementation of the function body to be include at a later point of time, it also allows us to catch incorrect calls at compile time instead of having to debug at run time. 

Now, for a real quick overview of what a function interface is made up of. 

A Function interface consists of:
- Functions name 
- Return type
- Parameters and their data types.

*But what is the stdio.h file, what is it doing exactly?*

*stdio.h* **aka** *the standard input/output library* implements functions such as `printf`, `scanf`, `getc` and many others (about 55 functions all together at the time of writing). The use of such libraries allow for the programmer to reuse code, to add a layer of abstraction between the programmer and the dirty nitty-gritty of how a operating system behaves and functions, while easing the overall code development. It usually also stops programmers reinventing the well, allowing for a more consistent experience between programs.

### Line 2:
```c
int main () {
```
This line is the entry point for any program.

*Why main(), and not something like FooBar()? Why int instead of void as the return type?*

Well, for the first question, it is because the language standard doesn't permit it. main() is the agreed upon entry point for the operating system to hand off execution too. Now lets say you control the operating system (E.g. embedded programming), there is nothing stopping you changing the logic so the operating system looks for FooBar() as the entry point into the program instead of main().

Now onto the second question, why int instead of void (or double or whatever) as the return type? Well that is simple, a program will report back to the user/calling process/operating system if the application successfully executed or if an error occurred during execution. So when you reach the end of the main function, you will see `return 0;`. If the compiled program has reached this point then it has successfully executed everything it has required and will will tell the operating system everything is all good when passing the control back to the operating system, if any other value is returned, then some error has occurred and the program has not ran successfully. 

*But that still doesn't explain why it must be int, why not `void main() { ... exit()}` to return an exit code to the environment?*

Again, as stated earlier in a different paragraph, its a simple case of the language standard not permitting it, and just like earlier, if you control the environment then there is nothing stopping you modifying the standard to allow for this to happen.

### Line 3:
```c
// Will print "Hello, World!" to console
```

This is a comment, and its intended for you, the developer, not for the compiler or anything else. Comments aren't included in the compiled binary as the compiler just ignores it when processing the source code, the usual use cases for a comment are to describe the logic of what some piece of code is doing, or a copyright or  license notice, or to document who the author is of a certain piece of code. \

In rare cases, comments are used to **warn about dragons** that you may find lurking in some code. Below are some funny examples of this type of comment, all of which have been grabbed off the internet:

```c
// Here be dragons!
// DO NOT OPTIMIZE THIS CODE
// YOU WILL BREAK SOMETHING ON SOMEBODY'S MACHINE
// LEAVE IT AS IT IS, LEST YOU WASTE YOUR OWN TIME  
```


```c
// 
// Dear maintainer:
//
// When I wrote this, only God and I understood what I was doing
// Now, God only knows
//
// Once you are done trying to 'optimize' this routine,
// and have realized what a terrible mistake that was,
// please increment the following counter as a warning
// to the next guy:
// 
// total_hours_wasted_here = 42
// 
```

While these sort of comments may give you a good laugh reading them, they are usually indicators of code smell and put there by programmers *"to indicate that the next section somehow works even though they don't know why, so please, please don't touch it."*

So if you are putting these types of comments in your code you really need to stop and have a good long think about what the code is try to doing and *fix it* so it makes sense and does what is required.

Anyways, lets get back on topic.

### Line 4:
```c
    printf ("Hello, World!\n");
```
Now the actual meat of the program, and it is pretty straight forward, you call the function `printf` and pass it the sting `Hello, World!\n` as the argument, for which it then writes output to stdout (stdout is typically attached to the user's terminal). 

**Note:** stdout is a wrapper around a thing called a File Descriptor, as called on Unix based Operating systems (Such as Linux, macOS, BSD etc), and in windows file descriptors are known as file handles (A deeper discussion of these types of abstractions are best left for another blog post). 

Remember a paragraph ago where I said you passed printf a string? What I really meant to say was that you pass a bunch of chars (Characters) and a newline (\n) character to indicate end-of-line (**Note:** End of line terminators may differ between operating systems, especially older systems) so that each character gets printed out to stdout via the function.

### Line 5 & 6:
```c
    return 0;
}
```
As described earlier in the blog, `return 0;` is used to indicate that the program has successfully ran and finished without error. And the lone curly bracket "`}`" indicates the end of the function.


## Compiling and running the program
Below is a quick example of compiling the source code into machine code via the gnu C compiler and then executing the compiled binary to see the lovely "Hello, World!" text being printed out to the terminal.
```bash

$ cat example.c 
#include <stdio.h>

int main () {
    // Will print "Hello, World" to console
    printf ("Hello, World!\n");
    return 0;
}
$ gcc example.c 
$ ./a.out 
Hello, World!

$ 

``` 
# Final Thoughts
To paraphrase chapter 1 of of K&R's The C Programming Language: *The only way to learn a new programming language is by writing programs in it, and the first program to write is the same for all languages, print the worlds **hello, world***
 
In reality these hello world programs aren't the easiest pieces of code to under for a complete beginner due to the layers and layers of abstractions we have created to make it easier on us, but it is the fastest way to introduce people to a language and syntax, and it shows you how complicated the syntax for a language is, and useful to illustrate the differences between languages. 

To me personally, `Hello, World!` is the *"gateway drug"* of programming, it is the thing *almost* every programmer has done at least once (especially when they first started learning), and I hope we don't lose it to the sands of time.
