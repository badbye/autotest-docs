=============
Develop Guide
=============

The ``expect_equal`` function use the ``compare`` function to compare two objects.
``compare`` is a generic function, that why it support multiple data types.

.. code-block:: r

> library(autotest)
> methods(compare)
 [1] compare.POSIXt*     compare.character*  compare.data.frame*
 [4] compare.default*    compare.factor*     compare.integer*
 [7] compare.list*       compare.logical*    compare.matrix*
[10] compare.numeric*    compare.tbl_df*

To extend the ``expect_equal`` function to support more data types, you just need to
implement a new ``compare.XXX`` function.

For example,

.. code-block:: r


compare.ggplot <- function(x, y, ...){

}