---
title: 'Spring 2026 in Review'
date: 2026-05-12
permalink: /posts/2026/spring/semester-in-review-stevie/
author: Steven Kim
teaser: Stevie Kim's report of his progress and changes to the CXip List project.
tags:
  - steven kim
  - cxip list
  - semester in review
---

This is a report of the research done by Stevie Kim as an RA during the Spring 2026 semester.

# Project Title: CXip List: A CXL-Aware Skip List

## Description

Modern workloads require systems that have access to both large and fast memory. However, developments in memory systems lag behind developments in compute, leading memory to be the primary bottleneck of such systems. One of the state-of-the-art solutions that can provide both large and fast memory is Compute Express Link (CXL), an open interconnect standard that allows for the attachment of remote memory modules to a system via IO, offering high bandwidth and low latency. Each memory module has their own latency, and thus systems that incorporate CXL have a memory topology where certain remote memory modules have lower latencies than other. Thus, to design a performant system for CXL systems, applications should use this topology for optimizations.

This leads us to our main problem: current interfaces cannot distinguish between individual remote memory modules. Instead, they view all remote memory modules as one large node, as shown by the image below.

![zNUMA Interface](/images/blog-posts/zNUMA_interface.png)

Since we cannot distinguish between individual remote memory modules, we cannot design systems around the unique memory topology that CXL provides. Thus, applications running on CXL systems will have inefficiencies, as it is highly likely that memory accesses occur across individual remote nodes, which is proven to be unoptimal by prior literature.

Thus, this project has two research goals: (1) to create a new interface in the `gem5` simulator for CXL systems that distinguishes between individual memory modules, and (2) to create *CXip List*, a skip list that is optimized for CXL systems. We hope that this work will provide the necessary tools and guidelines that can enable others to develop more CXL-aware applications.

Current Progress: Stevie worked on this project throughout the Spring 2026 semester. He worked on implementing skip lists in `C++`, starting from a simple sequential implementation and moving to a concurrent implementation using locks. He also gave a 15-minute talk about this project at the SoCal Programming Languages and Systems Workshop in February.
