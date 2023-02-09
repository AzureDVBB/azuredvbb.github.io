---
title: "Challenge 001 - Fibonacci Numbers"
date: 2023-02-09T12:44:03+01:00
draft: false
---

## Objectives

- create a command-line application to calculate the `n`-th fibonacci number
	+ accepts 1 command-line argument `n` (integer between 1 and 80)
	+ result is printed to the console (standard output)

## Introduction

The Fibonacci numbers which form the Fibonacci sequence are a classic mathematical concept that is both easy to implement in code and hard to actually compute for any `n`-th Fibonacci number.

Devilishly simple to understand and having a myraid of formulas to calculate it, and even more ways to implement it, each having it's own trade-offs make it a good challenge candidate.

To learn more about the topic and it's applications visit it's [excellent wikipedia article](https://en.wikipedia.org/wiki/Fibonacci_number).

## Details

### Calculating the `n`-th Fibonacci number

The Fibonacci numbers can start from either 0 or 1, here for simplicity we will start the sequence from 0.
Negative numbers are undefined and should cause an error.

Mathematically we define the first two fibonacci numbers as follows:
$$F_{n=0} = 0 ~~~ {and} ~~~ F_{n=1} = 1$$

To calculate the `n`-th Fibonacci number we can use the following formula:
$$F_n = F_{n-1} + F_{n-2}$$

That is, adding the previous two numbers together forms the next one.

**Hint**: recursive programs can have a stack overflow or recursion error if a function refers to itself too many times (too deep a recursion) in most languages.

### Command-line arguments

To specify `n` for the formula we will use the syntax `-n NUMBER` to pass in the variable.

In our case n is equal to or higher then 0, and can go up to 80.
Numbers that are outside the bounds should cause a program error, as numbers in the negative are undefined, while numbers greater then 80 would cause integer overflows.

Full example syntaxt (your mileage may vary depending on platform and language):
```
fib.exe -n 55
```

### Error handling

The program should handle general errors that arise from faulty input from the user, such as but not limited to:

- Inputting text instead of a number
- floating point instead of integer number
- input nothing