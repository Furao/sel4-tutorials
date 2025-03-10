/*#
#
# Copyright 2017, Data61
# Commonwealth Scientific and Industrial Research Organisation (CSIRO)
# ABN 41 687 119 230.
#
# This software may be distributed and modified according to the terms of
# the BSD 2-Clause license. Note that NO WARRANTY is provided.
# See "LICENSE_BSD2.txt" for details.
#
# @TAG(DATA61_BSD)
#
-#*/
/*? declare_task_ordering(['hello-world', 'hello-world-mod']) ?*/
# Hello, World!

This tutorial guides you through getting a simple "Hello World" program running as a user-level application on seL4. 
It allows you to test your local set up and make sure all the tools are working before moving onto more detailed tutorials.

## Prerequisites

1. [Set up your machine](https://docs.sel4.systems/HostDependencies). 

## Initialising

/*? macros.tutorial_init("hello-world") ?*/

## Outcomes

By the end of this tutorial, you should:

* Be familiar with the jargon *root task*.
* Be able to build and simulate seL4 projects.
* Have a basic understanding of the role of the `CMakeLists.txt` file in applications.

## Building your first program

seL4 is a microkernel, not an operating system, and as a result only provides very minimal services. 
After the kernel boots, an initial thread called the *root task* is started, which is then responsible for
 setting up the user-level system.
When the root task starts there are no available drivers, however a minimal C library is provided. 

The tutorial is already set up to print "Hello, world!", so at this point 
all you need to do is build and run the tutorial:

/*? macros.ninja_block() ?*/

If successful, you should see the final ninja rule passing:
```
[150/150] objcopy kernel into bootable elf
$
```

The final image can be run by:
/*? macros.simulate_block() ?*/

This will run the result on an instance of the [QEMU](https://www.qemu.org) simulator. 
If everything has worked, you should see:

```
Booting all finished, dropped to user space
/*- filter TaskCompletion("hello-world", TaskContentType.ALL) -*/
Hello, World!
/*- endfilter --*/
```

## Looking at the sources

In your tutorial directory, you will find the following files:
 * `CMakeLists.txt` - the file that incorporates the root task into the wider seL4 build system.
 * `src/main.c` - the single source file for the initial task.
 * `hello-world.md` - A generated README for the tutorial.

### `CMakeLists.txt`

Every application and library in an seL4 project requires a `CMakeLists.txt` file in order to be
 incorporated into the project build system.
 
```cmake
/*-- set build_file --*/
# @TAG(DATA61_BSD)
cmake_minimum_required(VERSION 3.7.2)
# declare the hello-world CMake project and the languages it is written in (just C)
project(hello-world C)

# Name the executable and list source files required to build it
add_executable(hello-world src/main.c)

# List of libraries to link with the application.
target_link_libraries(hello-world
    sel4runtime sel4
    muslc utils sel4tutorials
    sel4muslcsys sel4platsupport sel4utils sel4debug)

# Tell the build system that this application is the root task. 
DeclareRootserver(hello-world)
/*- endset -*/
/*? build_file ?*/
```

### `main.c`

The main C is a very typical C file. For a basic root server application, the only requirement is that 
a `main` function is provided. 

```c
#include <stdio.h>

/*-- filter TaskContent("hello-world", TaskContentType.ALL) -*/
int main(int argc, char *argv[]) {
    printf("Hello, World!\n");

    return 0;
}
/*- endfilter --*/
```

## Making a change

Test making a change to `main.c` by adding a second printf to output `"Second hello\n"`.

```c
/*-- filter TaskContent("hello-world-mod", TaskContentType.COMPLETED) -*/
int main(int argc, char *argv[]) {
    printf("Hello, World!\n");

    printf("Second hello\n");
    return 0;
}
/*- endfilter --*/
```
Once you have made your change, rerun `ninja` to rebuild the project:
/*? macros.ninja_block() ?*/
Then run the simulator again:

/*? macros.simulate_block() ?*/

On success, you should see the following:

```
/*- filter TaskCompletion("hello-world-mod", TaskContentType.COMPLETED) -*/
Second hello
/*- endfilter -*/
```

/*? macros.help_block() ?*/

/*- filter ExcludeDocs() -*/

```c
/*-- filter File("src/main.c") -*/
#include <stdio.h>

/*? include_task_type_replace(["hello-world", "hello-world-mod"]) ?*/

/*- endfilter -*/
```
```cmake
/*- filter File("CMakeLists.txt") -*/
/*? build_file ?*/

# utility CMake functions for the tutorials (not required in normal, non-tutorial applications)
/*? macros.cmake_check_script(state) ?*/
/*- endfilter -*/
```
/*- endfilter -*/
