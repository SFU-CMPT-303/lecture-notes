# Page Replacement Algorithms

* The question is when the memory is full (i.e., all page frames are used) and we need to bring in a
  new page, which page do we swap out from memory to disk?
* *Page fault*: when a memory location is accessed but the corresponding page is not found in
  physical memory, we say that a *page fault* has occurred. In this case, we need to bring in
  the right page to memory. If memory is full and there is no space left for a new page, we run a
  page replacement algorithm to swap out a memory page (from memory to disk) and make room.
* You can read more about this topic in the following chapters of
  [OSTEP](https://pages.cs.wisc.edu/~remzi/OSTEP/).
    * [Chapter 21 Swapping: Mechanisms](https://pages.cs.wisc.edu/~remzi/OSTEP/vm-beyondphys.pdf)
    * [Chapter 22 Swapping: Policies](https://pages.cs.wisc.edu/~remzi/OSTEP/vm-beyondphys-policy.pdf)


## The Optimal Page Replacement Algorithm

* The optimal page replacement algorithm picks the page that will not be used in the future at all
  or that will be used last among all pages currently in memory.
* This assumes that we know the future, which is impossible. Thus, this is only a theoretical
  exercise.
* Example
    * Suppose our memory has 4 page frames.
    * Suppose memory page access occurs like this (by page number): 1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4,
      5
    * The page frames will be used like the following according to the optimal page replacement
      algorithm. `*` indicates a page replacement.

      ```bash
                                                       *             *
      Page access:  1,      2,      3,      4,  1, 2,  5,  1, 2, 3,  4,  5
                  +---+   +---+   +---+   +---+      +---+         +---+
                  | 1 |   | 1 |   | 1 |   | 1 |      | 1 |         | 4 |
                  |---|   |---|   |---|   |---|      |---|         |---|
                  |   |   | 2 |   | 2 |   | 2 |      | 2 |         | 2 |
                  |---|   |---|   |---|   |---|      |---|         |---|
                  |   |   |   |   | 3 |   | 3 |      | 3 |         | 3 |
                  |---|   |---|   |---|   |---|      |---|         |---|
                  |   |   |   |   |   |   | 4 |      | 5 |         | 5 |
                  +---+   +---+   +---+   +---+      +---+         +---+
      ```

        * Accessing page 5 is when we run the replacement algorithm. Page 4 is used last in the
          future, so we swap it out.
        * Later, accessing page 4 also triggers the replacement algorithm. Pages 1, 2, and 3 are not
          used in the future, so we can swap out any one of those.
    * There are 6 page faults.
* Page replacement algorithms try to approximate this as much as possible.

## FIFO (First In, First Out)

* This algorithm keeps track of when a page was brought in to memory. The first one that was brought
  in gets swapped out first.
* Using the same page access sequence as above,

  ```bash
                                                   *       *       *       *       *       *
  Page access:  1,      2,      3,      4,  1, 2,  5,      1,      2,      3,      4,      5
              +---+   +---+   +---+   +---+      +---+   +---+   +---+   +---+   +---+   +---+
              | 1 |   | 1 |   | 1 |   | 1 |      | 5 |   | 5 |   | 5 |   | 5 |   | 4 |   | 4 |
              |---|   |---|   |---|   |---|      |---|   |---|   |---|   |---|   |---|   |---|
              |   |   | 2 |   | 2 |   | 2 |      | 2 |   | 1 |   | 1 |   | 1 |   | 1 |   | 5 |
              |---|   |---|   |---|   |---|      |---|   |---|   |---|   |---|   |---|   |---|
              |   |   |   |   | 3 |   | 3 |      | 3 |   | 3 |   | 2 |   | 2 |   | 2 |   | 2 |
              |---|   |---|   |---|   |---|      |---|   |---|   |---|   |---|   |---|   |---|
              |   |   |   |   |   |   | 4 |      | 4 |   | 4 |   | 4 |   | 3 |   | 3 |   | 3 |
              +---+   +---+   +---+   +---+      +---+   +---+   +---+   +---+   +---+   +---+
  ```

* 10 page faults
* This is simple but does not consider useful properties like locality.

## LRU (Least Recently Used)

* This algorithm replaces page that has not been used for longest period.
    * It tries to approximate the optimal algorithm.
    * It tries to infer the future based on past.
* Using the same page access sequence,

  ```bash
                                                   *           *       *       *
  Page access:  1,      2,      3,      4,  1, 2,  5,  1, 2,   3,      4,      5
              +---+   +---+   +---+   +---+      +---+       +---+   +---+   +---+
              | 1 |   | 1 |   | 1 |   | 1 |      | 1 |       | 1 |   | 1 |   | 5 |
              |---|   |---|   |---|   |---|      |---|       |---|   |---|   |---|
              |   |   | 2 |   | 2 |   | 2 |      | 2 |       | 2 |   | 2 |   | 2 |
              |---|   |---|   |---|   |---|      |---|       |---|   |---|   |---|
              |   |   |   |   | 3 |   | 3 |      | 5 |       | 5 |   | 4 |   | 4 |
              |---|   |---|   |---|   |---|      |---|       |---|   |---|   |---|
              |   |   |   |   |   |   | 4 |      | 4 |       | 3 |   | 3 |   | 3 |
              +---+   +---+   +---+   +---+      +---+       +---+   +---+   +---+
  ```

* 8 page faults
* Keeping track of access time is not simple to implement. The next algorithm approximates this.

## Second-Chance

* This algorithm is an approximation of LRU.
    * Each page has a reference bit (ref_bit), initially = 0
    * When a page is accessed, we set ref_bit to 1.
    * We maintain a moving pointer to the next (candidate) victim.
    * When choosing a page to replace, check ref_bit of victim:
        * If ref_bit == 0, replace it.
        * Else set ref_bit to 0.
            * Leave page in memory (give it another chance).
            * Move pointer to next page.
            * Repeat till a victim is found.

* Example
    * Assume the following state before running the algorithm. We move the next victim pointer down.
      If it reaches the bottom, we move the pointer to the top in a circular fashion.

      ```bash

                  reference bits        pages

                                      +-------+
                      +---+           |       |
                      | 0 |           |       |
                      +---+           |       |
                                      +-------+

                                      +-------+
                      +---+           |       |
      next victim --> | 1 |           |       |
                      +---+           |       |
                                      +-------+

                                      +-------+
                      +---+           |       |
                      | 1 |           |       |
                      +---+           |       |
                                      +-------+

                                      +-------+
                      +---+           |       |
                      | 0 |           |       |
                      +---+           |       |
                                      +-------+
      ```

    * If we need to replace a page, we look at the next victim pointer and check ref_bit according
      to the algorithm. If it is 0, we replace it. In the above case, it is not 0, so we set ref_bit
      to 0 and move the pointer down as follows.

      ```bash

                  reference bits        pages

                                      +-------+
                      +---+           |       |
                      | 0 |           |       |
                      +---+           |       |
                                      +-------+

                                      +-------+
                      +---+           |       |
                      | 0 |           |       |
                      +---+           |       |
                                      +-------+

                                      +-------+
                      +---+           |       |
      next victim --> | 1 |           |       |
                      +---+           |       |
                                      +-------+

                                      +-------+
                      +---+           |       |
                      | 0 |           |       |
                      +---+           |       |
                                      +-------+
      ```

    * We look at the next victim and ref_bit is still not 0. We set it to 0 and move the pointer
      down as follows.

      ```bash

                  reference bits        pages

                                      +-------+
                      +---+           |       |
                      | 0 |           |       |
                      +---+           |       |
                                      +-------+

                                      +-------+
                      +---+           |       |
                      | 0 |           |       |
                      +---+           |       |
                                      +-------+

                                      +-------+
                      +---+           |       |
                      | 0 |           |       |
                      +---+           |       |
                                      +-------+

                                      +-------+
                      +---+           |       |
      next victim --> | 0 |           |       |
                      +---+           |       |
                                      +-------+
      ```

    * Now the next victim's ref_bit is 0. We replace it with a new page.

## Thrashing

* If you run a process that accesses a large amount of memory, you might run into a situation where
  you need to keep bringing in new pages and keep replacing existing pages.
* Thrashing: a process is in a situation where it is too busy swapping in and out pages and not
  really executing its program on the CPU.
