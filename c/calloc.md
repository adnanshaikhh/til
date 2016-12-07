# Why use Calloc?

When programming in C, there are two standard ways to allocate some new memory on
the heap:

```
void* buffer1 = malloc(size);
void* buffer2 = calloc(count, size);
```
```calloc``` is different in two ways: first, it takes two arguments instead of
one, and second, it returns memory that is pre-initialized to be all-zeros.
One would assume that ```calloc``` = ```malloc``` + ```memset```

## Difference 1: Computers are bad at arithmetic

When ```calloc``` multiplies ```count * size```, it checks for overflow, and errors out if
the multiplication returns a value that can't fit into a 32- or 64-bit integer
(whichever one is relevant for your platform).
This is good. If you do the multiplication the naive way by just writing
```count * size```, then if the values are too large then the multiplication
will silently wrap around, and malloc will happily allocate a smaller buffer than
we expected.

Try this out

```
#include <stdlib.h>
#include <stdio.h>
#include <errno.h>
#include <stdint.h>

int main(int argc, char** argv)
{
    size_t huge = INTPTR_MAX;

    void* buf = malloc(huge * huge);
    if (errno) perror("malloc failed");
    printf("malloc(huge * huge) returned: %p\n", buf);
    free(buf);

    buf = calloc(huge, huge);
    if (errno) perror("calloc failed");
    printf("calloc(huge, huge) returned: %p\n", buf);
    free(buf);
}
```
Results
```
~$ gcc calloc-overflow-demo.c -o calloc-overflow-demo
~$ ./calloc-overflow-demo
malloc(huge * huge) returned: 0x55c389d94010
calloc failed: Cannot allocate memory
calloc(huge, huge) returned: (nil)
```

So yeah, apparently malloc successfully allocated a 73786976294838206464
exbiyte array? I'm sure that will work out well. This is a nice thing about
```calloc```: it helps avoid terrible security flaws.

## Difference 2: Lies, damned lies and virtual memory

Here's a little benchmark program that measures how long it takes to calloc a
1 gibibyte buffer versus ```malloc+memset``` a 1 gibibyte buffer. (Make sure you
compile without optimization, because modern compilers are clever enough to
know that ```free(calloc(...))``` is a no-op and optimize it out!)

```
#include <stdlib.h>
#include <stdio.h>
#include <time.h>
#include <string.h>

const int LOOPS = 100;

float now()
{
    struct timespec t;
    if (clock_gettime(CLOCK_MONOTONIC, &t) < 0) {
        perror("clock_gettime");
        exit(1);
    }
    return t.tv_sec + (t.tv_nsec / 1e9);
}

int main(int argc, char** argv)
{
    float start = now();
    for (int i = 0; i < LOOPS; ++i) {
        free(calloc(1, 1 << 30));
    }
    float stop = now();
    printf("calloc+free 1 GiB: %0.2f ms\n", (stop - start) / LOOPS * 1000);

    start = now();
    for (int i = 0; i < LOOPS; ++i) {
        void* buf = malloc(1 << 30);
        memset(buf, 0, 1 << 30);
        free(buf);
    }
    stop = now();
    printf("malloc+memset+free 1 GiB: %0.2f ms\n", (stop - start) / LOOPS * 1000);
}
```
Results
```
~$ gcc calloc-1GiB-demo.c -o calloc-1GiB-demo
~$ ./calloc-1GiB-demo
calloc+free 1 GiB: 3.44 ms
malloc+memset+free 1 GiB: 365.00 ms
```

i.e., ```calloc``` is more than 100x faster. Our textbooks and manual pages
says they're equivalent. What the heck is going on?  
The answer, of course, is that calloc is cheating.

For small allocations, ```calloc``` literally will just call ```malloc+memset```,
so it'll be the same speed. But for larger allocations, most memory allocators
will for various reasons make a special request to the operating system to
fetch more memory just for this allocation. ("Small" and "large" here are
determined by some heuristics inside your memory allocator; for glibc "large"
is anything >128 KiB, at least in its default configuration).

When the operating system hands out memory to a process, it always zeros it out
first, because otherwise our process would be able to peek at whatever detritus
was left in that memory by the last process to use it.
So that's the first way that ```calloc``` cheats: when you call ```malloc```
to allocate a large buffer, then probably the memory will come from the
operating system and already be zeroed, so there's no need to call ```memset```.
But you don't know that for sure! Memory allocators are pretty inscrutable.
So you have to call memset every time just in case. But ```calloc``` lives inside
the memory allocator, so it knows whether the memory it's returning is fresh
from the operating system, and if it is then it skips calling ```memset```. And
this is why ```calloc``` has to be built into the standard library, and you can't
fake it yourself.

But this only explains part of the speedup: ```memset+malloc``` is actually
clearing the memory twice, and ```calloc``` is clearing it once, so we might
expect ```calloc``` to be 2x faster at best. Instead... it's 100x faster.
What the heck?

It turns out that the kernel is also cheating! When we ask it for 1 GiB of
memory, it doesn't actually go out and find that much RAM and write zeros
to it and then hand it to our process. Instead, it fakes it, using virtual
memory: it takes a single 4 KiB page of memory that is already full of zeros
(which it keeps around for just this purpose), and maps 1 GiB / 4 KiB = 262144
copy-on-write copies of it into our process's address space. So the first time
we actually write to each of those 262144 pages, then at that point the kernel
has to go and find a real page of RAM, write zeros to it, and then quickly swap
it in place of the "virtual" page that was there before. But this happens
*lazily*, on a page-by-page basis.

So in real life, the difference won't be as stark as it looks in our benchmark
up above – part of the trick is that ```calloc``` is shifting some of the cost
of zero'ing out pages until later, while ```malloc+memset``` is paying the full
price up front.

BUT, at least we aren't zero'ing them out twice. And at least we aren't trashing
the cache hierarchy up front – if we delay the zero'ing until we were going
to write to the pages anyway, then that means both writes happen at the same
time, so we only have to pay one set of TLB / L2 cache / etc. misses. And, most
importantly, it's possible we might never get around to writing to all of those
pages at all, in which case ```calloc``` + the kernel's sneaky trickery is a
huge win!

So basically, ```calloc``` exists because it lets the memory allocator and
kernel engage in a sneaky conspiracy to make your code faster and use less
memory. You should let it! Don't use ```malloc+memset```!