---
title: Transform Markdown
description: Introdcution to Markdown and Apache POI XWPF
tags: JSON, data types
layout: article
---

## Introducing a Way to transform Markdown to an Word document
The purpose of this post is to show a way to transform Markdown
to a Word document using Frameworks like [Apache POI] (https://poi.apache.org/document/), 
[JSoup] (https://jsoup.org/) and [Pegdown] (https://github.com/sirthias/pegdown).
This blogpost will cover a simple example where a piece of markdown is transformed into a Word document. 
In this simple example I will show how simple markdown - formatting like **bold**, *italic* and ~~strikethrough~~ - can be transformed into a docx file.
## Markdown and Word documents
If you never faced markdown before, you can read everything about it at [daring fireball](https://daringfireball.net/projects/markdown/). To keep it short, Markdown is a markup language created for fast, easy formatting of text *on the fly*. Markdown lets the author of text format his text while writing it. So this:

`
This text is **bold** and this is *italic*, while this is ~~strikethrough~~. 
`

becomes:

This text is **bold** and this is *italic*, while this is ~~strikethrough~~. 

As you might know, in Microsoft Word formatting text like this have extra buttons in menus. 
## Why transform Markdown to a Word document?
In an workflow management system like [Signavio Workflow](http://www.signavio.com/products/workflow/) we face the frontier of digital and non-digital age quite often. As we try to do everything digital, Markdown is our best friend when it comes to formatting text online, espacially in a workflow management system. However in everyday life we are working with a lot of non-techy guys who love using office products like the Office Suite provided by Microsoft or Google.
Sometimes these two worlds collide and in the event of clashing there should be a solution suitable for both worlds, leading directly into a markdown to docx transformer. 
## Transform simple markdown text to a word document 
As mentioned in the introduction, the goal is to transform this:

`
This text is **bold** and this is *italic*, while this is ~~strikethrough~~. 
`

into this:



