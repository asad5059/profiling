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

**Understanding Memory Snapshot Modes**
* **Summary** — Groups objects by their constructor names. This mode is particularly useful for identifying memory usage patterns and tracking down objects based on their types. For instance, we can see how many instances of a particular object are present and their total memory consumption.
<img width="720" height="269" alt="image" src="https://github.com/user-attachments/assets/2fdccfc1-2317-48de-b476-7142c7741481" />

* **Containment** — Also known as the Heap Contents view, provides a hierarchical representation of objects. It shows how objects are referenced and retained in memory. This mode is beneficial for analyzing the structure of your objects and understanding the relationships between them.
<img width="720" height="285" alt="image" src="https://github.com/user-attachments/assets/c12e34ef-47b2-4d0f-9d3f-724d5bdda5aa" />

* **Statistics** — View presents a pie chart of memory allocation, showing the relative sizes of different memory parts allocated to code, strings, JS arrays, typed arrays, and system objects. This visual representation helps you quickly understand the distribution of memory usage across different types of objects.

<img width="720" height="250" alt="image" src="https://github.com/user-attachments/assets/7586e999-3e12-474f-bd37-691576164062" />

* **Distance** — measures the number of references (or “hops”) needed to reach an object from a root (global object, window object).
* **Shallow Size** — represents the memory occupied directly by the object itself, which includes the memory needed for the name property. This metric does not account for the memory used by objects that the main object references.
  <img width="720" height="277" alt="image" src="https://github.com/user-attachments/assets/e5941dca-1324-403f-bb1b-ea208c77ea01" />
* **Retained Size** is a broader measure that includes the memory occupied by the object itself (Shallow Size) plus all objects that would be garbage-collected if this object were deleted.

### Allocations on timeline
>This tool is for detecting which user action caused memory to be allocated and not released?

The heap allocation profile shows where objects are being created and identifies the retaining path. In the following snapshot, the bars at the top indicate when new objects are found in the heap.

The height of each bar corresponds to the size of the recently allocated objects, and the color of the bars indicate whether or not those objects are still live in the final heap snapshot. Blue bars indicate objects that are still live at the end of the timeline, Gray bars indicate objects that were allocated during the timeline, but have since been garbage collected:
<img width="856" height="499" alt="image" src="https://github.com/user-attachments/assets/d9762765-2717-4bbb-93b3-5bd9c3a0a738" />

### Allocation Sampling
Allocation Sampling is a statistical, periodic approximation of what Allocations on Timeline records exhaustively.

| Key point            | Allocations on Timeline                                         | Allocation Sampling                                                |
| -------------------- | --------------------------------------------------------------- | ------------------------------------------------------------------ |
| **What it answers**  | *When* did memory increase and during which user action?        | *Where* is memory pressure coming from overall?                    |
| **How it works**     | Records **every allocation** with exact timing and stack traces | **Periodically samples** allocations and builds a statistical view |
| **Performance cost** | High (slows the app, heavy)                                     | Low (safe for longer recordings)                                   |
| **Best use case**    | Debugging a **specific, reproducible leak**                     | Finding **memory-heavy code paths** in large apps                  |



## Python - Memory Profile

```python
from memory_profiler import profile
import time

@profile
def create_large_array(n):
    print("Starting function")

    list1 = ['hello'] * n
    list2 = ['world'] * n

    del list2

    print("Finished function")
    return list1


if __name__ == "__main__":
    create_large_array(1_000_000)
```

**Command** : `python -m memory_profiler name_of_script.py`

<img width="862" height="375" alt="image" src="https://github.com/user-attachments/assets/65138014-f3eb-40a0-9200-6ef902c5d760" />


| Column            | Meaning                   |
| ----------------- | ------------------------- |
| **Mem usage**     | Total memory at that line |
| **Increment**     | Memory added by this line |
| **Line contents** | The actual code line      |

