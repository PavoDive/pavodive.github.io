---
author: "Pavodive"
title: "Swirly: a CLI-based tool to learn python"
date: "2025-04-08"
description: "Swirly is a CLI tool to learn any python package, similar to the R package Swirl"
tags: ["python", "pandas"]
categories: ["python", "pandas"]
series: ["How-to"]
aliases: ["swirly"]
ShowToc: true
TocOpen: true
---

This is a very short article to share **Swirly**, a Command Line Interface-based tool to learn python (or any python package) interactively.

<!--more-->

## Swirl

[Swirl](https://swirlstats.com/) is a great package by Nick Carchedi and others. I've used it many times to learn about some R packages, and just about R when I was new to it.

Swirl allows you to interact with the package, but guides you through questions and checking your answers.

I missed Swirl in Python, and thought that developing something similar was a a good idea.

## Swirly

Swirly is a python CLI-based tool that behaves in a similar way to Swirl. It has some pre-defined lessons **that can be extended, created and modified** that guide you trhough questions and validation of your answers. It's entirely developed in python, and the lessons are very simple YAML files.

Take a look at [Swirly's repo](https://github.com/PavoDive/swirly), I hope it's useful!
