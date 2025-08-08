# A better spreadsheet addressing syntax

Consider all these ideas in public domain.

The ideas below come both from my own musings and with some inspiration from pyspread ( https://pyspread.gitlab.io/tutorial.html ) 

## Motivation

Whoever invented the A1 or even R1C1 addressing syntax perhaps made it easy for getting started or trivial formulas, but hard for anything else.

The main problem is that it does not allow for arbitrary math integer expressions for indices. This leads to the frequent need to use indirect specifications of index values, via INDEX, INDIRECT,ADDRESS, OFFSET... and formulas that are harder to understand, write and read.
The R1C1 syntax appears to be in the right direction, as at least you have numerical indices for both column and row. However, no expressions for indices are allowed (not even R1C[1+1] ). 
A further issue is that R1C1 syntax is quite longer than A1 equivalent. Consider the frequent case of relative references: assuming we are row 4, column 4 and refering to cell in row 1 column 1, then it's:
`=A1` 
vs
`=R[-3]C[-3]`


## Deriving the syntax
Let's assume for now that we work in a single sheet.

*Let `[1,2]` mean the reference to cell in column 1 and row 2*
This is the shortest syntax I could think of, hence no letter/name in front, so that it allows for any math expression in place of the 2 indices, but also is readable enough, and reminiscent of a cell's box.
Autoclosing of the "[" will help.

Why column before row? A tipical table (with several variables/attributes and many observations/records) contains many more columns than rows. Thinking hierarchicaly, we first specify the biggest subdivisions the the smaller ones (ex: countries, states/provinces, districts, cities, streets, houses). We'll respect this when we'll later introduce sheet index too.

More specifically, `[1,2]` will be an **absolute reference** , equivalent to , and replacing the need for, `INDIRECT(ADDRESS(2,1))`, which returns cell `$A$2`.


Importantly, `[17-16, 2*1]`, as well as any `[expression1, expression2]`, where the 2 expression return integers 1,2 will be equivalent to `[1,2]` .

**Relative references**: borrowing from pyspread, let **c**, **r** be "magic variables" that always stand for the current cell's column,row index ;
thus
`r` <=> `ROW()`, `c` <=> `COLUMN()`

Thus, in a cell `[1,2]`
`=c+r`
should return 3;

`=[c,r-1]` means the value of the cell in the same column, previous row (and sheet), relative to the formula cell.


Will Still need row/column functions, because sometimes need to compute offsets relative to an absolute address:
`=c([1,2])` to give 1, `=r([1,2])` gives 2

**Mixed absolute-relative references**
ex, if formula is in column 1, row 1, and want to refer to column 4  and 3 rows down, then '=[4, r+3]` <=> `=$D4`

There will also be the opportunity to accept ranges of indices, as in: 
`[2:5, r]`
, meaning all cells in current row at columns 2 to 5. Could also have  `[2:2:15, r]`, specifying a step, and perhaps other nice ideas borrowed from common programming languages.


### Sheet index and name
`=[sheetIndex, columnIndex, rowIndex]`
in hierarchical order, as mentioned earlier. 
**s** will stand for current sheet, while `=[1,2,3]` will refer, absolutely, to a cell in first sheet.

However, users could name their sheets as usual, and those names could be programmed to be constants standing for the respective sheet index, only cheangeable when the user shuffles or renames the sheets. 
Thus
`=[1, 2, r-1]`
is same as
`=[myfirstsheet, 2, r-1]`
and the user will index the sheet in the way that's convenient for them.

**Shortcut syntax**
Since specifying the sheet index is optional while in same sheet, i.e 
`[s,colInd,rowInd]` <=> `[colInd, rowInd]`
by extension, specifying the colInd is optional when referring to a cell in same sheet and column: 
`[c, rowInd]` <=> `[rowInd]`

## Examples and how they compare to A1 & R1C1 syntax:

**1)** Fibonacci sequence in a column, where first 2 rows hold 0; 1: 
`=A6+A5`
`=R[-1]C + R[-2]C
vs
 `=[c, r-1] + [c, r-2]` or just `=[r-1] + [r-2]`
("current value is sum of the previous 2 values")

**2)** maximum of the numbers in previous 7 columns and previous row:
`=MAX(B7:E7)`
`=MAX(R[-1]C[-7]:R[-1]C[-1]
vs:
`=MAX([c-7,r-1]:[c-1,r-1])` or just `=MAX([c-7 : c-1, r-1])`

**3)** you'll never again have to use INDIRECT(ADDRESS(...) constructs:

`=INDIRECT(ADDRESS(ROW(),RANDBETWEEN.NV(2,8)))`
vs
`=[RANDBETWEEN.NV(2,8) , r]`

**4)** you probably will never have to use INDEX or OFFSET either:

The following is for selecting first and then every 3rd number in a column B range in sheet1, assumming the result (formula) is in column J in sheet2:
`=INDEX($sheet1.B$2:B$100, 3*(row()-row(J$2))+1,1)`
`=INDEX($sheet1.R2C[-8]:R100C[-8], 3*(row()-row(R2C))+1,1)`
vs
`=[1, c-8, 3*(r-r([2]))+2 ]`

## Summary specification
`=[sheetIndex, columnIndex, rowIndex]`
Each of sheetIndex, columnIndex, rowIndex is integer starting at 1 or any math expression resulting in a valid index.
**s**, **c**,**r** are variables always standing for the respective index of the current cell (the one with the formula); as well as names for the functions returning row/column/sheet index given a reference. 

The expression for *rowIndex*: if it contains **r** then it is a relative reference. Otherwise, it's an absolute reference (even if it contains **c** or **s** ).
Similarly for columnIndex and sheetIndex.

The sheetIndex can be specified specified using the sheet name as well, the name being a constant standing for the respective sheet index (only cheangeable when the user shuffles or renames the sheets). 

**Shorter forms**
`=[columnIndex, rowIndex]`  equal to `=[s, columnIndex, rowIndex]`  (refering to cell in same sheet)
`=[rowIndex]` equal to `=[s, c, rowIndex]`  (refering to cell in same sheet and same column)

**Refering to an external workbook:**
`"filepath to workbook"@[sheetIndex, columnIndex, rowIndex]` (the `@` is just an example of possible syntax)



## Conclusion
IMHO, this way of relative addressing results in not only shorter, simpler formulas, but has the advantage that it matches the meaning of the dependency/relationship ("previous 2 values " etc) as opposed to making you stare at the row and column indices every time.

While at that, there is the opportunity to remove other syntax warts inherited from Excel and its granparents and borrow more from modern programming languages. 
