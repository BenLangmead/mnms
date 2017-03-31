# Strings and pattern matching
[Ben Langmead](www.langmead-lab.org), March 2017

Strings are a way of abstracting sequential data, data consisting of many symbols in a row.  A word, a tweet, a web page, or a book could all be considered strings.  A chess game could be considered a string.  OK, maybe that's a little harder to see, but there is a way of writing a chess move as a string ([chess notation](https://en.wikipedia.org/wiki/Chess_notation)), and a whole chess game is a sequence of moves, so you could convert the whole game into one long string.  Examples of strings are everywhere: sequences of stock trades, phonemes in an utterance, bytes in a file, etc.

Here I will draw motivation from two problem areas I worked on early in my early career as a computer scientist.  The first is [network intrusion detection](https://en.wikipedia.org/wiki/Intrusion_detection_system#Signature-based): computer processors living in firewalls and other network infrastructure can scan passing traffic for signs of malicious behavior.  The stream of network traffic being scanned can be thought of as a string.

The second area is computational genomics, my area of research.  A [DNA molecule](https://en.wikipedia.org/wiki/DNA) consists of a sequence of nucleic acids.  And even though DNA has two complementary strands of nucleic acids, and a complicated three dimensional shape, you can ignore all that and choose to think of the DNA simply as a sequence of letters corresponding to the four nucleic acids: A (for adenine), C (for cytosine), G (for guanine) and T (for thymine).

This is a very brief introduction to strings and pattern matching: what they are, and why they are useful in these two problem domains. As a bonus, we'll learn the basics of working with strings in Python.  We'll even write down a pattern matching algorithm: an algorithm that allows us to find copies of a short string in a longer string.  In lecture, I'll focus on a few more powerful techniques, involving approximate matching and text indexing, and how they apply these contexts.

## String definitions and Python syntax

A string _S_ is a finite _sequence_ of _characters_.
Those characters are drawn from an _alphabet_ which we refer to with the greek symbol Σ (capital sigma).  In the world of DNA our alphabet is the set of possible nucleic acids: Adenine (A), Cytosine (C), Guanine (G) and Thymine (T).  In other words: Σ = {A, C, G, T}.

_|S|_ represents the _length_ of _S_, that is, the number of characters in _S_.
In Python, we get a string's length with the special `len` function:

    > s = "hello"
    > len(s)
    5

To refer to a particular character in a string, we use an _offset_.
We use the convention that the first (leftmost) offset is zero.
This is also the convention in Python, C and C++.
In this Python example we define a string `s` and then use square-bracket syntax to get the character at offset 0 within `s`, which is `A`.

    > s = "ACGT"
    > s[0]
    'A'

And here we get the character at offset 2 (third from the left), which is `G`:

    > s[2]
    'G'

The _concatenation_ of two strings _S_ and _T_ consists of the characters of _S_ followed by the characters of _T_.
Concatenation simply "glues" strings together.
In Python, we use `+` (plus) to concatenate:

    > s1 = 'hello'
    > s2 = 'there'
    s1 + ' ' + s2
    'hello there'

A _substring_ is a shorter string occurring within a longer string.
In this example, we define a string `seq` and use square-bracket syntax to extract a substring starting offset 2, up to (but not including) offset 6. In Python, this is also called _slicing_.

    > s = 'AACC'
    > t = 'GGTT'
    > s + t
    'AACCGGTT'

A _prefix_ is a substring that begins at the beginning of another string.
For example, when we get the substring of `s` starting at offset 0, up to offset 6:

    > s = 'AACCGGTT'
    > s[0:6]
    'AACCGG'

we get a prefix of `s`. A _suffix_ is a substring that ends at the end. For example, when we extract the substring of `s` that begins at offset 4 and ends at offset 8:

    > s = 'AACCGGTT'
    > s[4:8]
    'GGTT'

we get a suffix of `s`.

## Exact string matching

Let's formulate a computational problem.  Say we're scanning an email passing by on the network, looking for signs of an [advance-fee scam](https://en.wikipedia.org/wiki/Advance-fee_scam). If we see the string "send money", we suspect the email is carrying an advance-fee scam and we want to warn the receiver.

This motivates the _exact matching_ problem.  Exact matching is concerned with finding all the places where a short string occurs in a longer string.  The shorter string is called the _pattern_ and we refer to it as _P_ or `p`.  The longer string is called the _text_ and we refer to it as _T_ or `T`.

More specifically, we want to report a list of offsets, where each offset where an occurrence of P _begins_ at that offset.  In this example:

    > t = 'to do to die today'
    > p = 'to'

A solution to the exact matching problem would be the following list: `[0, 6, 13]`.  Those are the starting offsets for all occurrences of `p` in `t`.

Python gives us some tools for solving this problem.  In this example, we call the text's `.find()` method to find the first (leftmost) occurrence of the pattern in the text.

    > t = 'to do to die today'
    > p = 'to'
    > t.find(p)
    0

If we want Python to report _all_ offsets where `p` occurs, we have to do something a bit more complicated.  A good exercise is to implement a Python function that reports all offsets where `p` occurs within `t` using `.find()`.  Hint: look in the Python documentation for what arguments can be passed to the `.find()` function, then write a loop that calls it repeatedly.

But rather than rely on functions Python provides, let's write it out ourselves.  Here's a suggested algorithm.  Try every possible way of lining `p` up with a substring of `t`. For each each of these "alignments", check whether all the characters of `p` match the corresponding characters of `p`. If they do, then `p` matches `t` at that offset. If one or more of the characters differ between `p` and `t`, then it's not a match.

```
p: word
t: there would have been a time for such a word
   word word word word word word word word word*
    word word word word word word word word
     word word word word word word word word
      word word word word word word word word
       word word word word word word word word
```

The asterisk * marks the match, which is at offset 40.

Here's an implementation of this idea in Python. The function is called `naive`, because this  is sometimes called the "naive exact matching algorithm." It takes two arguments, the pattern `p` and the text `t`.  We initialize this list `occurrences` to be empty.  We will eventually fill `occurrences` with all the offsets where P matches T.

```
def naive(p, t):
    occurrences = []
    for i in range(len(t) - len(p) + 1):  # loop over alignments
        match = True
        for j in range(len(p)):           # loop over characters
            if t[i+j] != p[j]:            # compare characters
                match = False             # mismatch; reject alignment
                break
        if match:
          occurrences.append(i)           # all chars matched; record
    return occurrences
```

I added code comments -- everything from `#` to the end of the line -- to help you understand what's going on.  It's fine if not everything makes perfect sense. We'll discuss it again in class.

The `naive` function returns a list of all the offsets where a pattern occurs in the text.

