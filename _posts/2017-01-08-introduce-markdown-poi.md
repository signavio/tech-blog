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
In this simple example we'll explain the basic structure of Markdown and docx-documents and how special pieces of like tables,
lists, quotes and codeblocks can be transformed to a Word document.
## Why transform Markdown to a Word document?
In an workflow management system like [Signavio Workflow] (http://www.signavio.com/products/workflow/) we face the frontier of digital 
and non-digital age quite often. As we try to do everything digital, Markdown is our best friend when it comes to formatting text online, espacially in a workflow management system. But usually in an office there are also working a lot of non-tech guys who love using Microsoft Office products like PowerPoint, Excel and Word.  

