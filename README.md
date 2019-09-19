# C-programming-pre-lab

Pre-lab to get started on compiling and running C programs and valgrind

-   [Background](#Background)
    -   [Checking vs. Exploration](#Checking_vs_Exploration)
    -   [Compiling and running a C program](#Compiling_and_running_a_C_progra)
    -   [Using valgrind to find memory leaks](#Using_valgrind_to_find_memory_le)
-   [Exercise](#Exercise)

Background
----------------------------------------

This pre-lab is for the _C and memory management_ labs, parts 1 and 2, 
which includes several C programming exercises with an emphasis on arrays, pointers,
and memory management. The Internet is chock full of C tutorials, etc.;
some are listed on the
[CSci3403 Resources Page](https://github.umn.edu/UMM-CSci3403-F15/Resources/wiki), but there are no doubt zillions of good resources out there we've never heard of.
There are also several examples of C that are written to deliberately
illustrate some relevant concepts in `~mcphee/pub/CSci3401/C_examples`.
We'll be happy to go over any of these in class or lab if you have
questions, so feel free to ask.

### Checking vs. Exploration

[As this article points out nicely](http://www.developsense.com/2009/08/testing-vs-checking.html),
it's useful to make distinction between checking (which is what we
typically call testing in our courses) and exploration (he calls it
testing, but I prefer exploration given that "testing" means other
things). Checking is what we do to see if our code (still) works.
Exploration is what we do to learn more about a domain or a tool or a
language. Exploration is often crucial when we're new to a space, and
it's important to recognize that the stuff we're writing when we explore
is often pretty crappy (because we don't know what we're doing yet). As
a result one often does the exploring off to the side, with the
intention of throwing it away. I bring all this up because I suspect
there will be a fair amount of exploring that goes on during this lab.
Try to be intentional and honest about that. Step off to the side and
try a little exploratory code to figure out if you've got an idea worked
out correctly. Then throw away that "quick and dirty" code, and bring
your new knowledge back to the project at hand.

### Compiling and running a C program

In the exercise below you'll need to edit, (re)compile, and run a C
program that you can find at
`~mcphee/pub/CSci3401/Lab3_prelab/check_whitespace.c`. Assuming you've
copied this to a directory in your space, you can compile this using the
command

```bash
gcc -g -Wall -o check_whitespace check_whitespace.c
```

`gcc` is the GNU C Compiler, which is pretty much the only C compiler
people use on Linux boxes these days. The meaning of the flags:

-   `-g` tells `gcc` to include debugging information in the generated
    executable. This is allows, for example, programs like `valgrind`
    (described below) to list line numbers where it thinks there are
    problems. Without `-g` it will identify functions, but not have line
    numbers.
-   `-Wall` (it's a capital 'W') is short for "Warnings all" and turns
    on *all* the warnings that `gcc` supports. This is a Very Good Idea
    because there are a ton of crazy things that C will try to
    "understand" for you, and `-Wall` tells the compiler to warn you
    about those things instead of just quietly (mis)interpreting them.
    You should typically use `-Wall` and make sure to figure out and
    clean up any warnings you do get.
-   `-o <name>` tells `gcc` to put the resulting executable in a file
    with the given name. If you don't provide the `-o` flag then `gcc`
    will write the executable to a file called `a.out` for strange
    historical reasons.

Assuming your program compiled correctly (**check the output!**) then you
should be able to run the program like any other executable:

```{bash}
./check_whitespace
```

### Using valgrind to find memory leaks

One of the more subtle problems with explicit memory management is that
you can allocate memory that you never free up again when you're done
with it. This will typically never lead to an error, but can cause a
long-running process to consume more and more memory over time until its
performance begins to degrade or it ultimately crashes the system. Since
system processes (e.g. file servers, authentication servers, and web servers) 
often run for days, weeks, or months
between restarts, a memory leak in such a program can be quite serious.
As a simple example, consider the (silly) function:

```C
void f(char *str) {
    char *c = calloc(100, sizeof(char));
    /* Do stuff with c */
    return 0;
  }
```

The problem here is the fact that `f` allocates 100 bytes (100
characters) for `c` to point to which are never freed. This means that
every time we call `f`, 100 bytes will be allocated to this process that
we'll *never* be able to get back because we have no way of accessing
that pointer outside of `f`. To fix that problem (assuming we really
need to allocate that space) we need to free it before we return:

```C
void f(char *str) {
    char *c = calloc(100, sizeof(char));
    /* Do stuff with c */
    free(c);
    return 0;
  }
```

These sorts of memory leaks can actually be really nasty to spot, so
happily there's a nice program called `valgrind` that can help identify
them. If your executable is `my_prog`, then running

``` {bash}
valgrind ./my_prog
```

will run the program as normal, and then print out a memory usage/leak
report at the end. To get more detailed information, including what
lines generate a leak, 

* Make sure to compile your program with the `-g` flag, and
* Add the `--leak-check=full` flag when running `valgrind`:

```bash
valgrind --leak-check=full ./my_prog
```

This generates lots of output of the form:

    ==28587== 18 bytes in 1 blocks are definitely lost in loss record 50 of 50
    ==28587==    at 0x400522F: calloc (vg_replace_malloc.c:418)
    ==28587==    by 0x80486AE: str_reverse (palindrome.c:12)
    ==28587==    by 0x804870A: palindrome (palindrome.c:27)
    ==28587==    by 0x80487FF: not_palindrome (palindrome_test.c:13)
    ==28587==    by 0x8048963: test_long_strings (palindrome_test.c:54)
    ==28587==    by 0x804A1B8: _run_test (cmockery.c:1519)
    ==28587==    by 0x804A5A7: _run_tests (cmockery.c:1624)
    ==28587==    by 0x80489B3: main (palindrome_test.c:68)

This tells you that 18 bytes were lost, and that were allocated by
`calloc` (the top line of the trace), which was called on line 12 of
`palindrome.c` in the function `str_reverse` (next to top line of the
trace), etc. Note that this tells you where the lost bytes were
*allocated*, which doesn't always tell you much about where they should
be *freed*, as that's going to depend on how they're used after they're
allocated.

Exercise
------------------------------------

1. Take the code we gave you for `check_whitespace.c`. Compile the program
and run `valgrind` on it to find any leaks it may have (hint: it has at
least one). 
2. In `Leaks.md` describe why the memory errors happen, and how to fix them. 
3. Actually fix the code.
4. Commit, push, etc.

```
Peter Dolan - 16 Sep 2015
KK Lamberty - 12 Sep 2012 
Nic McPhee - 12 Sep 2012 
Vincent Borchardt - 16 Aug 2012
```
