# Disclaimer

This is WORK IN PROGRESS. That is, although some basic parts are already working,
most features described here or in the specification are not working yet.
As soon as a first usable version is ready, this disclaimer is to be removed.


![](https://travis-ci.org/jpilgrim/spex.svg?branch=master)

# Overview

Spex is an extensible markdown language and processor written in N4JS (<https://numberfour.github.io/n4js/>). N4JS is an extension of JavaScript, and the code is transpiled to plain JavaScript. Thus extensions can be written in N4JS or JavaScript (or other languages transpiling to JavaScript).

The goal of Spex is to provide an unicode based textual language which is
as readable as markdown (and, as far as possible, compatible with GitHub Flavored Markdown, <https://github.github.com/gfm>) yet powerful enough to enable writing technical specifications,
in particular for software projects . If possible, extensions are to be designed compatible with
Asciidoc <http://asciidoctor.org/docs/asciidoc-writers-guide>.
For the target document types, namely software requirement specifications, it will come with a set of predefined extensions. For mathematical formulars, Tex-like syntax is to be supported.

The output format of Spex documents is created by output processors. 
Although different output formats may be produced by different processors, 
the focus is on a single output processor creating high-quality printable
HTML, using CSS and JavaScript (for screen reading).

The Spec processor consists of a very small core processor which can parse block structures. 
This processor can be extended by so called extensions. 
Thus there is no clearly predefined syntax. 
However all extensions should follow similar syntax patterns.

An important feature of documents is linking. 
A reference links to a named element or a target outside the document.
Elements can be nested and typed and they are referred to by name or ID. 
Spex supports the notion scoping and fully (or partially) qualified names.

The following snippet shows a simple Spex example using pre-defined extensions and links:

```spex
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


# Frequently Asked Questions

*Question:* When will it be usable?

*Answer:* As soon as the Spex specification found at <https://github.com/jpilgrim/spex/blob/master/spex.doc/index.spex> is successfully processed and the result is available via the GitHub pages.

# License

 Copyright (c) 2017 Jens von Pilgrim.
 All rights reserved. This program and the accompanying materials
 are made available under the terms of the Eclipse Public License v1.0
 which accompanies this distribution, and is available at
 	<http://www.eclipse.org/legal/epl-v10.html>
 