---
title: "Transcript: Vector embeddings — Blackboard AI"
description: Today we are going to demystify one of the most fundamental concepts in modern artificial intelligence, vector embeddings.
type: source
source_url: User-provided transcript in chat
media_type: text/plain
date_fetched: 2026-10-30
preservation: text-extracted
tags:
  - source
  - immutable
  - layer-ingest
  - text
---
Hello and welcome to Blackboard AI.
Today we are going to demystify one of
the most fundamental concepts in modern
artificial intelligence, vector
embeddings. How does a computer which
only understands numbers learn the
meaning of a word like king? How does it
know that king is more related to queen
than it is to apple?
The answer lies in a clever trick of
turning words into special lists of
numbers called vectors. But how exactly
is a word assigned its unique vector?
Let us draw it out. Before we see the
modern solution, let us understand the
problem. The simplest way to represent
words with numbers is a technique called
one hot encoding.
Imagine our entire dictionary has just
four words. We give each word a unique
index. To represent the word cat, we
create a vector with a length equal to
our vocabulary size. In this case, four.
Similarly, the vector for dog would have
a one in the fourth position. But this
has two massive problems. First, real
vocabularies have 50,000 words or more,
which is incredibly inefficient.
Second, and more importantly, these
vectors have no sense of meaning as all
words are equally different.
This brings us to the core of how modern
models work using something called an
embedding matrix. First, we define our
vocabulary size and a smaller embedding
dimension like 300. Our lookup table
will therefore have 50,000 rows, one for
each word, and 300 columns. At the very
beginning of training a model, we fill
this giant table with small random
numbers. So how is a token assigned its
vector? It is surprisingly simple. It
is just a lookup. The model finds the
words ID, goes to the matrix and simply
grabs the entire corresponding row.
Okay, so we are just looking up rows of
random numbers. How does that help us
understand meaning? The key is that
these numbers are not random forever.
They are learned through training. Let
us say based on the random numbers, it
predicts the next word is shu, which is
wrong. The model calculates an error,
then goes back to make tiny adjustments
to the numbers in the vector for queen.
Now repeat this process millions of
times with vast amounts of text from the
internet.
Words that appear in similar contexts
like king and queen will have their
vectors nudged in similar directions.
Eventually, the vectors for related
words end up close to each other,
filling the numbers with meaning. So let
us quickly recap how an embedding model
assigns a vector to a token. First,
every word is given a unique ID and a
large embedding matrix is created with
random numbers.
The assignment is a simple lookup. The
model takes the words ID and fetches the
corresponding row from the matrix.
Finally, through training, the model
constantly adjusts the numbers, causing
vectors for similar words to become
mathematically close. And that is it. It
is a lookup table that starts off random
and gets smarter through experience. If
this explanation helped you, please
consider liking this video and
subscribing to Blackboard AI for more
simple breakdowns of complex AI topics.
Thanks for watching.