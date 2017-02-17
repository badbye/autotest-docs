==================
Error Message APIs
==================


There are four APIs for inserting sef-defined messages in this packages:

- ``registerPreMsg``
- ``registerPostMsg``
- ``addPreMsg``
- ``addPostMsg``

There are three APIs for controlling the *automatic tracing* feature:

- The ``trace`` argument and the ``suppressErr`` argument in ``expect_XXX`` functions
- ``turnOnTraceback`` and ``turnOffTraceback``


Pre Message
-----------

Pre message means insert **before** the default error.
There is a vector storing the pre messages, let us call this vector ``pre_vector``.
The ``registerPreMsg`` function always overwrite ``pre_vector``, while the
``addPreMsg`` function append messages in ``pre_vector``.

.. code-block:: r
    :emphasize-lines: 5

    library(autotest)
    f <- function(x) ifelse(x < 5, x + 1, x)
    test_that('testing f', {
        for (i in 1:10){
            registerPreMsg('In testing f(%d):', i)
            expect_equal(f(i), i + 1, trace=FALSE)
        }
    })

.. error::
    | AutoTestCaseError:
    | In testing f(5):
    | Your answer is 5, which is not equal to the correct answer 6

As we know error occurs when ``i = 5``, which means the code have called the
``registerPreMsg`` function 5 times. According to the error message, you can
see that only the last registered one is returned.


.. code-block:: r
    :emphasize-lines: 5

    library(autotest)
    f <- function(x) ifelse(x < 5, x + 1, x)
    test_that('testing f', {
        for (i in 1:10){
            addPreMsg('In testing f(%d):', i)
            expect_equal(f(i), i + 1, trace=FALSE)
        }
    })

Here is the error message if you replace ``registerPreMsg`` with ``addPreMsg``.

.. error::
    | AutoTestCaseError:
    | In testing f(1):
    | In testing f(2):
    | In testing f(3):
    | In testing f(4):
    | In testing f(5):
    | Your answer is 5, which is not equal to the correct answer 6

It will keep appending messages in the ``pre_vector``.
You should not use ``addPreMsg`` in this case.


Post Message
------------

Post message means insert **after** the default error.
There is a vector storing the post messages, let us call this vector ``post_vector``.
The ``registerPostMsg`` function always overwrite ``post_vector``, while the
``addPostMsg`` function append messages in ``post_vector``.

Suppose we want to records the log of testing in the for loop. Let us add post message
after one test is passed.

.. code-block:: r
    :emphasize-lines: 7

    library(autotest)
    f <- function(x) ifelse(x < 5, x + 1, x)
    test_that('testing f', {
        for (i in 1:10){
            registerPreMsg('In testing f(%d): ', i)
            expect_equal(f(i), i + 1, trace=FALSE)
            registerPostMsg('Testing f(%d): pass', i)
        }
    })

By using ``registerPostMsg``, only `Testing f(4): pass` is shown. Because in the 4th loop
this post message overwrite the previous messages, and in the 5th loop error occurs the
register line does not run any more.

.. error::
    | AutoTestCaseError:
    | In testing f(5):
    | Your answer is 5, which is not equal to the correct answer 6
    |
    | Testing f(4): pass


.. code-block:: r
    :emphasize-lines: 7

    library(autotest)
    f <- function(x) ifelse(x < 5, x + 1, x)
    test_that('testing f', {
        for (i in 1:10){
            registerPreMsg('In testing f(%d): ', i)
            expect_equal(f(i), i + 1, trace=FALSE)
            addPostMsg('Testing f(%d): pass', i)
        }
    })

This time we switch to ``addPostMsg``, the error message below shows it was appending messages
after the default error.

.. error::
    | AutoTestCaseError:
    | In testing f(5):
    | Your answer is 5, which is not equal to the correct answer 6
    |
    | Testing f(1): pass
    | Testing f(2): pass
    | Testing f(3): pass
    | Testing f(4): pass


More about Messages Hooks
-------------------------
Previously, we said that the pre message and post message are stored in ``pre_vector`` and
``post_vector``. The vectors are really exist in the ``autotest`` package, however,
you can not access to them directly.

Here is an example shows you how to access to them:

.. code-block:: r

    library(autotest)
    autotest:::ErrorHandler$pre
    ## NULL
    autotest:::ErrorHandler$post
    ## NULL

    registerPreMsg('Register pre message')
    addPreMsg('--------------')
    autotest:::ErrorHandler$pre
    ## [1] "Register pre message" "--------------"

    registerPostMsg('Register pre message')
    addPostMsg('**************')
    autotest:::ErrorHandler$post
    ## [1] "Register pre message" "**************"

.. note::

    One more thing, the customized messages **does not** work outside the ``test_that`` function.
    A ``test_that`` call is a testing block that tests a specific variable(or function).

Consider the following example that testing ``x`` and ``y``. We know that the first test block
will pass and the second test block will have a trouble.
The registered message in the first test should not work in the second test block.

.. code-block:: r

    library(autotest)
    x = 1; y = 2
    test_that('', {
      registerPreMsg('testing x')
      expect_equal(x, 1, trace=FALSE)  # this test will pass
    })
    test_that('', {
      expect_equal(y, 1)  # error, the registered message 'testing x' should not show here
    })

.. error::
    | AutoTestCaseError:
    | Testing variable/expression:  y
    | Your answer is 2, which is not equal to the correct answer 1


Turn On/Off Automatic Tracing
-----------------------------

- Turn on/off locally:

.. code-block:: r

    expect_equal(x, y, trace=FALSE)  # only close tracing in this


- Turn on/off globally:

.. code-block:: r

    turnOnTraceback()   # turn on globally
    turnOffTraceback()  # turn off globally

.. warning::

    You should be careful do **not** use it unless you readlly know what you are doing.


- Local VS Global

    - If you turn off globally, then set ``trace=TRUE`` **does not** work.
    - If you turn on globally, set ``trace=FALSE`` works.


Turn On/Off Default Error
-------------------------

In the ``expect_XXX`` functions, the error message usually means something.

.. code-block:: r

    x = 1
    expect_equal(x, 2)
    ## AutoTestCaseError:
    ## Testing variable/expression:  x
    ## Your answer is 1, which is not equal to the correct answer 2
    ## The maximum tolerance is 0.00001

The second line of the error is **automatic tracing**, and the last two lines are defined
in the ``expect_equal`` function, you can not change it.

However, you can set ``suppressErr=TRUE`` to remove it.
In some cases, if you really want to change the error message,
register a post message and disable the default error.

Here is an example:

.. code-block:: r

    test_that('', {
        registerPostMsg('You answer is wrong')
        expect_equal(x, 2, suppressErr=TRUE)
    })
    ## AutoTestCaseError:
    ## Testing variable/expression:  x
    ## You answer is wrong


Conclusion
----------

Usually, the default error will be understandable without doing any extra argument settings in the ``expect_XXX`` function.

However, by using the APIs, you can fully control the error messages. Enjoy!


