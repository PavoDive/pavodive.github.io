---
author: "Pavodive"
title: "Automating RMarkdown Rendering and PDF Merging with Bash"
date: "2025-02-06"
description: "I used bash to automate the rendering of hundreds of Rmarkdown files and the merging of the resulting PDFs"
tags: ["shell", "bash"]
categories: ["shell"]
series: ["How-to"]
aliases: ["bash-scripts-rmarkdown"]
ShowToc: true
TocOpen: true
---

I work with many RMarkdown files structured within a hierarchy of directories. I needed to render these files to PDF and then use Ghostscript to merge them. This article explains the two small shell scripts I used for the task.

<!--more-->

## The Problem

For a client project, I need to produce several PDF documents consisting of:

- A cover letter
- A report

For reasons irrelevant to this article, the cover letter and report use different templates for rendering, so they cannot be combined before rendering. Both documents are written in RMarkdown (`.Rmd` files), which renders directly to PDF.

Each project involves _hundreds_ of cover–report pairs, making manual rendering impractical. The directory structure follows this pattern:

```
📂--client-root
    📂--project-1
    |  📂--report
    |  |  |--cover.Rmd
    |  |  |--report_project-1.Rmd
    |  📂--data
    📂--project-2
    |  📂--report
    |  |  |--cover.Rmd
    |  |  |--report_project-2.Rmd
    |  📂--data
```

Of course, in reality, my directories aren't named "project-n"; they have real, meaningful names.

> I never, **ever**, use spaces or non-ASCII characters in any directory or file names.

## The Solution

I used a one-liner to render all `Rmd` files to PDF:

```sh
find client-root -type f -name "*.Rmd" | xargs -I{} Rscript -e 'rmarkdown::render("{}")'
```

### How It Works

- `find client-root -type f -name "*.Rmd"` is a standard `find` command that:
  - Searches within `client-root`
  - Looks for files (`-type f`)
  - Matches filenames ending in `.Rmd` (`-name "*.Rmd"`)
  - The output is a list of file paths, e.g., `./client-root/project-1/report/cover.Rmd`.

- The `|` (pipe) sends this list to the next command.

- `xargs -I{} Rscript -e 'rmarkdown::render("{}")'` processes each file:
  - `xargs` constructs and executes commands for each file found.
  - `-I{}` tells `xargs` to replace `{}` with each filename.
  - `Rscript -e` runs an R expression (`-e` denotes inline execution).
  - `rmarkdown::render("{}")` calls the R function to process each file dynamically, replacing `{}` with the actual filename.

After running this, the directory structure now contains corresponding PDFs:

```
📂--client-root
    📂--project-1
    |  📂--report
    |  |  |--cover.Rmd
    |  |  |--cover.pdf
    |  |  |--report_project-1.Rmd
    |  |  L--report_project-1.pdf
    |  📂--data
    📂--project-2
    |  📂--report
    |  |  |--cover.Rmd
    |  |  |--cover.pdf
    |  |  |--report_project-2.Rmd
    |  |  L--report_project-2.pdf
    |  📂--data
```

## A New Problem
Now I needed to merge the cover and report PDFs for each project.

For a single project, I could do this manually using Ghostscript (`gs`):

```sh
gs -dBATCH -dNOPAUSE -sDEVICE=pdfwrite \
   -sOutputFile=merged_report_project-1.pdf \
   cover.pdf report_project-1.pdf
```

But since I had multiple projects, I needed to automate the process using Bash.

Additionally, I had to follow a naming convention:
The merged file should start with `"merged_"`, followed by the report’s filename, e.g.:

```
merged_report_project-1.pdf
```

## Merging the PDFs

To merge the PDFs, my approach was:

1. Locate all `report` directories across projects.
2. Extract file paths for the cover and report PDFs.
3. Construct the merged filename dynamically.
4. Use Ghostscript to merge the files.

Here’s the script:

```sh
find client-root -type d -name "report" | \
    while read -r dir; do
        cover_pdf="$dir/cover.pdf"
        report_pdf=("$dir/report_"*.pdf)
        output_pdf="$dir/merged_$(basename "${report_pdf[0]}")"
        gs -dBATCH -dNOPAUSE -sDEVICE=pdfwrite \
           -sOutputFile="$output_pdf" \
           "$cover_pdf" \
           "${report_pdf[0]}"
    done
```

### Understanding the Script

- **Finding directories**
  ```sh
  find client-root -type d -name "report"
  ```
  - Searches under `client-root`
  - Finds only **directories** (`-type d`) named `"report"`.
  - The results are piped to the next command.

- **Processing each directory**
  ```sh
  while read -r dir; do ... done
  ```
  - Iterates over each directory found.
  - `read -r dir` assigns each directory path to `dir`.
  - The `-r` flag ensures the path is read literally, preventing unintended escape sequences.

- **Defining the file paths**
  ```sh
  cover_pdf="$dir/cover.pdf"
  ```
  - Constructs the path for the cover PDF.
  - Quotes ensure correct handling if spaces exist (even though I avoid them).

  ```sh
  report_pdf=("$dir/report_"*.pdf)
  ```
  - Uses a wildcard (`report_*.pdf`) to match the report file.
  - The parentheses **create an array**, allowing for multiple matches (though only one is expected).

- **Constructing the merged filename**
  ```sh
  output_pdf="$dir/merged_$(basename "${report_pdf[0]}")"
  ```
  - `${report_pdf[0]}` selects the first (and expected only) match.
  - `basename` strips the directory path, keeping only the filename.
  - `$( ... )` performs **command substitution**, inserting the result dynamically.
  - `"merged_"` is prepended to create the final merged filename.

- **Merging with Ghostscript**
  ```sh
  gs -dBATCH -dNOPAUSE -sDEVICE=pdfwrite \
     -sOutputFile="$output_pdf" \
     "$cover_pdf" \
     "${report_pdf[0]}"
  ```
  - Combines the cover and report PDFs, saving as `merged_report_project-N.pdf`.
  - If you're curious about `gs` flags, check them out using `man gs`.


And that’s it! Now all my `merged_report_project-x.pdf` files are generated automatically.

**Bash saved me a lot of time**, which I then used to write this post. Now back to work! 😇
