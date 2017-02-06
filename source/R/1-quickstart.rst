===========
Quick Start
===========

There are mainly two functions in this package: ``test_that`` and ``expect_equal``.
The ``test_that`` function is a block of testing that may contains one or more expectations.
Expectations are usually ``expect_equal`` function calls.


Basic Example
-------------

Suppose we want to test whether a variable ``x`` equals to 1,
and whether a variable ``y`` equals to 2.
Here is the test code:

.. code-block:: r

    library(autotest)
    test_that('testing x', {
        expect_equal(x, 1)
    })
    test_that('testing y', {
        expect_equal(y, 1)
    })

- If the ``x`` variable is not defined, it will given an error:

.. error::
    | AutoTestCaseError:
    | Testing variable/expression:  x
    | object 'x' not found

- If ``x`` does not equal to 1, it will given an error:

.. error::
    | AutoTestCaseError:
    | Testing variable/expression:  x
    | Your answer is 2, which is not equal to the correct answer 1


Automatic Tracing
-----------------

The words ``Testing variable/expression:  x`` in previous example is automatically generated.
When error occurs, it tells users what we are testing and where they are wrong.
This feature is called **Automatic Tracing**.

The last character ``x`` is the first argument of ``expect_equal(x, 1)``.
If you run ``expect_euqla(f(i), 1)``, it will be ``Testing variable/expression:  f(i)``.
The last character ``f(i)`` is always exactly the same with the first argument.
You **must NOT** write ``expect_equal(1, f(i))`` instead of ``expect_equal(f(i), 1)``!

Here is another example of testing a function. Assume we need a function ``f``, which
returns ``x+1`` given a number ``x``.

.. code-block:: r

    library(autotest)
    f <- function(x) ifelse(x < 5, x + 1, x)  # give a wrong solution to see the error message
    test_that('testing x', {
        for (i in 1:10){
            expect_equal(f(i), i + 1)
        }
    })

In the for loop, error occurs when ``i = 5``.

.. error::
    | AutoTestCaseError:
    | Testing variable/expression:  f(i)
    | Your answer is 5, which is not equal to the correct answer 6


You can set ``trace=FALSE`` in the ``expect_equal`` function to close the automatic tracing.

.. code-block:: python
    :emphasize-lines: 5

    library(autotest)
    f <- function(x) ifelse(x < 5, x + 1, x)
    test_that('testing x', {
        for (i in 1:10){
            expect_equal(f(i), i + 1, trace=FALSE)  # trace=FALSE
        }
    })

.. error::
    | AutoTestCaseError:
    | Your answer is 5, which is not equal to the correct answer 6

Given this error, no one could figure out what is going on if he/she do not the test code.
Do not worry, keep reading the next following section.

Self-defined Error Message
--------------------------

In the previous example, error occurs when ``i = 5``. However, the tracing message
``Testing variable/expression:  f(i)`` does not tell what ``i`` is. Users may be
confusing that what input triggers the error.

Here we introduce a function ``registerPreMsg``, which is used to insert messages before
the default error.

- It only shows when there is an error.

- It should be defined before the ``expect_equal`` function.

- The arguments is exactly the same with the built-in function `sprintf`.

.. code-block:: python
    :emphasize-lines: 5

    library(autotest)
    f <- function(x) ifelse(x < 5, x + 1, x)
    test_that('testing x', {
        for (i in 1:10){
            registerPreMsg('In testing f(%d):', i)
            expect_equal(f(i), i + 1)
        }
    })

Here is the error message:

.. error::

    | AutoTestCaseError:
    | Testing variable/expression:  f(i)
    | In testing f(5):
    | Your answer is 5, which is not equal to the correct answer 6


After defining our own error message, the automatic tracing message is useless.
Set ``trace=FALSE`` to remove it.

.. code-block:: python
    :emphasize-lines: 5,6

    library(autotest)
    f <- function(x) ifelse(x < 5, x + 1, x)
    test_that('testing x', {
        for (i in 1:10){
            registerPreMsg('In testing f(%d):', i)
            expect_equal(f(i), i + 1, trace=FALSE)
        }
    })

Now it is perfect!

.. error::

    | AutoTestCaseError:
    | In testing f(5):
    | Your answer is 5, which is not equal to the correct answer 6



Read More
---------

- :doc:`2-expect_functions`

The ``expect_equal`` function is used to test whether two objects/expressions are equal.
It supports a lot of data types: `numeric`, `character`, `matrix`, and even `data.frame`.
There are also more functions like ``expect_true`` and ``expect_false`` testing whether
an expression returns true or false.

- :doc:`3-error_message`

To customize error messages, more APIs are designed.
The ``registerPreMsg`` function is just one of them.
We also have a function ``registerPostMsg`` insert messages after the default error.