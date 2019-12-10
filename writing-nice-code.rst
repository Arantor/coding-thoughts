===========================
 Writing High Quality Code
===========================

Regardless of what platform, environment or even which language you're using,
there are a few things that go a really long way into making good quality code.

The first rule of writing high quality code is to remember your audience.
The audience of your code is whoever comes after you wrote it, which may or may
not be you. Its future developers/maintainers will honestly appreciate well
written code that is clear and logical to follow. (And after a year away from
the code, it might as well be written by someone who wasn't you, so you too will
appreciate your past self writing good code.)

Commenting
----------

Comments in code are probably the number one skill to have in your toolkit.
Good comments habits and styles go a long way in writing good code.

For example, a function should generally have its purpose, inputs and outputs
described in a docblock before the function, and if you find you can't
adequately summarise the function's fundamental job in one line (perhaps with
a large number of lines of specificity under it), the function is probably
doing too many things and suggests it should be broken down into several
functions each doing one job.

It's a well understood mentality that no comments is the wrong approach, and
that commenting every single line is usually the wrong approach, and so some
kind of happy medium.

One strong recommendation is to look at the code and ask the two following
questions:

1. Is the code *superficially* transparent as to what it is doing? Could
   someone who is vaguely familiar with the language and environment make
   a reasonably accurate guess as to the function of the code?

2. Is it *superficially* clear why the code is necessary?

These two questions are a reasonable guide, if subjectively so, as to when a
comment is necessary.

For example::

    function swap_integers(&$x, &$y) {
        $x ^= $y ^= $x ^= $y;
    }

Even for someone familiar with PHP syntax, this function's methodology is likely
to be utterly opaque and if you ever encounter anything like this, something
has probably gone tragically wrong.

Let's fix it up the function to have some sensible comments::

    /**
     * Swaps two integers quickly.
     *
     * Uses the XOR algorithm to switch the values of two integer
     * variables without having to declare a third integer variable.
     * For extremely tight loops, this will be more memory efficient
     * than any implementation with three variables, assuming that
     * $x and $y are explicitly integers.
     *
     * @param int $x The first variable to swap
     * @param int $y The second variable, to swap with the first
     */
    function swap_integers(int &$x, int &$y) {
        $x ^= $y ^= $x ^= $y;
    }

This describes the function's purpose in one line, it now has some context as to
what it's doing and how, and it explains why it does it this way. Hopefully
whatever code calls this should also explain why it needs to do it efficiently,
e.g. maybe this is running in a loop with processing millions of items and the
creation and teardown of those variables becomes statistically significant.

It also adds type-hinting, which is not just good practice for reducing errors
by disallowing things of the correct type, but it also gives clear documentation
at the source level what values can be expected to be inserted.

Perhaps a better example of some comments that explains what a piece of code is
doing, and why it is necessary, in one go::

    // Before we add anything to the plan, we need to sort out the progress meter.
    // The issue is, we're adding tasks to the progress queue but the size of the queue
    // for the progress meter was set up before we started the queue, so we have to fit it.
    // Unfortunately, it's a protected array inside the progress instance, so better
    // pop the lid and get ourselves access to it with Reflection.
    $progress = $this->get_progress();
    $progressclass = new ReflectionClass($progress);
    $progressproperty = $progressclass->getProperty('maxes');
    $progressproperty->setAccessible(true);
    $maxes = $progressproperty->getValue($progress);
    $maxes[count($maxes) - 1] += count($subblocks);
    $progressproperty->setValue($progress, $maxes);

The comment explains what the code does without simply restating the code
itself, it explains why it is necessary and it becomes possible to think about
the code as a block of logic.

That is really the goal of the above two questions:

1. Is the code superficially transparent as to what it is doing?
   If not, it needs explaining **what** it is doing.

2. Is it superficially clear why the code is necessary?
   If not, it needs explaining **why** it is doing it.

Often these comments work best if delivered as a line of comments per logical
task within the code - very often several lines make one task, and the logical
unit of work is the scope that will need to be commented.

Comments generally should be reasonably professional; other people will see your
comments in future, so putting things in that are a little informal is usually
fine, but leaving multi-line apologies or venting frustration is usually not.

And usually, if you have some doubt as to whether something needs a comment,
that probably means it should have a comment, and it's always better to err on
the side of slightly more rather than slightly less.


Code complexity
---------------

Code has a natural habit of sprawling, in time and space complexity. Several
methods exist to try to describe the complexity of a piece of code and these
suggest some guidelines as to how to write code that doesn't sprawl and is thus
manageable.

1. Individual functions should be small, ideally under 50 lines in length, but
   with maybe 100 as a reasonably hard upper limit.

2. Individual functions should have one job, and one job only. It's not even
   recommended to have a function that does two variants of the same thing,
   e.g. some processing that can either be returned or displayed. That should be
   two functions - one that does the processing and returns it, and one that
   fetches and outputs. [#SOLID_S]_

3. When considering a function's complexity, there are two principle metrics
   that come up. Both should be kept as low as possible to keep a function
   manageable and if a function has too high a score, it probably should be
   split up into separate functions.

   a) nPath complexity - simply put, the maximum total number of ways a given
      function can possibly execute - for every if true/false, that creates
      2 branches of code, both of which should be considered in terms of
      testing::

        if (x) {
            if (y) {
                if (z) {
                    echo 'x and y and z';
                } else {
                    echo 'x and y';
                }
            } else {
                if (z) {
                    echo 'x and z';
                } else {
                    echo 'x';
                }
            }
        } else {
            if (y) {
                if (z) {
                    echo 'y and z';
                } else {
                    echo 'y';
                }
            } else {
                if (z) {
                    echo 'z';
                } else {
                    echo 'none of them';
                }
            }
        }

      This simplistic structure shows a surprising number of outcomes that
      potentially need unit tests or some kind of review to ensure that they all
      function correctly.

      We had three variables, and those three resulted in us having 2x2x2
      possible paths - nPath score 8.

      A general rule of guidance is that a function whose nPath score is much
      beyond 200 almost certainly needs refactoring to not have decisions
      that are directly contingent on each other (as in practice, two parts of
      a high-scoring function probably aren't functionally dependent on the
      outcomes of each other directly).

   b) cyclomatic complexity - similar to nPath in that it's a measure of the
      number of decisions through a piece of code, but instead of measuring the
      number of possible outcomes, it's a measure of the number of decisions
      being made through the code. Ifs, while loops, for loops, these are all
      counted as part of the cyclomatic complexity score.

      For example::

        void foo(void)
        {
            if (a && b)
                x=1;
            else
                x=2;
        }

      The if has two choices (if a, if b) and a 'if neither' choice in the code
      so this has a cyclomatic complexity of 3.

      A given function should usually have a cyclomatic complexity of no more
      than 10.

   The reason these metrics exist is to provide a benchmark for understanding
   how complex a piece of code is, and whether it needs splitting up. These
   should not be considered the final word in anything; sometimes the most
   logical and descriptive way of solving a problem is to write a 150 line
   function that has a cyclomatic count of 25 and an nPath complexity of 5000.

   Where it is more clear for someone else to read, follow and understand, that
   should always be taken first over any arbitrary metric, but experience
   suggests that writing more, smaller functions with explicit jobs usually
   trends towards code that is easier to understand, easier to maintain and
   easier to expand later.



.. [#SOLID_S] There is a paradigm of engineering discipline called SOLID. This
   is the S, Single Responsibility, namely that a function does one thing only.