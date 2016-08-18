---
title: Linux 运维
layout: page
date: 2016-08-18
---
[TOC]

## 负载 load average
the number of jobs in the run queue (state R) or waiting for disk I/O (state D) averaged over 1, 5, and 15 minutes.

### PROCESS STATE CODES
- R running or runnable (on run queue)
- D uninterruptible sleep (usually IO)
- S interruptible sleep (waiting for an event to complete)
- Z defunct/zombie, terminated but not reaped by its parent
- T stopped, either by a job control signal or because it is being traced

