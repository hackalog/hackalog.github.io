---
layout: post
title: Reproducible PDFs
date: 2022-07-19
categories: [pdf, reproducibility]
excerpt: Reproducibly generating a PDF from the same source file can't possibly be a hard problem, can it?
use_math: true
---

I got stuck in a rabbit hole today, and it was this: **Can I reproducibly generate a PDF?**

I mean, starting from the same input document (a markdown document, in this case), can I get the same PDF out?
That can't possibly be a hard problem, can it?

Short answer: *it's a lot harder than I thought.*

## The Pipeline

We generate our PDF using [pandoc] from a set of markdown source files. It's an academic paper, so there's a mix of filters in there: latex for equations, [pandoc-crossref] for intra-document references, and citeproc (bibtex) for citations and references. It's also multilingual, so we throw [xetex] into the mix. It may seem like a crazy way to do it (you may ask, _why not just use pure LaTeX?_) but the result is an easy-to-edit, easy-to-diff document that can be easily maintained (_and_ viewed) in GitHub.

[pandoc]: https://pandoc.org/
[pandoc-crossref]: https://github.com/lierdakil/pandoc-crossref

It also feels (maybe this is just my own bias) more reliable converting from markdown to other document formats, so when a client asks for, say,  a Word document to send for translation, we can easily do the conversion and expect that everything will render properly.

## The Problem

``` bash
>>> make clean && make && md5 document.pdf
MD5 (document.pdf) = fdfeefe8eb0df92162342271ad4cacc2
>>> make clean && make && md5 document.pdf
MD5 (document.pdf) = 90360b00c4f1ef08e57135e6b866e392
```

Basically, every time the PDF is generated, the hash is different. That's a little embarrassing for a guy who does reproducibility research. I need to fix that.

## The Fix

Of course, there's no guarantee the PDF is the culprit, so before digging in that grave, I should check the generation upstream:

```bash
>>> make clean && make document.tex && cp document.tex orig.tex
>>> make clean && make document.tex && cp document.tex next.tex
>>> diff orig.txt next.txt
```
No output, so the generated $\TeX$ is the same. A good start.

Next, check the file sizes (yeah, I probably should have done this first):

```bash
>>> make clean && make && mv document.pdf orig.pdf
>>> make clean && make && mv document.pdf next.pdf
>>> ls -la *.pdf
-rw-r--r--  1 kjell  staff  6709688 19 Jul 15:35 next.pdf
-rw-r--r--  1 kjell  staff  6709688 19 Jul 15:34 orig.pdf
```
Okay, so there's a good chance the bulk of those files are identical.
Since the upstream contents are the same, I assume it's some kind of metadata difference.
Sure enough, [stackoverflow confirms][tfa] that these three fields are to blame:

+ `/CreationDate`
+ `/ModDate`
+ `/ID`

[tfa]: https://tex.stackexchange.com/questions/229605/reproducible-latex-builds-compile-to-a-file-which-always-hashes-to-the-same-va

Two of these are easy to fix, by hard-coding something reasonable into the `SOURCE_DATE_EPOCH` environment variable before running `pandoc`. (like the suggested output of `date +%s`). According to [exiftool], the creation and modification dates now match. I can add that to the `Makefile`. Unfortunately, that's not enough.

 Annoyingly, `exiftool` doesn't seem to give me the `ID` field. Time to get dirty. (I'm actually impressed I made it this far without a [hex dump][xxd]).

[xxd]: https://github.com/vim/vim/blob/master/src/xxd/xxd.c

```bash
>>> diff <(xxd document-1.pdf) <(xxd document.pdf)
418936,418940c418936,418940
< 00664770: 662f 4944 5b3c 3935 3361 3763 3266 6531  f/ID[<953a7c2fe1
< 00664780: 3363 3431 3139 3231 6236 3265 6635 3065  3c411921b62ef50e
< 00664790: 3962 6334 3134 3e3c 3935 3361 3763 3266  9bc414><953a7c2f
< 006647a0: 6531 3363 3431 3139 3231 6236 3265 6635  e13c411921b62ef5
< 006647b0: 3065 3962 6334 3134 3e5d 2f52 6f6f 740a  0e9bc414>]/Root.
---
> 00664770: 662f 4944 5b3c 6130 6665 6131 3762 3361  f/ID[<a0fea17b3a
> 00664780: 3039 3436 3330 6561 3536 6364 3366 6539  094630ea56cd3fe9
> 00664790: 6363 3734 3434 3e3c 6130 6665 6131 3762  cc7444><a0fea17b
> 006647a0: 3361 3039 3436 3330 6561 3536 6364 3366  3a094630ea56cd3f
> 006647b0: 6539 6363 3734 3434 3e5d 2f52 6f6f 740a  e9cc7444>]/Root.
```

That's annoying. The ID changes every time. According to the [aforementioned stackoverflow answer][tfa], there is a solution, but it depends on which PDF backend is compiling the $\LaTeX$. I suppose I can patch the [eisvogel.tex] template I'm using to generate the book, and add something to the $\TeX$ header. Technically, I only use [xetex], but so I don't have to look it up again, I'll put it all in:

```TeX
\ifnum 0\ifxetex 1\fi\ifluatex 1\fi=0 % if pdftexe
  \pdfinfoomitdate=1
  \pdftrailerid{}
\else % if not pdftex
  \ifxetex
    \special{pdf:trailerid [
      <00112233445566778899aabbccddeeff>
      <00112233445566778899aabbccddeeff>
    ]}
  \fi
  \ifluatex
    \pdfvariable suppressoptionalinfo \numexpr32+64+512\relax
  \fi
\fi
```

And now finally, my PDF generation is reproducible.

```bash
>>> make clean && make && mv document.pdf orig.pdf
>>> make clean && make && mv document.pdf next.pdf
>>> md5 *.pdf
MD5 (next.pdf) = c3bf99530a35eab6f9adafb08c24acbd
MD5 (orig.pdf) = c3bf99530a35eab6f9adafb08c24acbd
```

That wasn't so hard, was it?

(yikes).

[exiftool]: https://exiftool.org/
[xetex]: https://tug.org/xetex/
[eisvogel.tex]: https://github.com/Wandmalfarbe/pandoc-latex-template
