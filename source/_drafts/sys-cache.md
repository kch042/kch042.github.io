---
title: "Cache from theory to practice"
cover: /img/work.png
mathjax: true
categories: HPC
tags: 
    - Cache
abbrlink: 342893247
date: 2025-04-23 00:00:00
updated: 2025-02-27 00:00:00
---


# TODO
- cache theory
    * memory hierarchy
    * cache design
- experiments
    * step size test
    * false sharing
    * measure the cache size (a classic interview question!)


# Cache Theory

# Experiments

In this section, we are gonna measure
- L1 data cache size
- L1 data cache line size
- L1 set associativity

## Sysyem under test

### Linux
On linux system, we could use `getconf` to get the cache info 
```bash
(base) kch@Ubuntu1604:/data3/kch$ getconf -a | grep CACHE
LEVEL1_ICACHE_SIZE                 32768
LEVEL1_ICACHE_ASSOC                8
LEVEL1_ICACHE_LINESIZE             64
LEVEL1_DCACHE_SIZE                 32768
LEVEL1_DCACHE_ASSOC                8
LEVEL1_DCACHE_LINESIZE             64
LEVEL2_CACHE_SIZE                  1048576
LEVEL2_CACHE_ASSOC                 16
LEVEL2_CACHE_LINESIZE              64
LEVEL3_CACHE_SIZE                  11534336
LEVEL3_CACHE_ASSOC                 11
LEVEL3_CACHE_LINESIZE              64
LEVEL4_CACHE_SIZE                  0
LEVEL4_CACHE_ASSOC                 0
LEVEL4_CACHE_LINESIZE              0
```

As we can see, we have
- L1 cache size: 32 KB
- L1 cache line size: 64 B
- L1 associativity: 8

So we have `32KB / 64B = 512` L1 cache lines, `512 / 8 = 64` L1 sets

### MacOS

On MacOS, we could check out the cache info by

```bash
$ sysctl hw | grep cache
hw.perflevel1.l1icachesize: 131072
hw.perflevel1.l1dcachesize: 65536
hw.perflevel1.l2cachesize: 4194304
hw.perflevel0.l1icachesize: 196608
hw.perflevel0.l1dcachesize: 131072
hw.perflevel0.l2cachesize: 12582912
hw.cacheconfig: 8 1 4 0 0 0 0 0 0 0
hw.cachesize: 3708731392 65536 4194304 0 0 0 0 0 0 0
hw.cachelinesize: 128
hw.l1icachesize: 131072
hw.l1dcachesize: 65536
hw.l2cachesize: 4194304
```

In the following section, we are gonna design and write some code on Linux to check out the cache.

## L1 cache size

The idea here is to 

## L1 cache line size

## L1 set associativity