## How is memory managed?
A memory refers to the physical space where data is stored. Most of the time, as an application developer, you’re dealing with either RAM or drive.

When you’re running an application in Chrome, Chrome uses the V8 Javascript engine, which parses your code and uses your computer’s RAM or drive to store data. Most of what you see is stored in the RAM, things in localStorage and IndexDB are stored on the drive.

There are several types of memories that reside in RAM, amongst which are:
1. Stack memory
2. Heap memory

**Stack memory** refers to variables that exist only within a function itself. They live in the RAM’s stack.

**Heap memory** refers to objects that must persist beyond the lifetime of a single function call. They live in the RAM’s heap.

Most of the time, memory leaks happen when some buggy code introduces objects to get accumulated in the heap. In both the frontend (browser) and the backend (server), it might lead to out of memory crashes.


## JS Heap (i.e. heap live size)
Before we look at the Memory tab, we first go to the Performance tab. When we’re on the Performance tab, we have the option to check the **“Memory”** checkbox when collecting profiles.

If we check that **“Memory”** box and run the profiler, it gives us some graphs that look like this:
<img width="720" height="552" alt="image" src="https://github.com/user-attachments/assets/b86a92f3-38d6-401e-9824-793058b6a546" />

This graph shows the size of the JS heap over time as we interact with the page.

As we interact with a page, memory can grow for many reasons. React, for instance, maintains a virtual DOM representation of the actual DOM, and this virtual DOM lives within the heap memory. As a result, when we add more and more elements to the application (for example, via infinite scroll), we increase the size of this virtual DOM and, in turn, the JS heap. When we create Redux stores, those also live in the heap.

Looking at which user interactions are associated with JS heap growth is a good starting point for investigating memory leaks.

The **“JS heap”** graph here tells us the size of JavaScript heap memory currently in use over time. However, it doesn’t tell us which objects in memory have grown over time.

For that, we need a heap snapshot from the **Memory** tab.

## Dev Tools - Memory Tab
### Heap Snapshot
>Taking a photo of memory at one exact moment

It shows:
1. All JS objects currently in memory
2. How many of each
3. How big they are
4. Who is referencing whom

**When to use it**
1. Memory keeps growing
2. We suspect a leak
3. We want to compare before vs after

