---
title:   "The Microservice-ization of gem5"
authors: "Connor Wang and Samuel Thomas"
pubtype: "poster"
poster:  microservice-gem5-poster_16x9
pubdate: "February 27, 2026"
---

**Abstract.** In computer architecture, gem5 represents the state-of-the-art microarchitectural simulator. It allows for running real programs end-to-end under timing-accurate full system hardware modeling with configurable components in terms of size, latency, bandwidth, and customizable logic. As such, the simulator has tens of thousands of active users in academia and industry. Unfortunately, the tool is extraordinarily arduous to use. Executing a simple “Hello world!” runs for over 300× longer on the simulated platform as it takes for the simulator to run. For more complex simulations, this implies hours of execution to simulate a few seconds of software. This work identifies a significant opportunity for improvement. At its core, gem5 is an single threaded, event-based simulator: hardware routines are encoded as lambda functions with a timestamp and placed in a central priority queue
