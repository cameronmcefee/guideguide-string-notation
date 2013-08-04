# GuideGuide String Notation

```
| 10px | ~ | 10px | ( H, 100px )
| 10px | ~ | 10px | ( V, 100px )
```

GuideGuide is awesome, but due to the fact that its values come from static inputs, users are provided a limited number of options as to how they can create grids. With the GuideGuide form it is not possible to create grids with arbitrary grids elements such as sidebars.

This spec defines a written language called GuideGuide String Notation (GGSN) that can be used to "write" custom grids. GuideGuide itself will run using GGSN. The GuideGuide form will simply be a frontend that outputs GGSN to be parsed by the underlying logic.

Users that only want the basic GuideGuide features will likely never know GGSN exists. Power users will be able to vertically expand the GuideGuide window revealing a text field that will show the output GGSN from the form above. If the user would like to create their own custom grid, they can do so by editing the GGSN in the field.

## Grid

> \<hunks\> (\<options\>, \<grid width\>, \<grid offset\> )

A grid is a collection of gaps and guides (hunks) across a single dimentional
plane. GuideGuide will split the string into an array of hunks and iterate
through them, following them like instructions. Starting at 0, for each gap
hunk, GuideGuide will advance its insertion point by the value of the hunk.
When the current hunk is a guide hunk, GuideGuide will place a guide at the
current location of the insertion point. This will continue until all hunks
have been parsed.

GuideGuide will take into acount the size of the document or selection when
procecing percentages, wildcards, and fills. GuideGuide will not take the
document or selection into account when rendering the grid, therefore allowing
guides to be placed outside of it.

Each hunk is separated by a space character. Newlines are used to define
multiple grids in one string.

It is possible to change the way GuideGuide renders the grid by specifiying
options after the right most pipe character. A width for the grid can be
specified, as well as an offset to start rendering the grid. Whitespace in
options is ignored.


#### Examples

- """
  | ~ | (v)
  | ~ | (h)
  """  
  a guide at the top, right, bottom, and left of the document.

- `| ~ | ~ | ~ | ( v )`  
  a three row vertical grid

- `| 10px | ~ | 20px | ~ | 10px| ( h, 100px`  
  a one hundred pixel horizontal grid with a ten pixel left margin, ten pixel right margin, and a twenty pixel column centered in the middle

- `| 10px | 200px | 10px | ~ | ~ | ~ |`  
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

---

## Hunks

Hunks represent objects that define a cell or group of cells. An optional multiplier can be added to the end of the hunk to indicate that the hunk should be repated.

### Guide Hunks

> |

Guide hunks are represented by a pipe `|`. These hunks tell GuideGuide to place a guide at the current insertion point.

### Arbitrary hunks

> \<value\>\<unit\>*\<multiplier\>

Arbitrary hunks are simple gaps that are the width of the unit specified. Arbitrary can be positive or negative. Due to this, it is possible to traverse backwards and forwards 

#### Examples

- `| 10px | 10px | 10px|`  
  three ten pixel columns

- `| .5in | 1in | .5in|`  
  one half inch column, one inch column, one half inch column

### Wildcards

> ~

A wildcard hunk is represented by a tilde `~`. Any area within a grid that remains after all of the arbitrary hunks have been calculated will be evenly distributed between amongst the wildcards present in a grid.

#### Examples

- `| ~ |`  
  A guide on the left and right side of the document or selection.

- `| ~ | ~ | ~ |`  
  A three column grid

### Variables

Variables allow you to define and reuse collections of hunks within a grid. Variables are composed of a definition and a call.

#### Definition

> $\<id\> = \<hunks\>

A variable definition is represented by a dollar sign `$`, an optional id, an equals sign, and then a collection of hunks separated by spaces.

#### Call

> $\<id\>*\<multiplier\>

A variable call is represented by a dollar sign `$`, an optional id, and an optional multiplier. Anywhere a variable call occurs GuideGuide will replace its contents with the contents of its variable definition. A variable must be defined before it is called.

#### Example

"""
$ = ~ |
| $*2
"""

expands to:

"""
| ~ | ~ | ~ |
"""

a three column grid

### Multiples and fills

Arbitray, wildcard, and variable hunks can accept a final modifier that will duplicate that hunk the number of times specified. These are most helpful when used with variables, as it is possible to specify both gap and guide hunks together. Multiples and fills can be specified on gap hunks, but since the result of the mutiplied gap is not visible, their usefulness is rare.

#### Multiple

A multiple is represented by an asterisk `*` followed by a number. The hunk will be recreated sequentially the number of times specified by the multiple

- `10px*3`  
  Three ten pixel gaps

- """
  $ =  ~ | 10px |
  | $*2 ~ |
  """  
  A three column grid with ten pixel gutters

#### Fill

A fill is represented by a asterisk `*` folowed by nothing and is a hunk that will be recreated squentially until it fills the remaining space in the grid. This is useful for cases such as creating a baseline grid, or filling a space with as many columns and gutters of a certain width as will fit.

- """
  $ = 16px |
  | $* ( V )
  """  
  A sixteen pixel baseline grid

## Grid Options

Optional values to modify how the grid is created.

### Orientation

Determines the direction the grid will be rendered, whether horizontal or vertical. Orientation options are *not* case sensitve.

#### Values:

- `h` *(default)*  
  horizontal

- `v`  
  vertical

### Origin

Determines the position where GuideGuide renders the grid in cases where the grid specified is not as wide as the document or selection. Note the capital letters.

#### Values:

- `F`*(default)*  
  first (left/top)

- `C`  
  center

- `L`  
  last (right/bottom)

### Remainder pixel distribution

Determines to which columns GuideGuide adds remainder pixels when the columns do not divide equally into the total width of the grid area. Note the lowercase letters.

#### Values:

- `f`*(default)*  
  first (left/top)

- `c`  
  center

- `l`  
  last (right/bottom)

### Calculation

Determines whether GuideGuide is strict about integers when calculating pixels

#### Values:

- `n` *(default)*  
  normal

- `p`  
  pixel specific

### Grid width

Optional unit object that specifies the width of the grid area to be used for the calculation. Must be a positive value. The width option can be left blank.

#### Examples:

- `| ~ | ~ | ~ | ( h, 100px )`  
  A three column grid that is one hundred pixels wide.

### Grid offset

Optional unit object that specifies how far from the origin the grid will be offset. When the origin of a grid is set to `center` the offset property will be ignored.

#### Examples

- `| 10px | ( hF, , 50px )`  
  A ten pixel column that sits 50px from the left side of the document/selection

- `| 10px | ( hL, , 50px )`  
  A ten pixel column that sits 50px from the right site of the doucment/selection

- `| 10px | ( hL, 100px, 30px)`  
  A ten pixel wide column that sits 30px from the right side of a 100px selection.

## Errors

GuideGuide String Notation errors will be denoted in curly brackets. Directly following a bracketed error will be a set of brackets containing a comma separated list of error IDs. Explanation of the errors will be printed below the grid.


### Hunk errors

```
| 10px | { 10foo [1]} | 10px|

# 1. Unrecognzed unit type
```
