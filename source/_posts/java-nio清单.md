---
title: java nio清单
date: 2018-09-10 11:59:21
tags: [java]
categories: [java]
---

Executors

// ThreadPoolExecutor的核心线程数与最大线程数的缺别
ThreadPoolExecutor(CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE,
                    TimeUnit.SECONDS, sPoolWorkQueue, sThreadFactory)


BlockingQueue

ArrayBlockingQueue,LinkedBlockingQueue


FutureTask