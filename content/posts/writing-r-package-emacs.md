---
author: "Pavodive"
title: "Writing an R package with Emacs and ESS"
date: "2025-01-11"
description: "Step by step guide to write R packages with emacs"
tags: ["R", "emacs", "ESS"]
categories: ["R", "emacs"]
series: ["How-to"]
aliases: ["write-r-package-ess-emacs"]
ShowToc: true
TocOpen: true
---

This article provides a step-by-step guide to writing R packages using Emacs' ESS (Emacs Speaks Statistics) mode. By the end, you'll have created a simple package called `dummyaddition`, which can perform the simple task of adding two numbers or paste two strings.

<!--more-->

## Why Emacs?

Emacs is one of those pieces of software that has stood the test of time. While some may argue that its age is a disadvantage, I would say that the fact it remains relevant after more than 40 years is proof of its robust design and adaptability to modern needs.

It’s far more than just a text editor or an IDE—it’s a powerful tool that has allowed me to develop a highly efficient workflow.

Here are some reasons why I prefer Emacs:

### Keyboard-Centric Efficiency
Emacs is command-line-based, meaning almost every possible action can be executed via the keyboard. I rarely use the mouse during my workflow. Relying on the mouse can strain your hands and arms, and it wastes time in unnecessary movements.

### Extensibility and Customization
Emacs is incredibly extensible and highly customizable. Whether you prefer to keep it simple or turn it into a complex powerhouse, you can tweak it to fit your needs and fulfill your specific desires.

### Easy to Learn
Although some people say the learning curve for Emacs is steep, I disagree. Vanilla (unmodified) Emacs is very simple to use, and it comes with excellent documentation. You can start by point-and-clicking while familiarizing yourself with the keyboard shortcuts. It’s not a radical difference from what you're likely using now. Over time, you’ll gradually learn to enhance and tailor Emacs to make it more efficient for your workflow.

### Free and Open Source
Emacs is free—both as in 🍺 (beer) and as in **freedom**. You can install it on any operating system at no cost. Not only _do you not have to pay_, but also **nothing is taken from you**: not your data, your software interactions, or your privacy.

Being open-source means you can modify Emacs and even share your tweaks with others—without anyone coming after you with legal concerns. It embodies true software freedom.

## Why a Package?


One of the fundamental goals of software development is **automation**. Tasks we do frequently should be made quicker and easier to execute. It's a common scenario: you write a function once, then you need it again, and again...

Instead of rewriting the same function multiple times (trust me, I’ve been there—it’s not fun), or wasting time searching for where you wrote it last, it’s better to consolidate those functions into a package. A package allows you to load and reuse those functions effortlessly whenever you need them.

Creating a package also means you can share your functions with others who might be facing the same problem you solved. By writing packages, you’re contributing to a collaborative environment. After all, I’m sure you’ve relied heavily on R packages created by others—so why not give back?

## End Result


By following this step-by-step guide, you will create a new package called `dummyaddition`. This package, while basic, will demonstrate the essential steps involved in creating an R package. You’ll host your version in your GitHub repository, but if you’d like to see mine for reference, here’s the link: [`dummyaddition`](https://github.com/pavodive/dummyaddition).

## Step by Step

### Magit for Version Control
#### Create a Repository on Github

The first step is to create a new repository on your GitHub account. I've named mine `dummyaddition`, but feel free to choose a name that makes sense for your project. While filling out the **description** field is optional, I highly recommend doing so—it’s helpful to have a clear description for each repository.

Keep in mind that the R package naming conventions (required by CRAN) allow only ASCII characters and numbers. Avoid using special characters like dashes, underscores, or periods in the name.

You also have the option to make the repository **public** or **private**. If you're planning to share your code (which I strongly encourage), make the repository public.

Before creating the repository, configure the following minor but important settings:

- **Initialize with a `README.md`**: Check this box. We'll use the README later.
- **Add a `.gitignore` file**: Select the R template. This ensures that Git ignores unnecessary files, such as `.history`, which don’t add value to your project.
- **Choose a License**: GitHub offers several license options. Two common ones are the _MIT License_ and the _GNU General Public License v3.0_. Be sure to read about their differences, as they may affect how your software can be used. For this tutorial, I’ve chosen the _GNU General Public License v3.0_.
    - Additionally, check the licenses of the R packages you plan to use in your code. Some licenses require you to adopt compatible or less restrictive licenses for your package.

Once you've configured the settings, click the **Create Repository** button.

#### Magit

**Magit** is an Emacs package that acts as a hybrid between a graphical Git client and the standard command-line Git interface. It provides user-friendly commands for Git actions. I use Magit because it simplifies interacting with Git, and I only need to remember a few basic commands. It’s user-friendly, and if you want to explore the actual Git commands it runs under the hood, they’re readily available for auditing and study.

If Git is not installed on your system, you can follow the official installation instructions [here](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git).

After installing Git, configure your name and email to associate commits with your identity:

```shell
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
```

Next, install **Magit** for Emacs. If you haven’t installed it yet, I recommend doing so via [MELPA](https://melpa.org/), the Emacs package repository. Add the following lines to your `.emacs` file:

```lisp
(require 'package)
(add-to-list 'package-archives
             '("melpa" . "https://melpa.org/packages/") t)
```

Select this block of text, evaluate it in Emacs by pressing `M-x eval-region RET`, and then refresh your package list with:

```lisp
M-x package-refresh-contents RET
```

You can now install Magit (and its dependencies) by running:

```lisp
M-x package-install RET magit RET
```

To ensure everything works as expected, restart your Emacs session. This resets the `load-path` and avoids potential issues with outdated configurations.

#### Cloning the Repository

Now it's time to clone the GitHub repository onto your computer.

Navigate to the directory where you’d like to clone the repository. For example, if you'd like to clone it into `~/Documents`, open Emacs’ **Dired** mode with `C-x D` and enter `~/Documents/` to open the corresponding directory.

Next, run the following command in Emacs:

```lisp
M-x magit-clone
```

Magit will ask where you'd like to clone the repository from. Choose `[u]` for **URL**. Then, paste the repository URL from your GitHub page. You can find the URL by clicking the green "<> Code" button and copying the HTTPS URL. For example:

```plaintext
https://github.com/PavoDive/dummyaddition.git
```

Paste this URL into Magit’s minibuffer.

Magit will then ask you to confirm the name of the subdirectory where the repository will be cloned. By default, it suggests the repository's name (e.g., `dummyaddition`). I recommend keeping this to maintain consistency.

It will also ask, **"Set remote.pushDefault to origin? (y or n)"**. Since this is the start of a new repository, you can safely choose **yes (y)**.

After a short while, Magit will finish the cloning process and display a new buffer with the repository's details and the most recent commit. You might need to press `TAB` to expand the details of the commits.

To verify the repository has been cloned, refresh the Dired buffer by pressing `g`. You should see a new directory—`dummyaddition`—inside your `~/Documents` folder. Navigate to this directory, and you'll find:

```plaintext
drwxrwxr-x  8 gp gp 4.0K Jan 11 17:44 .git
-rw-rw-r--  1 gp gp  671 Jan 11 17:44 .gitignore
-rw-rw-r--  1 gp gp  35K Jan 11 17:44 LICENSE
-rw-rw-r--  1 gp gp   26 Jan 11 17:44 README.md
```

These include the files and folders created during the GitHub setup:

- `.gitignore`: Preconfigured to ignore unnecessary files.
- `LICENSE`: The license you selected during setup.
- `README.md`: The initial README file.
- `.git/`: A hidden directory containing version control metadata.


### Ready to Start Developing

You are now ready to start developing your R package! 🎉

### Required Packages

To make writing R packages easier and more efficient, we need to install two essential packages:

- **`rmarkdown`**: Converts R Markdown documents into various formats.
- **`devtools`**: A collection of package development tools—a package to help you develop packages!

If these aren't installed yet, start a new R session in your `Documents` directory by running `M-x R` in Emacs. Once the R session starts, enter the following commands:

```r
install.packages("rmarkdown")
install.packages("devtools")
```

These packages may require additional dependencies. Since the installation process varies depending on your system, you might encounter errors. If so, don’t worry! Take a deep breath, search for the errors, and troubleshoot any issues you encounter.

{{< figure src="/images/keep-calm-try-again.jpg" attr="Keep Calm and Try Again" align=center link="/images/keep-calm-try-again.jpg" target="_blank" >}}

After successfully installing the packages, load them in your R session:

```r
library(rmarkdown)
library(devtools)
```

### Create a Package

Now that we’ve set up the necessary tools, let's create the package. In your R session, run:

```r
create("dummyaddition")
```

This will create a new R package named `dummyaddition`, inside a corresponding `dummyaddition` directory.

⚠ **Important Note**: Ensure that your R session is **not** inside the `dummyaddition` directory when running `devtools::create`. Otherwise, it will attempt to create a nested package (e.g., a `dummyaddition` folder inside another `dummyaddition` folder), which can be problematic.

When executed, the function prints the following information in your R console:

```
✔ Setting active project to "/home/gp/Documents/dummyaddition".
✔ Creating R/.
✔ Writing DESCRIPTION.
Package: dummyaddition
Title: What the Package Does (One Line, Title Case)
Version: 0.0.0.9000
Authors@R (parsed):
    * First Last <first.last@example.com> [aut, cre]
Description: What the package does (one paragraph).
License: `use_mit_license()`, `use_gpl3_license()` or friends to
    pick a license
Encoding: UTF-8
Roxygen: list(markdown = TRUE)
RoxygenNote: 7.3.2
✔ Writing NAMESPACE.
✔ Setting active project to "<no active project>".
```


#### What Gets Created?

You'll notice the following new elements in your `dummyaddition` folder:

- **DESCRIPTION**: Describes the metadata of your package (e.g., name, author, version, and description). Edit this file following the instructions within it.
- **NAMESPACE**: Manages exported and imported functions. This file is automatically generated—avoid editing it manually.
- **R/**: A folder to store your package functions and code. Currently, it is empty.

At this point, we have the skeleton of the package. Let’s move on to writing the code!

### Writing the Code

Let's define the core function of our package. Create a new file called `addition.R` inside the `R/` directory and add the following code:

```r{linenos=true}
addition <- function(a, b) {
  if (length(a) > 1 || length(b) > 1) {
    stop("I can't accept a vector. Sorry.")
  }
  if (is.numeric(a) == TRUE && is.numeric(b) == TRUE) {
    a + b
  }else {
    paste0(as.character(a), as.character(b))
  }
}
```

Of course, your package can include multiple complex functions. As a general recommendation, place each **exported** function in its own file. It’s fine to include auxiliary (internal) functions inside the same file as the main function that uses them.

### Documenting the Functions

Documentation is a crucial aspect of package development, and with `devtools`, you can easily manage it using `roxygen2`. This involves placing special comments at the top of each function to describe its purpose, usage, and arguments.

Let’s document our `addition` function by adding the following **above** its definition in `addition.R`:

```r
#' Add two numeric values or paste two character values.
#' 
#' This function takes two single values and adds them,
#' if they are numeric, or pastes them together, otherwise.
#' The function checks the length of each argument, and
#' returns an error if any of the arguments has length
#' greater than one.
#' 
#' @usage addition(a, b)
#' @keywords addition, pasting, sum.
#' @param a A single value that is numeric, character or that
#'  can be coerced to character.
#' @param b A single value that is numeric, character or that
#'  can be coerced to character.
#' @return A numeric value with the sum of a and b, if both
#'  are numeric, or a string if both values can be coerced to
#'  to string.
#' @examples
#' # Adding numerc values
#' addition(1, 6)
#' # Pasting strings together
#' addition("nice ", "function")
#' @export
```


#### Key Points About Documentation:

- `#'` denotes a documentation comment.
- The `@export` tag is essential to make the function available to users of the package. Functions without this tag are treated as internal.
- Tags like `@param`, `@return`, `@keywords`, and `@examples` provide structured details about the function.


### A Note on Imports

If your package relies on external libraries, you should include the `@import` tag in the documentation, like this:

```r
#' @import data.table
```

This ensures the package correctly imports external functionality.


With the core function implemented and documented, you’re well on your way to building a functional R package. Great work so far! 🎉

### Building the Documentation

To build the documentation for your package, ensure your working directory is set to the package's root directory (`dummyaddition`). If you haven’t closed your R session, you can simply use:

```r
setwd("dummyaddition")
```

If you’ve already closed the session, just start a new R session in the correct directory and reload the required libraries (`devtools` and `rmarkdown`) using `library()`.

Now, use the `devtools::document()` function, a convenient wrapper for the `roxygen2::roxygenize()` function, to generate the documentation. Run:

```r
document()
```

This produces the following output:

```
ℹ Updating dummyaddition documentation
ℹ Loading dummyaddition
Writing NAMESPACE
Writing addition.Rd
```

Let's break down what happens when you run this command:

- The **`NAMESPACE` file** is updated automatically: it now includes the information needed to export the `addition` function.
- The **`man/` directory** is created, if it doesn’t already exist. This folder contains documentation files for your exported functions, written in `.Rd` format. For example:
    - `addition.Rd`: This file was generated from the special comments in the `addition.R` file. It contains documentation in a format recognized by R. **Note:** Do not modify this file directly, as it is automatically generated.

At this point, you can now access the help for your function, even though the package isn’t installed yet. To do this, use a specific process:

```r
> ?
+ addition
```

In this code:
- `>` represents the R prompt.
- `+` indicates a continuation line (a feature of Emacs).

This means: write `?` and when emacs offers you the continuation line, write `addition`.

When you run this command, you’ll see a nicely formatted help page for your function, rendered as if the package were fully installed:

```
ℹ Rendering development documentation for "addition"
addition             package:dummyaddition             R Documentation

Add two numeric values or paste two character values.

Description:

     This function takes two single values and adds them, if they are
     numeric, or pastes them together, otherwise. The function checks
     the length of each argument, and returns an error if any of the
     arguments has length greater than one.

Usage:

     addition(a, b)
     
Arguments:

       a: A single value that is numeric, character or that can be
          coerced to character.

       b: A single value that is numeric, character or that can be
          coerced to character.

Value:

     A numeric value with the sum of a and b, if both are numeric, or a
     string if both values can be coerced to to string.

Examples:

     # Adding numerc values
     addition(1, 6)
     # Pasting strings together
     addition("nice ", "function")
```

This confirms that your documentation is functioning correctly and your package is well-structured.

#### A Beautiful pdf


Let's generate a nicely formatted PDF for our package—just like the ones you see with well-established R packages.

To produce this PDF, you'll need to use a shell (or Emacs’ Eshell). Open a shell using `M-x shell` or `M-x eshell` and navigate to the parent directory of your project (`Documents` in this example). Then, run the following command in the shell:

```shell
R CMD Rd2pdf dummyaddition
```

This will generate a PDF file named `dummyaddition.pdf` in the `Documents` folder.

⚠ **Important Note**: You may need LaTeX and Pandoc installed on your system to successfully create the PDF. If they’re not already installed, search online for platform-specific installation instructions (and make yourself a coffee, installing them will require some time and patience).

### Update the `README.md` File


The `README.md` file is the "face" of your package. It’s what people see first when they visit your repository (especially on GitHub), so it’s important to update it with relevant information.

At a minimum, your `README.md` file should include:

- **What your package does**: Provide a clear and simple explanation of what problem the package solves.
- **How to install it**: Include installation instructions. For example, you can add the following if your package is hosted on GitHub:

```r
devtools::install_github("yourusername/dummyaddition")
```

Replace `"yourusername"` with your GitHub username and `"dummyaddition"` with your package name.

### Pushing Your Updated Package to Github


Once you've completed your code, documentation, and updates to your `README.md`, it’s time to push the changes to GitHub. Magit makes this process smooth and efficient.

1. **Open the Magit buffer** for your repository, typically titled `magit: dummyaddition`. If it's not already open, you can activate it with the command:

   ```
   M-x magit-status
   ```

2. **Refresh the repository status** by pressing `g` in the Magit buffer. You'll see something like this:

   ```
   Head:     main Initial commit
   Merge:    origin/main Initial commit
   Push:     origin/main Initial commit
   
   Untracked files (4)
   Unstaged changes (1)
   
   Recent commits
   ```

   You may expand specific sections (e.g., untracked files, unstaged changes, recent commits) by pressing `TAB`.


#### Staging and Committing Changes

- **Untracked files**: When you expand the "Untracked files" section, you'll see files like `DESCRIPTION`, `NAMESPACE`, and directories like `man/` and `R/`. These are the new files we created during the development process.
  - Stage these changes by selecting the file or directory and pressing `s` (for _stage_). Staged files will now appear under the "Staged changes" section.

- **Unstaged changes**: You’ll notice that the `README.md` file appears here because it was modified (not newly created). Stage these changes with `s` as well. Magit may ask if you'd like to stage all changes—press **y** to confirm.

- Once all changes are staged under the "Staged changes" section, commit them by pressing `c` (for _commit_). A menu will appear. Press `c` again to create the commit.


#### Writing the Commit Message

Once you start the commit process, a buffer will open for you to write your commit message. Commit messages should be meaningful and concise. For example:

- **Title**: Clearly summarize the commit in one line.
- **Body (optional)**: Add more details if necessary, especially if the changes are complex.

Here’s an example commit message for our work:

```
Code and Documentation Generated
- Added the core function `addition`.
- Created auto-generated documentation with roxygen2.
- Updated the README.md file.
```

After writing your commit message, save it and press `C-c C-c` to confirm and close the buffer.

This will update the Magit buffer, which may now say something like:

```
Unmerged into origin/main (1)
56d6bd3 main Code and Documentation Generated
```

This means your changes have been committed locally but not yet pushed to the GitHub repository.


#### Pushing to GitHub

To push your changes to GitHub, press `P` (for _push_) in the Magit buffer. A menu will appear—press `p` to push your changes to `origin/main`.

Magit may ask for your GitHub credentials (username and password) if you haven’t configured them for SSH or token-based authentication. Enter the required information, and after a moment, the Magit buffer will reflect that the changes have been pushed:

```
Recent commits
56d6bd3 origin/main Code and Documentation Generated
4b1b6c8 Initial commit
```

At this point, your changes are live on GitHub. You can verify them by visiting your repository on GitHub.

## Share Your Package

Your journey isn’t complete until you share your work! While this package may solve a particular problem for you, it’s likely someone else out there (out of the 8 billion people on Earth 🌍) may face the same challenge in the future. Sharing your work could help them too.

Here are some ways to share your package:

- Post about it on your **social networks**.
- Write a **blog post** about what it accomplishes.
- Share it on platforms such as [Hacker News](https://news.ycombinator.com) or [Lobste.rs](https://lobste.rs).
- Mention it in relevant **forums or communities**.

Whichever method you choose, don’t forget to share your amazing work with the world!
