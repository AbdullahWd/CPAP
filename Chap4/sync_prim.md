# Synchronization Primitives
## My answer to Exercise 4-11

### Locks
`Lock`s are perhaps the simplest synchronization primitives in Python. A `Lock` has
only two states - locked and unlocked (surprise). It is created in the unlocked
state and has two principal methods - `acquire()` and `release()`. The `acquire()`
method locks the `Lock` and blocks execution until the `require()` method in some
other coroutine sets it to unlocked. Then it locks the `Lock` again
and returns `True`. The `release()` method should only be called in the locked
state, it sets the state to unlocked and returns immediately. If `release()` is
called in the unlocked state, a `RunTimeError` is raised.

Snippet:
```
lock = Lock()
g = 0

def add_one():
    lock.acquire()
    g += 1
    lock.release()

def add_two():
    lock.acquire()
    g += 2
    lock.release()

threads = []
for func in [add_one, add_two]:
    threads.append(Thread(target=func))
    threads[-1].start()

print(g)
```

### RLocks
The standard `Lock` doesn’t know which thread is currently holding the
lock. If the lock is held, any thread that attempts to acquire it will
block, even if the same thread itself is already holding the lock. (Source: [effbot.org]
[effbot]).
In such cases, `RLock` (re-entrant lock) is used.

Snippet:
```
import threading

num = 0

lock = Threading.Lock()
lock.acquire()
num += 1
lock.acquire()  # This will block.
num += 2
lock.release()

# With RLock, that problem doesn't happen.
lock = Threading.RLock()
lock.acquire()
num += 3
lock.acquire()  # This won't block.
num += 4
lock.release()
lock.release() # You need to call release once for each call to acquire.
```

One good use case for `RLock`s is recursion, when a parent call  of a function would otherwise
block its nested call. Thus, the main use for `RLock`s is nested access to shared
resources.

##Events
The `Event` synchronization primitive acts as a simple communicator between
threads. They are based on an internal flag which threads can `set()` or `clear()`.
Other threads can `wait()` for the internal flag to be `set()`. The `wait()`
method blocks until the flag becomes true. Following snippet demonstrates how
`Event`s can be used to trigger actions.

Snippet:
```
import random, time
from threading import Event, Thread

event = Event()

def waiter(event, nloops):
    for i in range(nloops):
        print("%s. Waiting for the flag to be set." % (i+1))
        event.wait()  # Blocks until the flag becomes true.
        print("Wait complete at:", time.ctime(), "\n")
        event.clear()  # Resets the flag.

def setter(event, nloops):
    for i in range(nloops):
        time.sleep(random.randrange(2, 5))  # Sleeps for some time.
        event.set()

threads = []
nloops = random.randrange(3, 6)
threads.append(Thread(target=waiter, args=(event, nloops)))
threads[-1].start()
threads.append(Thread(target=setter, args=(event, nloops)))
threads[-1].start()

for thread in threads:
    thread.join()
print("All done.")
```

[effbot]: http://effbot.org/zone/thread-synchronization.htm