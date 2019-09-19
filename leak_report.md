# Leak report

_Use this document to describe whatever memory leaks you find in `clean_whitespace.c` and how you might fix them. You should also probably remove this explanatory text._

The memory leak is whenever is_cleaned is calling strip because strip is actually allocating memory. However, we can't just free memory in strip because the result which has the allocation is being passed to is_cleaned to be evaluated. So, in is_cleaned once the value cleaned which has the allocation is done being used in the result comparison we can then free(cleaned); so it frees up the memory that it creates.
