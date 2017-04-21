**Eeny-meeny-miny-moe problem**

**Task:**
 
Consider the following children’s game:
 
    * n children stand around a circle. 
    * Starting with a given child and working clockwise, each child gets a 
     sequential number, which we will refer to as it’s id. 
    * Then starting with the first child, they count out from 1 until k. The 
     k’th child is now out and leaves the circle. The count starts again 
     with the child immediately next to the eliminated one.
    * Children are so removed from the circle one by one. The winner is the 
     child left standing last.
 
Write a method on a new class, which, when given n and k, returns the 
sequence of children as they go out, and the id of the winning child. Create any
additional classes, tests, etc, you need to support the design of your solution.
 
Please document design decisions you have made i.e. general approach, 
where and why you have sacrificed performance for maintainability or visa versa etc.

**Solution:**

The problem is quite closely resembles well known _Josephus problem_.
Game simulation can be described as follows:

1) Initialize N children data structure
2) Retrieve Child at position K
3) Store Child in elimination sequence
4) Remove Child at position K
5) Repeat from step (2)

Following are the key operations used by game simulation on main data structure:

1) Retrieve Child at position K
2) Remove Child at position K

**Problem classification analysis**

* Realtime / Batch

Solution required to produce winner and elimination sequence as result. Based on this assumption was made that it is required to produce result of batch job 
and not required to be optimised for real-time (e.g. defined response time of N ms) response producing elimination elements in stream. 
However we still can make an assessment for each proposed solution if it will have better real-time response or should be used for batch result calculation.

* Not paralellisable

This problem solved by N sequential iterations and can't be paralellised as every subsequent step depends on the outcome of previous.
There might be some optimisations done but in general it is not a Map/Reduce problem. Some conditions of the problem may need to be relaxed to allow parallelism. 
For example: group of N divided to groups of K then same problem applied to every of those groups, 
then result combined and problem applied again to a result to reduce the set to a single winner.

* Memory impact

TODO: (Is this correct?) what do we need to store for this simulation to run? below is not correct. 
What if we store only holes still eventually we will need to store N elements in memory. 
what if those are stored as bits (of bytes) optimised structures.

Since actual N, K range is not defined (can be anything) below is an attempt to classify problem from memory perspective:

* For N small enough to fit in memory (N < ...) - in memory data structures can be used
* For large N that do not fit in memory (N > ...) - data may need to be partially cached in memory and mostly stored on disk.


**N is small enough to fit in memory solution space**

In memory data structure should support quick random look up, remove operations.
Different performance for those operations reflects on CPU cycles and memory performance. It is briefly discussed for each of the data structures.

1) Array list. 

Array list is backed by arrays. This structure is optimised for O(1) look up at K index position. 
However removal of the elements is expensive O(N) operation. 

As remove operation is used almost on every iterations of N elements overall complexity is O(N^2).

This structure / algorithm works fine for small N, large K.
Each removal of element triggers resizing of array. Resizing of array is done by creating a copy. 
It utilises max of 2 * N elements storage at every step. Doing it for large sets for every of N iterations is quite expensive in terms of total CPU cycles 
(as well creates memory waste which is usually not a big problem for GC but still requires CPU).
Removal of elements at every iteration is the reason algorithm performs poorly for large N.

2) Linked list

Or slightly more convenient Circular list. Circular List is doubly linked list that jumps to first element of sequence after last element was reached and next element requested.

This structure is backed by list / chain of linked elements. 
Look up by index at K position is expensive O(K) operation (as it requires skipping K positions). 
Removal of elements is optimised O(1) operation. 

As look up by index K is used for N iterations overall complexity is O(KN)

This structure works well for large N, small K as average complexity works out to be roughly O(N). e.g. K=1 = O(N)
This structure as well does not have as much impact on memory waste compared to an Array List as removal operation simply 
re-links A-B-C chain to A-C when B removed requiring to GC element B only.


**Implementation of 1 and 2**

As initial implementation I've decided to implement set of tests that can be used to evaluate solution. 
And implemented (1), (2) - used reasonably simple solution as well easy to understand. 

TODO: Tests executed

Array list solution for large N does not run to completion.

Linked list solution performance yield roughly linear time to complete trend for same N, linearly increased K:

    // N = 2,147,483, K = 1 runs around 17 seconds
    // N = 2,147,483, K = 10 runs around 23 seconds
    // N = 2,147,483, K = 100 runs around 1m 40 seconds
    // N = 2,147,483, K = 1000 runs around 15m

**TODO: More advanced data structures:**

3) Trees-like index structures optimised for traversals and removal of elements. 

Complexity is O(n log n)


**TODO: N is large to fit in memory solution space**

If we take it even further and generalise for N that is large enough to fit in memory there are two scenarios:

* For data sets large enough to be handled by enterprise DB or caching solution as ...

I believe this space solutions will be quite similar as bottleneck will be disk operations. As data can't fit in memory
and in order to complete iteration one or several exchanges of data from memory-disk will be required per one of N iterations.

* For data sets large enough and would require more than one DB

As discussed earlier, solution to this problem can't be described in map-reduce way and does not scale horizontally.  
Therefor it should not be attempted to be resolved on map-reduce cluster...
