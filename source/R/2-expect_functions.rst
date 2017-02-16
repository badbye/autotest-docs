=================
Testing Functions
=================

This part will introduce all the expect functions:

- ``expect_equal``: test whether two objects are equal
- ``expect_false``, ``expect_true``, ``expect_na``: test whether a variable/expression returns *TRUE*/*FALSE*/*NA*
- ``expect_is``, ``expect_length``: test *class* and *length*
- ``expect_match``, ``expect_output``, ``expect_error``: test *returned strings*/*printed strings*/*errors*

.. note::

    All those ``expect_XXX`` introduced here have an argument ``trace``, which is used to
    control the **automatic tracing** feature.

.. danger::

    Do **not** use the other ``expect_XXX`` functions that are not introduced here!


expect_equal
------------

It support a lot of data types (``integer``, ``character``, ``logical``,
``double``, ``factor``, ``list``, ``matrix`` and ``data.frame``).

.. note::
    When checking the missing values, it is better to use the ``expect_na`` function instead of ``expect_equal``.

In R, each object has the *class* and *length* attributes.
To test whether two variables/expressions are equal, it always test the *class* attribute first.

Basically, Here are the steps of testing:

1. Test class.
2. Test length/dimension.
3. Test value.

So ``expect_equal(rep(1, 10), 1)`` will raise an error because of the length diff.


numeric
^^^^^^^

The ``expect_equal`` function test class first. It will give an error if the target(the first argumnt)
and answer(the second argument) do not have the same class.

However, this is the only exception that it ignores the difference between `numeric` and `integer`.
Both integer and float belong to numeric. ``is.numeric(1)`` and ``is.numeric(1L)`` returns TRUE.

.. code-block:: r

    class(1)
    ## [1] "numeric"
    class(1L)
    ## [1] "integer"
    expect_equal(1, 1L)
    expect_equal(1L, 1)
    expect_equal(1 / 0, Inf)
    expect_equal(-1 / 0, -Inf)


There is an argument ``tolerance`` which is used to control the maximum threshold between target and answer.
In default, it is :math:`10^{-5}`.

.. code-block:: r

    expect_equal(1, 0.8, tolerance=0.2)
    expect_equal(1, 0.8, tolerance=0.1999)
    ## AutoTestCaseError:
    ## Testing variable/expression:  1
    ## Your answer is 1, which is not equal to the correct answer 0.8


character & logical
^^^^^^^^^^^^^^^^^^^
We only test the *values* of character and logical variables,
Which means we does not check the name or any other attributes.

.. code-block:: r

    x = letters
    names(x) = 1:length(x)
    test_that('', {
        expect_equal(x, letters)  # pass, does not check names
        ## if you want to check the names
        # registerPreMsg('Testing the name attribute of x')
        # expect_equal(names(x), as.character(1:26))
    })


factor
^^^^^^

Test steps:

1. Test length
2. Test the levels
3. Test whether it is ordered or unordered
4. Convert to character and then test the character values

If the correct answer is an ordered factor, then the checked variable must be ordered.

.. code-block:: r

    x = factor(1:10, order=T)
    expect_equal(x, factor(1:10))  # pass

    x = factor(1:10)
    expect_equal(x, factor(1:10, order=T))
    ## AutoTestCaseError:
    ## Testing variable/expression:  x
    ## The answer is an ordered factor, your factor is not unordered.
    ## Use the `as.ordered` function to convert you answer to an ordered factor.


matrix
^^^^^^

Test steps:

1. Test dimension
2. Test data types
3. Test values


.. code-block:: r

    x = matrix(1:9, 3)
    y = matrix(as.character(1:9), 3)
    expect_equal(x, y)
    ## AutoTestCaseError:
    ## Testing variable/expression:  x
    ## The type of the data in your matrix is `integer`, which should be `character`

You can also set the `tolerance` argument if the matrix is a numeric matrix.

.. code-block:: r

    x = matrix(1:9, 3)
    y = x + 1
    expect_equal(x, y, tolerance=1) # no error because the tolerance is large enough


list
^^^^

Test steps:

1. Test length
2. Test names of the elements
3. Test values of each element

You can set ``test.name=FALSE`` to skip the second step.

Since the ``matrix``, ``list`` and ``data.frame`` objects may contains numeric values, you can always
set the ``tolerance`` argument when testing them.

Here is one more example, you could also try to testing it on a data frame.

.. code-block:: r

    x = list(a = 1)
    y = list(a = 2)
    expect_equal(x, y, tolerance = 1)  # pass
    expect_equal(x, y, tolerance = 0.999)  # pass
    ## AutoTestCaseError:
    ## Testing variable/expression:  x
    ## The type of the 1th element in your list is `numeric`.
    ## In testing the 1th element:
    ## Your answer is 1, which is not equal to the correct answer 2
    ## The maximum tolerance is 0.999


data.frame
^^^^^^^^^^

Test steps:

1. Test dimension
2. Test column names
3. Test the values in each column

Set ``test.colname=FALSE`` to skip testing the column names, in default it is true.

.. note::
    In testing a data frame, it does not care about the order of the columns.
    ``expect_equal(iris, iris[, c(1, 3, 2, 5, 4)])`` does not raise an error.





expect_true & expect_false & expect_na
--------------------------------------

``expect_true(x)`` is *almost equivalent* to ``expect_equal(x, TRUE)``,
while ``expect_false(x)`` is *almost equivalent* to ``expect_equal(x, FALSE)``.

The difference is that ``expect_true`` supports a vector of TRUE, while ``expect_equal``
will test the length first. So it is equivalent to ``expect_equal(x, rep(TRUE, length(x)))``.


.. code-block:: r

    library(autotest)
    x = 1:10 > 0
    expect_true(x)
    expect_equal(x, rep(TRUE, length(x))) # same with previous one
    expect_equal(x, TRUE) # error
    ## AutoTestCaseError:
    ## Testing variable/expression:  x
    ## The length of your answer is 10, which should be 1 in the correct answer

    x = 1:10 < 0
    expect_false(x)
    expect_equal(x, rep(FALSE, length(x))) # same with previous one
    expect_equal(x, FALSE) # error
    ## AutoTestCaseError:
    ## Testing variable/expression:  x
    ## The length of your answer is 10, which should be 1 in the correct answer


In R, ``NA`` is a special *logical* value. You could run ``class(NA)`` to confirm it.
The point is that ``NA == NA`` returns ``NA``, not ``TRUE``,
that is why we design the ``expect_na`` function.
Similarly, it also supports a vector of NA.

.. code-block:: r

    library(autotest)
    expect_na(NA)
    expect_na(rep(10, NA))

    x = 1:10; x[5] = NA
    expect_na(x, method = 'any') # 'any' means x contains NA


Lastly, if you want to check ``x`` to be a single missing value, here is the test code:

.. code-block:: r

    library(autotest)
    test_that('', {
        expect_equal(length(x), 1)  # or `expect_length(x, 1)`
        expect_na(x)
    })


expect_is & expect_length
-------------------------

``expect_is(x, 'integer')`` is equivalent to ``expect_equal(class(x), 'integer')``.
``expect_length(x, 10)`` is equivalent to ``expect_equal(length(x), 10)``.


We know ``expect_equal`` ignore the difference class between ``integer`` and ``numeric``,
so if you want to test whether ``x`` is an integer 1, it is necessary to make sure its
class is integer.

.. code-block:: r

    library(autotest)
    x = 1L
    test_that('', {
        expect_is(x, 'integer')
        expect_equal(x, 1)
    })

If you want to test whether ``y`` is a vector of TRUE with length 10:

.. code-block:: r

    test_that('', {
        expect_length(y, 10)
        expect_true(y)
    })

    # alternatively
    test_that('', {
        expect_equal(y, rep(TRUE, 10))
    })


expect_match & expect_output & expect_error
-------------------------------------------

The testing functions we introduced previously usually ask the submission should be exactly
the same with the correct solutions.

Here we introduce some more testing functions that you may rarely need them, however, they are
still very useful.


expect_match
^^^^^^^^^^^^

It expect the submission should be a character and match some pattern (could be a regular expression).
In the backend, it is using the ``grepl`` function to check whether it matches. So you can also pass
the argument from ``grepl`` to this function.


.. code-block:: r

    expect_match('abc', 'a')  # pass
    expect_match('abc', '[a-z]{3}')  # using regular expression

    expect_match('abc', 'A', ignore.case=T)  # using the `ignore.case` argument from `grepl`

    # fixed=T means exactly match, not using regular expression
    expect_match('a(b', '(b', fixed=T)

    # support vector
    expect_match(letters, '[a-z]')

expect_output
^^^^^^^^^^^^^

``expect_output(exp, char)`` expects the first `exp` argument prints out something that matches
the second argument `char`. The second argument could be a regular expression. In default it is
using regular expression, and the backend is also the ``grepl`` expression.


.. code-block:: r

    expect_output(print('abc'), 'a')  # pass
    expect_output(cat('abc'), '[a-z]{3}')  # using regular expression

    expect_output(cat('abc'), 'A', ignore.case=T)  # using the `ignore.case` argument from `grepl`

    # fixed=T means exactly match, not using regular expression
    expect_output(print('a(b'), '(b', fixed=T)

    # support vector
    expect_output(cat(letters), '[a-z]')


expect_error
^^^^^^^^^^^^

``expect_error(exp, char)`` expects the first `exp` argument raise an error, the second argument
is an optional argument used to matches the error message.

.. code-block:: r

    # check if it raise an error
    expect_error(1 - '1')

    # check if it raise an error with message ...
    expect_error(1 - '1', 'non-numeric argument to binary operator')

    # error message could be a regular expression
    expect_error(1 - '1', 'non-numeric.*')
