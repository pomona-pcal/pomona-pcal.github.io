---
title: 'Spring 2026 in Review'
date: 2026-05-30
permalink: /posts/2026/spring/semester-in-review-henry/
author: Henry Cannon
teaser: Henry Cannon's report of his progress and changes to the Secure-Compressed Co-Designed Memory project.
tags:
  - henry cannon
  - scr mmy
  - semester in review
---

This is a report of the research done by Henry Cannon as an RA during the Spring 2026 semester.

# Project Title: Scr Mmy: A Secure Compressed Co-Designed Memory

As the world increasingly uses more and more RAM in daily life, it is pivotal that the information remains secure. However, the methods that are currently available to ensure this come at a cost of time. Secure Memory, a method that uses merkle trees to verify the data is secure in RAM, while proven to accomplish the security, has also been proven to take too long to be able to be used in production.

To solve this problem, our focus moved away from creating a faster method, as security is inherently slow, and instead focused on adding other features into the process that are able to make the slower speed a viable option. The first approach that we began looking into this summer was compression. We aimed to create an architecture that takes a secure memory architecture and co designs it with a memory compression architecture.

The memory compression that we are using as the basis for our compression architecture is DyLeCT compression. We chose this architecture as it uses similar structures that the secure memory uses. Because of this, we can continue to use the memory controller as the main component, and build it in a way that allows for both compression and security.

The work that we did this past summer was the foundation for the direction of the research. I worked to understand the needs of the secure system as well as the different compression systems that have been published. Designing the theoretical model, I began working to recreate the architecture from DyLeCT, using gem5, in a way that would integrate with the Secure Memory. Moving forward in the fall, I will continue to implement the two architectures together to be able to run analysis on the theoretical model. 
