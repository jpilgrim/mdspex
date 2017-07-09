# Disclaimer

This is WORK IN PROGRESS!

This document specified the markdown language MDSpex with default extensions. The MDSpex processor is currently under developed and not done yet.
The features described in this document link to github issues at https://github.com/jpilgrim/mdspex/issues . These issues may be not closed yet,
meaning that the corresponding features are not implemented yet.

# Overview

MDSpex is an extensible markdown language and processor written in N4JS/JavaScript.
The goal of MDSpex is to provide an unicode based language which is
as readable as *markdown* (and compatible with [https://github.github.com/gfm|GitHub Flavored Markdown])
and powerful enough to enable writing technical *specifications*,
in particular for software projects. If possible, extensions are to be designed compatible with
[http://asciidoctor.org/docs/asciidoc-writers-guide|Asciidoc] .
For the target document type, it comes with a set of predefined extensions.
For mathematical equations, Tex-like syntax is supported.

The output format of MDSpex documents is created by output processors. 
Although different output formats may be produced by different processors, 
the focus is on a single output processor creating high-quality printable
HTML, using CSS and JavaScript (for screen reading).

It consists of a very small core processor which can parse block structures. 
This processor can be extended by so called extensions. 
Thus there is no clearly predefined syntax. 
However all extensions should follow similar syntax patterns.

An important feature of documents is linking. 
A reference links to a named element or a target outside the document.
Elements can be nested and typed and they are referred to by name or ID. 
MDSpex supports the notion scoping and fully (or partially) qualified names.

The following snippet shows a simple MDSpex example using pre-defined extensions and links:

```mdspex
# Main

## Intro
This is an example to illustrate
* basic formattings (headings, lists etc.)
* how commands can be used
* how linking is working

## More
As described in [Intro] this is an example. The main class is >Application, it contains a method
>main. The algorithm is described in [Knuth1969]. [RPRJ-123] defines a requirement.

REQ RPRJ-123 (Version 1): Some Requirement
	TASK:PRJ-1001
	
	This is a requirement for a project. It has a defined ID (for referencing) and is related to a task
	of some task tracker [1].
	
	FOOTNOTE 1: That may be Github or Jira for example.
```


# Character Codes

Similar to Tex, MDSpex is based on the notion of characters with different categories. The following character 
categories are the most important once:

	LETTER: a-z, A-Z, umlauts etc. 
		Spec supports unicode, and all characters with a unicode >255 are interpreted as letters by default.
	OTHER: Digits, punctuation and other special characters
	CMD: A command is a named extension. It must consist of a sequence of LETTER characters (the name of the command), 
		followed by a colon ':' by default. Parameters may be provided inside parenthesis. 
		A command is only recognized if an extension registered the command with the given name.
	CTRL: A control sequence is an extension identified by OTHER characters. Similar to commands an extension
		must register the control sequence. Its concrete syntax then depends on the extension. 	 
	ESC: '^' by default is used to enforce the following character to be either a LETTER or OTHER, but not
		a command or ctrl sequence. 
	EOL: End of line, '\n' by default, '\r' is ignored (but added to EOL token)
	INDENT: Tabs and four spaces at beginning of line are interpreted as indents. The number of indents
		defines the indent level of a block.
	SPACE: Space character including tabs if not at beginning of line, does not contain not newline characters!

The following categories are for internal or special use, these characters cannot be occur in a document directly:
	
	IGNORE: Ignore, character 0
	INVALID: Invalid character
	
For mathematical equations, a subset of Tex is supported. Since this is done similar to Tex, the following
categories known from Tex are used:	
	
	TEX_ESC: Tex: Escape character in Tex, '\' by default.
	TEX_BEGINGROUP:Tex: Beginning of group {
	TEX_ENDGROUP: Tex: End of group }
	TEX_PARAM: Tex: Parameter character #
	TEX_SUPER: Tex: Superscript,  '^' by default 
	TEX_SUB: Tex: Subscript '_' by default

These Tex categories are only recognized in Tex blocks, though.

# Syntax notation

In the following, we use an EBNF to express the syntax rules. Due to the nature of markdown, 
these rules are pragmatically extended to allow for defining more flexible behavior. In general,
the rules are more pseudo-rules for describing the syntax, additional constraints may alter the behavior.

We write additional constraints inside guillements (french quotation marks), including pseudo code or
if certain pre- or suffixes are required (but not consumed).

The result of a rule reference may be assigned to a value. 

We use upper case names for terminal tokens (identified by the lexer), and lower case rules for non-terminals
identified by the parser. In many cases, the number of elements is important. We use regular-expression like 
syntax to restrict the number of repetitions. Additionally, elements can be assigned to variables to which we refer
the rule, and we use the prefix '#' to refer to the number of repetitions.

The rules use the [Character Codes] as terminal tokens, and we define the following helper rules:

EBNF:
	NONSPACE: 	LETTER | OTHER | CMD | CTRL | ESC;
	NONEOL:		NONSPACE | SPACE;
	EMPTYLINE: 	EOL INDENT* SPACE* EOL	
	.:			«any character»
	DIGIT:		[0-9]
	
For the sake if simplicity we ignore escaped characters. An escaped character is treated as LETTER 
unless otherwise specified.	

# Document Structure

A MDSpex document consists of a elements.
An element has a name and may has attributes and may contain other elements. We call the contained elements children and the
container parent. Attributes have a name and a value.

Blocks are elements usually separated by empty lines. Actually it is the output processor that
defined which element is rendered as a block or not, but usually this difference is already reflected in
the MDSpex source code. Children are usually visualized by indentation.

The core processor creates only two kind of elements:

	Paragraphs: Paragraphs have the same indentation level as their parent, separated by empty lines.
	Code Blocks: Code blocks have a greater indention level as their parent, they are also separated by empty lines.
		Inside code blocks, extensions and further formatting is disabled except for certain features described below.   
 
Other kind of elements are created by extensions. Note that neither the name of the element nor
the name of the attributes (or its values) have to explicitly exist in the source code.
Instead extensions may derive these values from special syntax or the context in general. 
 
The default document structure can be defined as follows:
 
REQ SPEX-101: Core document structure
	TASK: #1, #13
	EBNF:
		document creates root element "mdspex": 		
						blocks«indent=0»;
		blocks«indent»: EMPTYLINE* block«indent» (EMPTYLINE+ block«indent»)* EMPTYLINE*;
		block«indent»:	paragraph«indent» | codeblock«indent» | extBlock«indent»;
		paragraph«indent» creates "p":
						INDENT{indent} content 
						(EOL INDENT{indent} content)*;
		codeblock«indent»: «disable most extensions»
						INDENT{indent} INDENT content
						(EOL+ INDENT{indent+1} content)*;
	
		content:		(extInline|NONSPACE) ( extInline|NONEOL )*;
		  	
In these rules we introduces two non-terminals: `extBlock`and `extInline`.
This refers to extensions in general as specified in the next section.

# Extensions

Extensions are additions to the MDSpex system, written in JavaScript (or N4JS). The need to 
implement the interface [src:Extension].

Extensions identified by [Character Codes/LETTER] tokens are called commands
and the lexer produces [Character Codes/CMD] tokens.

Extensions started by a [Character Codes/OTHER] tokens are called controls 
and the lexer produces [Character Codes/CTRL] tokens. Although these extensions must 
start with a OTHER character, the extension may use arbitrary patterns to identify the start of the 
extension. The sequence of characters identified by the extension is called the control sequence, and this
is the token recognized by the lexer and returned as value of the CTRL token.

When an extension is registered, is must have a name even if it is a control. We use these names to identify 
the extension in this specification. The name of the element the extension creates is usually but necessarily 
similar to its name. 

Besides the syntax pattern, extension can define arbitrary constraints.

SAMPLE: 
	The heading extension is a control which implicitly defined the heading level as follows:

		heading«indent»: «indent=0» level=#('='+) SPACE title=NONEOL* EOL;
		
	Besides the dynamic control sequence, it is restricted to indent level 0 and it ends with a single EOL.	
		 
Due to this flexible definition of extensions it is impossible to define general EBNF rules. We will 
discuss extensions and the default syntax of extensions in the following.
  
An extensions defining a special block are called block extensions. It must start at the beginning of the line after the indentation.
There may be block commands or block controls. 
Extensions decorating characters or appearing inline otherwise are called inline extensions. 
There may be inline commands or inline controls.

The following rules summarize these different kind of extension elements:

REQ SPEX-201: Basic Extension Syntax
	TASK:#1
	EBNF:
		extBlock«indent»:	cmdBlock«indent» | ctrlBlock«indent»;
		cmdBlock«indent»: «INDENT» name=LETTER* «extension specific content» ':' «extension specific content» 
		ctrlBlock«indent» «INDENT» control=(OTHER «extension specific characters») «extension specific content»
		
		extInline:	cmdInline | ctrlInline;
		cmdInline:	name=LETTER* «extension specific content» ':' «extension specific content»
		cmdInline:	control=(OTHER «extension specific characters») «extension specific content»

## Standard Attribute Definitions

It is recommended to use the following rules for attributes in general:

EBNF: 
	attributes:  '(' (attribute (',' attribute)* )* ')';
	attribute: (name=LETTER* '='?)? value=value;
	value:	(LETTER|OTHER) | '"' [^"]* '"';

The name of attributes is optional, quite often extensions do have only one or two attributes and in these
case the position should be sufficient.

Quite often, an extension defines a special attribute `id` used for linking. The following rules
are predefined for id attribute values:

EBNF:
	idvalue:  	digitid | prefixid | otherid;
	digitid: 	[0-9]*;
	prefixid:	LETTER+ '-' [0-9]*;
	otherid: 	value
	
Digit or prefixed ids are recognized by the linking extension and enable very short references.	

## Standard Command Format

Standard block commands use a fixed syntax pattern as follows:

REQ SPEX-1201 Standard Block Command 
	TASK:#12

	EBNF:
		stdBlock«indent» creates name-element: 	
			«INDENT»
				name=LETTER«uppercase» 
				«if supportsID» idvalue? «endif» 
				«if hasAttributes» attributes «endif»
				':' 
				«if hasTitle» 
					title=NONEOL*
					EOL
				«else» 
					^EOL «creates P» 
						content 
						(EOL INDENT{indent+1} content)*
				«endif»
				blocks«indent+1»*
		;
	
This standard block command starts with the optional id and attributes, followed by the name of the
extension in upper case letters and a colon. If the block has a title, then this is the text
in following the colon. Otherwise, the text after the colon is the first paragraph of the element.
		
SAMPLE:
	The following snippet shows a requirement block with an prefix id, a version and a title:
	
	```mdspex
	This is normal text.
	
	REQ SPEX-123 (version 1): Some Requirement
		This is a paragraph of a requirement.
		
		Second requirement paragraph
	
	This is normal text.
	```
	   
Extensions may change this default behavior, e.g., modifies the indent level of the block.

# Macros

Macros are similar to extensions except that they are written using the MDSpex extension block `SPEX_MACRO`.

TODO: Specify macros, possible macros: alias, rename, block


# Linking

MDSpex provides a powerful linking mechanism. The general rational behind the linking algorithms is to allow for specifying links as short as possible to write, and to reduce the need of explicitly defining any anchors.


## Anchors and Scopes

Linking consists of two parts: an anchor and a link. 
Anchors are implicitly defined by certain elements. 
They must have an ID, which may be derived from other attributes, e.g. the CDATA of a section (with certain normalizations). 
For linking it makes no difference how the anchor is defined. 
The extension creating the anchor also defines the type of an anchor and it is responsible for deriving the id.

In general if an element defines an ID, an anchor is created for that element.
In contrast to HTML anchors and element IDs, anchors in MDSpex are hierarchically scoped. Every anchor can define a scope itself, causing children of the element or, depending on the extension defining the element and the anchor, succeeding anchors (of succeeding elements) may become sub-scopes.

That leads to the notion of fully qualified IDs of an anchor.
	
REQ SPEX-1001: Syntax of FQIDs and PQIDs	
	TASK: #10
	The syntax of fqids, pqids and links is defined as follows:
	
	EBNF:
		idsymbol«allowWildcard»: LETTER|DIGIT|'_'|OTHER «including '…' if allowWildcard»«endif»
		segment«allowWildcard»: idsymbol«allowWildcard»*;

		qid: fqid|pqid;
		fqid: type ':' segment«false» (SEP segment«false»)*;
		pqid: [type ':'] (segment«true» (SEP segment«true»)*))?;
		
		SEP: '/' | '.'; // '.' is used if no '/' is found, only one character can be used in one qid

REQ SPEX-1008: FQIDs of Anchors
	TASK: #10
	
	The fully qualified id (fqid) of an anchor is the concatenation of the simple id of the anchor's element, preceded by the fqid of the element (or respective anchor) defining the scope if such a scope exists. The type of the fqid is derived from the element type by the extension.

## Links

A link to an element is done via a so called partially qualified id (pqid). This may be similar to the fqid, or a shorter version. That is, the ids of outer segments may be omitted. It is even possible to use wildcards.

Since anchors are scopes it is usually not required for anchor 
names to be globally unique. However certain extensions may add this constraint for
enabling indexes and easier linking.

NOTE: 
	By default, the HTML element created for an MDSpex element will have the id attribute set for actual linking. Its value will be the string representation of the FQID. Accordingly HTML links will have to use the FQID of an anchor. This may be changed by extensions though. In particular extensions enforcing document wide unique (simple) ids may alter this default behavior in order to enable shorter (HTML) links. However linking to MDSpex created HTML elements is problematic anyway, since the final document may be split-up into several documents automatically. Moving a linked element from one section to another might then lead to moving the HTML element to another file, breaking existing links from the outside to that element even if the id attribute of that element has not been changed at all.

Note that although segments can be almost an character, extensions computing the id may restrict the name further. This is true for the standard heading extension as described below.

A default extension is provided for defining links with the following syntax:

REQ SPEX-1002: Default link syntax
	TASK: #10

	EBNF:
		link: prefixLink | braketLink;
		prefixLink: ('>>' | ^) pqid;
		braketLink: '[' pqid ']'

REQ SPEX-1003: Qualified Name Matching
	TASK: #10

	A FQID typeF:segFn...segF1 matches a PQID typeP:segPk..segP1 if and only iff
	- if typeP is specified, then typeF must exactly equals typeP
	- n>=k, that is, the FQID must have at least as much segments as the PQID
	- for all segFi, segPi, i=0..k the segments match. That is, segFi must match
		a regular rexpression Ri derived from segPi, in which all wildcards "…" are
		replaced with ".*" expressions.

REQ SPEX-1004: Linking Algorithm
	TASK: #10

	The linking mechanisms works as follows:
	1. The scope hierarchy (from link location up to document root) is used
		to locally resolve a link.
	2. If no anchor is found in the scope hierarchy, 
		the link is globally resolved.
	
	If not otherwise specified, the first match is used in case several anchors match a PQID.
		
REQ SPEX-1005: Default anchor ID normalization
	Task: #10

	The default anchor ID normalization used for all anchors of AST elements converts any given string into an anchor ID with
	the following properties:
	- all spaces, punctuation and other symbols are converted to underscores "_"
	- trailing and leading underscores are omitted
	- multiple succeeding underscores are merged to a single underscore

The default header extension defines an anchor of type 'sec' (for section), ignoring the heading level. The anchor is is derived from the heading title using ^SPEC-1005.

Note that in general, anchor ids may contain arbitrary characters including spaces. A  normalized ID does not contain spaces. This allows for using the prefix link syntax.


SAMPLE 1: Given the following snippet:
	```
	# Big Equations
	## Pythagoras
	EQ 1: $a^2 + b^2 = c^2$
	Equation [eq:1] is known from geometry.
	## Einstein
	EQ 1: $e = m * c^2$
	Equation [1] is known from physics. It also contains a $c^2$ as ^Pyth….1.
	```

	This examples defines 5 anchors, we list the fqids of here. Sections also defines scopes:
	- sec:Big_Equations -- with scope
	- sec:Big_Equations/Pyhtagoras -- with scope
	- eq:Big_Equations/Pyhtagoras/1
	- sec:Big_Equations/Einstein -- with scope
	- eq:Big_Equations/Einstein/1

	Rhe links are resolved as follows:
	- `[eq:1]` is resolved locally to the equation defined above that line.
	- `[1]` is also resolved locally, but to the second equation. 
		Since no other anchor with that id is defined, no type is required for linking.
	- `^Pyth….1` is resolved globally: 
		'Pyth….1' matches 'eq:Big_Equations/Pyhtagoras/1' since the last segments are similar, the second segment is matched by wildcard, and the first segment 'Big_Equations' can be omitted in the pqid. Also the type is not required in that case. Since the heading extension removes spaces, it is possible to use the prefix link syntax.

The text and layout of the links in the final text is created by extensions registered to handle certain types of links.


## Dictionaries and External Links

So far all anchors were defined in MDSpex documents directly. Often it is required to link to elements "outside" the document.

The most obvious case is to link to arbitrary URLs. 
This is simply possible by using generic references and "http" (or "https") as element type.

These links directly link to the URL defined by the reference. In other words: The reference needs to be fully qualified.

In many cases it is convenient to use pqids for linking. This is true in particular for source code elements. MDSpex supports dictionaries for
enabling pqids to arbitrary elements outside the MDSpex document. 

In case of internal links, a link has a single purpose: The reader of the documents wants to navigate to the element defining the anchor.
Thus, most internal links simply navigate to the anchor.

In case of external links, there may be multiple targets: In the case of source code elements for instance one may want to navigate to the
source code located in a code repository, to the API documentation of an element, or to the local source code location (e.g., to be shown in an IDE).

TODO: Dictionaries similar to AsciiSpec linking


# Counters

MDSpex supports counters. A counter is a named integer variable. An extension may assign the current value of a named counter to an element's attribute,
incrementing the counter automatically or reset a counter. The value of the counter can be used to compute numbers of list items, headings, 
or to make unique IDs (or names) of elements. The counters are used by the extensions internally, see API for details.
 

# Output

MDSpex focuses on high-quality HTML output (with JavaScript for screen and special formatting for print). The parser initially creates a simple tree
with elements named according to the names of the extensions (or the elements created as specified). Unless specified otherwise, these elements are
all converted to HTML `div` elements with the element type as class name. If the element defines an ID explicitly, it is set as ID. Otherwise an ID
is computed from the title.

TODO: how to compute titles.

# Built-In Extensions

## Headings

MDSpex supports so-called ATX headings, using '#' as control sequence. The number of '#' characters defines the level of the heading.

REQ SPEX-601: ATX Headings
	TASK:#6

	EBNF: ATX-Heading Extension
		headingATX := ^ level=#'#' space title EOL

REQ SPEX-1006: Heading Anchors
	Task: #10

	For each heading an anchor is automatically created. Its id is the title of
	the anchor in normalized form (cf. ^SPEC-1005).

	The type of headings is "sec", independent of their level.

	A heading H with level n defines a scope for all succeeding elements E as follows:
	- if E is a heading itself with level k:
		- if k<=n, the scope of H ends before element E
		- if k>n, the scope defined by E becomes a sub-scope of H's scope
	- if E is an element defining a scope itself, 
		then this scope becomes a sub-scope of H's scope 
		unless otherwise specified

REQ SPEX-1007: Heading Links
	Task: #10

	The link text to a heading is its title.

## Lists

There are three kind of lists supported by default: unordered lists, ordered list and description lists.

The unordered list extension uses '*' or '-' as control sequences. It detects list items via that pattern and automatically creates surrounding list elements:

REQ SPEX-701: Unordered Lists
	TASK:#7

	EBNF: Unordered List
		unorderedListItem«indent» creates unordered list if first item: 	
			«INDENT»
				CTRL='*'|'-' SPACE+ 
					(^EOL «creates P» content (EOL INDENT{indent+1} content)*)
					blocks«indent+1»*
		;

The ordered list extensions uses '.' or ')' as control sequence. If this sequene is found, it examines the preceeding element. 
If this is content which matches a numeration type, and if no space is preceeding, it transform this preceeding element into the number item type.

REQ SPEX-801: Ordered Lists
	TASK:#8

	EBNF:
		orderedListItem«indent» creates ordered list if first item:
			«INDENT» numerationType CTRL=('.'|')' SPACE+ 
					(^EOL «creates P» content (EOL INDENT{indent+1} content)*)
					blocks«indent+1»*
		numerationType: DIGIT+ | 'a'-'z';
	
	
The description list extensions uses ':' as control sequence. If this sequene is found, it examines the preceeding element. 
If this is content with only letters or spaces, it transform this preceeding element into the description title.

REQ SPEX-901: Description Lists
	TASK:#9

	EBNF: Description List
		descriptionListItem«indent» creates description list if first item:
			«INDENT» descriptionTitle CTRL=':' SPACE+ 
					(^EOL «creates P» content (EOL INDENT{indent+1} content)*)
					blocks«indent+1»*
		descriptionTitle: LETTER+ (SPACE LETTER+)*


## Generic References

MDSpex comes with a built-in extension enabling generic references. These references are placed in square brackets and enable linking via FQN or PQN.
The output of the link is defined by the extension defining the anchor type. The example [sample:-1] shows this kind of references.

REQ SPEX-1001:
	TASK:#10

	EBNF:
		genericReference: '[' refTarget  ']' | '<' refTarget  '>';
		refTarget: (PQN | FQN) ['|' title]
	
The generic reference may define an optional title which is used to create the link text. How the title is used depends on the extension defining the type, however.	  


## HTTP References

REQ SPEX-1002:
	TASK:#10

	The following anchor types are predefined: "http", "https". They simply create links to URLs, optionally using the title specified in the
	generic link format.


## Images

Images are included by the image extension. It follows the [Standard Command Format] with id and attributes:

REQ SPEX-1101
	TASK:#11
	
	EBNF: Image Extension
		image: 'IMAGE' idvalue? attributes ':' «path to image»
		
## Todo Blocks

Todo blocks are standard command blocks. They are visually marked as "to do".

REQ SPEX-1202 Todo Extension
	TASK:#12
	
	The Todo extension adds a standard block command `TODO` without title and without attributes.		 

## Task References

Task references are references to tasks (or issues) managed by an external task tracking system such as Jira or GitHub.

TODO: Describe task references as used in this document, e.g. `TASK: #1, #13`


## Source References

TODO:
	For links to source code, a typed reference extension is provided. It assumes that source code element names contain no spaces or other "problematic" characters.
	Thus it allows linking with only a prefix.
	
	EBNF:
		srcRef: '>' PQN | FQN «simpleNames only»
	
	For elements defining anchors with name according to prefix identifiers (see [Standard Attribute Definitions]), the extension defining the prefix identifier
	may enable automatic-linking simply by recognition of the prefix identifier.



## Tables

TODO: describe tables


## Forms

TODO: E.g. use case, properties etc.

## Decorations

# Math and Tex Support

TODO: $some_1 math$

## Tagged Blocks

TODO: REQ, INFO, WARNING, TODO, etc.


## Modules

TODO:
	MDSpex support splitting up a document into modules and combining these modules via the "include" extension. This extension simply includes the referenced
	document into the current one. It follows the [Standard Command Format] without id or attributes.
	
	EBNF:
		include: 'INCLUDE' ':' «path to included document»
		
	Instead of creating output, the included file is expanded at the current location.	
	

# Source Maps and Traceability

TODO: Enable traces and links such as 
		<a href="subl:/Users/account/file.md:10:5">subl</a>
	which could open an external editor (e.g., Sublime) in combination with a script as follows:
		
		on open location this_URL
			set SUBL to "/usr/local/bin/subl"
			set x to the offset of ":" in this_URL
			set the argument_string to text from (x + 1) to -1 of this_URL
			set cmdline to SUBL & " " & argument_string
			do shell script cmdline
		end open location

	This needs to be saved as "Application" with the following lines in "info.plist":
	
		<key>CFBundleURLTypes</key>
		<array>
			<dict>
				<key>CFBundleURLName</key>
				<string>Sublime Opener</string>
				<key>CFBundleURLSchemes</key>
				<array>
					<string>subl</string>
				</array>
			</dict>
		</array>