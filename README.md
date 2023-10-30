# topX

While it will sort a text file full of ints,
It's utterly naive in every way.

## Usage

```bash
./topX <x> <file>
```

`topX` takes 2 arguments: `x` and `file`, where `x` is an integer, and `file` is
the path to a file full of integers, one per line, and prints the `x` largest
numbers in `file`, one per line.

## Run time/space complexity

This script is naive in a number of ways:

- It's Ruby, and therefore single threaded
- Sorts and reverses the array every time it adds a new number, which can
  probably be avoided

### CPU / run time

Run time is (probably?) proportional to:

```
x * lines_in_file * ascending_orderedness_of_file
```

where `ascending_orderedness_of_file` increases when more numbers in the file
are in ascending order. In a perfectly ascendingly ordered file, the results
array would have to be changed on every iteration, resulting in a lot more
computation.

When I tried sorting one of my test files of randomly-ordered integers and
presenting that to `topX`, the execution time increased by about 15%.

### Memory consumption

The biggest change I made to reduce memory consumption was to switch from using
`File.readlines`, which reads the whole file into memory, to
`File.open().each_line`, which streams the file as needed. This probably made
very little difference for the 10K-line files I tested on, but would in theory
make it possible to iterate over very large files, because we can
garbage-collect each number that is rejected from the top list (either without
being added, or after being ousted by a larger number).

### How I'd test further

I used a few samples from random.org, which generates lists of up to 10,000
numbers between 0 and 100,000. My best effort was about 50K numbers.

If I was getting this ready for production, I'd test with more, larger numbers,
and prod a few of the 2^n boundaries, especially around 2^16 and 2^32.

### Room for improvement

I have several ideas about improving performance:

- Implement in a compiled language, such as Go or C. Everyone knows Ruby is
  slow, but great for prototyping ;)
- See if I can avoid parsing integers, and instead use characteristics like the
  length of the string to determine whether a>b. See if this improves
  performance compared to parsing every string into an integer.
- Use a map/reduce algorithm to divide the work among multiple processors:
  1. Split the data into N chunks (and hope each chunk has a fairly random
     distribution of numbers?)
  1. Run `topX` on each chunk, with the same X, so that we're sure to always
     include the top X results even if they're all in the same chunk
  1. Concatenate all the "sub-results" together
  1. Run `topX` over that, to get the "top of the top"
