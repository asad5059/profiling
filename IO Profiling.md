# I/O Profiling

When our app feels slow, it's tempting to just guess what's wrong and start fixing things. But more often than not, we end up optimizing the wrong part and wasting hours. That's exactly the problem I/O profiling solves.

---

## What Is I/O Profiling?

**I/O (Input/Output)** refers to any operation where our code talks to something outside itself — a database, a file, an external API, or a network request. These operations involve *waiting*, and waiting is where slowness hides.

**I/O profiling** is simply the act of *measuring* those operations — how many happen, how long they take, and which ones we can speed up. Think of it like putting a stopwatch on different parts of our app to find the slow spots. We never optimize what we haven't measured first.

---

## Tools We'll Use

**Backend (Django)**
- **Django Debug Toolbar** — Shows every SQL query fired during a request, right in the browser. Great for spotting too many queries at a glance.
- **Django Silk** — Records detailed profiling data for every request (queries, timing, payloads) and stores it in a dashboard we can browse later.

**Frontend (React)**
- **Browser DevTools Network Tab** — The simplest and most powerful tool. Shows every request our app makes, how long each took, and whether they're running in parallel or one after another.
- **`performance.now()`** — A built-in browser API that lets us time specific fetch calls right in our code.

---

## Backend Examples

### Example 1: The N+1 Query Problem

This is the most common I/O issue in any app that talks to a database. Let's say we're displaying a list of blog posts with their author names, using SQLAlchemy as our ORM.

```python
# models.py — SQLAlchemy models
from sqlalchemy import Column, Integer, String, ForeignKey
from sqlalchemy.orm import relationship, declarative_base

Base = declarative_base()

class Author(Base):
    __tablename__ = 'authors'
    id = Column(Integer, primary_key=True)
    name = Column(String)

class Post(Base):
    __tablename__ = 'posts'
    id = Column(Integer, primary_key=True)
    title = Column(String)
    author_id = Column(Integer, ForeignKey('authors.id'))
    author = relationship('Author')  # Lazy-loaded by default
```

```python
# ❌ Problematic — runs 1 + N queries
class PostListView(APIView):
    def get(self, request):
        posts = db.query(Post).all()        # 1 query to get all posts
        data = []
        for post in posts:
            data.append({
                'title': post.title,
                'author': post.author.name  # 1 extra query PER post! (lazy load)
            })
        return Response(data)
```

If we have 50 posts, this runs **51 queries**. SQLAlchemy's lazy loading is convenient but silently fires a separate `SELECT` every time we access `post.author`. Django Debug Toolbar (or SQLAlchemy's own query logging) will make this very obvious.

The fix is `joinedload`, which tells SQLAlchemy to eagerly JOIN the authors table and fetch everything in one shot:

```python
# ✅ Fixed — runs just 1 query
from sqlalchemy.orm import joinedload

class PostListView(APIView):
    def get(self, request):
        posts = db.query(Post).options(joinedload(Post.author)).all()  # JOIN in one query
        data = []
        for post in posts:
            data.append({
                'title': post.title,
                'author': post.author.name  # Already loaded, no extra query
            })
        return Response(data)
```

51 queries → 1 query. That's the kind of win profiling reveals.

---

### Example 2: Slow External API Calls Blocking the Response

Imagine our registration view sends a welcome email via an external service. That API call might take 400–600ms, and right now our user is sitting there waiting for it.

```python
# ❌ Problematic — user waits for the email to send
class RegisterView(APIView):
    def post(self, request):
        user = User.objects.create_user(...)
        send_welcome_email(user.email)  # Slow! Blocks the response.
        return Response({'message': 'Account created!'})
```

We can confirm this is the culprit by timing it:

```python
import time

start = time.monotonic()
send_welcome_email(user.email)
elapsed = time.monotonic() - start
print(f"Email API took {elapsed:.3f}s")  # e.g. "Email API took 0.487s"
```

The fix is to move it to a **background task** with Celery so the user gets their response immediately:

```python
# tasks.py
from celery import shared_task

@shared_task
def send_welcome_email_async(email):
    send_welcome_email(email)

# ✅ Fixed — user gets response right away
class RegisterView(APIView):
    def post(self, request):
        user = User.objects.create_user(...)
        send_welcome_email_async.delay(user.email)  # Runs in background
        return Response({'message': 'Account created!'})
```

The email still gets sent — just not on the user's time.

---

## Frontend Examples

### Example 3: Waterfall Requests

A waterfall happens when we fetch data *sequentially* — each request waits for the previous one to finish — when they could all run at the same time. We can spot this in the DevTools Network tab: requests that form a staircase pattern instead of lining up together.

```jsx
// ❌ Problematic — requests fire one after another (waterfall)
useEffect(() => {
  fetch('/api/user/')
    .then(res => res.json())
    .then(user => {
      setUser(user);
      return fetch('/api/posts/');       // Starts only after user loads
    })
    .then(res => res.json())
    .then(posts => {
      setPosts(posts);
      return fetch('/api/notifications/'); // Starts only after posts load
    })
    .then(res => res.json())
    .then(setNotifications);
}, []);
```

If each request takes 200ms, we're waiting 600ms total. The fix is `Promise.all`, which fires all requests at the same time:

```jsx
// ✅ Fixed — all requests fire in parallel
useEffect(() => {
  const fetchAll = async () => {
    const [userRes, postsRes, notifRes] = await Promise.all([
      fetch('/api/user/'),
      fetch('/api/posts/'),
      fetch('/api/notifications/'),
    ]);

    const [user, posts, notifications] = await Promise.all([
      userRes.json(),
      postsRes.json(),
      notifRes.json(),
    ]);

    setUser(user);
    setPosts(posts);
    setNotifications(notifications);
  };

  fetchAll();
}, []);
```

Now total time is determined by the *slowest* request, not the *sum* of all of them. 600ms → ~200ms.

---

### Example 4: Timing Fetch Calls with `performance.now()`

Sometimes we want to measure exactly how long a specific API call takes right from our component — without opening DevTools. We can build a simple timing wrapper using the browser's built-in `performance.now()`:

```jsx
// hooks/useTimedFetch.js
export function useTimedFetch() {
  const timedFetch = async (url, label) => {
    const start = performance.now();
    const response = await fetch(url);
    const data = await response.json();
    const elapsed = performance.now() - start;

    console.log(`[${label}] took ${elapsed.toFixed(1)}ms`);

    if (elapsed > 500) {
      console.warn(`⚠️ Slow request detected: ${label} (${elapsed.toFixed(1)}ms)`);
    }

    return data;
  };

  return timedFetch;
}
```

Using it in a component:

```jsx
function Dashboard() {
  const timedFetch = useTimedFetch();
  const [orders, setOrders] = useState([]);

  useEffect(() => {
    timedFetch('/api/orders/', 'Fetch Orders').then(setOrders);
    // Console: [Fetch Orders] took 843.2ms
    // Console: ⚠️ Slow request detected: Fetch Orders (843.2ms)
  }, []);
}
```

Now whenever a request is suspiciously slow, our console tells us exactly which one and by how much — without us having to manually dig through the Network tab every time.

---

## The Takeaway

I/O profiling doesn't require fancy infrastructure. Start simple:

- Open **Django Debug Toolbar** and look at how many SQL queries your views are firing
- Open the **Network tab** and check if your requests are running in parallel or forming a waterfall
- Add a `time.monotonic()` or `performance.now()` call anywhere something feels slow

Find the bottleneck, fix it, measure again. That's the whole loop.
