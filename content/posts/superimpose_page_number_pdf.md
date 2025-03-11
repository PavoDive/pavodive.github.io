---
author: "Pavodive"
title: "Adding Page Numbers to a PDF Document"
date: "2025-03-10"
description: "Add page numbers to an existing pdf with Latex"
tags: ["latex"]
categories: ["latex"]
series: ["How-to"]
aliases: ["superimpose-page-number"]
ShowToc: true
TocOpen: true
---

I recently faced the following problem: I had a PDF document created by merging several other documents, many of them containing multiple pages. I needed to display the page numbers in the document. **Latex** and **pdftk** to the rescue!

<!--more-->

# The Problem

I needed to generate a PDF by merging multiple unrelated documents from different sources, resulting in a 57-page file. After that, I had to display the page number in the header as "Folio x/57", where "x" represented the current page number ("Folio is the Spanish legalese for "page").

Manually adding the headers with the page numbers was too tedious, so I devised a two-step solution.

# The Solution

The approach I came up with had two steps:

1. **Generate a separate PDF with only the headers** ("Folio x/57") on each page.
2. **Merge** this with the original document, so that the headers are superimposed on top.

## Generating a PDF with Only Page Numbers

I needed to create **57 mostly empty pages** with "Folio x/57" in the top-right corner. The page number needed to update dynamically for each page. While this could be done in different ways, generating a PDF file was the most sensible options.

There were several tools to create such a document, but I choose `Latex`, because it offered the simplest and most customizable solution. Customization was crucial, particularly for adjusting margins, since the original document had inconsistent ones. The `geometry` and `fancyhdr` packages in `Latex` provided the flexibility I needed.

Here's the code I used in `pagination.tex`:

```latex
\documentclass[12pt]{article}
\usepackage[paperwidth=8.5in,
  paperheight=11in,
  top=0.7in,
  right=0.5in,
  left=0.5in,
  bottom=0.5in]{geometry}
\usepackage{fancyhdr}
\usepackage{pgffor}
\pagestyle{fancy}
\fancyhf{}
\rhead{Folio \thepage/57}

\begin{document}
  \setcounter{page}{1}
    \foreach \n in {1,...,57} {
      \null\vfill\eject
    }
\end{document}
```

This script sets the document's paper size and margins, defines the header format and then generates the pages using a loop, placing a new page break in each iteration.

If you are wondering why use `\vfill\eject` instead of `\newpage`, I recommend checking out [this question in tex.stackexchange.com](https://tex.stackexchange.com/questions/208698/difference-between-newpage-and-vfill-eject).

After compiling the LaTeX file, I obtained `pagination.pdf`, a beautifully blank document that displayed the page numbers in the appropriate places.

## Merging the New Document with the Original PDF

The final step was merging the two PDFs –not just **appending** them, but actually **overlaying** them so that the headers from my new document appeared on top of the original content.

For this, I used `pdftk`, a fantastic tool for manipulating PDFs. The following one-liner did the trick:

```shell
pdftk original.pdf multistamp pagination.pdf output numbered.pdf
```

- `original.pdf` is the main document.
- `pagination.pdf` contains the headers.
- The `multistamp` option tells **pdftk** to overlay each page of `pagination.pdf` onto the corresponding page in `original.pdf`.
- The `output` flag generates the new, numbered PDF as `numbered.pdf`.

And that's it! I managed to solve a tedious problem with just a few lines of code. Now I'm sharing them with you –and with _future me_ 😉.
