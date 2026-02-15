---
title: DSA - Introduction to Queues
category: DSA
tags: DSA, algorithms
excerpt: An Introduction to arrays for beginners in which we get to discover a very interesting data structure that you deal with everyday as an engineer.
published_at: 2026-02-09
is_featured: true
---

## What are arrays

An array is just a simple contiguous memory chunks.
They are fixed size, which means it cannot grow.
There is no insert at or push or pop.

## Traversing an array

- Acting on data in a specific index

  1) takes the width of the type
  2) multiplies it by the offset
  3) puts it in the memory addressing
  4) goes and grabs/delete/insert the value at that address

![White](images/oob-white.png)
![Black](images/oob-black.png)

  ### Magic formula

  ```tex
    VALUE + WIDTH (in bytes) * OFFSET
  ```
  - Value: the value to be inserted (in case of insertion)
  - Width: the size of the array
  - Offset: <!--TODO: Understand what this is exactly -->

  This gives a **CONSTANT** time complexity

## Array algorithms

  - Refer to the linked repo

```bash
git clone git@github.com:ThePrimeagen/kata-machine.git
cd kata-machine
yarn install
yarn generate
nvim $(yarn -s day)
```

### Linear Search

```ts
export default function linear_search(haystack: number[], needle: number): boolean {
    for (let i = 0; i < haystack.length; i++) {
    if (haystack[i] === needle) {
      return true;
    }
  }
  return false;
}
```
- Run test

```bash
npx jest <Test name> 
```

### Binary Search

Say we want to search for a value K in an array. The size of the array is N.

In order to be able to find that value, you need to have an efficient way to search maintaining the lowest number of steps to execute. This implies that searching the array value by value is far away from achieving that objective.

#### Prerequisite

At the very beginning, we need an extra step to be able to work efficiently. Our first rule of thumb would be that the array must be sorted (that already adds a O(N) right from the start but it is okay).

This gives us the power of establishing a normalized distribution of elements inside the array. 

- **THE GOLDEN RULE**: The array must be sorted.

### Application

We can leverage on the fact that when we pick a random element from the array, we can certainly know that the elements on the left are less and the ones on the right are greater.

This gives us the power of doing so while evaluating the chosen element against the our target element and decide which side of the array we can illuminate.

That's actually already a great progress. Now we only need to decide that random element and start our process.

We can absolutely move randomly inside the array, but is it effective?

Remember that we need to achieve our goal at the lowest steps possible. And when humans seek to simplify things, they definitely bring it down to a **BINARY** decision.

Yeah, the minimum amount of steps and it is easy. We get to ditch the half of our array each time we choose a position. And that is why we need to pick the middle of the remaining parts of the array each time we evaluate against our target element.

```
    [-----------------------------X----------------------------] 32 element

    [--------------X--------------] 16 element

    [-------x-------] 8 element

    [---x---] 4 elements
  
    [-x-] 2 elements

    [x] 1 elements
```

#### Mathematical Representation


N/2^k = 1

N is the elements in the array

K is the number of steps that we need to perform

2 is the divider that we chose

So we can set both sides to the power of (2^K), this gives:

N = 1^(2K)

N = 2K

This can be also written as:

K = Log(N)


$$
\begin{align*}
&\frac{N}{2^k} = 1 \\
&\text{where:} \\
&\quad N = \text{number of elements in the array} \\
&\quad k = \text{number of steps to perform} \\
&\quad 2 = \text{divider chosen} \\
\\
&\text{Set both sides to the power of } 2^k: \\
&N = 1 \times 2^k \\
&N = 2^k \\
\\
&\text{Taking logarithm on both sides:} \\
&k = \log_2 N \\
\end{align*}
$$
