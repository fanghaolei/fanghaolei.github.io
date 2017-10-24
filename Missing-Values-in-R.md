# Missing Values in R

Something interesting I found out about `NA` values in R. Typically, this won't cause any bug or undesired results in data analysis, but there are certain cases that this should be aware of, especially when using `rpy2` python package to call R code from a python script. 

### The type coercion

As opposed to python's `None` is registered as a `None` type, R's `NA` does not have a distinct type, and surprisingly it's a `logical` type. The class is `logical` in a pure `NA` vector as well:

```R
> class(NA)
[1] "logical"
> class(c(NA, NA, NA))
[1] "logical"
```

However, this logical type is not like `TRUE` OR `FALSE`. It can't be evaluated:

```R
> if(NA) print('!!!')
Error in if (NA) print("!!!") : missing value where TRUE/FALSE needed
```

And it will be coerced to any other non-NA types in the vector:

```R
> class(c(2, NA))
[1] "numeric"
```
You can also coerce itself it other types:

```R
> class(as.numeric(NA))
[1] "numeric"
> class(as.character(NA))
[1] "character"
```

This flexibility of `NA`'s type is actually very useful and convenient in data analysis, where you do not need to worry about type conversion all the time. 

### A potential bug with `rpy2`

Normally, this `NA` type issue won't cause any problems; however, I found a bug in using pure `NA` vectors with `rpy2` package. When I use `pandas2ri.ri2py()` function to convert R a data frame to a `pandas` DataFrame, a pure `NA` logical vector/column would turn into a Series of min values of type `int32`, which is -2147483648. 

However, if the NA value is pre-converted into `numeric` or `character` type in R, the converted `pandas` DataFrame will be normal, showing as `NaN`.  

This is a caveat to note when using `rpy2` to convert data frames from R. If there is a pure `NA` column, be sure to convert it to numeric or character type beforehand. 

Also, there is another trick: R has a constant `NA_character_` stored in the global environment, which is essentially a character typed `NA`. You can simply assign the column with this constant as `df$MyNACol <- NA_character_` to replace the default logical `NA`. 

