---
title: "Java_gc"
date: 2025-02-20T00:01:43+08:00
draft: false
---

# measure
throughput indicate percentage of time used in application against gc. the more frequent gc work, the more resource it take away from application which affect throughput even if with concurrent. If reduce the frequency of gc and don't gc before garbage has accumulate to large amount, it will decrease total cpu and footprint cost on gc but may increase the paused time in this big gc work round.

response time, as what said above, gc more frequent with short pause time, with small effect to user interactive application.

> 

# design
most of time, we don't want the ultimate, long time paused gc and was supposed to increase gc oppotunity but not influence throughput that much.
1. Generation hypothesis. most objects live a short time, objects age get to more long, the less oppotunity they will be collected. it increase times to gc young objects but decrease times to gc old objects to achieve the balance.
2. concurrent with application at some phase can reduce pause time to gain low latency.


# key
## how to check live objects

### Reachability analysis.  
1. list gc root
2. mark phase(track the reference tree from gc root)
consisder reference as  alive only if there is the path from gc root to the object. gc roots are stack variable, class static variable, string table, JNI, class object.  
finding gc roots phase must stop the world, 
and optimizition direction was about to reduce the range to find gc root, e.g. minor gc only need to find reference in eden space(also inter-generation reference)


mark phase should also start from the consistent snapshot, otherwise the reachability of the reference would change, from live to dead, even worse garbage become referenced by other live ref.

as heap grows, serialize mark phase seek a way to become concurrent, arise the requisite to solve reference relation change through phase.
consisder these scenes: 1. object was scanned but origin reference relation was break. It would become a float garbage can't be collected until next gc round 2. object was not scanned yet and change to be referenced by an already scanned ref, so it won't be scanned in this initial mark round.  

java collection use tri-color marking algorithm in marking phase and to solve above problem which called "lost-object-problem" introduce two other solutions: incremental update and snatshot at the begining(SATB).each solution break one of two condition that must meet if the lost object happens: ·赋值器插入了一条或多条从黑色对象到白色对象的新引用； ·赋值器删除了全部从灰色对象到该白色对象的直接或间接引用。

### SATB
- write barrier to detect b.c = null, speculate c is alive, 
- preserve the b->c relation at marking start
- process c in remarking phase?
- can not handle floating garbage(new garbage)

### multiple marking
take CMS as an example: initial marking phase list gc roots and gc directly reached object. concurrent marking means gc threads work concurrent with application threads. remarking phase handle incremental update during concurent marking phase.


## how to evacuate space  

there are 3 methods: sweep, copy, compact.  
sweep was not efficient when more objs were short-live and also will cause memory fragement.
copy live objects to one place, and sweep last place. not fit into old generation because of high percentage of live objects.
compact was designed for old gen to solve 分配担保.

consisder 3 method, copy & move was not efficient at gc time, but sweep need to do more work(空闲分配链表) when allocate memory to handle memory fragement. And because memory alloc and access are more frequent we can conclude sweep method has low throughput then copy method.  

## phases & STW 

different algorithm and gc implementations pause at stw time in different phase. there are also pause free alogrithm that do not require pause.

commons phases

initial mark
remark
evacuate
re-allocate


# implementation(different collectors)
earlier to latest. for different generation. for throughput or low lattency or balance.

## serialize
- young (copy), old (sweep)
- 
## parnew
- young (copy)
- parallel
- used with cms for old gc
## parallel scavenge
- young (copy)
- compare parnew friendly to cpu resource sensitive application
- not compatible with cms. instead parallel scavenge + parallel old

detail:
`-XX: GCTimeRatio`, lower ratio higher throughput
`-XX: MaxGCPauseTime`, higher throughput means more space to collect and higher pause time. 
`-XX: UseAdaptiveSizePolicy`, GC Ergonomics auto-tune generation sizes by specify preferer

see more detail [here](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/parallel.html)

## cms
- old (sweep)
- phases: initial mark, concurrent mark, re-mark, concurrent sweep
- cons: cpu sensitive, contention resource with application. cannot collect floating garbage may encounter concurrent mode failure(no more free space to allocate for application thread) and will go back to full gc. sweep cause memory fragmentation.

## g1
- region based. collection set instead of whole generation, so called mixed gc.
- region size 1-32MB
- inter-region reference, remember set and card table

### remember set & card table
for the purpose to remember which old region obj has pointers to current region, jvm add write barrier to pointer assigment statement. 
for memory efficiency, instead of recording each object, card table record only high level of object address, which is bucket the obj reside in. 
each bit in card table indicate the bucket at the index is dirty.

remember set is k-v structure, k to find region, v to find card 

### phases

### tuning



## shenandoah

## zgc

generational logic heap memory was implemented in all kinds of collectors and different generation may use different algorithm as discussed before, old generation mark compact, young gen mark copy.

collectors are for specific generation and require other collector to work together.

since cms there


g1 remember set consume much memory 

g1 2nd phase need build rset after checkout candidate set to reclaim , then move to space reclaimation phase collect and reclaiming space in old region from collection set candidate 

rset maintain 



card table each bit says some portion is dirty-has pointers to other generation.

Per region has remember set track these card tables related to its region. When scan the region it will also need to scan the card table region too.

Card table structure maintains?

As remember set will consume much memory if it has many track information so young gen was divided into small regions 

# ref
1. 深入理解Java虚拟机
2. G1 Garbage Collector Details and Tuning by Simone Bordet(https://www.youtube.com/watch?v=Gee7QfoY8ys)

other gc related materials:  
Shenandoah GC Part I ： The Garbage Collector That Could
https://gchandbook.org/contents.html