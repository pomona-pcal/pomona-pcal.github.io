---
title: 'Spring 2026 in Review'
date: 2026-05-13
permalink: /posts/2026/spring/semester-in-review-connor/
author: Connor Wang
teaser: Connor Wang's report of his progress and changes to the gem5 project in order to parallelize it.
tags:
  - connor wang
  - parallel gem5
  - semester in review
---
This is a report of the research done by Connor Wang as an RA during the Spring 2026 semester.

# gem5 Parallelization

## Overview
This project explores how to parallelize gem5 through microservicing. The original gem5 implementation executes events sequentially through the `serviceOne()` function, which both removes the next event from the event queue and then executes it. As a result, each event is serviced one at a time. The goal of this project is to separate the process of popping events from the queue and executing the popped events. Ideally, this split will allow multiple worker threads to pull events from the queue and process them in parallel. 

## Approach
Our approach to parallelize gem5 is centered around multiple worker threads executing events at the same time as long as the events do not interact with the same simulated hardware component. To do this, the event queue logic is split into two steps, where one step pops the next event from queue and the second step executes the event. Since multiple threads could run at once, the implementation needs to prevent two threads from modifying the same hardware state at the same time. To handle this, we create a lock for each `event->name()` because in gem5 the event name corresponds to the hardware component associated with the event. The worker threads share the same event queue, with a global queue lock to protect the process of checking the queue. The lock per event protects the actual execution of each event. In theory, this means that multiple events can run in parallel as long as they are associated with different hardware components.

## Progress
The current implementation does not work. It fails the assert `(when >= tick)` in `schedule` and `reschedule`. When commenting these asserts out, the functions do not run properly.

To get gem5 to run properly, edit `simulationWorker` in `simulate.cc`.

The working code that calls `serviceOne` is commented out, so uncomment this and comment out the calls to `popNextEvent` and `executePoppedEvent`.

The functional lines have a comment before them that says:

```cpp
// this is WORKING simulate.cc line
```

The test lines say:

```cpp
// this is for split serviceOne events (DOESNT WORK)
```

---

### `src/sim/eventq.hh`

Lines 863 and 865 — function declarations:

```cpp
// remove head event
Event *popNextEvent();

// does everything that serviceOne would do
Event *executePoppedEvent(Event *event);
```

---

### `src/sim/eventq.cc`

Lines 55–56 — create a dictionary for a lock per event and a lock for the table as a whole. This is meant to prevent multiple events from trying to create locks at the same time.

```cpp
std::mutex mutexTableLock;
std::unordered_map<std::string, std::mutex> eventLocksTable;
```

Lines 58–66 — retrieve or create a mutex for a given event:

```cpp
std::mutex &getEventLock(const std::string &name)
{
    // unlock guard when done adding event to map
    std::lock_guard<std::mutex> guard(mutexTableLock);
    auto result = eventLocksTable.try_emplace(name);
    return result.first->second;
}
```

Lines 293–294 — acquire the event’s mutex and make it unlock once the event is done running:

```cpp
std::mutex &eventLock = getEventLock(event->name());
std::lock_guard<std::mutex> eventGuard(eventLock);
```

Lines 257–279 — first half of `serviceOne`, the part that manages the queue:

```cpp
Event *
EventQueue::popNextEvent()
{
    if (head == NULL)
        return nullptr;

    --queuedEvents;

    Event *event = head;
    Event *next = head->nextInBin;

    event->flags.clear(Event::Scheduled);

    if (next) {
        next->nextBin = head->nextBin;
        head = next;
    } else {
        head = head->nextBin;
    }

    setCurTick(event->when());

    return event;
}
```

Lines 281–309 — second half of `serviceOne`, the part that executes the event’s process:

```cpp
Event *
EventQueue::executePoppedEvent(Event *event)
{
    if (!event) {
        return NULL;
    }

    if (!event->squashed()) {
        // setCurTick(event->when());

        if (debug::Event)
            event->trace("executed");

        currEvent = event;

        std::mutex &eventLock = getEventLock(event->name());
        std::lock_guard<std::mutex> eventGuard(eventLock);

        event->process();

        currEvent = nullptr;

        if (event->isExitEvent()) {
            assert(!event->flags.isSet(Event::Managed) ||
                   !event->flags.isSet(Event::IsMainQueue)); // would be silly

            return event;
        }
    } else {
        event->flags.clear(Event::Squashed);
    }

    event->release();

    return NULL;
}
```

---

### `src/sim/simulate.cc`

Lines 174–178 — struct that holds the event queue, mutex to protect access, and atomic flag to tell worker threads when to exit:

```cpp
struct multiThreads {
    EventQueue *eventq;
    std::mutex lock;
    std::atomic<bool> stop{false};
};
```

Lines 180–234 — worker thread main loop.

This attaches the thread to the event queue and keeps pulling events and running them. It locks the queue while checking if it is empty, then unlocks before executing. It stops once it hits an exit event.

```cpp
static void
simulationWorker(multiThreads *shared, int worker_id)
{
    std::cout << "Worker thread " << worker_id << " running..." << std::endl;

    // Found in eventq.hh; sets the thread-local pointer to the current event queue.
    // This is for single thread, when eventq is a parameter to the function.
    // curEventQueue(eventq);

    curEventQueue(shared->eventq);

    // memory order relaxed is
    while (!shared->stop.load()) {
        Event *event = nullptr;

        {
            // take event queue lock
            std::lock_guard<std::mutex> guard(shared->lock);

            // std::cout << "Work being done by" << worker_id << std::endl;

            if (shared->eventq->empty()) {
                // std::cout << "Worker thread " << worker_id
                //           << " sees an empty event queue." << std::endl;
                break;
            }

            // Optional full queue dump while debugging.
            // shared->eventq->dump();

            // this is WORKING simulate.cc line
            // event = shared->eventq->serviceOne();

            // this is for split serviceOne events (DOESNT WORK)
            event = shared->eventq->popNextEvent();
        } // queue lock is released

        // testing with split serviceOne events (DOESNT WORK)
        Event *exit_event = shared->eventq->executePoppedEvent(event);

        if (exit_event != nullptr) {
            shared->stop.store(true);
            break;
        }

        // this is WORKING simulate.cc line
        // if (event != nullptr) {
        //     shared->stop.store(true);
        //     break;
        // }

        // this is for single thread when eventq is a parameter
        // while (!eventq->empty()) {
        //     Event *event = eventq->serviceOne();
        //     if (event != NULL) {
        //         break;
        //     }
        // }

        std::cout << "Worker thread " << worker_id << " done." << std::endl;
    }
}
```

Lines 238–241 — starts thread using a helper function:

```cpp
static std::thread
startSimulationWorker(multiThreads *shared, int worker_id)
{
    return std::thread(simulationWorker, shared, worker_id);
}
```

Lines 320–339 — sets up worker threads, starts the workers, and joins them once they finish:

```cpp
const int num_workers = 1;

multiThreads shared{getEventQueue(0), {}, false};

// Dynamic array of thread objects; eventually pushes each worker to this.
std::vector<std::thread> worker_threads;

// Capacity for the number of workers.
worker_threads.reserve(num_workers);

for (int i = 0; i < num_workers; i++) {
    int worker_id = next_worker_id.fetch_add(1);

    // New element to end of vector, stores each thread.
    worker_threads.emplace_back(startSimulationWorker(&shared, worker_id));
}

// For each element in worker_threads, call each thread t.
for (std::thread &curr_thread : worker_threads) {
    if (curr_thread.joinable()) {
        curr_thread.join();
    }
}
```
