# GuideGuide String Notation

```
|<10px, $*3, >10px|H100px
|<10px, $*3, >10px|V100px
```

GuideGuide is awesome, but due to the fact that its values come from static inputs, users are provided a limited number of options as to how they can create grids. With the GuideGuide form it is not possible to create grids with arbitrary such as sidebars.

This spec defines a written language called GuideGuide String Notation (GGSN) that can be used to "write" custom grids. GuideGuide itself will run using GGSN. The GuideGuide form will simply be a frontend that outputs GGSN to be parsed by the underlying logic.

Users that only want the basic GuideGuide features will likely never know GGSN exists. Power users will be able to vertically expand the GuideGuide window revealing a text field that will show the output GGSN from the form above. If the user would like to create their own custom grid, they can do so by editing the GGSN in the field.

## Grid

> |\<hunks\>|\<options\>\<grid width\>

A grid is a collection of cells (hunks) across a single dimentional plane. The format for a grid is bases loosely on that of regular expressions. A grid is a collection of hunk objects bookended by pipe `|` characters. It is possible to change the way GuideGuide renders the grid by specifiying options after the right most pipe character. A width for the grid can be specified as a single unit object to the right of the options. Each hunk declaration is separated by a comma. Whitespace is ignored. Newlines are used to define multiple grids in one string.

In cases where the user does not need to specify options for the grid, the pipes and options can be omitted.


#### Examples

- `|$*3|V`  
  a three row vertical grid

- `|<10px, 20px*3, >10px|HC100px`  
  a one hundred pixel horizontal grid with a ten pixel left margin, ten pixel right margin, and three twenty pixel columns centered in the middle

- `10px, 200px, 10px, $*5`  
  a grid with a left side bar with 10px on either side, and a five columns filling the gap.



## Unit objects

> \<value\>\<unit\>

Unit objects are value-unit pairs that indicate a measurement.

#### Examples

- `72px`
- `1in`
- `2.54cm`
- `25.4mm`
- `72pts`
- `6pica`
- `100%`

## Hunks

Hunks represent objects that define a cell or group of cells. An optional multiplier can be added to the end of the hunk to indicate that the hunk should be repated.

### Arbitrary hunks

> \<value\>\<unit\>*\<multiplier\>

Arbitrary hunks are simple grid cells that are the width of the unit specified. Arbitrary hunks must have a positive value.

#### Examples

- `|10px, 10px, 10px|`  
  three ten pixel columns

- `|.5in, 1in, .5in|`  
  one half inch column, one inch column, one half inch column

### Margins

> \<side\>\<value\>\<unit\>*\<multiplier\>

Margins are hunks that attach to the specified side of the grid area. While it's possible to delcare margins anywhere in the grid declaration, it's less confusing if they are declared on the side of the declaration that they represent. Margins are additive from the edge of the grid inward. For example a value such as `<10px <-10px` would result in only a ten pixel margin, as the second margin returns to 0. Negative values can be used to place the margins outside the grid area.

#### Values:

- `<`  
  first margin (left/top)

- `>`  
  last margin (right/bottom)

#### Examples

- `|<10px, >10px|`  
  ten pixel left margin, ten pixel right margin
  
- `|<10px, <10px, >10px, >10px|`  
  two ten pixel left margins, two ten pixel right margins

### Wildcards

> $\<id\>{\<hunks\>}\*\<multiplier\>

A wildcard is a way to define variables within a grid. When specified in its simplest form `$` the value remaing after all defined grid hunks have been applied to the grid area will be distributed evenly between the wildcards. If the grid is rendered as pixel specific, the value will be spread evently across the wildcards, with the remaining pixels being distributed amongst the grid based on GuideGuide's pixel remainder settings. Values must be positive.

#### Examples

- `| $, $, $ |`  
  a three column grid


In cases where a user would like to define a repeating collection of hunks, a wildcard combined with an id and curly brackets can be used. The wildcard's hunks should be declared in the first instance of the hunk. In cases where only a single wildcard of an id is declared, GuideGuide will attempt to repeat the wildcard as many consecutive times as will fit in the given area, after all other calculations are made.

*Note:* I haven't yet figured out how to handle cases of multiple wildcard ids. This portion is still in flux.

#### Examples

- `|${100px}|`  
  as many one hundred pixel columns as will fit

- `|$, $G{20px}, $, $G, $|`  
  three colums with twenty pixel gutters
  
- `|$B{10px $ 10px}, 20px, $B, 20px, $B|`  
  three columns, ten pixel column padding, and twenty pixel gutters 

### Multiples and fills

Arbitray and wildcard hunks can accept a final modifier that determines how many of said hunk exist.

#### Multiple

A multiple is represented by an asterisk `*` followed by a number. The hunk will be recreated sequentially the number of times specified by the multiple

- `|<10px*3|`  
  Three ten pixel right margins

- `|$*3|`  
  A three column grid

- `|$A{ $, 10px }*2, $|`
  A three column grid with ten pixel gutters

#### Fill

A fill is represented by a plus `+` and is a hunk that will be recreated squentially until it fills the remaining space in the grid. This is useful for cases such as creating a baseline grid, or filling a space with as many columns and gutters of a width as will fit.

- `|16px+|V`  
  A sixteen pixel baseline grid

- `|<100px, 16px+, >100px|V`  
  A one hundred pixel header, a sixteen pixel baseline grid, and a one hundred pixel footer

- `|${ 100px, 10px }+ 100px|H`  
  As many one hundred pixel columns with 10 pixel gutters as will fit in the area given.

## Grid Options

Optional values to modify how the grid is created. 

### Orientation

Determines the direction the grid will be rendered, whether horizontal or vertical.

#### Values:

- `H` *(default)*  
  horizontal

- `V`  
  vertical

### Position

Determines the position where GuideGuide renders the grid in cases where the grid specified does not fill the available area. GuideGuide factors this available area as the remainder of *area - margins*.

#### Values:

- `F`*(default)*  
  first

- `C`  
  center

- `L`  
  last

### Calculation

Determines whether GuideGuide is strict about integers when calculating pixels

#### Values:

- `N` *(default)*  
  normal

- `P`  
  pixel specific

### Grid width

A unit object that can specify the width of the grid area to be used for the calculation.
Must be a positive value.

#### Examples:

- `| $*3 |100px`  
  A three column grid that is one hundred pixels wide.

## Errors

GuideGuide String Notation errors will be denoted in brackets. Directly following a bracketed error will be a set of parens containing a comma separated list of error IDs. Explanation of the errors will be printed below the grid.


### Hunk errors

```
|10px, [10foo(1)], 10px|

(1) Unrecognzed unit type
```

### Param errors

```
|$*3|N[E(1)]

(1) Unrecognized option
```

### Grid Errors
```
[|$*3NL10px(1)]

(Improperly formatted grid)
```
