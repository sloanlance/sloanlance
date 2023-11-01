---
title: "My AI Teacher"
date: 2023-11-01
---

Not too surprisingly, AI tools designed to help people learn have taught me
something.

I developed and maintain a Django app for staff (e.g., secretaries and
librarians) to maintain a searchable index for volumes in a digital library.  I
had been asked to implement a feature to produce printable indices for volumes,
showing all the topics, their items, and pages.

> Without getting too deep, the data model of the app has topics, which contain
> items, which contain item-pages.  Topics and their items may appear on multiple
> pages and in multiple volumes.  Therefore, only item-pages are related to
> volumes.

I'm a little rusty with this type of Django query, but I worked at it and came
up with the following first draft…

## Before

{% gist 522952e01975ebd9d744c6ca8b60646c indexformatter1.py %}

Lines 4–5 are the main query, loading all the data related to a volume ID
number.  Each line of the printable index begins with a topic followed by each
of its items and pages.  That solution worked, but I knew it wasn't particularly
good.  It ran slowly, for one thing.  I suspected the function calls to filter
the query results into topics and into page and item name were the main reason
for the slowness.

Walking away from that code until the next day gave me time to think about what
I could do to improve it.  I was sure I could eliminate the extraction of
topics.  I could iterate over the full query results and when the topic changed,
start a new line.  That's much the way the code uses `previousLetter` to show a
heading when the initial of the topic changes.

## Inspiration

The next day, I decided to make a copy of the `format()` method named
`format2()`.  That would let me test them side by side and time them with
Python's `timeit` module.  In my IDE, IntelliJ IDEA, I copied the code and IDEA
immediately warned me that the code contains a lot of duplication.  That was
expected, but I idly wondered whether changing the docstring in `format2()`
would change that warning.  So, I began to add a line to it, "Second attempt,
using…".  I was about to write "fewer function calls" or "simpler loop".

That's when inspiration in the form of AI happened.  GitHub Copilot, which had
been active all along, making moderately interesting suggestions, suggested my
docstring should continue "using itertools.groupby()".  That's not what I had
been thinking at all.  However, I wondered whether GitHub Copilot may have
stumbled onto something.  I know that using a library module to reduce looping
and simplify logic would improve performance.  So, I looked into the
documentation for `itertools.groupby()`, which I had rarely ever used.

After a few minutes of reading and considering code examples, it was clear that
`itertools.groupby()` would definitely be helpful here.  I ended up with code
that looked almost like the following…

## After

{% gist 522952e01975ebd9d744c6ca8b60646c indexformatter2.py %}

When I ran each version of the code and timed it using `timeit.timeit(i.format,
number=10)`, I saw that the calls to the original method took 30–40 seconds
while calls to the new method took 5 seconds or less!

I had inspiration from other sources as well.  I found my way to an episode of
Podcast.__init__, in which Python FLUFL Barry Warsaw was being interviewed.  He
mentioned the "walrus operator" ("`:=`"), which I had read about before, but
hadn't used in a practical application.  I used that on line 18 of the "after"
code to help me combine the topic initial logic into the main loop.  Later, I
used the operator again on line 16 to help me move the topic initial selection
out of the loop and do it with a short line of code.

## More optimization?

I thought of using `itertools.groupby()` again for extracting the topic initial.
I think I would need to make another loop for that, though.  I started to
implement that, but it was getting late, I was getting tired, and I just
couldn't grok it.  I decided it's probably a folly anyway, so I'll leave that
for a future experiment.  Along those lines, I think I could probably eliminate
the `for` loop and do everything in a list comprehension, but I suspect it would
hurt code readability.

## I, for one, embrace our AI overlords

OK, maybe they're not "overlords".  I've seen so much weird stuff produced by
AI.  I know AI can't be used blindly, but when employed by an experienced user,
it can be a helpful, inspiring tool.  I will continue using it and I expect it
will improve with time.
