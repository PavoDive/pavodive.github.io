---
author: "Pavodive"
title: "Paquete spanish2 para R"
date: "2025-01-11"
description: "Paquete para convertir números a su equivalente en texto en Español con R"
tags: ["R"]
categories: ["R"]
series: ["How-to"]
aliases: ["package-spanish2"]
ShowToc: true
TocOpen: true
---

In legal documents in Spanish language it is often necessary to include the "value in words" of a number—for example, $2400 (dos mil cuatrocientos pesos). I created the package `spanish2` to automate this task. If you are curious to learn more about this package, please keep reading. If you'd like to install it or view its code, you can access its [GitHub repository](https://github.com/pavodive/spanish2). Writing this package inspired [this article]({{< ref "writing-r-package-emacs" >}} "Writing an R package with Emacs' ESS") about how to write an R package using Emacs and ESS (Emacs Speaks Statistics). You might want to check it out!

<!--more-->

## Why Write the Value in Words?

I'm not sure if this is just a custom in my home country 🇨🇴, but in Colombia, it is common practice to include the value in words immediately after its numeric representation in legal documents. For example:

> ...con un precio de $1.000.000 (un millón de pesos)...

or

> ...el lote tiene un área de 43 ha (cuarenta y tres hectáreas)...

Over the past few months, I encountered this requirement repeatedly in the context of some ongoing collaborations.

## What alternatives did I have?

There is a package in [CRAN](https://cran.r-project.org/web/packages/spanish/index.html) and in [github](https://github.com/rOpenSpain/spanish) called `spanish` that almost met my needs. This package provides a function, `spanish::to_words()`, for converting numbers into text. However, the function presented some limitations.


### First Issue: Incorrect Number Representation

In certain cases, the package generated incorrect word representations for numbers. For instance:

```r{linenos=true}
spanish::to_words(40000000)
# "cuarenta millones mil "
```

I managed to identify and suggest fixes for some of these issues, but additional errors emerged under less-than-ideal circumstances—such as when I had already delivered a finished document to a client. 😵

### Second Issue: Limited Range

Another significant limitation of the package was that it could not handle numbers larger than 999,999,999.


## Creating a Solution: `spanish2`

Faced with these challenges, I decided to create my own solution. **There's nothing better than standardizing solutions you find helpful for yourself**, as there's a good chance others might find them useful as well. This is how the `spanish2` package was born.

**Important Note**: While the `spanish` package did not meet my specific needs for this task, it offers several unique and valuable functions. For example, `to_number()` converts text-based numbers back into numeric values, and the package also includes functionality related to geolocation in Spain. If these features sound interesting, I encourage you to give the `spanish` package a try!


## Solution Strategy

To convert numbers into words, I followed the same logical principle we use when reading numbers in Spanish:

1. Group the digits in sets of three, from right to left.
2. Read these groups as "hundreds" (this concept was key—bear with me 😉).
3. Combine the different "hundred" blocks.
4. Finally, clean the resulting text.

### Group Digits in Threes and Read Them as "Hundreds"

The first task involves converting the number into a string and splitting it into groups of three digits. We use a regular expression (regex) to accomplish this:

```r
groups = regmatches(y, gregexpr(".{1,3}(?=(.{3})*$)", y, perl = TRUE))[[1]]
```

Once the number is split into groups of three digits, we pass each group to the function `convert_3_digits`. This function is responsible for "reading" the hundreds, tens, and units:

```r{linenos=true, hl_lines=[7,8]}
convert_3_digits = function(string_value){
  units = c("cero", "uno", "dos", "tres", "cuatro",
            "cinco", "seis", "siete", "ocho", "nueve")

  string_value = sprintf("%03d", as.integer(string_value))
  digits = strsplit(string_value, "")[[1]]
  a = sapply(digits, function(x) units[as.integer(x) + 1])
  raw_text = mapply(paste0, a, c("cientos", "diez y", ""))
  clean_text(paste(raw_text, collapse = " "))
}
```

Here’s what is happening in detail:
- The `sapply` function converts each digit into its corresponding word. For example, `234` becomes `c("dos", "tres", "cuatro")`.
- The `mapply` function maps this vector of words to another vector specifying hundreds, tens, and units. This results in `"dos cientos tres diez y cuatro"`.
- Finally, the function `clean_text` handles edge cases such as "tres diez" (which should become "treinta") to ensure the text is properly formatted.


### Map the Big Units

At this point, we have the number converted into text groups of three. For instance, the number `1,234,567` would be converted into `c("cero cientos cero diez y uno", "dos cientos tres diez y cuatro", "cinco cientos seis diez y siete")`.

Clearly, this output doesn’t quite make sense. To address this, we need to map each group to its corresponding "big units" (i.e., thousands or millions).

The second group (from right to left) represents thousands (`mil`), and the third group corresponds to millions (`millones`). After applying `mapply`, the text becomes "cero cientos cero diez y uno **millones** dos cientos tres diez y cuatro **mil** cinco cientos seis diez y siete". While it may still look rough, the structure is now correct.


### Clean the Text

In the final step, we use the `gsub` function extensively to clean up the text. This involves applying regular expression replacements to handle edge cases and improve readability. Here’s what we do:

1. **Remove unnecessary elements**: For example, "cero cientos" and "cero diez y" should be removed entirely, as they are redundant.
2. **Fix special pronunciations**: Certain phrases like "diez y cinco" are replaced with their proper counterparts (e.g., "diez y cinco" becomes "quince," and "diez y tres" becomes "trece").


## What We Achieved

The function `spanish2::to_words()` can convert numbers up to `1e22` or strings with numeric values up to 60 characters in length. Those are **really big numbers**!

The conversion to text adheres to the conventions of the Spanish language, specifically using the [long scale](https://en.wikipedia.org/wiki/Long_and_short_scales). As a result, the number `1e9` (1,000,000,000) is converted to **"mil millones"**, not **"un billón"**, which may be the expectation of English speakers who are used to the short scale.

Additionally, the conversion produces the most standard **and simple** way of expressing a number in Spanish, even though alternative representations may exist. Here's a comparison of some examples:

| Number | Common Style 1  | Other Styles              | spanish2 Output |
|:------:|:---------------:|:-------------------------:|-----------------|
| 77     | Setenta y siete |                           | setenta y siete |
| 16     | Dieciséis       | Diez y seis               | diez y seis     |
| 27     | Veintisiete     | Veinte y siete            | veinte y siete  |
| 1100   | Mil cien        | Mil ciento / mil y ciento | mil cien        |

I hope this package proves to be useful to you! If it does, how about giving it a ⭐ on GitHub? 😊

[Star the repository on GitHub ⭐](https://github.com/pavodive/spanish2)
