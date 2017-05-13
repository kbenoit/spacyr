[![CRAN Version](https://www.r-pkg.org/badges/version/spacyr)](https://CRAN.R-project.org/package=spacyr) ![Downloads](https://cranlogs.r-pkg.org/badges/spacyr) [![Travis-CI Build Status](https://travis-ci.org/kbenoit/spacyr.svg?branch=master)](https://travis-ci.org/kbenoit/spacyr) [![Appveyor Build status](https://ci.appveyor.com/api/projects/status/jqt2atp1wqtxy5xd/branch/master?svg=true)](https://ci.appveyor.com/project/kbenoit/spacyr/branch/master) [![codecov.io](https://codecov.io/github/kbenoit/spacyr/spacyr.svg?branch=master)](https://codecov.io/github/kbenoit/spacyr/coverage.svg?branch=master)

spacyr: an R wrapper for spaCy
==============================

This package is an R wrapper to the spaCy "industrial strength natural language processing" Python library from <http://spacy.io>.

Installing the package
----------------------

For the installation of `spaCy` and `spacyr` in Mac OS X (in homebrew and default Pythons) and Windows you can find more detailed instructions in [Mac OS X Installation](inst/docs/MAC.md) and [Windows Installation](inst/docs/WINDOWS.md).

1.  Python (&gt; 2.7 or 3) must be installed on your system.

    **(Windows only)** If you have not yet installed Python, Download and install [Python for Windows](https://www.python.org/downloads/windows/). We strongly recommend to use Python 3, and the following instructions is based on the use of Python 3. We recommend the latest 3.6.\* release (currently 3.6.1). During the installation process, be sure to scroll down in the installation option window and find the "Add Python.exe to Path", and click on the small red "x."

2.  A C++ compiler must be installed on your system.

    -   **(Mac only)** Install XTools. Either get the full XTools from the App Store, or install the command-line XTools using this command from the Terminal:

        ``` bash
        xcode-select --install
        ```

    -   **(Windows only)** Install the [Rtools](https://CRAN.R-project.org/bin/windows/Rtools/) software available from CRAN

        You will also need to install the [Visual Studio Express 2015](https://www.visualstudio.com/post-download-vs/?sku=xdesk&clcid=0x409&telem=ga#).

3.  You will need to install spaCy.

    Install spaCy and the English language model using these commands at the command line:

    ``` bash
    pip install -U spacy
    python -m spacy download en
    ```

    Test your installation at the command line using:

    ``` bash
    python -c "import spacy; spacy.load('en'); print('OK')"
    ```

    There are alternative methods of installing spaCy, especially if you have installed a different Python (e.g. through Anaconda). Full installation instructions are available from the [spaCy page](http://spacy.io/docs/).

4.  Installing the **spacyr** R package:

    To install the package from source, you can simply run the following.

    ``` r
    devtools::install_github("kbenoit/spacyr")
    ```

Examples
--------

When initializing spaCy, you need to set the python path if in your system, spaCy is installed in a Python which is not the system default. A detailed discussion about it is found in [Multiple Pythons](#multiplepythons) below.

``` r
require(spacyr)
#> Loading required package: spacyr
# start a python process and initialize spaCy in it.
# it takes several seconds for initialization.
# spacyr attempts to find python with spaCy
spacy_initialize()
#> No python executable is specified, spacyr tries to find a python executable with spacy
#> spaCy (language model: en) is installed in /usr/local/bin/python
#> spaCy is successfully initialized (spaCy Version: 1.8.1, language model: en)
```

The `spacy_parse()` function calls spaCy to both tokenize and tag the texts. In addition, it provides a functionalities of dependency parsing and named entity recognition. The function returns a `data.table` of the results. The approach to tokenizing taken by spaCy is inclusive: it includes all tokens without restrictions. The default method for `tag()` is the [Google tagset for parts-of-speech](https://github.com/slavpetrov/universal-pos-tags).

``` r

txt <- c(fastest = "spaCy excells at large-scale information extraction tasks. It is written from the ground up in carefully memory-managed Cython. Independent research has confirmed that spaCy is the fastest in the world. If your application needs to process entire web dumps, spaCy is the library you want to be using.",
         getdone = "spaCy is designed to help you do real work — to build real products, or gather real insights. The library respects your time, and tries to avoid wasting it. It is easy to install, and its API is simple and productive. I like to think of spaCy as the Ruby on Rails of Natural Language Processing.")

# process documents and obtain a data.table
parsedtxt <- spacy_parse(txt)
head(parsedtxt)
#>    docname sentence_id token_id  tokens tag_detailed tag_google
#> 1: fastest           1        1   spaCy           NN       NOUN
#> 2: fastest           1        2 excells          NNS       NOUN
#> 3: fastest           1        3      at           IN        ADP
#> 4: fastest           1        4   large           JJ        ADJ
#> 5: fastest           1        5       -         HYPH      PUNCT
#> 6: fastest           1        6   scale           NN       NOUN
```

By default, `spacy_parse()` conduct tokenization and part-of-speech (POS) tagging. spacyr provides two tagsets, coarse-grained [Google](https://github.com/slavpetrov/universal-pos-tags) tagsets and finer-grained [Penn Treebank](https://www.ling.upenn.edu/courses/Fall_2003/ling001/penn_treebank_pos.html) tagsets. The `tag_google` or `tag_detailed` field in the data.table corresponds to each of these tagsets.

Many of the standard methods from [**quanteda**](http://githiub.com/kbenoit/quanteda) work on the new tagged token objects:

``` r
require(quanteda, warn.conflicts = FALSE, quietly = TRUE)
#> quanteda version 0.9.9.50
#> Using 7 of 8 cores for parallel computing
docnames(parsedtxt)
#> [1] "fastest" "getdone"
ndoc(parsedtxt)
#> [1] 2
ntoken(parsedtxt)
#> fastest getdone 
#>      57      63
ntype(parsedtxt)
#> fastest getdone 
#>      44      46
```

### Document processing with additional features

The following codes conduct more detailed document processing, including dependency parsing and named entitiy recognition.

``` r
results_detailed <- spacy_parse(txt,
                                pos_tag = TRUE,
                                lemma = TRUE,
                                named_entity = TRUE,
                                dependency = TRUE)
head(results_detailed, 30)
#>     docname sentence_id token_id      tokens       lemma tag_detailed
#>  1: fastest           1        1       spaCy       spacy           NN
#>  2: fastest           1        2     excells      excell          NNS
#>  3: fastest           1        3          at          at           IN
#>  4: fastest           1        4       large       large           JJ
#>  5: fastest           1        5           -           -         HYPH
#>  6: fastest           1        6       scale       scale           NN
#>  7: fastest           1        7 information information           NN
#>  8: fastest           1        8  extraction  extraction           NN
#>  9: fastest           1        9       tasks        task          NNS
#> 10: fastest           1       10           .           .            .
#> 11: fastest           2        1          It      -PRON-          PRP
#> 12: fastest           2        2          is          be          VBZ
#> 13: fastest           2        3     written       write          VBN
#> 14: fastest           2        4        from        from           IN
#> 15: fastest           2        5         the         the           DT
#> 16: fastest           2        6      ground      ground           NN
#> 17: fastest           2        7          up          up           RB
#> 18: fastest           2        8          in          in           IN
#> 19: fastest           2        9   carefully   carefully           RB
#> 20: fastest           2       10      memory      memory           NN
#> 21: fastest           2       11           -           -         HYPH
#> 22: fastest           2       12     managed      manage          VBN
#> 23: fastest           2       13      Cython      cython          NNP
#> 24: fastest           2       14           .           .            .
#> 25: fastest           3        1 Independent independent           JJ
#> 26: fastest           3        2    research    research           NN
#> 27: fastest           3        3         has        have          VBZ
#> 28: fastest           3        4   confirmed     confirm          VBN
#> 29: fastest           3        5        that        that           IN
#> 30: fastest           3        6       spaCy       spacy          NNP
#>     docname sentence_id token_id      tokens       lemma tag_detailed
#>     tag_google head_token_id   dep_rel named_entity
#>  1:       NOUN             2  compound             
#>  2:       NOUN             2      ROOT             
#>  3:        ADP             2      prep             
#>  4:        ADJ             6      amod             
#>  5:      PUNCT             6     punct             
#>  6:       NOUN             7  compound             
#>  7:       NOUN             8  compound             
#>  8:       NOUN             9  compound             
#>  9:       NOUN             3      pobj             
#> 10:      PUNCT             2     punct             
#> 11:       PRON             3 nsubjpass             
#> 12:       VERB             3   auxpass             
#> 13:       VERB             3      ROOT             
#> 14:        ADP             3      prep             
#> 15:        DET             6       det             
#> 16:       NOUN             4      pobj             
#> 17:        ADV             3       prt             
#> 18:        ADP             7      prep             
#> 19:        ADV            12    advmod             
#> 20:       NOUN            12  npadvmod             
#> 21:      PUNCT            12     punct             
#> 22:       VERB             3      prep             
#> 23:      PROPN            12      dobj        ORG_B
#> 24:      PUNCT             3     punct             
#> 25:        ADJ             2      amod             
#> 26:       NOUN             4     nsubj             
#> 27:       VERB             4       aux             
#> 28:       VERB             4      ROOT             
#> 29:        ADP             7      mark             
#> 30:      PROPN             7     nsubj             
#>     tag_google head_token_id   dep_rel named_entity
```

### Use other language models

In default, `spacyr` load an English language model in spacy, but you also can load a German language model instead by specifying `model` option when `spacy_initialize` is called.

``` r
## first finalize the spacy if it's loaded
spacy_finalize()
spacy_initialize(model = 'de')
#> Python space is already attached to R. You cannot switch Python.
#> If you'd like to switch to other Python, please restart R
#> spaCy is successfully initialized (spaCy Version: 1.8.1, language model: de)

txt_german = c(R = "R ist eine freie Programmiersprache für statistische Berechnungen und Grafiken. Sie wurde von Statistikern für Anwender mit statistischen Aufgaben entwickelt. Die Syntax orientiert sich an der Programmiersprache S, mit der R weitgehend kompatibel ist, und die Semantik an Scheme. Als Standarddistribution kommt R mit einem Interpreter als Kommandozeilenumgebung mit rudimentären grafischen Schaltflächen. So ist R auf vielen Plattformen verfügbar; die Umgebung wird von den Entwicklern ausdrücklich ebenfalls als R bezeichnet. R ist Teil des GNU-Projekts.",
               python = "Python ist eine universelle, üblicherweise interpretierte höhere Programmiersprache. Sie will einen gut lesbaren, knappen Programmierstil fördern. So wird beispielsweise der Code nicht durch geschweifte Klammern, sondern durch Einrückungen strukturiert.")
results_german <- spacy_parse(txt_german,
                              pos_tag = TRUE,
                              lemma = TRUE,
                              named_entity = TRUE,
                              dependency = TRUE)
head(results_german, 30)
#>     docname sentence_id token_id             tokens              lemma
#>  1:       R           1        1                  R                  r
#>  2:       R           1        2                ist                ist
#>  3:       R           1        3               eine               eine
#>  4:       R           1        4              freie              freie
#>  5:       R           1        5 Programmiersprache programmiersprache
#>  6:       R           1        6                 fr                 fr
#>  7:       R           1        7       statistische       statistische
#>  8:       R           1        8       Berechnungen       berechnungen
#>  9:       R           1        9                und                und
#> 10:       R           1       10           Grafiken           grafiken
#> 11:       R           1       11                  .                  .
#> 12:       R           2        1                Sie                sie
#> 13:       R           2        2              wurde              wurde
#> 14:       R           2        3                von                von
#> 15:       R           2        4       Statistikern       statistikern
#> 16:       R           2        5                 fr                 fr
#> 17:       R           2        6           Anwender           anwender
#> 18:       R           2        7                mit                mit
#> 19:       R           2        8      statistischen      statistischen
#> 20:       R           2        9           Aufgaben           aufgaben
#> 21:       R           2       10         entwickelt         entwickelt
#> 22:       R           2       11                  .                  .
#> 23:       R           3        1                Die                die
#> 24:       R           3        2             Syntax             syntax
#> 25:       R           3        3         orientiert         orientiert
#> 26:       R           3        4               sich               sich
#> 27:       R           3        5                 an                 an
#> 28:       R           3        6                der                der
#> 29:       R           3        7 Programmiersprache programmiersprache
#> 30:       R           3        8                  S                  s
#>     docname sentence_id token_id             tokens              lemma
#>     tag_detailed tag_google head_token_id dep_rel named_entity
#>  1:           XY          X             2      sb             
#>  2:        VAFIN        AUX             2    ROOT             
#>  3:          ART        DET             5      nk             
#>  4:         ADJA        ADJ             5      nk             
#>  5:           NN       NOUN             2      pd             
#>  6:           NE      PROPN             5      nk             
#>  7:         ADJA        ADJ             8      nk             
#>  8:           NN       NOUN             2      pd             
#>  9:          KON       CONJ             8      cd             
#> 10:           NN       NOUN             9      cj             
#> 11:           $.      PUNCT             2   punct             
#> 12:         PPER       PRON             2      sb             
#> 13:        VAFIN        AUX             2    ROOT             
#> 14:         APPR        ADP             6      pg             
#> 15:           NN       NOUN             3      nk             
#> 16:           NE      PROPN             6      nk             
#> 17:           NN       NOUN            10      oa             
#> 18:         APPR        ADP            10      mo             
#> 19:         ADJA        ADJ             9      nk             
#> 20:           NN       NOUN             7      nk             
#> 21:         VVPP       VERB             2      oc             
#> 22:           $.      PUNCT             2   punct             
#> 23:          ART        DET             2      nk             
#> 24:           NN       NOUN             3      sb             
#> 25:        VVFIN       VERB             3    ROOT             
#> 26:          PRF       PRON             3      oa             
#> 27:         APPR        ADP             3      mo             
#> 28:          ART        DET             7      nk             
#> 29:           NN       NOUN             5      nk             
#> 30:           NE      PROPN             7      nk             
#>     tag_detailed tag_google head_token_id dep_rel named_entity
```

The German language model has to be installed (`python -m spacy download de`) before you call `spacy_initialize`.

### When you finish

A background process of spaCy is initiated when you ran `spacy_initialize`. Because of the size of language models of `spaCy`, this takes up a lot of memory (typically 1.5GB). When you do not need the python connection any longer, you can finalize the python (and terminate terminate the process) by running `spacy_finalize()` function.

``` r
spacy_finalize()
```

By calling `spacy_initialize()` again, you can restart the backend spaCy.

<a name="multiplepythons"></a>Multiple Python executables in your system
------------------------------------------------------------------------

If you have multiple Python executables in your systems (e.g. you, a Mac user, have brewed python2 or python3), `spacy_initialize` function will check whether each of them have spaCy installed or not. To save the time for this checking, you can specify the particular python when initializing `spaCy` by executing `spacy_initialize()`. Suppose that your python with spaCy is `/usr/local/bin/python`, run the following:

``` r
library(spacyr)
spacy_initialize(use_python = "/usr/local/bin/python")
```

Comments and feedback
---------------------

We welcome your comments and feedback. Please file issues on the [issues](https://github.com/kbenoit/spacyr/issues) page, and/or send us comments at <kbenoit@lse.ac.uk> and <A.Matsuo@lse.ac.uk>.
