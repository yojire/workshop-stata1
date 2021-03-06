^#^ Data Manipulation

Let's reload the "auto" data to discard any changes made in previous sections and to start fresh.

~~~~
<<dd_do>>
sysuse auto, clear
<</dd_do>>
~~~~


^#^^#^ `generate`

The `generate` command can be used to create new variables which are functions of existing variables. For example, if we look at the variable label
for `weight`, we see that it is measured in pounds.

~~~~
<<dd_do>>
describe weight
<</dd_do>>
~~~~

Let's create a second weight variable measured in tons. The syntax for `generate` is straightforward,

```
generate <new varname> = <function of old variables>
```

~~~~
<<dd_do>>
generate weight2 = weight/2000
<</dd_do>>
~~~~

The `list` command can be used to output some data, let's use it here to output the first 5 rows' `weight` and `weight2` variables:


~~~~
<<dd_do>>
list weight* in 1/5
<</dd_do>>
~~~~

If you check the arithmetic, you'll see that we've got the right answer. We should probably add a variable label to our new `weight`

~~~~
<<dd_do>>
label variable weight2 "Weight (tons)"
describe weight*
<</dd_do>>
~~~~

In addition to direct arithmetic equations, we can use a number of functions to perform calculations. For example, a common transformation is to take
the log of any dollar amount variable, in our case `price`. This is done because typical dollar amount variables, such as price or salary, tend to be
very right-skewed - most people make $30k-50k, and a few people make 6 or 7 digit incomes.

~~~~
<<dd_do>>
generate logprice = log(price)
label variable logprice "Log price"
list *price in 1/5
<</dd_do>>
~~~~

In that command, `log` is the function name, and it is immediately followed by parentheses which enclose the variable to operate on. Read the
parentheses as "of", so that `log(price)` is read as "log of price".

There are a lot of functions that can be used. We list some commonly used mathematical functions below for your convenience:

- `+`, `-`, `*`, `/`: Standard arithmetic
- `abs( )`: returns the absolute value
- `exp( )`: returns the exponential function of \(e^x\)
- `log( )` or `ln( )`: returns the natural logarithm of the argument^[If you want log with a different base, you can use the transformation that
  dividing by `log(b)` is equivalent to using `b` as a base. In other words, if you need log base 10, use `gen newvar = log(oldvar)/log(10)`.]
- `round( )`, `ceil( )`, `floor( )`: returns the rounded value (rounded to nearest integer, rounded up, and rounded down)
- `sqrt( )`: returns the square root


You can see a full accounting of all functions you can use in this setting in

```
help functions
```


^#^^#^^#^ Creating dummies

Dummy variables (also known as indicator variables or binary variables) are variables which take on two values, 0 and 1^[Technically and mathematically
they can take on any two values, but your life will be easier if you stick with the 0/1 convention.]. These are typically used in a setting where the
0 represents an absence of something (or an answer of "no") and 1 represents the presence (or an answer of "yes"). When naming dummy variables, you
should keep this in mind to make understanding the variable easier, as well as extracting interpretations regarding the variable in a model.

For example, "gender" is a poor dummy variable - what does 0 gender or 1 gender represent? Obviously we could (and should) use [value
labels](data-management.html#label-values) to associate 0 and 1 with particular genders, but it is more straightforward to use "female" as the dummy
variable - a 0 represents "no" to the question of "Female?", hence male; and a 1 represents a "yes", hence female.

If you are collecting data, consider collecting data as dummies where appropriate - if the question has a binary response, encode it as a dummy
instead of strings. If a question has categorical responses, consider encoding them as a series of dummy variables instead (e.g. "Are you from MI?",
"Are you from OH?" etc). These changes will (usually) need to be made later anyways.

Before we discuss creating dummy variables, we need to understand logical operators. Consider the following statements

^$$^
  4 \gt 2
^$$^
^$$^
  1 \gt 2
^$$^


Remembering back to middle school math classes that ^$^\gt^$^ means "greater than", clearly the first statement is true and the second statement is
false. We can assign values of true and false to any such conditional statements, which use the following set of conditional operators:

| Sign       | Definition                     | True example                        | False example                      |
|:----------:|:-------------------------------|:-----------------------------------:|:----------------------------------:|
| ^$^==^$^   | equality                       | ^$^3 == 3^$^                        | ^$^3 == 2^$^                       |
| ^$^!=^$^   | not equal                      | ^$^3 != 4^$^                        | ^$^3 != 3^$^                       |
| ^$^\gt^$^  | greater than                   | ^$^4 \gt 2^$^                       | ^$^1 \gt 2^$^                      |
| ^$^\lt^$^  | less  than                     | ^$^1 \lt 2^$^                       | ^$^4 \lt 2^$^                      |
| ^$^\gt=^$^ | greater than or equal to       | ^$^4 \gt= 4^$^                      | ^$^1 \gt= 2^$^                     |
| ^$^\lt=^$^ | less than or equal to          | ^$^1 \lt= 1^$^                      | ^$^4 \lt= 2^$^                     |
| \&         | and (both statements are true) | ^$^(4 \gt 2)^$^ \& ^$^(3 == 3)^$^   | ^$^(4 \gt 2)^$^ \& ^$^(1 \gt 2)^$^ |
| ^$^\|^$^   | or (either statement is true)  | ^$^(3 == 2) \| (1 \lt= 2)^$^        | ^$^(4 \lt 2) \| (1 \gt 2)^$^       |

You can also use parentheses in combination with \& and ^$^\|^$^ to create more logical statements (e.g. `True & (False | True)` returns true).

Now here's the trick: In Stata^[This is true of most statistical software in fact.], conditional statements return 1 (True) and 0 (False). So we can
use them in `generate` statements to create binary variables easily.

~~~~
<<dd_do>>
generate price4k = price > 4000
list price* in 1/5
<</dd_do>>
~~~~


Now, `price4k` takes on values 1 and 0 depending on whether the conditional statement was true.

For a slightly more complicated example, lets create a dummy variable representing cheap cars. There are two possible definitions of cheap cars - cars
which have a low cost, or cars which have low maintenance costs (high mileage and low repairs).

~~~~
<<dd_do>>
generate cheap = price < 3500 | (rep78 <= 2 & mpg > 20)
list make price rep78 mpg if cheap == 1
<</dd_do>>
~~~~

The `list` commands conditions on `cheap == 1` because again, the conditional statement will return 1 for true and 0 for false. So we see three cheap
cars, two with low cost and one with low maintenance.

^#^^#^^#^ Hidden variables

The name "hidden variables" may be slightly more dramatic than need be. In Stata, under the [One Data](basics.html#one-data) principal, any
information in the data^[We'll see some exceptions to this in the [programming](programming.html) section.] must be in a variable. This includes the
so called "hidden variables" of `_n` and `_N`. You can imagine that each row of your data has two additional columns of data, one for `_n` and one for `_N`.

`_n` represents the row number currently. Currently, meaning if the data is re-sorted, `_n` can change.

`_N` represents the total number of rows in the data, hence this is the same for every row. Again, if the data changes (e.g. you [drop](#keep-drop)
some data) then `_N` may be updated.

While you cannot access these hidden variables normally, you can use them in generating variables or conditional statements. For example, we've seen
that `list` can use `in` to restrict the rows it outputs, and we've seen that it can use `if` to choose conditionally. We can combine these:

~~~~
<<dd_do>>
list make in 1/2
list make if _n <= 2
<</dd_do>>
~~~~

A more useful example is to save the initial row numbering in your data. When we discuss [sorting](#sorting) later, it may be useful to be able to
return to the original ordering. Since `_n` changes when the data is re-sorted, if we save the initial row numbers to a permanent variable, we can
always re-sort by it later. `_N` is slightly less useful but can be used similarly.

~~~~
<<dd_do>>
generate row = _n
generate totalobs = _N
list row totalobs in 1/5
<</dd_do>>
~~~~

^#^^#^^#^ Extension to `generate`

The command `egen` offers some functionality that `generate` lacks, for example creating the mean of several variables

```
egen <newvar> = rowmean(var1, var2, var3)
```

The functions which `egen` support are fairly esoteric; you can see the full list in the help:

```
help egen
```

^#^^#^ `replace`

[Earlier](#generate) we created the `weight2` variable which changed the units on weight from pounds to tons. What if, instead of creating a new variable,
we tried to just change the existing `weight` variable.

~~~~
<<dd_do>>
generate weight = weight/2000
<</dd_do>>
~~~~

Here Stata refuses to proceed since `weight` is already defined. To overwrite `weight`, we'll instead need to use the `replace` command.

~~~~
<<dd_do>>
replace weight = weight/2000
list weight in 1/5
<</dd_do>>
~~~~

`replace` features syntax identical to `generate`.^[`generate` has a few features we do not discuss which `replace` does not support. Namely,
`generate` can set the [type](data-management.html#compress) manually (instead of letting Stata choose the best type automatically), and `generate`
can place the new variable as desired rather than [using `order`](data-management.html#changing-variable-ordering). Clearly, neither of these features
are needed for `replace`.]

^#^^#^^#^ Conditional variable generation

One frequent task is recoding variables. This can be "binning" continuous variables into a few categories, re-ordering an ordinal variables, or
collapsing categories in an already-categorical variable. There are also multi-variable versions; e.g. combining multiple variables into one.

The general workflow with these cases will be to optionally use `generate` to create the new variable, then use `replace` to conditional replace the
original or new variable.

As a first example, let's collapse the `rep78` variable into a low/mid/high cost of maintenance categorical variable (1 repair, 2-3 repairs, 4-5
repairs). First, we'll generate the new variable.

~~~~
<<dd_do>>
generate maintcost = 1
tab maintcost
<</dd_do>>
~~~~

Without any variables or conditions, every row is set to 1. We'll let the 1 represent the "low" category, so next we'll replace it with 2 for cars in
the "mid" category.

~~~~
<<dd_do>>
replace maintcost = 2 if rep78 >= 2 & rep78 <= 3
tab maintcost
<</dd_do>>
~~~~

Finish with the "high" category.

~~~~
<<dd_do>>
replace maintcost = 3 if rep78 > 3
tab maintcost
<</dd_do>>
~~~~

There's one additional complication. Stata represents missing values by `.`, and `.` has a value of positive infinity. That means that

^$$^
  400 \lt .
^$$^

is true! There is some discussion [on the Stata FAQs](http://www.stata.com/support/faqs/data-management/logical-expressions-and-missing-values/) that
goes into the rationale behind it, but the short version is that this slightly complicates variable generation but greatly simplifies and protects
some data management tasks.

The complication referred to can be seen in row 3 here:

~~~~
<<dd_do>>
list make rep78 maintcost in 1/5
<</dd_do>>
~~~~

The AMC Spirit has a high repair cost even though we do not have its repair record. We can fix this easily.

~~~~
<<dd_do>>
replace maintcost = . if missing(rep78)
tab maintcost, missing
<</dd_do>>
~~~~

The `missing()` function returns true for each row with a missing value, and false for each row with an observed value, for the variable inside the
parantheses (in this case, `rep78`).

You may occasionally see `if rep78 != .` or `if rep78 <= .` instead of the `missing()` function. Recall that missing values are sorted to be larger
than the largest observed value, so this works just as well as `missing()`. However, Stata allows you to define "reasons" for missing, specifically
`.a`, `.b`, all the way through `.z`. These are sorted such that `.` < `.a` < `.b` < ... < `.z`. For this reason, `!= .` is not suggested, as while
`.` will be captured as missing, `.a`, etc will not be. Using `missing()` removes the temptation to write `!=` instead of `<=`.

The `missing()` function can be proceeded with an exclamation point to indicate not missing. For example

```
replace maintcost = 2 if !missing(rep78)
```

The `missing` option to `tab` forces it to show a row for any missing values. Without it, missing rows are suppressed.

To summarize, we used the following commands:

```
generate maintcost = 1
replace maintcost = 2 if rep78 >= 2 & rep78 <= 3
replace maintcost = 3 if rep78 > 3
replace maintcost = . if missing(rep78)
```

There are various other ways it could have been done, such as

```
generate maintcost = 1 if rep78 == 1
replace maintcost = 2 if rep78 >= 2 & rep78 <= 3
replace maintcost = 3 if rep78 > 3 & !missing(rep78)
```

```
generate maintcost = .
replace maintcost = 1 if rep78 == 1
replace maintcost = 2 if rep78 >= 2 & rep78 <= 3
replace maintcost = 3 if rep78 > 3 & !missing(rep78)
```

Of course, we could also generate it in the reverse order (3 to 1). There are also alternate ways to write the various conditionals, such as replacing
`rep78 > 3` with `rep78 >=4` or even `rep78 == 4 | rep78 == 5` (though the last is a bit wordy). There are usually multiple correct ways to specify
any conditional.

^#^^#^ `recode`

The above example for `replace` was fairly simplistic, but you can imagine the need for a much more complicated replacing structure (perhaps based on
the value of multiple variables). If, however, you do have something this simple, the `recode` command could be used instead.

The `recode` command syntax is fairly simple,

```
recode <oldvar> (<rule 1>) (<rule 2>) ...., generate(<newvar>)
```

The different rules define the recoding to take place. For example, the above creation of `maintcost` can be written as

```
recode rep78 (1 = 1) (2 3 = 2) (4 5 = 3) (missing = .), generate(maintcost)
```

Each rule has the form of `old value(s) = new value`, where the old values can be either a single number, several numbers (either listed as above, `1
5 10`, or a [numlist](working-with-data-sets.html#loading-subsets-of-the-data) like `7/12` to include all values between 7 and 12), the phrases
"missing", "nonmissing" or "else" to capture anything not elsewhere defined. The new value must be a single number or a missing value (`.` or `.a`,
etc). "else" cannot be used if "missing" or "nonmissing" is defined (and vice-versa), and all of those must be the last rules defined. E.g.,

```
recode x (missing = 5) (2 = 4) (else = 3) (1 = 2), generate(y)
```

will not run because "missing" and "else" are both simultaneously defined, and the `1 = 2` rule is last instead of "else" or "missing".

Note that if you see older code you may see either the parantheses or the `generate` option excluded. You should include both of these.

Finally, the rules are executed left-to-right. So if you have two rules referring to the same values, the first one is used, and the second is
not. For example,

```
recode x (1/5 = 7) (2 = 4), generate(y)
```

The `2 = 4` rule will never take place because 2 is already recoded to 7 in the `1/5 = 7` rule.

^#^^#^ Subsetting

Almost any Stata command which operates on variables can operate on a subset of the data instead of the entire data, using the conditional statements
we just learned. Specifically, we can append the `if <condition>` to a command, and the command will be executed as if the data for which the
conditional does not return True does not exist. This is equivalent to throwing away some data and then performing the command. In general, you should
avoid discarding data as you never know when you will possible use it. Of course, you could
use [`preserve` and `restore`](working-with-data-sets.html#preserverestore) to temporarily remove the data, but using the conditional subsetting is
more straightforward.

~~~~
<<dd_do>>
summ price
<</dd_do>>
~~~~

This gives us a summary of price across all cars. What if we wanted to look at the summary only for non-foreign cars?

~~~~
<<dd_do>>
codebook foreign
summ price if foreign == 0
<</dd_do>>
~~~~

First, we check the label to make sure we're looking at the appropriate value of `foreign`. Now, notice that "Obs", the number of rows of the data, is
only 52 as we expect! If we look also at foreign cars,

~~~~
<<dd_do>>
summ price if foreign == 1
<</dd_do>>
~~~~

we see that American cars are cheaper on average^[Note that this is not a statistical claim, we would have to do a two-sample t-test to make any
statistical claim.].

^#^^#^^#^ `by` and `bysort`

To look at the average price for American and foreign cars, we ran two individual commands. If we wanted to look at the summaries by `rep78`, that
would take 6 commands (values 1 through 5 and `.`)!

Instead, we can use `by` and `bysort` to perform the same operation over each unique value in a variable. For example, we could repeat the above with:

~~~~
<<dd_do>>
by foreign: summ price
<</dd_do>>
~~~~

There is a strong assumption here that the data is already sorted by the variables we are splitting `by` on (e.g. `foreign`). If `foreign` were not
sorted (or if you simply did not want to check/assume it was), you could instead use

```
bysort foreign: summ price
```

`bysort` is identical to [sorting](#sorting) first and running the `by` statement afterwards. In general, it is recommended to always use `bysort`
instead of `by`, *unless* you believe the data is already sorted and want an error if that assumption is violated.

Before running these commands, consider generating a [original ordering variable](#hidden-variables) first.

`bysort`'s variables cannot be conditional statements, so if you wanted to for example get summaries by low and high mileage cars, you'd need to
generate a dummy variable first.

~~~~
<<dd_do>>
gen highmileage = mpg > 20
bysort highmileage: summ price
<</dd_do>>
~~~~

`bysort` can take two or more variables, and performs its commands within each unique combination of the variable. For example,

~~~~
<<dd_do>>
bysort foreign highmileage: summ price
<</dd_do>>
~~~~

When specifying `bysort`, you can optionally specify a variable to sort on but not to group by. For example, let's say your data consisted of doctors
visits for patients, where patients may have more than one appointment. You want to generate a variable indicating whether a visit is the first by the
patient. Say `id` stores the patient id, `date` stores the date of the visit.

```
bysort id (date): gen firstvisit = _n == 1
```

By placing `date` in parentheses, this ensures that within each `id`, the data is sorted by `date`. Therefore the first row for each patient is their
first visit, so `_n == 1` evaluates to 1 only in that first row and 0 zero otherwise.

^#^^#^^#^ `keep`, `drop`

If you do want to discard data, you can use `keep` or `drop` to do so. They each can perform on variables:

```
keep <var1> <var2> ...
drop <var1> <var2> ...
```

or on observations:

```
keep if <conditional>
drop if <conditional>
```

Note that these *cannot* be combined:

~~~~
<<dd_do>>
drop turn if mpg > 20
<</dd_do>>
~~~~

`keep` removes all variables except the listed variables, or removes any row which the conditional does not return true.

`drop` removes any listed variables, or removes any row which the conditional returns true.

^#^^#^ Dealing with duplicates

If your data is not expected to have duplicates, either across all variables or within certain variables, the `duplicates` command can make their
detection and correction easier. The most basic command is `duplicates report`, which simply reports on the status of duplicate rows. Let's use the
built-in "bplong" data. This data contains 120 patients (`patient`) with measures of blood pressure (`bp`) at two different time points (`when`, a
Before and After), and some descriptive variables (`sex` and `agegrp`).

~~~~
<<dd_do>>
sysuse bplong, clear
duplicates report
<</dd_do>>
~~~~

This report is not very interesting; it reports that there are 240 observations which have 1 copy (e.g. are unique), and hence no surplus. Given that
each row should be unique (just in patient ID and before/after alone), this is not surprising. Let's instead look at the duplicates just for `bp` and
`when`:

~~~~
<<dd_do>>
duplicates report bp when
<</dd_do>>
~~~~

Here we have some duplicates. First, there are 23 observations which are fully unique. All other observations have repeats to some extent.

The second row of the output tells of us that there are 42 observations which have 2 copies. The language here can be a bit confusing; all it is
saying is that there are 42 rows, each of which has a single duplicate *within that same 42*. So if we have values 1, 1, 2, 2, that would be reported
as 4 observations with 2 surplus.

The number of surplus is the number of non-unique rows in that category. We could compute it ourselves - we know that there are 42 rows with 2 copies,
so that means that half of the rows are "unique" and the other half are "duplicates" (which is unique and which is duplicate is not clear). So 42/2 =
21, and we have 21 surplus.

Consider the row for 4 copies. There are 48 rows, each of which belongs to a set of four duplicates. For example, 1, 1, 1, 1, 2, 2, 2, 2, has
observations 8 and copies 2. In this row, 48/4 = 12, so there are 12 unique values, meaning 36 surplus.

Other useful commands include

- `duplicates list`: Shows every set of duplicates, including its row number and value. Obviously for something like this the output would be massive
    as of the 240 total rows, only 23 are not duplicated to some degree!
- `duplicates tag <vars>, gen(<newvar>)`: Adds a new variable which represents the number of other copies for each row. For unique rows, this will
  be 0. For any duplicated rows, it will essentially be "copies" from `duplicates report` minus 1. This can be useful for examining duplicates or
  dropping them.
- `duplicates drop`: Be cautious with this, as it drops any row which is a duplicate of a previous row (in other words keeps the first entry of every
  set of duplicates).

^#^^#^ Sorting

We already saw sorting [in the context of `bysort`](#by-and-bysort). We can also sort as a standalone operation. As before, consider generating
a [original ordering variable](#hidden-variables) first.

We'll switch back to "auto" first.

~~~~
<<dd_do>>
sysuse auto, clear
gen order = _n
<</dd_do>>
~~~~

The `gsort` function takes a list of variables to order by.

~~~~
<<dd_do>>
gsort rep78 price
list rep78 price in 1/5
<</dd_do>>
~~~~

Stata first sorts the data by `rep78`, ascending (so the lowest value is in row 1). Then within each set of rows that have a common value of `rep78`,
it sorts by `price`.

You can append "+" or "-" to each variable to change whether it is ascending or descending. Without a prefix, the variable is sorted ascending.

~~~~
<<dd_do>>
gsort +rep78 -price
list rep78 price in 1/5
<</dd_do>>
~~~~

Recall that missing values (`.`) are [larger than any other values](data-manipulation.html#conditional-variable-generation). When sorting with missing
values, they follow this rule as well. If you want to treat missing values as smaller than all other values, you can pass the `mfirst` option to
`gsort`. Note this does *not* make missingness "less than" anywhere else, only for the purposes of the current sort.

Sorting strings does work and is done alphabetically. All capital letters are "less than" all lower case letters, and a blank string ("") is the
"smallest". For example, if you have the strings "DBC", "Daa", "", "EEE", the sorted ascending order would be "", "DBC", "Daa", "EEE". The blank is
first; the two strings starting with "D" are before the string "EEE", and the upper case "B" precedes the lower case "a".

As a side note, there is an additional command, `sort`, which can perform sorting. It does not allow sorting in descending order, however it does allow
you to conditionally sort; that is, passing something like `sort <varname> in <condition>` would sort only those rows for which the condition is true,
the remaining rows remain *in their exact same position*.

^#^^#^ Working with strings and categorical variables

String variables are commonly used during data collection but are ultimately not very useful from a statistical point of view. Typically string
variables should be represented as [categorical variables with value labels](data-management.html#label-values) as we've previously discussed. Here
are some useful commands for operating on strings and categorical variables.

^#^^#^^#^ `destring` and `tostring`

These two commands convert strings and numerics between each other.

```
destring <variable>, gen(<newvar>)
tostring <variable>, replace
```

Both commands can take the options `replace` (to replace the existing variable with the new one) or `gen( )` (to generate a new variable). I would
recommend always using `gen` to double-check that the conversion worked as expected, then using `drop`, `rename` and `order` to replace the existing
variable.

~~~~
<<dd_do>>
desc mpg
tostring mpg, gen(mpg2)
desc mpg2
list mpg* in 1/5
<</dd_do>>
~~~~
Now that the new string is correct, we can replace the existing `mpg`.
~~~~
<<dd_do>>
drop mpg
rename mpg2 mpg
order mpg, after(price)
<</dd_do>>
~~~~
Let's go the other way around:
~~~~
<<dd_do>>
desc mpg
destring mpg, gen(mpg2)
desc mpg2
list mpg* in 1/5
drop mpg
rename mpg2 mpg
order mpg, after(price)
<</dd_do>>
~~~~
And we're back to the original set-up.^[If you are sharp-eyed, you may have noticed that the original `mpg` was an "int" whereas the final one is a
"byte". If we had called [`compress`](data-management.html#compress) on the original data, it would have done that type conversion anyways - so we're
ok!]

When using `destring` to convert a string variable (that it storing numeric data as strings - "13", "14") to a numeric variable, if there are *any*
non-numeric entries, `destring` will fail:

~~~~
<<dd_do>>
destring make, gen(make2)
<</dd_do>>
~~~~

We must pass the `force` option. With this option, any strings which have non-numeric variables will be marked as missing.

~~~~
<<dd_do>>
destring make, gen(make2) force
tab make2, mi
<</dd_do>>
~~~~

`tostring` also accepts the `force` option when using `replace`, we recommend instead to **never** use `replace` with `tostring` (you probably should
not use it with `destring` either!).

^#^^#^^#^ `encode` and `decode`

If we have a string variable which has non-numerical values (e.g. `race` with values "white", "black", "Hispanic", etc), the ideal way to store it is
as numerical with [value labels](data-management.html#label-values) attached. While we could do this manually using a combination of `gen` and
`replace` with some conditionals, a less tedious way to do so is via `encode`.

We'll switch to the "surface" data set, which contains sea surface temperature measurements from a number of locations over two days. (Remember this
will erase any existing unsaved changes! You will not need any modifications you've made to other built-in datasets going forward [except `census9`
from [exercise 3](data-management.html#execise-3), but if you do want to save it, do so first!)

~~~~
<<dd_do>>
sysuse surface, clear
desc date
tab date
<</dd_do>>
~~~~

The `date` variable is a string with two values. First, let's create a numeric with value labels version manually.

~~~~
<<dd_do>>
gen date2 = date == "01mar2011"
label define date 0 "01mar2011" 1 "11mar2011"
label values date2 date
desc date2
tab date2
<</dd_do>>
~~~~

Instead, we can easily use `encode`:

~~~~
<<dd_do>>
encode date, gen(date3)
desc date3
tab date3
<</dd_do>>
~~~~

However, `encode` starts numbering at 1 instead of 0, which is not ideal for dummy variables. To get around this, we can create our value label
manually first, then pass it as an argument to `encode`.

~~~~
<<dd_do>>
label define manualdate 0 "01mar2011" 1 "11mar2011"
encode date, gen(date4) label(manualdate)
tab date3
tab date3, nolab
tab date4
tab date4, nolab
<</dd_do>>
~~~~

This can be extended to allow any sort of ordering desired. For this trivial binary example, it might actually be faster to use `gen` and do it
manually, but for a variable with a large number of categories, this is much easier.

`decode` works in the reverse, creating a string out of a numeric vector with labels attached to it.

~~~~
<<dd_do>>
decode date4, gen(date5)
desc date5
tab date5
<</dd_do>>
~~~~

`decode` will fail if its target does not have value labels attached.

^#^^#^^#^ String manipulation

If you find yourself in a situation where you simply must manipulate the strings directly, there are a number of string functions. You can see the
full list in `help string functions`, but below we list a few commonly used ones.

- `strlen`: Returns the number of characters in the string
- `wordcount`: Returns the number of whitespace-separated words.
- "`+`": Adding two strings together concatenates them (e.g. "abc" + "def" = "abcdef").
- `strupper` and `strlower`: Converts to lower/upper case.
- `strtrim`: Removes white space before and after strings (e.g. `strtrim(" string ") = "string"`). To remove only left (preceding) or right (following)
  spaces, use `strltrim` or `strrtrim`.
- `substr`: Returns the substring starting at an index for a given number of characters (e.g. `substr("abcdefg", 2, 3) = "bcd").

These are used inside `gen` and `replace`, e.g.

~~~~
<<dd_do>>
gen date6 = strupper(date5)
gen month = substr(date5, 3, 3)
list date5 date6 month in 1/5
<</dd_do>>
~~~~

^#^^#^ Exercise 5

Open the *saved* version of "census9" with `use`, not the original version with `webuse`.

1. Generate a new variable, `deathperc`, which is the percentage of deaths in each state. (Remember that `deathrate` is deaths per 10,000.)
2. The average age of all Americans in 1980 is roughly 30.11 years of age. [Generate a categorical](#conditional-variable-generation) with four values
   as described before, with appropriate [value labels](data-management.html#label-values).
    - Significantly below national average: `medage` equal to 26.20 or less
    - Below national average: `medage` greater than 26.20 and less than or equal to 30.10.
    - Above national average: `medage` greater than 30.10 and less than or equal to 32.80.
    - Significantly above national average: `medage` greater than 32.80.
3. What is the death rate in [each of those four categories](#by-and-bysort)? (You can use `summarize` to obtain the means.) Does there appear to be
   any pattern?
4. What state has the [lowest](#sorting) death rate? The highest? The lowest average age? The highest?
5. Each state has a single observation here, but if we had multiple years of data, then we could have ["long data"](#reshaping-files) with multiple
   rows per state. To prepare for this sort of data, [encode](#encode-and-decode) the two-letter state abbreviation into a numeric value with value
   labels.

^#^^#^ Merging Files

When managing data sets, the need often arises to merge two data sets together, either by matching two files together according to values on certain
variables, or by adding cases to an existing data set. We’ll start with the simpler case of adding cases to an existing data set.

^#^^#^^#^ Appending Files

Appending is straightforward. Observations in a new data set (called the "using" data) are appended to the current data set (called the "master"
data), matching variable names. If a variable in the using data exists already in the master data, its values are entered there. If a variable in the
using data does not exist in the master, the new variable is added to the appended data set which is missing for all members of the master data. This
is easiest to visualize. There are two data sets on [Stata's website](working-with-data-sets.html#stata-website-data) which we can append, "odd" and
"even".

~~~~
<<dd_do>>
webuse odd, clear
list
webuse even
list
<</dd_do>>
~~~~

It does not truly matter which data set is the master data and which is the using data (it will later in [match-merging](#match-merging-data)), it
will only affect the sorted order (the data in master is sorted first). The syntax is simply

```
append using <using data>
```

~~~~
<<dd_do>>
webuse even
append using http://www.stata-press.com/data/r15/odd
list
<</dd_do>>
~~~~

(We must specify the complete path to the data instead of the `webuse` shorthand of just the data name. Of course, with real data that you locally
have on your computer, you follow the [working directory](basics.html#working-directory) rules; if the file exists in your working directory you just
give the name, otherwise give the complete path. I obtained that path by visiting the Stata data website and finding the link to "odd".)

Note that the "number" variable, which exists in both data sets, has complete data, while "even" and "odd" both have missing data as expected. The
`using` part of the command is where the term the "using data" comes from.

Stata is case sensitive in its variable names, so "varname" and "Varname" are two unique variables. We could always fix this later with something like

```
replace varname = Varname if varname == .
```

but it is better to ensure before appending that the shared variables have identical names.

^#^^#^^#^ Match-merging Data

A more common data management necessity is to add variables from another data set to a master data set, match-merging the cases in the data sets by
values on a single ID variable (or by values on multiple variables). There are two general forms of this match-merging.

The first, one-to-one merging, occurs when in each data, each individual is represented by only one row. For example, one data set containing final
exam information and one data set containing demographic information. This "1:1" match takes rows which match on some variable(s) and places them
together.

The second match, many-to-one, occurs when the data are measured on different "levels". For example, consider if we have one data set containing
household level characteristics and another containing town level characters. Two households from the same town would want the same town level
data. This is either "1:m" or "m:1" depending on which data is the master data and which is the using data (e.g. 1:m indicates the household data is
the master data and town is the using data).

(Technically there is also many-to-many matching, "m:m", but it is rarely used in practice.)

We'll use data sets off Stata's website again to demonstrate, specifically the "autosize" and "autocost" data which splits the "auto" data into two
pieces.

~~~~
<<dd_do>>
webuse autosize, clear
list in 1/5
webuse autocost
list in 1/5
<</dd_do>>
~~~~

Now we can perform the merge using the syntax.

```
merge 1:1 <variables to match on> using <using data set>
```

All that needs to be specified is the variable to match cases by and the name of the data set with variables to be added. We could replace `1:1` with
`m:1` or `1:m` as desired.

~~~~
<<dd_do>>
merge 1:1 make using http://www.stata-press.com/data/r15/autoexpense
list in 1/7
<</dd_do>>
~~~~

(Again, we use the full path but for local files, following [working directory](basics.html#working-directory) rules.)

First, take a look at the output of the merge command. We see that 69 cars were not matched, which means they exist in only one of the two data
sets. In this case, they all exist in master data, but in general you could see a mix of the two. The remaining 5 observations were appropriately
matched. This is a terrible merge! Hopefully with your real data, the majority of data is matched and only a few outliers are not matched.

Notice the `(_merge==#)` tags. When you perform a merge, a new variable `_merge` is added which indicates the source for each row: 1 and 2 indicate
the data was only found in the master data or using data respectively, while 3 indicates a match. There are two other possible values (4 and 5) which
occur rarely, see the documentation at `help merge` for details.

A few notes:

- Stata will sort both files by the key variables before and after merging.
- You can match on more than one variable. For example, if you had data based upon year and state, you might run `merge 1:1 state year using ...`
- If you wanted to merge another file after the initial merge, you’ll need to drop the `_merge` variable first.
- IMPORTANT NOTE: Make sure that the only variables common to both files when performing a match-merge are the variables that will be used to match
  cases (like ID)! Stata will by default keep the variable in the master data when the merge is performed if the same variable appears in more than
  one file and is not defined as a matching variable. This may cause problems when performing merges. (You can overwrite this behavior with the
  `update` or `replace` options, see the documentation for details.)

^#^^#^ Reshaping Files


Different data formats are needed for various statistical methods. Stata prefers data in "Long" format, but also makes it easy to convert between Long
and "Wide". Stata uses the `reshape` command to convert data formats.

[![](../images/wide-vs-long.png)](../images/wide-vs-long.png)


In this example, the wide format of the data has each row representing a single observation. The variables "X1", "X2" and "X3" are what make this
"wide". These are typically variables measured at different time points, but don’t have to be. In the long format, each row represents an observation
*at a specific index*.


A nice feature of Stata's `reshape` command is that the syntax to convert from wide-to-long or from long-to-wide are identical, except for desired
format (`long` vs `wide`).

Convert to long:

```
reshape long <stub>, i(<ivar>) j(<jvar>)
```

Convert to wide:

```
reshape wide <stub>, i(<ivar>) j(<jvar>)
```

We need to identify three components, the `stub`, `ivar` and `jvar`.

- "stub": The stub in wide format is the common prefix of the repeated variables names. In the illustration above, "X1", "X2" and "X3" have the common
  prefix "X". In the long format, the stub is simply the name of the variable which is repeated. In the illustration above, "X" is this
  variable. Hence the stub is `X` for both.
- "ivar": The ivar is the id variable. In the long format, this should be constant across individuals. In both formats above, the id is `ID`.
- "jvar": The jvar in long format is the variable that distinguishes which index each repeated measure is from. In the illustration above, "Index" fills
  this role. In wide format, this does not exist. So when converting from wide to long, you can use any name for the jvar.

Putting this all together together, the two commands to convert between the illustrations above would be:

```
reshape long X, i(ID) j(Index)
reshape wide X, i(ID) j(Index)
```

As an example, we'll use the built-in data set "bplong"

~~~~
<<dd_do>>
sysuse bplong, clear
list in 1/5
<</dd_do>>
~~~~

Each patient has two rows representing their before and after measurements. `when` indicates which time period the measurement occurs in, and `bp` is
the only time-varying variable (both `sex` and `agegrp` are constant, presumably the "Before" and "After" occur within a short time period such that
neither of those can change). Let's identify the three components

- "stub": Since we're going from long to wide, the "stub" is any time-varying variables, here only `bp`.
- "ivar": `patient` identifies individuals.
- "jvar": `when` identifies time period.

Putting this together,

~~~~
<<dd_do>>
reshape wide bp, i(patient) j(when)
list in 1/5
<</dd_do>>
~~~~

Each row represents a single patient, and `bp1` and `bp2` represent the before and after measurements.

Let's generate the command to convert back to long.

- "stub": "bp" is the stub of `bp1` and `bp2`.
- "ivar": `patient` identifies individuals.
- "jvar": Since the data is currently wide, there is no existing jvar and we can call it whatever we like. For consistency, we'll call it "when" again.

The command is identical! Just swap `wide` for `long`.

~~~~
<<dd_do>>
reshape long bp, i(patient) j(when)
list in 1/5
<</dd_do>>
~~~~

The variables are slightly out of order, but we've completely recovered the original data.

After you've run a single `reshape` command, and assuming nothing has changed (you do not want to change "stub", "ivar" or "jvar", and the variables
in the data are the same), you can convert between wide and long without specifying anything. Try it:

```
reshape wide
reshape long
```

Now, notice that when we reshaped the original long data into wide format, the two "bp" variables where `bp1` and `bp2`, not something like
`bp_before` and `bp_after`. In most cases this is fine (as the common use case for this is repeated measures over time), but not always - what if we
wanted to save the "before" and "after" labels? Do note that thankfully Stata saves these labels, so when converting back to long, it restores the
"Before" and "After" tags.

If you do want to save the text instead of the count, you need to use strings. We'll use [`decode`](#encode-and-decode) to convert to a string, then
use that as the jvar.

~~~~
<<dd_do>>
decode when, gen(when2)
drop when
rename when2 when
reshape wide bp, i(patient) j(when) string
list in 1/5
<</dd_do>>
~~~~

Note the `string` option. When converting back to long, you'll need to `encode` the string to get it back to numeric.


~~~~
<<dd_do>>
reshape long
desc when
encode when, gen(when2)
drop when
rename when2 when
desc when
<</dd_do>>
~~~~

A few notes:

- If you have wide data and some individuals are missing some repeated measures, when converting to tall, they will still have a row with a blank
  value. You can easily drop it:

    ```
    reshape long
    drop if <varname> == .
    ```

    Converting back to wide will re-enter those missing values; `reshape` does not care if the long data is "complete".

- More than one stub can be entered if you have more than one repeated measurement. For example, if the variables were {"id", "x1", "x2", "y1", "y2"},
  you could enter

    ```
    reshape long x y, i(id) j(index)
    ```

    Note that they technically don’t have to have the same indices (e.g. you could have "x1", "x2", "y3", "y4") although that would create a weird
    result where each row of index 1 or 2 is missing `y` and each row of index 3 or 4 is missing `x`.

- If you have wide data and many time-varying variables, there is no shorthand for entering all the stubs. For large data, this is **extremely**
  frustrating. I'd recommend using `describe, simple` to get a list of all variable names, then using find \& replace to remove the indices. If you
  know a better way, [let me know](index.html#contact-information)!
