# Find the Longest Continuous Subsequence with SQL 

I've recently come across an interesting algorithmic problem in work, that is: given a sequence of integers, find the longest continuous subsequence. Here is a concrete example of this problem:

Assume we have a series of years in the numeric form. To be more specific:

`1999, 2000, 2001, 2004, 2005, 2007, 2010, 2011, 2012, 2013, 2014, 2015, 2016, 2017, 2019`

The longest continuous subsequence is 2010 - 2017, which is 8 years. 

### The iterative solution

This is a fairly simple problem to solve in any real programming languages with a for loop. For example, in python:

```python
from typing import List

def find_subseq(seq:List[int]) -> int:
    # set a counter for continuous sub seq, starts as 1.
    counter = 1
    continuous_seq_lengths = []
    for i in range(len(seq)):
        if i == 0:
            continue
            
        # add to counter if the increment is exactly 1
        # otherwise, archive the current length and reset the counter
        if seq[i] - seq[i-1] == 1:
            counter += 1
        else:
            continuous_seq_lengths.append(counter)
            counter = 1

    # append the final count
    continuous_seq_lengths.append(counter)
    return max(continuous_seq_lengths)
```

You can also get all sub sequences by telling the function to return `continuous_seq_lengths`. 

### The SQL tabular solution

However, it's much less intuitive to solve in SQL without the ability to iterate. Suppose the data is given in a tabular format as follows:

```sql
create table years (
    "year" integer
);
insert into years 
values 
    (1999), 
    (2000),
    (2001),
    (2004),
    (2005),
    (2007),
    (2010),
    (2011),
    (2012),
    (2013),
    (2014),
    (2015),
    (2016),
    (2017),
    (2019)
```

The trick here is to use the exact 1-year lag that we are trying to identify.  The solution can be outlined in the following steps:

1. Use select statement to create a new year column with values as original year + 1.
2. Self join the original years with the sub query created in 1.
3. There will be nulls from both sides of the join, so we make a coalesced new year column.
4. Make a new column to flag the NULLs by different types after the join (see the code below for details).
5. Call the `row_number()` function twice, one count straight down over the years, the other partitioned by the flag column in 4.
6. Calculate the differences/lags between the two row numbers, the rows with the same differences/lags and the 'continue' flag should belong to the same group of a continuous subsequence.
7. Use a group by statement to find lengths of continuous subsequences under each lag + flag combination. The max length should be the length + 1 (the starting year).

Here is the code:

```sql
-- A CTE for 1 & 2
with year_flags as (
	select 
        coalesce(t1.year, t2.year) as year,
        t1.year as year1,
        t2.year as year2,
        case when t1.year notnull and t2.year notnull then 'continue'
             when t1.year notnull and t2.year is null then 'new seq'
             when t1.year is null and t2.year notnull then 'break' end as "flag"
    from years t1
    full join (
        select year+1 as year
        from years
	) t2
    on t1.year = t2.year
)

select 
    flag, count(1) + 1 as seq_len
from (
    select *, 
        row_number() over(order by year) as row1,
        row_number() over(partition by flag order by year) as row2,
        row_number() over(order by year) - row_number() over(partition by flag order by year) as diff
    from year_flags
    order by year
) x
where flag = 'continue'
-- records with same flag + diff combo would be a continuous subsequence
group by flag, diff

```

The output of this query per the given data would be as follows, and the start year and break year can also be selected with ease if needed:

```
flag  seq_len
continue 3
continue 2
continue 8
```

### Conclusion

This algorithm can be easily generalized in situations which involves group aggregations too. One can simply add additional fields in the `partition by` and `group by `statements. 

However, this is totally not recommended to be done in pure SQL as it can be solved within minutes with any formal programming languages. This write-up is more of a demonstration piece that SQL can be used to solve a problem that seemingly iterative in nature.



