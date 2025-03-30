---
author: "Pavodive"
title: "Bash Script for Edition and Exporting of an Org-mode Template"
date: "2025-03-30"
description: "I have an org-mode template of a document. I have to edit some information in it, and export it to text and then to pdf. I do it with a bash script"
tags: ["shell", "bash"]
categories: ["shell"]
series: ["How-to"]
aliases: ["bash-org-mode-document"]
ShowToc: true
TocOpen: true
---

I have a template of a document written as an org file. I have to edit some fields in that file to be able to generate different versions of that document for different people. I export it as a text file to give the feel of a document written with an old-school software and printed with a dot matrix printer. Finally I convert the produced document to PDF.

<!--more-->

## The Problem

Every now and then I have to produce a document that has to have the feel of old-school software and dot matrix printing. It has to have some fixed data, and some dynamic data, that is related to the special case at-hand. Without any automation, the process is like this:

1. Open the org file and edit the fields. Try **not to** forget about editing any of the fields. _Not that it has ever happened to me 🙄_.
2. Export it to text `C-c C-e t a`.
3. Convert the text file to pdf.

I wondered if I could do this without needing to open the org file at all, and specially without **forgetting about any of the fields**. One command, type the substitutions and go! That would be ideal.

## The Solution

> This solution is one of those cases where plain text is the king of formats.

The solution I devised is a bash script that I will call from the terminal, and it comes in four parts:

- Ask the user for the dynamic fields using bash's `read`.
- Substitute the fields with the user input using `sed`.
- Export a temporary text file using `emacs` from the command line, and
- Convert the text file to PDF using `cups`.

### Ask the User for Dynamic Fields

```shell
#!/usr/bin/bash

read -p "Please input the year: " year
read -p "Please input the name of the person: " nombre_sujeto
read -p "Please input the text for other field: " other_field
# Some more read commands similar to those above
```

### Substitute Fields in the Org File

The plan in this step is to substitute the occurrences of "field" in the org file with the user-supplied value for "field" (`$field`), and do it with `sed`. The step after this will be to run the org-export command on the text produced by the substitution. Here we need to take into account that emacs will not read from standard input, but from a file. So we need to make the substitutions with `sed`, _export the resulting text to a temporary file_, and then run the org export command on that file.

Let's go over the first two parts:

```shell
# Create the temporary file
tempfile=$(mktemp /tmp/docXXXXXXXXXXXXXXX.org)

# Make the substitutions with sed
sed -e "s/year/$year/g" -e "s/nombre_sujeto/$nombre_sujeto/g" -e "s/other_field/$other_field/g" /full-path-to/template_document.org > "$tempfile"
```

Some interesting things here:

- `mktemp`: It's very useful for cases like this, where you need some intermediate output. Remember to delete after working with it!
- The use of double quotes ["] in `sed`. If you use single quotes, you'll replace any occurrence of `year` with the literal text `$year` **and not with your intended text**, say `2025`.
- We directed the results of the substitution to the temporary file.

### Run the Org Export Command on the Temporary File

Emacs offers a very cool feature, that is allowing you to call a function on the input file. We'll use that feature to run the org export command on our temporary file.

```shell
emacs "$tempfile" --batch \
      -f org-ascii-export-to-ascii \
      --kill
```

The `-f` flag is the one allowing you to define the function you want to run (`org-ascii-export-to-ascii` in our case).

The `--batch` flag prevents emacs from doing an interactive display, and the `--kill` quits without asking for confirmation (see `emacs --help` or `man emacs`).

Now we need to do some minor housekeeping before our next and final step:

```shell
# Create the name of the output file
output_name="document_${nombre_sujeto// /_}_$year.txt"

# Rename the file produced by emacs to the new name
mv "${tempfile%.org}.txt" "./$output_name"

# Remove the Temporary file produced by sed
rm "$tempfile"
```

- The `${nombre_sujeto// /_}` part takes the value in `nombre_sujeto` and replaces any spaces in it with underscores. Thus, the value `John Smith` will produce `John_Smith`.
- The `${tempfile%.org}` drops the `.org` part, just leaving the name of the file (without extension), adding then `.txt`. So say `tempfile` was `docAFGERWESF42424.org`, the name of the file we're renaming is `docAFGERWESF42424.txt`, which is in fact the file produced by the org export command. We don't want to keep that ugly name, and that's why we're renaming it to a more user-friendly name.
- Finally we delete the temporary file to leave no traces.

> "Take only pictures and leave only footprints."

### Convert the Text File to PDF

As I said before, one of the requirements of the project was to produce a document that had the feeling of being printed in a dot matrix printer. Exporting from org directly to PDF would not achieve that without some heavy LaTeX template work.

So the idea here is to retain as much as possible of the plain text format and feel while being able to produce a PDF file. While there are several options here, `cups` is the one that, in my opinion, produces the result I wanted. It's also a very simple one-liner:

```shell
cupsfilter "$output_name" > "${output_name%.txt}.pdf"
```

And that's it, I produced a PDF document with the feel of dot matrix printer from an org file, without even opening emacs.
