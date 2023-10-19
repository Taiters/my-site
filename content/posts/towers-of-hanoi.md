+++
title = 'Towers of Hanoi'
date = 2023-10-19T10:37:01Z
draft = true
+++
So there you are, in an interview, palms are sweaty, mum's spaghetti... or you're just brushing up on some algorithms, either way, you've come across a question asking you to solve the Towers of Hanoi problem. What now?

## The problem

Let's start with a quick refresher on the problem. You are given three towers. The first tower has a set of disks on it which increase in size from top to bottom. The challenge is to move these disks to the third tower, following these rules:

* You can only move one disk at a time
    
* A disk cannot be placed on another disk which is smaller than it
    

Take a moment to think about how you would approach this. What do you do with the first disc? Once you've moved this disc, where can you move the second disc? How have you moved closer to solving the puzzle?

You could just move the discs randomly, following the rules above, until you've managed to move every disk onto the third tower. This doesn't translate well into an algorithm though. How else can you think about this?

## Start simple

What's the simplest version of this problem? If we always have three towers and a variable number of disks, then a single disc is the simplest version. So let's start here.

Our function can take a stack (or list) for each tower. Each stack contains the discs currently on that tower. In the single disc case, our `origin` tower contains `[1]`, and the `destination` tower contains `[]`, as it's currently empty. This function can ignore the third tower for now.

```python
# Move a single disc from the origin to the destination
def towers_of_hanoi_1_disc(origin, destination):
    destination.append(origin.pop())
```

Mind blowing, isn't it? To move the disc to the destination, we can simply `pop` it off our `origin` tower, and `append` it to the `destination` tower.

How can we build on top of this to solve for 2 discs? First we can reintroduce our friend, the third tower. We can think of this tower as our `buffer` tower, which can act as our temporary storage. When we need to move 2 discs we can:

1. Move the small (top) disc from the `origin` tower onto the `buffer` tower (put it in storage)
    
2. Move the large (bottom) disc from the `origin` tower directly to the `destination` tower
    
3. Move the small disc from the `buffer` tower onto the `destination` tower
    
4. Take a moment to appreciate our ability to move discs between towers at will
    

Do you see anything familiar about the steps above? In step 2, we move a single disc from the `origin` tower to the `destination` tower, which we already covered in the single disc case. Our function for 2 discs looks looks like this:

```python
# Move 2 discs from the origin to the destination tower
def towers_of_hanoi_2_discs(origin, buffer, destination):
    buffer.append(origin.pop()) # Move the first disc into our buffer
    towers_of_hanoi_1_disc(origin, destination) # Move the second disc to the destination
    destination.append(buffer.pop()) # Move the first disc to the destination
```

## Scale up with recursion

We now know how to solve this for both a single disc as well as 2 discs. How about 3 or 4 discs? We could handcraft a solution for every case... but what about `N` discs? Instead, can we break these larger cases down into cases we've already solved? Yes... That's why this post exists.

We've actually already done this. Our solution for 2 discs (`N = 2`) reused our solution for a single disc (`N = 1`). This single disc case is our "base case" in recursion terms. Our 2 disc case builds on this by using the `buffer` tower to store the top disc, allowing us to simply solve the single disc case for the remaining disc. Let's have a look at a function which solves for 3 discs:

```python
def towers_of_hanoi_3_discs(origin, buffer, destination):
    towers_of_hanoi_2_discs(origin, destination, buffer) # Move 2 discs onto the buffer
    destination.append(origin.pop()) # Move the third disc onto the destination
    towers_of_hanoi_2_discs(buffer, origin, destination) # Move 2 discs onto the destination
```

What's happening here? We're reusing our `towers_of_hanoi_2_discs` function, but now we've jumbled up our arguments!? Let's walk through what's happening:

1. Our first call to `towers_of_hanoi_2_discs` says "I want to move 2 discs from `origin` into `buffer`, using `destination` for temporary storage"
    
2. We then move the third disk directly from `origin` to `destination` (Remember, `destination` is empty once `towers_of_hanoi_2_discs` returns, as the discs are moved into `buffer`)
    
3. We call `towers_of_hanoi_2_discs` again, saying "I want to move 2 discs from `buffer` into `destination`, using `origin` for temporary storage"
    

See below for an illustration of what's happening:

```plaintext
# Initial state when calling towers_of_hanoi_3_discs(A, B, C):
A: [3, 2, 1]
B: []
C: []

# After towers_of_hanoi_2_discs is called:
A: [3]
B: [2, 1]
C: []

# After moving the third disc into the destination tower (C):
A: []
B: [2, 1]
C: [3]

# After towers_of_hanoi_2_discs is called again:
A: []
B: []
C: [3, 2, 1]
```

So to recap:

* To move 2 discs, we move 1 disc to our `buffer`, then move the second disc to the `destination`, then move our first disc from the `buffer` to the `destination`
    
* To move 3 discs, we move 2 discs to our `buffer`, then move the third disc to the `destination`, then move the 2 discs from our `buffer` to the `destination`
    
* More generally, to move `N` discs, we move `N-1` discs to our `buffer`, then move 1 disc to our `destination`, then move `N-1` discs from our `buffer` to our `destination`
    

Using the final point above, we can combine our three separate functions into a single recursive function which solves for `N`. This function recursively calls itself for `N-1`, eventually reaching the base case, where `N = 1`:

```python
def towers_of_hanoi(n, origin, buffer, destination):
    if n == 1:
        destination.append(origin.pop())
    elif n == 2:
        buffer.append(origin.pop())
        towers_of_hanoi(n-1, origin, buffer, destination)
        destination.append(buffer.pop())
    else:
        towers_of_hanoi(n-1, origin, destination, buffer)
        destination.append(origin.pop())
        towers_of_hanoi(n-1, buffer, origin, destination)
```

Don't worry if you are having a hard time comprehending each disc move which happens when this function is called. Once you've figured out your base case, and how your function can solve smaller sub problems of itself, it is sometimes best to "trust" the recursion from there. If you *really* want to think about all the individual steps, you can also draw a recursion tree to map out each recursive call for a given input.

Also, if you *are* in an interview, don't forget your edge cases! (And maybe best not to read this right now)

## Next steps

* Rather than just moving values between arrays, write a function which outputs each move. For example "Disc **5** moved from tower **A** to tower **C**"
    
* Whats the runtime complexity (Big O) for this function?