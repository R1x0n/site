---
title: "Defl8"
date: 2021-02-17
tags: ["Experiences"]
draft: false
image: "posts/defl8/static/logo.png"
---

In this post, I want to speak about a project that I made as a semester-work for the “[Algorithms and data structures](https://www.supsi.ch/dti/bachelor/ingegneria-informatica/piano-studio-offerta-formativa/piano-studio/dettaglio-piano-studio/dettaglio-modulo.158172.backLink.b1046c04-728f-44a0-b4ad-5a72df905579&ps=3188.html)” module in SUPSI.
<br>I worked on this project with [Ignacio Utrilla](https://github.com/IgnacioUtrilla), which I want to thanks for all the work and the effort he put into this project.

If you are interested to see the project’s repository on GitHub [click here](https://github.com/IgnacioUtrilla/defl8)! 😺

## Project’s goal

The initial delivery of this project was to build in C language a compressor and decompressor using the [DEFLATE](https://tools.ietf.org/html/rfc1951) algorithm, which combines the Huffman algorithm with that of the LZ77.
After an initial analysis and discussion, we decided to modify, in agreement with the professor, the initial algorithm using LZ78 instead of LZ77, as an attempt to optimize the compression percentage.
An interesting gamble that did not, however, lead to an improvement in the algorithm 🙁

## The core of the Defl8’s algorithm

The idea behind Defl8 is to optimize the L78 coding sending the new character through the Huffman coding.

<u>So a normal LZ78 message will be the following:</u>
* [ index + new-char (8 bit) ]

<u>While a Defl8 message will be the following:</u>
* [ index + Huffman coding (mostly < 8bit) ]

Whit this structure we are trying to optimize the number of bits required to write a sequence of byte, obviously, there are some cases in which this approach will optimize the output and others where the output file becomes bigger than the primitive one.
To mitigate this behavior the algorithm splits into multiple blocks the file, each one of 64k, compresses each block individually, and compares the output size whit the native size of the block. If the output block becomes bigger, this specific block will be not compressed.

Once a block is compressed will be write in the following structure:

* [header (3 bit) + index + Huffman coding + … + end-of-block (11)]

## A comment to the source code

As previously said, all the project is written in C and the main topic was to create a single executable able to compress and decompress every type of file.
I want to mention that, all data structures have been implemented from scratch, no external library was used.

We tried to modulize the source code as much as possible, giving to him the following folder structure for each functionality:

```
lib
|– lz78
|   |   `– encoding
|   |       |– lz78.c
|   |       |– lz78.h
|   |       |– lz78.test.c
```

The following animation will shows an example of the executable at work:
<img src="static/defl8gif.gif" width="50%" height="50%" align="center" class="center">
