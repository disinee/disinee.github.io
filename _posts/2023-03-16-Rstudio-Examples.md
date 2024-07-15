---
layout: post
title:  Hashiwokakero
date: 2024-03-04
category: [Projects]
---

Hashiwokakero is a logic puzzle game where players are presented with a grid circle, each containing a number. As this was a constraint satisfaction problem, our team employed a backtracking search algorithm on a 2D array as the data structure. Recursion on this 2D array enabled us to deploy forward checking to detect failure (unsatisfied constraints). 

<!--more-->

The program begins by scanning the map given from standard input via the scan_map() function. It scans all the islands into a dictionary where the keys are the island number and the value is a class type Island which holds all the relevant information of that particular island. 
