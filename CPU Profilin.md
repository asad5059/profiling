## CPU Profiling

Measuring how much execution time each function or code path consumes while our program runs. It answers questions like:

* Which function is taking the most time?
* Why is this API endpoint slow?
* Is my React component re-rendering too much?
* Is my Django view doing heavy computation?

If we run a Django endpoint and it takes 5 seconds, what would we want to know? Not `it took 5 seconds.` But: `Which function inside that endpoint is responsible for most of those 2 seconds?` That’s exactly what CPU profiling tells you.

Let's say we have an API called `compute_kpi` in our django app inside our view. This API consists of:
* DB query
* Looping through the results
* Serialization
* JSON response

If our API above takes 5 seconds, then it could mean:
* The ORM query inside it is slow
* A loop inside the API is heavy
* Serialization is expensive

So CPU profiling helps us answer `Is the slowness in this function itself, or in something it calls?`

## Tools we use for profiling
### cProfile
<img width="1539" height="581" alt="image" src="https://github.com/user-attachments/assets/3f4af4b8-624d-4724-9976-0ba6e6ab0a67" />

When we profile a function or any API we see some terms like:
1. `ncalls`: number of calls. How many times the function was executed.
2. `tottime`: time spent in the function itself. (excluding time spent in functions it calls)
3. `percall (next to tottime`: On average, how expensive is one execution of this function (excluding children). tottime / ncalls
4. `cumtime`: time spent in the function including children
6. `percall (next to cumtime)`: It helps understand average total cost including children.

### SnakeViz
SnakeViz is a browser based graphical viewer for the output of Python’s cProfile module. We can create a profile using cProfile inside the program itself or using command line.

#### Creating profile using cProfile in command line
`python -m cProfile -o program.prof my_program.py`

#### Creating profile using cProfile in the program itself

```python
import cProfile

def my_program(a, b, c):

    profiler = cProfile.Profile()
    profiler.enable()    

    res = op1(a, c)

    for i in 10:
      res = res + op2(b, i)

    profiler.disable()
    profiler.dump_stats("program.prof")

    return res;
```

#### Visualizing the profile with SnakeViz
`python -m snakeviz create_conversation_note.prof`

<img width="1900" height="909" alt="image" src="https://github.com/user-attachments/assets/514349d6-6751-4dc6-b5ef-3ec64f41e36e" />

