# A better spreadsheet addressing syntax

Consider all these ideas in public domain.

## Motivation

Whoever invented the A1 or even R1C1 addressing syntax perhaps made it easy for getting started or trivial formulas, but hard for anything else.

The main problem is that it does not allow for arbitrary math integer expressions for indices. This leads to the frequent need to use indirect specifications of index values, via INDEX, INDIRECT,ADDRESS, OFFSET... and formulas that are harder to understand, write and read.
Another issue is that with A1 style, the syntax of  relative and absolute addressing is rather arbitrary, removed from the intent behind a particular formula. Consider " in this cell, I'm going to put  the previous 4 cells in same column, and repeat so for each row downward".  In A1 notation, you instead specify the particular index of rows that currently happen to be previous 4. 

## Specification for column and row indices
Drawing some inspiration from pyspread ( https://pyspread.gitlab.io/tutorial.html ) and my own musings on this:

*Let both row and column indices be numerical, starting either from 1 or 0.*

(it does not matter, but I'd choose from 0 since often in first row/col have headers)

Sheet index as well (but name will also be possible, later on this).

Let `[1,2]` mean the reference to cell in column 1 and row 2. 

More specifically, this will be an **absolute reference** , equivalent to , and replacing the need for, INDIRECT(ADDRESS(2,1)), which returns cell $A$2.

This is the shortest syntax I could think of, hence no letter/name in front, that allows for any math expression in place of the 2 indices, but also is readable enough, and kind-of reminiscent of a cells's box boundaries.

Importantly, `[17-16, 2*1]`, as well as any `[expression1, expression2]`, where the 2 expression return integers 1,2 will be allowed and mean the same thing.

**Relative addressing**: borrowing from pyspread', let **r**, **c**, **s** be "magic variables" that always stand for the current cell's row, column and sheet index ;

thus
`r` <=> `ROW()`, `c` <=> `COLUMN()`

(alternative letters could be: x,y,z (just not capital; please keep it easy to type), or i,j,k (usual math notation in linear algebra) )

Thus, for a cell with address 1,2,3, (meaning in third' sheet),
`=r+c+s`
will return 6;

`[r-1,c,s]` means the value of the cell in the previous row, same column and sheet.

If you ommit s, means current sheet: `[r-1,c]`

By extension, if you drop c as well, means current column, thus 
`=[r-1]`
refers to the cell in previous row, same column as the cell where the formula resides.

There will also be the opportunity to accept ranges of indices, as in: 
`[2:5, r]`
, meaning all cells in current row at columns 2 to 5. Could allow `[2:2:15, r]`, specifying a step, etc.

Will Still need a row/column/sheet function, because sometimes need to compute offsets relative to an absolute address:

`=r([1,2,3])` to give 1, `=c([1,2,3])` gives 2, `=s([1,2,3])` gives 3

Regarding **sheet names** :
the users could name their sheets with usual names as well, and those names could be programmed to be constants standing for the respective sheet index.
So that
`=[r-1,c, 0]`
(assuming indices from 0) is same as
`=[r-1,c,myfirstsheet]`

On the other hand, if it is the case that people almost never need relative sheet references, then could instead use sheen name in front as absolute address only:
`=myfirstsheet[r-1,c]`
or,again, just `[r-1,c]` if in same sheet as the formula cell.

--------------------------------

## Examples and how they compare to A1 syntax:

**1)** Fibonacci sequence in a column:

`0;1; =A6+A5`
vs:
`0;1; =[r-1]+[r-2]`
("current value is sum of the previous 2 values")

**2)** maximum of the previous 7 numbers in same row:

`=MAX(B7:E7)`
vs:
`=MAX([r, c-1]:[r, c-7])` or `=MAX([r, c-7 : c-1])`

**3)** you'll never again have to use INDIRECT(ADDRESS(...) constructs:

`=INDIRECT(ADDRESS(ROW(),RANDBETWEEN.NV(2,8)))`
vs
`=[r, RANDBETWEEN.NV(2,8) ]`

**4)** you probably will never have to use INDEX or OFFSET either:

The following is for selecting every 3rd number in a column B range in sheet1, assumming the result column is J in sheet2:
`=INDEX($sheet1.B$2:B$100, 3*(row()-row(J$2))+1,1)`
vs
`=[3*(r-r([2]))+2, c-8, 1]`

## Comparison with R1C1 style
The proposed syntax is in fact closer to R1C1.

Absolute-absolute:
`=R1C1` 
vs
`=[1,1]`

Absolute-relative mix
`=R1C[2]`
vs
`=[1,c+2]`

Relative-relative
`=R[-1]C[2]`
vs
`=[r-1,c+2]`


If it were just that, then I'd agree the proposal is is not worthy of the effort, although the new syntax is IMO a bit clearer and half of the time shorter as well.

But the MAIN change is to allow, for indices, any mathematical expression that results in an integer in valid index range. That is what would allow to do remove the need, at least in most cases, for any INDEX/INDIRECT/ADDRESS/OFFSET and probably other functions.

Classical R1C1 does not allow even : `RC[1+1]`

## Conclusion
IMHO, this way of relative addressing results in not only shorter, simpler formulas, but has the advantage that it matches the meaning of the dependency/relationship ("previous 2 values " etc) as opposed to making you stare at the row and column indices every time.

While at that, there is the opportunity to remove other syntax warts inherited from Excel and its granparents.
