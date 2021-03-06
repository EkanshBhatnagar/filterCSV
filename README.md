# filterCSV

iThoughts is a third-party application for creating and managing mind maps. It runs on iOS, iPad OS, Mac OS and Windows.

You can create a mind map either in the application itself or by importing files in a number of other formats. The most complete format is Comma-Separated Value (CSV). Being a text format, CSV can be programmatically created in a number of programming languages, such as Python.

The CSV format that iThoughts understands has a tree-like structure. As well as the tree of nodes, colour, position, node shape and other attributes of a node can be specified in the CSV file. To a very limited extent the format is documented [here](https://www.toketaware.com/ithoughts-howto-csv). A better way to understand the format is to export a mind map from iThoughts as CSV and look at the resulting file.

## About filterCSV

filterCSV is a set of tools to automatically edit a CSV file in the form used in iThoughts. filterCSV is written in Python 3.6+. It has been tested on a Raspberry Pi and a machine running macOS.

Based on matching regular expressions you can do things such as:

* Set colours for nodes
* Change their shape
* Delete them
* Set their positions

You can check the structure of the input CSV file is good for importing into iThoughts.

You can export the CSV file as a Markdown file consisting of headings and bulleted lists.

**NOTE:** In this document we will use terms such as "mind map" and "tree". Structurally the data represents a tree. \
It might or might not be used for mapping your mind.

## Using filterCSV

filterCSV reads from stdin and writes to stdout, with messages (including error messages) to stderr. For example:

    filterCSV '^A1$' 'triangle' < input.csv > output.csv


Command line parameters are pairs of:

1. A specifier. This is a regular expression to match. (A special value `all` matches any value)
1. An action or sequence of actions. 

### Specifiers

Specifiers are regular expressions in a format the Python `re` module understands.

A special value `all` matches all nodes.

If you want to match a cell's text exactly you can code something like `^A1$` where `^` means 'the start of the text' and `$` means 'the end of the text'.

### Actions

Actions you can take include:

* Specify a colour number
* Specify a colour RGB value
* `delete`
* `keep`
* Specify a shape
* Specify a position
* Promote all subtrees at a certain level by 1 level
* Computing basic statistics about the mind map

In the following action specifications are case-insensitive; If you specify, for example, an action in upper case it will be converted to lower case before being applied to matching nodes.

#### Colour Numbers

A colour number is a 1- or 2-digit number. It is specified relative to the top left of iThoughts' colour palette. (`1` is the first colour in the palette.)

You can also specify `nextcolour`, `nextcolor` or even `nc` and filterCSV will select the next colour in iThoughts' colour palette. `samecolour`, `samecolor` or `sc` can be used to specify the same colour again.

#### Colour RGB Values

This is a hexadecimal 6-character representation of the colour, in Red-Green-Blue (RGB) format. For example `FFAAFF`.

#### Delete

`delete` deletes the matching node and all its children.

####  Keep

`keep` retains the matching node, all of its children, and its parent, grandparent, great-grandparent, etc. The idea is to retain a workable tree.

For example

    filterCSV 'EXCPs' keep < input.csv > output.csv

would retain any nodes which match the string "EXCPs", and all the nodes below them. In addition, to ensure the tree remained valid (for import into iThoughts) any nodes leading from the root (level 0) to the matching nodes would be retained.

**Note:** You can use regular expression alternation to keep multiple subtrees. For example:

    filterCSV 'A1|X' keep < input.csv > output.csv

where `|` means either the term to the left (`A1`) or the term to the right (`X`) can be used to match.

If you use `keep` in a filterCSV action you can't use anything else. For example, you can't use `triangle`. You can use another specifier, perhaps `all`with `triangle` to get the same effect.

#### Shapes

Specify a shape as named by iThoughts.

Currently the shapes are `auto`,`rectangle`,`square`,`rounded`,`pill`,
`parallelogram`,`diamond`,`triangle`,`oval`,`circle`,
`underline`,`none`,`square bracket`,`curved bracket`.

You can also specify `nextshape`, or `ns` and filterCSV will select the next shape in iThoughts' set of shapes. `sameshape` or `ss` can be used to specify the same shape again.

#### Positions

Positions are specified in the form `{x,y}` where the braces are necessary.

At present setting the position only seems to work for Level 0 (root) nodes. You can have as many Level 0 nodes as you like.

#### Removing Notes, Shapes, Colours, And Positions

If you specify `noshape`, `nocolour`, `nonote` or `noposition` the corresponding attribute is removed from matching nodes.

Most usefully you could specify this with a match condition of `all` to reset an entire column. For example, `nonote` could clear all the notes from a mind map - to prepare it for exporting from iThoughts.

#### Eliminating A Level

Suppose you have some nodes at level 1 and you want to make them all level 0, retaining their subtrees. With `promote` you can do this.

If you specify, for example

    filterCSV promote 2 < input.csv > output.csv

The nodes at level 1 are deleted and all their direct children move up to level 1. These children might be the root of subtrees. All nodes in the subtrees are also promoted by 1 level.

####  Computing Statistics About A Mind Map

If you specify `stats` it will write basic statistics to an output file in one of the following forms:

* Flat file - if you specify `stats text`
* HTML table - if you specify `stats html`
* Markdown table - if you specify `stats markdown`
* Comma-Separated Value (CSV) - if you specify `stats csv`

The statistics are (for each level):

* Number of nodes at that level
* Number of distinct text values at that level

Here is an example - produced by specifying `stats text`:

    Level Nodes Distinct Nodes
        0     2              1
        1     2              2
        2     1              1
        3     1              1

####  Merging Nodes Into Their Parent node

You can merge a matching node into its parent as a bullet.

To do this specify `asbullet`. For example

    filterCSV ^A1$ asbullet < input.csv > output.csv

will merge any bullet whose text or note is 'A1' with its parent. The text of the note will be merged in, with the two characters '* ' denoting it's a bulleted item.

### Input Files

Input files can be in one of four formats:

* A CSV file that is already in a format supported by iThoughts' Import function.
* A flat file where each line is a new node. Spaces and tabs can be used to indent the text. Here the level of indentation is used to control what level the line is added at.
* A Markdown nested list where each line is a new node. Spaces and tabs can be used to indent the text. Here the level of indentation is used to control what level the line is added at. Only an asterisk (`*`) followed by a space is supported as a list item marker.
* An XML file, including one with namespaces (both default and named).

#### Nesting Level Detection

When parsing a non-CSV file the first line with leading white space (spaces and/or tabs) is used to detect the indentation scheme: Any white space on this line is used as a single indentation level marker. It is expected that all lines are indented in the same way.

For example, if the second line starts with two spaces this is taken to indicate that every line will have zero, two, four, etc spaces:

* Zero spaces denotes a level 0 node
* Two spaces denotes a level 1 node
* Four spaces denotes a level 2 node
* And so on

For an XML input file the nesting level is in the data stream; Elements' children are at a deeper nesting level. \
On import child nodes are created for the element's value (if it has one) and any attributes. \
These are colour coded and free-standing nodes marked "Element", "Value", and "Attribute" are added as a legend.

#### Checking

If the input data is not a CSV file filterCSV performs two checks on it:

* If the indentation of an individual line contains a number of whitespace characters that is not a multiple of the first indented line's leading whitespace diagnostic messages are produced. This line's indentation level is rounded down.
* If the indentation of an individual line contains a sequence that is not the first indented line's leading whitespace (perhaps repeated) diagnostic messages are produced.

These checks are intended to help debug an indentation problem.

If you specify `check` - in place of a regular specifier - filterCSV will check the level column of the data. It will detect bad levels in the following way: If the level of a node is greater than one more than the parent node it will consider this to be an error - which will be reported.

If a level error is detected one of three things can happen:

* If the action is specified as `repair` or `repairnode`, filterCSV will set the node's level to 1 more than its parent's.<br/><br/>
This is good for the case where you want all the gory details of badly leveled nodes.
* If the action is specified as `repairsubtree`, filterCSV will set the node's level to 1 more than its parent's and then adjust the subtree below it in a similar fashion.<br/><br/>
This is good for the case where you want the tree repaired with a minimum of fuss. But you still get informed when repairs were necessary.

* If the action is `stop`, filterCSV will terminate.

As an example, you might code

    filterCSV check repair < input.csv > output.csv

#### Spreading Out Level 0 (Root) Nodes

If you import a CSV file without specifying positions in the file iThoughts will place all the Level 0 (root) nodes on top of each other. \
This is probably not what you want. \
filterCSV can spread out the Level 0 nodes - either horizontally or vertically.

For example, if you specify `vspread 500` filterCSV will set the positions of the Level 0 nodes 500 units apart - one above the other.

For example, if you specify  `hspread 1000` filterCSV will set the positions of the Level 0 nodes 500 units apart - spaced out horizontally.

In both cases the children will be arranged as normal, relative to the root nodes.

vspread and hspread set the values for these nodes in the "position" column in the CSV file.\
Their format is of the form "{1000,0}". \
(In this example "1000" is the horizontal offset and "0" is the vertical offset.) \
If you specify vspread or hspread they will overwrite all Level 0 nodes' positions.

### Output Formats

While generally you would write a CSV file for importing into iThoughts, you can also export the data to:

* A Markdown file
* A HTML table
* A HTML nested list
* A Freemind mind map
* An OPML outline

#### Markdown Output

A CSV file can be exported to Markdown with each level rendered according to the following rules:

* The first few levels are rendered as headings. For example the first level (level 0) might be rendered as `#` to denote "heading level 1".
* Subsequent levels are rendered as nested bullets, with `*` as the indicator for a list item. Each level is indented by pairs of spaces.

You can specify Markdown output by invocations such as

    filterCSV markdown 3 < myfile.csv > myfile.md

or

    filterCSV markdown '2 3` < myfile.csv > myfile.md

In the first case three levels of heading are required, starting with heading level 1. (Heading level 1 is the default).

In the second case two levels of heading are required, starting with heading level 3.

#### HTML Output

You can export to HTML as either a nested list or a table. Colour is preserved on output, as are notes.

You can specify HTML table output by invocations such as

    filterCSV html table < myfile.csv > myfile.md

You can specify HTML nested list output by invocations such as

    filterCSV html list < myfile.csv > myfile.md

#### Freemind And OPML XML Output

filterCSV can output to Freemind XML format, including notes and colours:

    filterCSV xml freemind < myfile.csv > myfile.mm

filterCSV can output to OPML XML format, but support for notes and colours by other programs is mixed. For example Omnifocus will create a custom "Notes" column:

    filterCSV xml opml < myfile.csv > myfile.opml

## Test Files

[tests/README.md](./tests/README.md) describes test files that you can study to become familiar with filterCSV.
