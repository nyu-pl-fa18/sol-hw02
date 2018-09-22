# Homework 2 (20 Points)

Please submit your solution via NYU Classes.

The deadline for Homework 2 is Friday, September 21, 6pm.

To obtain a local copy of this repository. Open a terminal at the
location where you want the repository to reside and execute the
command

```bash
git clone https://github.com/nyu-pl-fa18/hw02.git
```

## Problem 1 (4 Points)

Consider the following pseudo code.

```scala
 1: def main () {
 2:   var a: Int = 1
 3:   var b: Int = 2

 4:   def middle () {
 5:     var b: Int = a

 6:     def inner () {
 7:       println(a); println(b)
 8:       var a: Int = 3
 9:     }
       
10:     var a: Int = 3

11:     inner()
12:     println(a); println(b)
13:   }

14:   middle()
15:   println(a); println(b)
16: }
```

Suppose this was code for a language with the declaration-order rules
of C (but with nested subroutines) - that is, names must be declared
before use, and the scope of a name extends from its declaration
through the end of the block. At each print statement, indicate which
declarations of `a` and `b` are in the static scope. What does the
program print when `main` is executed assuming static scoping (or will
the compiler identify static semantic errors)?  Repeat the exercise
for the declaration-order rules of C# (names must be declared before
use, but the static scope of a name is the entire block in which it is
declared).

### Solution

* C rules: in this case, the variables are bound as follows:
  * line 7: `a` -> line 2, `b` -> line 5
  * line 12: `a` -> line 10, `b` -> line 5
  * line 15: `a` -> line 2, `b` -> line 3

  The program first executes the print statements on line 12 when
  `inner` is called within `middle`, which prints `1` and `1`. Then
  the print statements on line 12 are executed after `inner` returns,
  printing `3` and `1`. Finally, the print statements on line 15 are
  executed printing `1` and `2`.
  
* C# rules: in this case, the variables are bound as follows:
  * line 7: `a` -> line 10, `b` -> line 5
  * line 12: `a` -> line 10, `b` -> line 5
  * line 15: `a` -> line 2, `b` -> line 3

  The compiler would produce a static semantic error because the
  occurrence of `a` on line 5 refers to the declaration on line `12`
  in the same block. Thus `a` on line `12` is used before its
  declaration, which is not allowed.

## Problem 2 (4 Points)

Consider the following pseudo code:

```scala
var x: Int // global variable

def set_x(n: Int) {
  x = n
}

def print_x () {
  println(x)
}

def first () {
  set_x(1)
  print_x()
}

def second () {
  var x: Int
  set_x(2)
  print_x()
}

def main () {
  set_x(0)
  first()
  print_x()
  second()
  print_x()
}
```

What does this program print when `main` is executed if the language
uses static scoping? What does it print with dynamic scoping? Why?

### Solution

* Static scoping: In this case, the program prints the sequence `1`,
  `1`, `2`, `2`. This is because with static scoping, the occurrences
  of `x` in `set_x` and `print_x` always refer to the global
  declaration of `x` in the first line. When the first `print_x` call
  within `first` executes, the global variable `x` has just been
  reassigned to value `1`. The second call to `print_x` in `main`
  after `first` returns, again prints the current value `1`. In
  `second`, the call to `set_x` sets the value of global `x` to
  `2`. So the following print statements then print that new value.
  
* Dynamic scoping: In this case, the program prints the sequence `1`,
  `1`, `2`, `1`. Now, the occurrence of `x` in `set_x` and `print_x`
  will always refer to the most recent declaration of `x` that is
  still alive during program execution. For the first two calls to
  `set_x` and `print_x`, this is the global variable `x` declared on
  the first line. These calls therefore behave just like in the case
  for static scoping. However, when the calls in `second` are
  processed, the new declaration of `x` in `second` is in the dynamic
  scope. Hence, the call to `set_x` within `second` sets the local
  variable `x` in that function to `2` but leaves the value of the
  global `x` unchanged (value `1`). The subsequent call to `print_x`
  in `second` then prints the variable of the local `x` (value
  `2`). After `second` returns to `main`, the declaration of `x` in the
  dynamic scope is again the global `x`, which still holds value
  `1`. The final call to `print_x` thus prints `1`.

## Problem 3 (6 Points)

Consider the following fragment of code in C:

```c
{
  int a, b, c;
  ...
  {
    int d, e;
    ...
    {
      int f;
      ...
    }
    ...
  }
  ...
  {
    int g, h, i;
    ...
  }
  ...
}
```

1. Assume that each integer variable occupies 4 bytes and that the
   shown variable declarations are the only variable declarations
   contained in this code. How much total space is required for the
   variables in this code when the program is executed?

2. Recall that the values of local variables for each call to a
   subroutine are stored on the stack in a stack frame (aka activation
   record). The compiler assigns to each local variable of the
   subroutine a static offset that determines the variable's location
   in the stack frame relative to the frame pointer that points to the
   beginning of the stack frame. Thus during the execution of the
   call, the value of each local variable can be accessed by adding
   its fixed static offset to the current value of the frame pointer.
   
   Describe an algorithm that a compiler could use to assign stack
   frame offsets to the variables of arbitrary nested blocks of a
   subroutine, in a way that minimizes the total space required to
   store the values of these variables in the stack frame.

### Solution

* We assume that the shown variables are the only variables that are
  declared in the code. It follows from `C`'s static scoping rules
  that there are at most 6 variables in scope at any time during
  execution of the code. Hence, 24 bytes are sufficient to store these
  variables.

* Without loss of generality, we assume that variable declarations
  only appear at the beginning of a block before any other nested
  blocks or statements. The algorithm could work as follows: we store
  in `offset` the next offset for available free space in the stack
  frame.  We initialize `offset` to 0. Blocks in the function body are
  processed recursively starting with the outermost block. When
  entering a block we first process all variables declared in the
  block in order of appearance. For each variable declaration of a
  variable `x`, we associate the current value of `offset` with this
  declaration of `x` and then increment `offset` by the size of `x`'s
  type (e.g. 4 bytes for an `int`). After processing all declarations,
  we recursively process all subblocks of the current block in the
  order in which they appear in the code. After processing an
  individual subblock, we always set `offset` back to the value it had
  before that subblock was entered. To determine the offset needed to
  implement a memory access to the stack frame for an individual
  occurrence of a variable `x` in the function body, we simply
  determine the declaration of `x` that is in the static scope at that
  point and use the offset calculated for that declaration.


## Problem 4 (6 Points)

As part of the development team at MumbleTech.com, Janet has written a
list manipulation library for C that contains, among other things, the
following code

```c
typedef struct list_node {
  void* data;
  struct list_node* next;
} 

list_node;
list_node* insert(void* d, list_node* L) {
  list_node* t = (list_node*) malloc(sizeof(list_node));
  t->data = d;
  t->next = L;
  return t;
}

list_node* reverse(list_node* L) {
  list_node* rtn = 0;
  while (L) {
    rtn = insert(L->data, rtn);
    L = L->next;
  }
  return rtn;
}

void delete_list(list_node* L) {
  while (L) {
    list_node* t = L;
    L = L->next;
    free(t->data);
    free(t);
  }
}
```

1. Accustomed to Java, new team member Brad includes the following
   code in the main loop of his program:

   ```c
   list_node* L = 0;

   while (more_widgets()) {
     L = insert(next_widget(), L);
   }
   
   L = reverse(L);
   ```

   Sadly, after running for a while, Brad's program always runs out of
   memory and crashes. Explain what's going wrong.

2. After Janet patiently explains the problem to him, Brad gives it another
   try:

   ```c
   list_node* L = 0;

   while (more_widgets()) {
     L = insert(next_widget(), L);
   }
   
   list_node* T = reverse(L);
   
   delete_list(L);
   
   L = T;
   ```
   
   This seems to solve the insufficient memory problem, but where the
   program used to produce correct results (before running out of
   memory), now its output is strangely corrupted, and Brad goes back
   to Janet for advice. What will she tell him this time?

### Solution

1. Evidently Brad assumed that the dynamically allocated memory in his
   program would be garbage collected as in a Java program. This is
   not the case. The function call `reverse(L)` in Brad's program does
   not reverse the list `L` *in place*.  Instead, it creates a new
   list with the reversed data elements from the list `L`. The
   assignment to `L` on the last line of the code snippet overwrites
   the only pointer to the old list `L` with the pointer pointing to
   the head of the reversed list that is returned by `reverse`. The
   list `L` thus remains in memory without being reachable from any
   stack pointer or global pointer via pointer look-ups. That is, the
   list can never be deallocated. The program therefore leaks
   memory. If this code is executed many times, the memory leak will
   accumulate, using up all available heap memory, which will cause
   the program to crash eventually with an "out of memory" error.
  
2. The problem with Brad's new program is that the data elements
   stored in list `L` are shared with the reversed list `T` created by
   `reverse`. Inspecting the code of `delete_list` reveals that this
   function does not just dispose the memory that is allocated for the
   nodes constituting list `L`, but also the memory of the actual data
   elements pointed to by the `data` pointers in each node of the
   list.  Hence, after the call to `delete_list(L)`, all `data`
   pointers in the nodes of the reversed list `T` are dangling, i.e.,
   they point to unallocated memory regions in the heap.  If Brad's
   program later dereferences any of these dangling pointers, the
   program may crash, e.g. with a segmentation fault because it
   accesses unallocated memory, or it may corrupt the state of the
   heap because, in the meantime, the memory pointed to by the
   dangling references has been reallocated for other purposes.
