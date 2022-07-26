---
layout: post
title: Reproducible PDFs
date: 2022-07-19
categories: [pdf, reproducibility]
excerpt: Reproducibly generating a PDF from the same source file can't possibly be a hard problem, can it?
use_math: true
---

For the last few weeks, I've been putting the final touches on a research report, intended to be published both as a print (like, *dead-tree*) publication, and as a digital artifact (PDF, including the sources needed to generate it.)

It's been a lot of back-and-forth, but we're finally at the point of production. As I happily regenerated the PDF to send to the printer "one last time" (`book-final-edited-fix2-AGAIN-reallyfinal-ugh.pdf`), I noticed something odd. I kept getting git conflicts with the PDF, even when the source material wasn't changing. (I don't usually check generated files into git, but this particular PDF, being the main output of the project in question, seemed like a reasonable exception to my rule).

## Down The Rabbit Hole

In hunting for the source of these git conflicts,  I unwittingly fell down a reproducibility rabbit hole with my PDF generation:  **Can I reproducibly generate a PDF from an unchanging source document?**

To be more precise, I mean: starting from _exactly_ the same input (a set of markdown documents), can I get the same PDF out? That can't possibly be a hard problem, can it?

Short answer: _**it's a lot harder than I thought**_ (and metadata is to blame).

## Our Pipeline

We generate our book PDF from a set of markdown source files using [pandoc](https://pandoc.org/). It's academic writing, so there's a mix of filters in there: LaTeX for equations, [pandoc-crossref](https://github.com/lierdakil/pandoc-crossref) for intra-document references, and Citeproc (BibTeX) for citations and references. It's also multilingual, so we throw [Xetex](https://tug.org/xetex/) into the mix to handle Unicode. It may seem like a crazy way to do it, but the result is an easy-to-edit, easy-to-diff document that can be easily maintained (_and_ viewed) in GitHub by a wide variety of people (even those for whom LaTeX isn't their first language).

The result is shockingly easy to convert to other document formats, so when a client asks for, say, a Word document to send for translation, we can easily do the conversion and expect that everything will render properly.

## Our Problem

I couldn't seem to generate the same PDF twice. Witness here:
```bash
>>> make clean && make && md5 document.pdf
MD5 (document.pdf) = fdfeefe8eb0df92162342271ad4cacc2
>>> make clean && make && md5 document.pdf
MD5 (document.pdf) = 90360b00c4f1ef08e57135e6b866e392
```
Basically, every time the PDF is generated, the hash is different. That's a little embarrassing for a guy who does reproducibility research. I need to fix this in our generation pipeline.

Pro-tip number 1: **make sure you're solving the right problem**. There's no guarantee the PDF is the culprit, so before digging in that grave, I should check the generation upstream. Is the source material actually unchanging?
```bash
>>> make clean && make document.tex && cp document.tex orig.tex
>>> make clean && make document.tex && cp document.tex next.tex
>>> diff orig.txt next.txt
(nothing)
```

As I hoped: no output, so the generated TEX is the same. A good start.

Next, let's figure out how different these files actually are, starting with my favourite hash function: file size.

```bash
>>> make clean && make && mv document.pdf orig.pdf
>>> make clean && make && mv document.pdf next.pdf
>>> ls -la *.pdf
-rw-r--r--  1 hackalog  staff  6709688 19 Jul 15:35 next.pdf
-rw-r--r--  1 hackalog  staff  6709688 19 Jul 15:34 orig.pdf
```

Since the upstream contents are the same, and the resulting PDFs are the same size, I'm going to assume the bulk of the files are identical and look for some kind of metadata difference.

## The Fix

Lo and behold, it's metadata. Google and [Stackoverflow confirm](https://tex.stackexchange.com/questions/229605/reproducible-latex-builds-compile-to-a-file-which-always-hashes-to-the-same-va) that these three fields are to blame:

-   `/CreationDate`
-   `/ModDate`
-   `/ID`

By reading the article, it seems that two of these are easy to fix, by hard-coding something reasonable into a `SOURCE_DATE_EPOCH` environment variable before running `pandoc`. (like the suggested output of `date +%s`). I can generate a fixed date, set the variable, and give it a try.

Sure enough, according to [exiftool](https://exiftool.org/), the creation and modification dates now match. Unfortunately, the hashes _still_ don't match.

What about that third one? ID? Annoyingly, [exiftool](https://exiftool.org/) doesn't let me view the `ID` field directly. Time to get dirty. (I'm actually impressed I made it this far without a [hex dump](https://github.com/vim/vim/blob/master/src/xxd/xxd.c)).
```
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

There it is, and sure enough, the ID changes every time. According to the [aforelinked stackoverflow article](https://tex.stackexchange.com/questions/229605/reproducible-latex-builds-compile-to-a-file-which-always-hashes-to-the-same-va), there is a solution, but it depends on which PDF backend is compiling the actual compiling for pandoc. I suppose I can patch the [eisvogel.tex](https://github.com/Wandmalfarbe/pandoc-latex-template) template I'm using to generate the book, and add a blurb the TeX header. Technically, I only use Xetex, but so I don't have to look it up again, I'll put it **all** in and cross my fingers if I ever need a different backend:
```tex
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

## The Result

A few fistfuls of hair, a few hours, a hex dump, and much googling later, and...

```bash
>>> make `clean` && make && mv `document.pdf orig.pdf`
>>> make `clean` && make && mv `document.pdf next.pdf`
>>> `md5 *.pdf MD5`
(`next.pdf`) = `c3bf99530a35eab6f9adafb08c24acbd MD5`
(`orig.pdf`) = `c3bf99530a35eab6f9adafb08c24acbd`
```

At last my PDF generation is reproducible. That wasn't so hard, was it?
